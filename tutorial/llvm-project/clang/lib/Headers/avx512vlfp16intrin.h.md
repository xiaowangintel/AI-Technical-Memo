# avx512vlfp16intrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx512vlfp16intrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: AVX512-FP16 intrinsics.
- **Purpose (CN)**: 提供 AVX512-FP16 intrinsic 接口。
- **Line Count / 行数**: 2090

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---------- avx512vlfp16intrin.h - AVX512-FP16 intrinsics --------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */
#ifndef __IMMINTRIN_H
#error                                                                         \
    "Never use <avx512vlfp16intrin.h> directly; include <immintrin.h> instead."
#endif

#ifdef __SSE2__

#ifndef __AVX512VLFP16INTRIN_H
#define __AVX512VLFP16INTRIN_H

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS256                                                  \
  __attribute__((__always_inline__, __nodebug__,                               \
                 __target__("avx512fp16,avx512vl"),                            \
                 __min_vector_width__(256)))
#define __DEFAULT_FN_ATTRS128                                                  \
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
- **L11 EN**: Continues the surrounding expression or declaration: `"Never use <avx512vlfp16intrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 继续构造周围的表达式或声明：`"Never use <avx512vlfp16intrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifdef __SSE2__`.
  **L14 CN**: 开始一个预处理条件块：`#ifdef __SSE2__`。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Starts a preprocessor conditional block: `#ifndef __AVX512VLFP16INTRIN_H`.
  **L16 CN**: 开始一个预处理条件块：`#ifndef __AVX512VLFP16INTRIN_H`。
- **L17 EN**: Defines macro `__AVX512VLFP16INTRIN_H` for conditional compilation, shorthand, or API generation.
  **L17 CN**: 定义宏 `__AVX512VLFP16INTRIN_H`，用于条件编译、简写或 API 生成。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L20 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L20 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L21 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__,                               \`.
  **L21 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__,                               \`。
- **L22 EN**: Continues logic associated with callable symbol `__target__`.
  **L22 CN**: 继续与可调用符号 `__target__` 相关的逻辑。
- **L23 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L23 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L24 EN**: Defines macro `__DEFAULT_FN_ATTRS128` for conditional compilation, shorthand, or API generation.
  **L24 CN**: 定义宏 `__DEFAULT_FN_ATTRS128`，用于条件编译、简写或 API 生成。

### Lines 25-48

````c
  __attribute__((__always_inline__, __nodebug__,                               \
                 __target__("avx512fp16,avx512vl"),                            \
                 __min_vector_width__(128)))

#if defined(__cplusplus) && (__cplusplus >= 201103L)
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256 constexpr
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128 constexpr
#else
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128
#endif

static __inline__ _Float16 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_cvtsh_h(__m128h __a) {
  return __a[0];
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtsh_h(__m256h __a) {
  return __a[0];
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_set_sh(_Float16 __h) {
````
- **L25 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__,                               \`.
  **L25 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__,                               \`。
- **L26 EN**: Continues logic associated with callable symbol `__target__`.
  **L26 CN**: 继续与可调用符号 `__target__` 相关的逻辑。
- **L27 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L27 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L29 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L29 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L30 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L30 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L31 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L31 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L32 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L32 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L33 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L33 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L34 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L34 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L35 EN**: Closes the current preprocessor conditional block.
  **L35 CN**: 结束当前预处理条件块。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L37 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L37 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L38 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsh_h(__m128h __a) {`.
  **L38 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsh_h(__m128h __a) {`。
- **L39 EN**: Returns from the current function with `__a[0]`.
  **L39 CN**: 以 `__a[0]` 从当前函数返回。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L42 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L43 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtsh_h(__m256h __a) {`.
  **L43 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtsh_h(__m256h __a) {`。
- **L44 EN**: Returns from the current function with `__a[0]`.
  **L44 CN**: 以 `__a[0]` 从当前函数返回。
- **L45 EN**: Closes the current lexical scope or compound statement.
  **L45 CN**: 结束当前词法作用域或复合语句块。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L47 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L48 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_sh(_Float16 __h) {`.
  **L48 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_sh(_Float16 __h) {`。

### Lines 49-72

````c
  return __extension__(__m128h){__h, 0, 0, 0, 0, 0, 0, 0};
}

static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_set1_ph(_Float16 __h) {
  return (__m128h)(__v8hf){__h, __h, __h, __h, __h, __h, __h, __h};
}

static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_set1_ph(_Float16 __h) {
  return (__m256h)(__v16hf){__h, __h, __h, __h, __h, __h, __h, __h,
                            __h, __h, __h, __h, __h, __h, __h, __h};
}

static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,
           _Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8) {
  return (__m128h)(__v8hf){__h8, __h7, __h6, __h5, __h4, __h3, __h2, __h1};
}

static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_set1_pch(_Float16 _Complex h) {
  return (__m256h)_mm256_set1_ps(__builtin_bit_cast(float, h));
}
````
- **L49 EN**: Returns from the current function with `__extension__(__m128h){__h, 0, 0, 0, 0, 0, 0, 0}`.
  **L49 CN**: 以 `__extension__(__m128h){__h, 0, 0, 0, 0, 0, 0, 0}` 从当前函数返回。
- **L50 EN**: Closes the current lexical scope or compound statement.
  **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Continues the surrounding expression or declaration: `static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L52 CN**: 继续构造周围的表达式或声明：`static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L53 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_ph(_Float16 __h) {`.
  **L53 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_ph(_Float16 __h) {`。
- **L54 EN**: Returns from the current function with `(__m128h)(__v8hf){__h, __h, __h, __h, __h, __h, __h, __h}`.
  **L54 CN**: 以 `(__m128h)(__v8hf){__h, __h, __h, __h, __h, __h, __h, __h}` 从当前函数返回。
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L57 EN**: Continues the surrounding expression or declaration: `static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L57 CN**: 继续构造周围的表达式或声明：`static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L58 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_set1_ph(_Float16 __h) {`.
  **L58 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_set1_ph(_Float16 __h) {`。
- **L59 EN**: Returns from the current function with `(__m256h)(__v16hf){__h, __h, __h, __h, __h, __h, __h, __h,`.
  **L59 CN**: 以 `(__m256h)(__v16hf){__h, __h, __h, __h, __h, __h, __h, __h,` 从当前函数返回。
- **L60 EN**: Adds a standalone statement or declaration: `__h, __h, __h, __h, __h, __h, __h, __h};`.
  **L60 CN**: 添加一条独立语句或声明：`__h, __h, __h, __h, __h, __h, __h, __h};`。
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Continues the surrounding expression or declaration: `static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L63 CN**: 继续构造周围的表达式或声明：`static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,`。
- **L65 EN**: Continues the surrounding expression or declaration: `_Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8) {`.
  **L65 CN**: 继续构造周围的表达式或声明：`_Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8) {`。
- **L66 EN**: Returns from the current function with `(__m128h)(__v8hf){__h8, __h7, __h6, __h5, __h4, __h3, __h2, __h1}`.
  **L66 CN**: 以 `(__m128h)(__v8hf){__h8, __h7, __h6, __h5, __h4, __h3, __h2, __h1}` 从当前函数返回。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L69 EN**: Continues the surrounding expression or declaration: `static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L69 CN**: 继续构造周围的表达式或声明：`static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L70 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_set1_pch(_Float16 _Complex h) {`.
  **L70 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_set1_pch(_Float16 _Complex h) {`。
- **L71 EN**: Returns from the current function with `(__m256h)_mm256_set1_ps(__builtin_bit_cast(float, h))`.
  **L71 CN**: 以 `(__m256h)_mm256_set1_ps(__builtin_bit_cast(float, h))` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-96

````c

static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_set1_pch(_Float16 _Complex h) {
  return (__m128h)_mm_set1_ps(__builtin_bit_cast(float, h));
}

static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,
              _Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8,
              _Float16 __h9, _Float16 __h10, _Float16 __h11, _Float16 __h12,
              _Float16 __h13, _Float16 __h14, _Float16 __h15, _Float16 __h16) {
  return (__m256h)(__v16hf){__h16, __h15, __h14, __h13, __h12, __h11,
                            __h10, __h9,  __h8,  __h7,  __h6,  __h5,
                            __h4,  __h3,  __h2,  __h1};
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_setr_ph(_Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,
            _Float16 e5, _Float16 e6, _Float16 e7) {
  return _mm_set_ph(e7, e6, e5, e4, e3, e2, e1, e0);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_setr_ph(_Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,
````
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L74 EN**: Continues the surrounding expression or declaration: `static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L74 CN**: 继续构造周围的表达式或声明：`static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L75 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_pch(_Float16 _Complex h) {`.
  **L75 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_pch(_Float16 _Complex h) {`。
- **L76 EN**: Returns from the current function with `(__m128h)_mm_set1_ps(__builtin_bit_cast(float, h))`.
  **L76 CN**: 以 `(__m128h)_mm_set1_ps(__builtin_bit_cast(float, h))` 从当前函数返回。
- **L77 EN**: Closes the current lexical scope or compound statement.
  **L77 CN**: 结束当前词法作用域或复合语句块。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L79 EN**: Continues the surrounding expression or declaration: `static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L79 CN**: 继续构造周围的表达式或声明：`static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,`.
  **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,`。
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8,`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8,`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 __h9, _Float16 __h10, _Float16 __h11, _Float16 __h12,`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 __h9, _Float16 __h10, _Float16 __h11, _Float16 __h12,`。
- **L83 EN**: Continues the surrounding expression or declaration: `_Float16 __h13, _Float16 __h14, _Float16 __h15, _Float16 __h16) {`.
  **L83 CN**: 继续构造周围的表达式或声明：`_Float16 __h13, _Float16 __h14, _Float16 __h15, _Float16 __h16) {`。
- **L84 EN**: Returns from the current function with `(__m256h)(__v16hf){__h16, __h15, __h14, __h13, __h12, __h11,`.
  **L84 CN**: 以 `(__m256h)(__v16hf){__h16, __h15, __h14, __h13, __h12, __h11,` 从当前函数返回。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__h10, __h9,  __h8,  __h7,  __h6,  __h5,`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`__h10, __h9,  __h8,  __h7,  __h6,  __h5,`。
- **L86 EN**: Adds a standalone statement or declaration: `__h4,  __h3,  __h2,  __h1};`.
  **L86 CN**: 添加一条独立语句或声明：`__h4,  __h3,  __h2,  __h1};`。
- **L87 EN**: Closes the current lexical scope or compound statement.
  **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L89 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L89 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_setr_ph(_Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_setr_ph(_Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,`。
- **L91 EN**: Continues the surrounding expression or declaration: `_Float16 e5, _Float16 e6, _Float16 e7) {`.
  **L91 CN**: 继续构造周围的表达式或声明：`_Float16 e5, _Float16 e6, _Float16 e7) {`。
- **L92 EN**: Returns from the current function with `_mm_set_ph(e7, e6, e5, e4, e3, e2, e1, e0)`.
  **L92 CN**: 以 `_mm_set_ph(e7, e6, e5, e4, e3, e2, e1, e0)` 从当前函数返回。
- **L93 EN**: Closes the current lexical scope or compound statement.
  **L93 CN**: 结束当前词法作用域或复合语句块。
- **L94 EN**: Blank line separating nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L95 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L95 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_setr_ph(_Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_setr_ph(_Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,`。

### Lines 97-120

````c
               _Float16 e5, _Float16 e6, _Float16 e7, _Float16 e8, _Float16 e9,
               _Float16 e10, _Float16 e11, _Float16 e12, _Float16 e13,
               _Float16 e14, _Float16 e15) {
  return _mm256_set_ph(e15, e14, e13, e12, e11, e10, e9, e8, e7, e6, e5, e4, e3,
                       e2, e1, e0);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_add_ph(__m256h __A,
                                                              __m256h __B) {
  return (__m256h)((__v16hf)__A + (__v16hf)__B);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_add_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      __U, (__v16hf)_mm256_add_ph(__A, __B), (__v16hf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_add_ph(__mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      __U, (__v16hf)_mm256_add_ph(__A, __B), (__v16hf)_mm256_setzero_ph());
}

````
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 e5, _Float16 e6, _Float16 e7, _Float16 e8, _Float16 e9,`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 e5, _Float16 e6, _Float16 e7, _Float16 e8, _Float16 e9,`。
- **L98 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 e10, _Float16 e11, _Float16 e12, _Float16 e13,`.
  **L98 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 e10, _Float16 e11, _Float16 e12, _Float16 e13,`。
- **L99 EN**: Continues the surrounding expression or declaration: `_Float16 e14, _Float16 e15) {`.
  **L99 CN**: 继续构造周围的表达式或声明：`_Float16 e14, _Float16 e15) {`。
- **L100 EN**: Returns from the current function with `_mm256_set_ph(e15, e14, e13, e12, e11, e10, e9, e8, e7, e6, e5, e4, e3,`.
  **L100 CN**: 以 `_mm256_set_ph(e15, e14, e13, e12, e11, e10, e9, e8, e7, e6, e5, e4, e3,` 从当前函数返回。
- **L101 EN**: Adds a standalone statement or declaration: `e2, e1, e0);`.
  **L101 CN**: 添加一条独立语句或声明：`e2, e1, e0);`。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_add_ph(__m256h __A,`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_add_ph(__m256h __A,`。
- **L105 EN**: Continues the surrounding expression or declaration: `__m256h __B) {`.
  **L105 CN**: 继续构造周围的表达式或声明：`__m256h __B) {`。
- **L106 EN**: Returns from the current function with `(__m256h)((__v16hf)__A + (__v16hf)__B)`.
  **L106 CN**: 以 `(__m256h)((__v16hf)__A + (__v16hf)__B)` 从当前函数返回。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L109 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L109 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L110 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_add_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`.
  **L110 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_add_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`。
- **L111 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L111 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L112 EN**: Executes a call or declaration centered on `__U,`.
  **L112 CN**: 执行以 `__U,` 为核心的调用或声明。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L115 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L116 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_add_ph(__mmask16 __U, __m256h __A, __m256h __B) {`.
  **L116 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_add_ph(__mmask16 __U, __m256h __A, __m256h __B) {`。
- **L117 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L117 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L118 EN**: Executes a call or declaration centered on `__U,`.
  **L118 CN**: 执行以 `__U,` 为核心的调用或声明。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 121-144

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_add_ph(__m128h __A,
                                                           __m128h __B) {
  return (__m128h)((__v8hf)__A + (__v8hf)__B);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_add_ph(__m128h __W,
                                                                __mmask8 __U,
                                                                __m128h __A,
                                                                __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_add_ph(__A, __B),
                                              (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_add_ph(__mmask8 __U,
                                                                 __m128h __A,
                                                                 __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_add_ph(__A, __B),
                                              (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_sub_ph(__m256h __A,
                                                              __m256h __B) {
  return (__m256h)((__v16hf)__A - (__v16hf)__B);
}
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_add_ph(__m128h __A,`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_add_ph(__m128h __A,`。
- **L122 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L122 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L123 EN**: Returns from the current function with `(__m128h)((__v8hf)__A + (__v8hf)__B)`.
  **L123 CN**: 以 `(__m128h)((__v8hf)__A + (__v8hf)__B)` 从当前函数返回。
- **L124 EN**: Closes the current lexical scope or compound statement.
  **L124 CN**: 结束当前词法作用域或复合语句块。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_add_ph(__m128h __W,`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_add_ph(__m128h __W,`。
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L129 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L129 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L130 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_add_ph(__A, __B),`.
  **L130 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_add_ph(__A, __B),` 从当前函数返回。
- **L131 EN**: Executes a call or declaration centered on `statement`.
  **L131 CN**: 执行以 `statement` 为核心的调用或声明。
- **L132 EN**: Closes the current lexical scope or compound statement.
  **L132 CN**: 结束当前词法作用域或复合语句块。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_add_ph(__mmask8 __U,`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_add_ph(__mmask8 __U,`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L136 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L136 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L137 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_add_ph(__A, __B),`.
  **L137 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_add_ph(__A, __B),` 从当前函数返回。
- **L138 EN**: Executes a call or declaration centered on `statement`.
  **L138 CN**: 执行以 `statement` 为核心的调用或声明。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_sub_ph(__m256h __A,`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_sub_ph(__m256h __A,`。
- **L142 EN**: Continues the surrounding expression or declaration: `__m256h __B) {`.
  **L142 CN**: 继续构造周围的表达式或声明：`__m256h __B) {`。
- **L143 EN**: Returns from the current function with `(__m256h)((__v16hf)__A - (__v16hf)__B)`.
  **L143 CN**: 以 `(__m256h)((__v16hf)__A - (__v16hf)__B)` 从当前函数返回。
- **L144 EN**: Closes the current lexical scope or compound statement.
  **L144 CN**: 结束当前词法作用域或复合语句块。

### Lines 145-168

````c

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_sub_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      __U, (__v16hf)_mm256_sub_ph(__A, __B), (__v16hf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_sub_ph(__mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      __U, (__v16hf)_mm256_sub_ph(__A, __B), (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sub_ph(__m128h __A,
                                                           __m128h __B) {
  return (__m128h)((__v8hf)__A - (__v8hf)__B);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sub_ph(__m128h __W,
                                                                __mmask8 __U,
                                                                __m128h __A,
                                                                __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_sub_ph(__A, __B),
                                              (__v8hf)__W);
````
- **L145 EN**: Blank line separating nearby declarations or logic blocks.
  **L145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L146 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L146 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L147 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_sub_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`.
  **L147 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_sub_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`。
- **L148 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L148 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L149 EN**: Executes a call or declaration centered on `__U,`.
  **L149 CN**: 执行以 `__U,` 为核心的调用或声明。
- **L150 EN**: Closes the current lexical scope or compound statement.
  **L150 CN**: 结束当前词法作用域或复合语句块。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L152 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L152 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L153 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_sub_ph(__mmask16 __U, __m256h __A, __m256h __B) {`.
  **L153 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_sub_ph(__mmask16 __U, __m256h __A, __m256h __B) {`。
- **L154 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L154 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L155 EN**: Executes a call or declaration centered on `__U,`.
  **L155 CN**: 执行以 `__U,` 为核心的调用或声明。
- **L156 EN**: Closes the current lexical scope or compound statement.
  **L156 CN**: 结束当前词法作用域或复合语句块。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sub_ph(__m128h __A,`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sub_ph(__m128h __A,`。
- **L159 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L159 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L160 EN**: Returns from the current function with `(__m128h)((__v8hf)__A - (__v8hf)__B)`.
  **L160 CN**: 以 `(__m128h)((__v8hf)__A - (__v8hf)__B)` 从当前函数返回。
- **L161 EN**: Closes the current lexical scope or compound statement.
  **L161 CN**: 结束当前词法作用域或复合语句块。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sub_ph(__m128h __W,`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sub_ph(__m128h __W,`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L166 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L166 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L167 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_sub_ph(__A, __B),`.
  **L167 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_sub_ph(__A, __B),` 从当前函数返回。
- **L168 EN**: Executes a call or declaration centered on `statement`.
  **L168 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 169-192

````c
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sub_ph(__mmask8 __U,
                                                                 __m128h __A,
                                                                 __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_sub_ph(__A, __B),
                                              (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_mul_ph(__m256h __A,
                                                              __m256h __B) {
  return (__m256h)((__v16hf)__A * (__v16hf)__B);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_mul_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      __U, (__v16hf)_mm256_mul_ph(__A, __B), (__v16hf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_mul_ph(__mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      __U, (__v16hf)_mm256_mul_ph(__A, __B), (__v16hf)_mm256_setzero_ph());
````
- **L169 EN**: Closes the current lexical scope or compound statement.
  **L169 CN**: 结束当前词法作用域或复合语句块。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sub_ph(__mmask8 __U,`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sub_ph(__mmask8 __U,`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L173 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L173 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L174 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_sub_ph(__A, __B),`.
  **L174 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_sub_ph(__A, __B),` 从当前函数返回。
- **L175 EN**: Executes a call or declaration centered on `statement`.
  **L175 CN**: 执行以 `statement` 为核心的调用或声明。
- **L176 EN**: Closes the current lexical scope or compound statement.
  **L176 CN**: 结束当前词法作用域或复合语句块。
- **L177 EN**: Blank line separating nearby declarations or logic blocks.
  **L177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_mul_ph(__m256h __A,`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_mul_ph(__m256h __A,`。
- **L179 EN**: Continues the surrounding expression or declaration: `__m256h __B) {`.
  **L179 CN**: 继续构造周围的表达式或声明：`__m256h __B) {`。
- **L180 EN**: Returns from the current function with `(__m256h)((__v16hf)__A * (__v16hf)__B)`.
  **L180 CN**: 以 `(__m256h)((__v16hf)__A * (__v16hf)__B)` 从当前函数返回。
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L183 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L183 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L184 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mul_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`.
  **L184 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mul_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`。
- **L185 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L185 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L186 EN**: Executes a call or declaration centered on `__U,`.
  **L186 CN**: 执行以 `__U,` 为核心的调用或声明。
- **L187 EN**: Closes the current lexical scope or compound statement.
  **L187 CN**: 结束当前词法作用域或复合语句块。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L189 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L189 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L190 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mul_ph(__mmask16 __U, __m256h __A, __m256h __B) {`.
  **L190 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mul_ph(__mmask16 __U, __m256h __A, __m256h __B) {`。
- **L191 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L191 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L192 EN**: Executes a call or declaration centered on `__U,`.
  **L192 CN**: 执行以 `__U,` 为核心的调用或声明。

### Lines 193-216

````c
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mul_ph(__m128h __A,
                                                           __m128h __B) {
  return (__m128h)((__v8hf)__A * (__v8hf)__B);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_mul_ph(__m128h __W,
                                                                __mmask8 __U,
                                                                __m128h __A,
                                                                __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_mul_ph(__A, __B),
                                              (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_mul_ph(__mmask8 __U,
                                                                 __m128h __A,
                                                                 __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_mul_ph(__A, __B),
                                              (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_div_ph(__m256h __A,
                                                              __m256h __B) {
````
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mul_ph(__m128h __A,`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mul_ph(__m128h __A,`。
- **L196 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L196 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L197 EN**: Returns from the current function with `(__m128h)((__v8hf)__A * (__v8hf)__B)`.
  **L197 CN**: 以 `(__m128h)((__v8hf)__A * (__v8hf)__B)` 从当前函数返回。
- **L198 EN**: Closes the current lexical scope or compound statement.
  **L198 CN**: 结束当前词法作用域或复合语句块。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_mul_ph(__m128h __W,`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_mul_ph(__m128h __W,`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L203 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L203 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L204 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_mul_ph(__A, __B),`.
  **L204 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_mul_ph(__A, __B),` 从当前函数返回。
- **L205 EN**: Executes a call or declaration centered on `statement`.
  **L205 CN**: 执行以 `statement` 为核心的调用或声明。
- **L206 EN**: Closes the current lexical scope or compound statement.
  **L206 CN**: 结束当前词法作用域或复合语句块。
- **L207 EN**: Blank line separating nearby declarations or logic blocks.
  **L207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_mul_ph(__mmask8 __U,`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_mul_ph(__mmask8 __U,`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L210 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L210 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L211 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_mul_ph(__A, __B),`.
  **L211 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_mul_ph(__A, __B),` 从当前函数返回。
- **L212 EN**: Executes a call or declaration centered on `statement`.
  **L212 CN**: 执行以 `statement` 为核心的调用或声明。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_div_ph(__m256h __A,`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_div_ph(__m256h __A,`。
- **L216 EN**: Continues the surrounding expression or declaration: `__m256h __B) {`.
  **L216 CN**: 继续构造周围的表达式或声明：`__m256h __B) {`。

### Lines 217-240

````c
  return (__m256h)((__v16hf)__A / (__v16hf)__B);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_div_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      __U, (__v16hf)_mm256_div_ph(__A, __B), (__v16hf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_div_ph(__mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      __U, (__v16hf)_mm256_div_ph(__A, __B), (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_div_ph(__m128h __A,
                                                           __m128h __B) {
  return (__m128h)((__v8hf)__A / (__v8hf)__B);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_div_ph(__m128h __W,
                                                                __mmask8 __U,
                                                                __m128h __A,
                                                                __m128h __B) {
````
- **L217 EN**: Returns from the current function with `(__m256h)((__v16hf)__A / (__v16hf)__B)`.
  **L217 CN**: 以 `(__m256h)((__v16hf)__A / (__v16hf)__B)` 从当前函数返回。
- **L218 EN**: Closes the current lexical scope or compound statement.
  **L218 CN**: 结束当前词法作用域或复合语句块。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L220 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L220 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L221 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_div_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`.
  **L221 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_div_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`。
- **L222 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L222 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L223 EN**: Executes a call or declaration centered on `__U,`.
  **L223 CN**: 执行以 `__U,` 为核心的调用或声明。
- **L224 EN**: Closes the current lexical scope or compound statement.
  **L224 CN**: 结束当前词法作用域或复合语句块。
- **L225 EN**: Blank line separating nearby declarations or logic blocks.
  **L225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L226 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L226 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_div_ph(__mmask16 __U, __m256h __A, __m256h __B) {`.
  **L227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_div_ph(__mmask16 __U, __m256h __A, __m256h __B) {`。
- **L228 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L228 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L229 EN**: Executes a call or declaration centered on `__U,`.
  **L229 CN**: 执行以 `__U,` 为核心的调用或声明。
- **L230 EN**: Closes the current lexical scope or compound statement.
  **L230 CN**: 结束当前词法作用域或复合语句块。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_div_ph(__m128h __A,`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_div_ph(__m128h __A,`。
- **L233 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L233 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L234 EN**: Returns from the current function with `(__m128h)((__v8hf)__A / (__v8hf)__B)`.
  **L234 CN**: 以 `(__m128h)((__v8hf)__A / (__v8hf)__B)` 从当前函数返回。
- **L235 EN**: Closes the current lexical scope or compound statement.
  **L235 CN**: 结束当前词法作用域或复合语句块。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_div_ph(__m128h __W,`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_div_ph(__m128h __W,`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L240 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L240 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。

### Lines 241-264

````c
  return (__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_div_ph(__A, __B),
                                              (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_div_ph(__mmask8 __U,
                                                                 __m128h __A,
                                                                 __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_div_ph(__A, __B),
                                              (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_min_ph(__m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_min_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      (__v16hf)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B),
      (__v16hf)__W);
}

````
- **L241 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_div_ph(__A, __B),`.
  **L241 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_div_ph(__A, __B),` 从当前函数返回。
- **L242 EN**: Executes a call or declaration centered on `statement`.
  **L242 CN**: 执行以 `statement` 为核心的调用或声明。
- **L243 EN**: Closes the current lexical scope or compound statement.
  **L243 CN**: 结束当前词法作用域或复合语句块。
- **L244 EN**: Blank line separating nearby declarations or logic blocks.
  **L244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_div_ph(__mmask8 __U,`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_div_ph(__mmask8 __U,`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L247 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L247 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L248 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_div_ph(__A, __B),`.
  **L248 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(__U, (__v8hf)_mm_div_ph(__A, __B),` 从当前函数返回。
- **L249 EN**: Executes a call or declaration centered on `statement`.
  **L249 CN**: 执行以 `statement` 为核心的调用或声明。
- **L250 EN**: Closes the current lexical scope or compound statement.
  **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L252 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L252 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L253 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_min_ph(__m256h __A, __m256h __B) {`.
  **L253 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_min_ph(__m256h __A, __m256h __B) {`。
- **L254 EN**: Returns from the current function with `(__m256h)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B)`.
  **L254 CN**: 以 `(__m256h)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B)` 从当前函数返回。
- **L255 EN**: Closes the current lexical scope or compound statement.
  **L255 CN**: 结束当前词法作用域或复合语句块。
- **L256 EN**: Blank line separating nearby declarations or logic blocks.
  **L256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L257 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L257 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L258 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_min_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`.
  **L258 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_min_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`。
- **L259 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L259 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B),`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B),`。
- **L262 EN**: Executes a call or declaration centered on `statement`.
  **L262 CN**: 执行以 `statement` 为核心的调用或声明。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 265-288

````c
static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_min_ph(__mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      (__v16hf)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B),
      (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_min_ph(__m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_min_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B),
      (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_min_ph(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B),
````
- **L265 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L265 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L266 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_min_ph(__mmask16 __U, __m256h __A, __m256h __B) {`.
  **L266 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_min_ph(__mmask16 __U, __m256h __A, __m256h __B) {`。
- **L267 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L267 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B),`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__builtin_ia32_minph256((__v16hf)__A, (__v16hf)__B),`。
- **L270 EN**: Executes a call or declaration centered on `statement`.
  **L270 CN**: 执行以 `statement` 为核心的调用或声明。
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L273 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L273 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L274 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_ph(__m128h __A, __m128h __B) {`.
  **L274 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_ph(__m128h __A, __m128h __B) {`。
- **L275 EN**: Returns from the current function with `(__m128h)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B)`.
  **L275 CN**: 以 `(__m128h)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B)` 从当前函数返回。
- **L276 EN**: Closes the current lexical scope or compound statement.
  **L276 CN**: 结束当前词法作用域或复合语句块。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L278 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L278 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L279 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_min_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L279 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_min_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L280 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L280 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, (__v8hf)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B),`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, (__v8hf)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B),`。
- **L282 EN**: Executes a call or declaration centered on `statement`.
  **L282 CN**: 执行以 `statement` 为核心的调用或声明。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L285 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L285 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L286 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_min_ph(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L286 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_min_ph(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L287 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L287 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, (__v8hf)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B),`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, (__v8hf)__builtin_ia32_minph128((__v8hf)__A, (__v8hf)__B),`。

### Lines 289-312

````c
      (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_max_ph(__m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_max_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      (__v16hf)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B),
      (__v16hf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_max_ph(__mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      (__v16hf)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B),
      (__v16hf)_mm256_setzero_ph());
}

````
- **L289 EN**: Executes a call or declaration centered on `statement`.
  **L289 CN**: 执行以 `statement` 为核心的调用或声明。
- **L290 EN**: Closes the current lexical scope or compound statement.
  **L290 CN**: 结束当前词法作用域或复合语句块。
- **L291 EN**: Blank line separating nearby declarations or logic blocks.
  **L291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L292 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L292 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L293 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_max_ph(__m256h __A, __m256h __B) {`.
  **L293 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_max_ph(__m256h __A, __m256h __B) {`。
- **L294 EN**: Returns from the current function with `(__m256h)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B)`.
  **L294 CN**: 以 `(__m256h)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B)` 从当前函数返回。
- **L295 EN**: Closes the current lexical scope or compound statement.
  **L295 CN**: 结束当前词法作用域或复合语句块。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L297 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L298 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_max_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`.
  **L298 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_max_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`。
- **L299 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L299 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B),`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B),`。
- **L302 EN**: Executes a call or declaration centered on `statement`.
  **L302 CN**: 执行以 `statement` 为核心的调用或声明。
- **L303 EN**: Closes the current lexical scope or compound statement.
  **L303 CN**: 结束当前词法作用域或复合语句块。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L305 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L305 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L306 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_max_ph(__mmask16 __U, __m256h __A, __m256h __B) {`.
  **L306 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_max_ph(__mmask16 __U, __m256h __A, __m256h __B) {`。
- **L307 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L307 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L308 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B),`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__builtin_ia32_maxph256((__v16hf)__A, (__v16hf)__B),`。
- **L310 EN**: Executes a call or declaration centered on `statement`.
  **L310 CN**: 执行以 `statement` 为核心的调用或声明。
- **L311 EN**: Closes the current lexical scope or compound statement.
  **L311 CN**: 结束当前词法作用域或复合语句块。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 313-336

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_max_ph(__m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_max_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B),
      (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_max_ph(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B),
      (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_abs_ph(__m256h __A) {
  return (__m256h)_mm256_and_epi32(_mm256_set1_epi32(0x7FFF7FFF), (__m256i)__A);
}

````
- **L313 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L313 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L314 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_ph(__m128h __A, __m128h __B) {`.
  **L314 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_ph(__m128h __A, __m128h __B) {`。
- **L315 EN**: Returns from the current function with `(__m128h)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B)`.
  **L315 CN**: 以 `(__m128h)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B)` 从当前函数返回。
- **L316 EN**: Closes the current lexical scope or compound statement.
  **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Blank line separating nearby declarations or logic blocks.
  **L317 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L318 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L318 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L319 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_max_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L319 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_max_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L320 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L320 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, (__v8hf)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B),`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, (__v8hf)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B),`。
- **L322 EN**: Executes a call or declaration centered on `statement`.
  **L322 CN**: 执行以 `statement` 为核心的调用或声明。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L325 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L325 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L326 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_max_ph(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L326 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_max_ph(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L327 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L327 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, (__v8hf)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B),`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, (__v8hf)__builtin_ia32_maxph128((__v8hf)__A, (__v8hf)__B),`。
- **L329 EN**: Executes a call or declaration centered on `statement`.
  **L329 CN**: 执行以 `statement` 为核心的调用或声明。
- **L330 EN**: Closes the current lexical scope or compound statement.
  **L330 CN**: 结束当前词法作用域或复合语句块。
- **L331 EN**: Blank line separating nearby declarations or logic blocks.
  **L331 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L332 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L332 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L333 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_abs_ph(__m256h __A) {`.
  **L333 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_abs_ph(__m256h __A) {`。
- **L334 EN**: Returns from the current function with `(__m256h)_mm256_and_epi32(_mm256_set1_epi32(0x7FFF7FFF), (__m256i)__A)`.
  **L334 CN**: 以 `(__m256h)_mm256_and_epi32(_mm256_set1_epi32(0x7FFF7FFF), (__m256i)__A)` 从当前函数返回。
- **L335 EN**: Closes the current lexical scope or compound statement.
  **L335 CN**: 结束当前词法作用域或复合语句块。
- **L336 EN**: Blank line separating nearby declarations or logic blocks.
  **L336 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 337-360

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_abs_ph(__m128h __A) {
  return (__m128h)_mm_and_epi32(_mm_set1_epi32(0x7FFF7FFF), (__m128i)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_conj_pch(__m256h __A) {
  return (__m256h)_mm256_xor_ps((__m256)__A, _mm256_set1_ps(-0.0f));
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_conj_pch(__m256h __W, __mmask8 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_selectps_256(
      (__mmask8)__U, (__v8sf)_mm256_conj_pch(__A), (__v8sf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_conj_pch(__mmask8 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_selectps_256(
      (__mmask8)__U, (__v8sf)_mm256_conj_pch(__A), (__v8sf)_mm256_setzero_ps());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_conj_pch(__m128h __A) {
  return (__m128h)_mm_xor_ps((__m128)__A, _mm_set1_ps(-0.0f));
}
````
- **L337 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L337 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L338 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_abs_ph(__m128h __A) {`.
  **L338 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_abs_ph(__m128h __A) {`。
- **L339 EN**: Returns from the current function with `(__m128h)_mm_and_epi32(_mm_set1_epi32(0x7FFF7FFF), (__m128i)__A)`.
  **L339 CN**: 以 `(__m128h)_mm_and_epi32(_mm_set1_epi32(0x7FFF7FFF), (__m128i)__A)` 从当前函数返回。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。
- **L341 EN**: Blank line separating nearby declarations or logic blocks.
  **L341 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L342 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_conj_pch(__m256h __A) {`.
  **L342 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_conj_pch(__m256h __A) {`。
- **L343 EN**: Returns from the current function with `(__m256h)_mm256_xor_ps((__m256)__A, _mm256_set1_ps(-0.0f))`.
  **L343 CN**: 以 `(__m256h)_mm256_xor_ps((__m256)__A, _mm256_set1_ps(-0.0f))` 从当前函数返回。
- **L344 EN**: Closes the current lexical scope or compound statement.
  **L344 CN**: 结束当前词法作用域或复合语句块。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L346 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L346 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L347 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_conj_pch(__m256h __W, __mmask8 __U, __m256h __A) {`.
  **L347 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_conj_pch(__m256h __W, __mmask8 __U, __m256h __A) {`。
- **L348 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectps_256(`.
  **L348 CN**: 以 `(__m256h)__builtin_ia32_selectps_256(` 从当前函数返回。
- **L349 EN**: Executes a call or declaration centered on `statement`.
  **L349 CN**: 执行以 `statement` 为核心的调用或声明。
- **L350 EN**: Closes the current lexical scope or compound statement.
  **L350 CN**: 结束当前词法作用域或复合语句块。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L352 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L352 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L353 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_conj_pch(__mmask8 __U, __m256h __A) {`.
  **L353 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_conj_pch(__mmask8 __U, __m256h __A) {`。
- **L354 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectps_256(`.
  **L354 CN**: 以 `(__m256h)__builtin_ia32_selectps_256(` 从当前函数返回。
- **L355 EN**: Executes a call or declaration centered on `statement`.
  **L355 CN**: 执行以 `statement` 为核心的调用或声明。
- **L356 EN**: Closes the current lexical scope or compound statement.
  **L356 CN**: 结束当前词法作用域或复合语句块。
- **L357 EN**: Blank line separating nearby declarations or logic blocks.
  **L357 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L358 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_conj_pch(__m128h __A) {`.
  **L358 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_conj_pch(__m128h __A) {`。
- **L359 EN**: Returns from the current function with `(__m128h)_mm_xor_ps((__m128)__A, _mm_set1_ps(-0.0f))`.
  **L359 CN**: 以 `(__m128h)_mm_xor_ps((__m128)__A, _mm_set1_ps(-0.0f))` 从当前函数返回。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_conj_pch(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A) {
  return (__m128h)__builtin_ia32_selectps_128(
      (__mmask8)__U, (__v4sf)_mm_conj_pch(__A), (__v4sf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_conj_pch(__mmask8 __U, __m128h __A) {
  return (__m128h)__builtin_ia32_selectps_128(
      (__mmask8)__U, (__v4sf)_mm_conj_pch(__A), (__v4sf)_mm_setzero_ps());
}

#define _mm256_cmp_ph_mask(a, b, p)                                            \
  ((__mmask16)__builtin_ia32_cmpph256_mask(                                    \
      (__v16hf)(__m256h)(a), (__v16hf)(__m256h)(b), (int)(p), (__mmask16)-1))

#define _mm256_mask_cmp_ph_mask(m, a, b, p)                                    \
  ((__mmask16)__builtin_ia32_cmpph256_mask(                                    \
      (__v16hf)(__m256h)(a), (__v16hf)(__m256h)(b), (int)(p), (__mmask16)(m)))

#define _mm_cmp_ph_mask(a, b, p)                                               \
  ((__mmask8)__builtin_ia32_cmpph128_mask(                                     \
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_conj_pch(__m128h __W,`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_conj_pch(__m128h __W,`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L364 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L364 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L365 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectps_128(`.
  **L365 CN**: 以 `(__m128h)__builtin_ia32_selectps_128(` 从当前函数返回。
- **L366 EN**: Executes a call or declaration centered on `statement`.
  **L366 CN**: 执行以 `statement` 为核心的调用或声明。
- **L367 EN**: Closes the current lexical scope or compound statement.
  **L367 CN**: 结束当前词法作用域或复合语句块。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L369 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L369 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L370 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_conj_pch(__mmask8 __U, __m128h __A) {`.
  **L370 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_conj_pch(__mmask8 __U, __m128h __A) {`。
- **L371 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectps_128(`.
  **L371 CN**: 以 `(__m128h)__builtin_ia32_selectps_128(` 从当前函数返回。
- **L372 EN**: Executes a call or declaration centered on `statement`.
  **L372 CN**: 执行以 `statement` 为核心的调用或声明。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L375 EN**: Defines macro `_mm256_cmp_ph_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L375 CN**: 定义宏 `_mm256_cmp_ph_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L376 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpph256_mask`.
  **L376 CN**: 继续与可调用符号 `__builtin_ia32_cmpph256_mask` 相关的逻辑。
- **L377 EN**: Continues the surrounding expression or declaration: `(__v16hf)(__m256h)(a), (__v16hf)(__m256h)(b), (int)(p), (__mmask16)-1))`.
  **L377 CN**: 继续构造周围的表达式或声明：`(__v16hf)(__m256h)(a), (__v16hf)(__m256h)(b), (int)(p), (__mmask16)-1))`。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L379 EN**: Defines macro `_mm256_mask_cmp_ph_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L379 CN**: 定义宏 `_mm256_mask_cmp_ph_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L380 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpph256_mask`.
  **L380 CN**: 继续与可调用符号 `__builtin_ia32_cmpph256_mask` 相关的逻辑。
- **L381 EN**: Continues the surrounding expression or declaration: `(__v16hf)(__m256h)(a), (__v16hf)(__m256h)(b), (int)(p), (__mmask16)(m)))`.
  **L381 CN**: 继续构造周围的表达式或声明：`(__v16hf)(__m256h)(a), (__v16hf)(__m256h)(b), (int)(p), (__mmask16)(m)))`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L383 EN**: Defines macro `_mm_cmp_ph_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L383 CN**: 定义宏 `_mm_cmp_ph_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L384 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpph128_mask`.
  **L384 CN**: 继续与可调用符号 `__builtin_ia32_cmpph128_mask` 相关的逻辑。

### Lines 385-408

````c
      (__v8hf)(__m128h)(a), (__v8hf)(__m128h)(b), (int)(p), (__mmask8)-1))

#define _mm_mask_cmp_ph_mask(m, a, b, p)                                       \
  ((__mmask8)__builtin_ia32_cmpph128_mask(                                     \
      (__v8hf)(__m128h)(a), (__v8hf)(__m128h)(b), (int)(p), (__mmask8)(m)))

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_rcp_ph(__m256h __A) {
  return (__m256h)__builtin_ia32_rcpph256_mask(
      (__v16hf)__A, (__v16hf)_mm256_undefined_ph(), (__mmask16)-1);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_rcp_ph(__m256h __W, __mmask16 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_rcpph256_mask((__v16hf)__A, (__v16hf)__W,
                                               (__mmask16)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_rcp_ph(__mmask16 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_rcpph256_mask(
      (__v16hf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rcp_ph(__m128h __A) {
````
- **L385 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(a), (__v8hf)(__m128h)(b), (int)(p), (__mmask8)-1))`.
  **L385 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(a), (__v8hf)(__m128h)(b), (int)(p), (__mmask8)-1))`。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L387 EN**: Defines macro `_mm_mask_cmp_ph_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L387 CN**: 定义宏 `_mm_mask_cmp_ph_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L388 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpph128_mask`.
  **L388 CN**: 继续与可调用符号 `__builtin_ia32_cmpph128_mask` 相关的逻辑。
- **L389 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(a), (__v8hf)(__m128h)(b), (int)(p), (__mmask8)(m)))`.
  **L389 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(a), (__v8hf)(__m128h)(b), (int)(p), (__mmask8)(m)))`。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_rcp_ph(__m256h __A) {`.
  **L391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_rcp_ph(__m256h __A) {`。
- **L392 EN**: Returns from the current function with `(__m256h)__builtin_ia32_rcpph256_mask(`.
  **L392 CN**: 以 `(__m256h)__builtin_ia32_rcpph256_mask(` 从当前函数返回。
- **L393 EN**: Executes a call or declaration centered on `statement`.
  **L393 CN**: 执行以 `statement` 为核心的调用或声明。
- **L394 EN**: Closes the current lexical scope or compound statement.
  **L394 CN**: 结束当前词法作用域或复合语句块。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L396 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L397 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_rcp_ph(__m256h __W, __mmask16 __U, __m256h __A) {`.
  **L397 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_rcp_ph(__m256h __W, __mmask16 __U, __m256h __A) {`。
- **L398 EN**: Returns from the current function with `(__m256h)__builtin_ia32_rcpph256_mask((__v16hf)__A, (__v16hf)__W,`.
  **L398 CN**: 以 `(__m256h)__builtin_ia32_rcpph256_mask((__v16hf)__A, (__v16hf)__W,` 从当前函数返回。
- **L399 EN**: Executes a call or declaration centered on `statement`.
  **L399 CN**: 执行以 `statement` 为核心的调用或声明。
- **L400 EN**: Closes the current lexical scope or compound statement.
  **L400 CN**: 结束当前词法作用域或复合语句块。
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L402 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L402 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L403 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_rcp_ph(__mmask16 __U, __m256h __A) {`.
  **L403 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_rcp_ph(__mmask16 __U, __m256h __A) {`。
- **L404 EN**: Returns from the current function with `(__m256h)__builtin_ia32_rcpph256_mask(`.
  **L404 CN**: 以 `(__m256h)__builtin_ia32_rcpph256_mask(` 从当前函数返回。
- **L405 EN**: Executes a call or declaration centered on `statement`.
  **L405 CN**: 执行以 `statement` 为核心的调用或声明。
- **L406 EN**: Closes the current lexical scope or compound statement.
  **L406 CN**: 结束当前词法作用域或复合语句块。
- **L407 EN**: Blank line separating nearby declarations or logic blocks.
  **L407 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L408 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rcp_ph(__m128h __A) {`.
  **L408 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rcp_ph(__m128h __A) {`。

### Lines 409-432

````c
  return (__m128h)__builtin_ia32_rcpph128_mask(
      (__v8hf)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rcp_ph(__m128h __W,
                                                                __mmask8 __U,
                                                                __m128h __A) {
  return (__m128h)__builtin_ia32_rcpph128_mask((__v8hf)__A, (__v8hf)__W,
                                               (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_rcp_ph(__mmask8 __U,
                                                                 __m128h __A) {
  return (__m128h)__builtin_ia32_rcpph128_mask(
      (__v8hf)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_rsqrt_ph(__m256h __A) {
  return (__m256h)__builtin_ia32_rsqrtph256_mask(
      (__v16hf)__A, (__v16hf)_mm256_undefined_ph(), (__mmask16)-1);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_rsqrt_ph(__m256h __W, __mmask16 __U, __m256h __A) {
````
- **L409 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rcpph128_mask(`.
  **L409 CN**: 以 `(__m128h)__builtin_ia32_rcpph128_mask(` 从当前函数返回。
- **L410 EN**: Executes a call or declaration centered on `statement`.
  **L410 CN**: 执行以 `statement` 为核心的调用或声明。
- **L411 EN**: Closes the current lexical scope or compound statement.
  **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rcp_ph(__m128h __W,`.
  **L413 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rcp_ph(__m128h __W,`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L415 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L415 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L416 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rcpph128_mask((__v8hf)__A, (__v8hf)__W,`.
  **L416 CN**: 以 `(__m128h)__builtin_ia32_rcpph128_mask((__v8hf)__A, (__v8hf)__W,` 从当前函数返回。
- **L417 EN**: Executes a call or declaration centered on `statement`.
  **L417 CN**: 执行以 `statement` 为核心的调用或声明。
- **L418 EN**: Closes the current lexical scope or compound statement.
  **L418 CN**: 结束当前词法作用域或复合语句块。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_rcp_ph(__mmask8 __U,`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_rcp_ph(__mmask8 __U,`。
- **L421 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L421 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L422 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rcpph128_mask(`.
  **L422 CN**: 以 `(__m128h)__builtin_ia32_rcpph128_mask(` 从当前函数返回。
- **L423 EN**: Executes a call or declaration centered on `statement`.
  **L423 CN**: 执行以 `statement` 为核心的调用或声明。
- **L424 EN**: Closes the current lexical scope or compound statement.
  **L424 CN**: 结束当前词法作用域或复合语句块。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L426 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_rsqrt_ph(__m256h __A) {`.
  **L426 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_rsqrt_ph(__m256h __A) {`。
- **L427 EN**: Returns from the current function with `(__m256h)__builtin_ia32_rsqrtph256_mask(`.
  **L427 CN**: 以 `(__m256h)__builtin_ia32_rsqrtph256_mask(` 从当前函数返回。
- **L428 EN**: Executes a call or declaration centered on `statement`.
  **L428 CN**: 执行以 `statement` 为核心的调用或声明。
- **L429 EN**: Closes the current lexical scope or compound statement.
  **L429 CN**: 结束当前词法作用域或复合语句块。
- **L430 EN**: Blank line separating nearby declarations or logic blocks.
  **L430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L431 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L431 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L432 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_rsqrt_ph(__m256h __W, __mmask16 __U, __m256h __A) {`.
  **L432 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_rsqrt_ph(__m256h __W, __mmask16 __U, __m256h __A) {`。

### Lines 433-456

````c
  return (__m256h)__builtin_ia32_rsqrtph256_mask((__v16hf)__A, (__v16hf)__W,
                                                 (__mmask16)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_rsqrt_ph(__mmask16 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_rsqrtph256_mask(
      (__v16hf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rsqrt_ph(__m128h __A) {
  return (__m128h)__builtin_ia32_rsqrtph128_mask(
      (__v8hf)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rsqrt_ph(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A) {
  return (__m128h)__builtin_ia32_rsqrtph128_mask((__v8hf)__A, (__v8hf)__W,
                                                 (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_rsqrt_ph(__mmask8 __U, __m128h __A) {
````
- **L433 EN**: Returns from the current function with `(__m256h)__builtin_ia32_rsqrtph256_mask((__v16hf)__A, (__v16hf)__W,`.
  **L433 CN**: 以 `(__m256h)__builtin_ia32_rsqrtph256_mask((__v16hf)__A, (__v16hf)__W,` 从当前函数返回。
- **L434 EN**: Executes a call or declaration centered on `statement`.
  **L434 CN**: 执行以 `statement` 为核心的调用或声明。
- **L435 EN**: Closes the current lexical scope or compound statement.
  **L435 CN**: 结束当前词法作用域或复合语句块。
- **L436 EN**: Blank line separating nearby declarations or logic blocks.
  **L436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L437 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L437 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L438 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_rsqrt_ph(__mmask16 __U, __m256h __A) {`.
  **L438 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_rsqrt_ph(__mmask16 __U, __m256h __A) {`。
- **L439 EN**: Returns from the current function with `(__m256h)__builtin_ia32_rsqrtph256_mask(`.
  **L439 CN**: 以 `(__m256h)__builtin_ia32_rsqrtph256_mask(` 从当前函数返回。
- **L440 EN**: Executes a call or declaration centered on `statement`.
  **L440 CN**: 执行以 `statement` 为核心的调用或声明。
- **L441 EN**: Closes the current lexical scope or compound statement.
  **L441 CN**: 结束当前词法作用域或复合语句块。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L443 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rsqrt_ph(__m128h __A) {`.
  **L443 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rsqrt_ph(__m128h __A) {`。
- **L444 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rsqrtph128_mask(`.
  **L444 CN**: 以 `(__m128h)__builtin_ia32_rsqrtph128_mask(` 从当前函数返回。
- **L445 EN**: Executes a call or declaration centered on `statement`.
  **L445 CN**: 执行以 `statement` 为核心的调用或声明。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rsqrt_ph(__m128h __W,`.
  **L448 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rsqrt_ph(__m128h __W,`。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L450 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L450 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L451 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rsqrtph128_mask((__v8hf)__A, (__v8hf)__W,`.
  **L451 CN**: 以 `(__m128h)__builtin_ia32_rsqrtph128_mask((__v8hf)__A, (__v8hf)__W,` 从当前函数返回。
- **L452 EN**: Executes a call or declaration centered on `statement`.
  **L452 CN**: 执行以 `statement` 为核心的调用或声明。
- **L453 EN**: Closes the current lexical scope or compound statement.
  **L453 CN**: 结束当前词法作用域或复合语句块。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L455 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L455 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L456 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_rsqrt_ph(__mmask8 __U, __m128h __A) {`.
  **L456 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_rsqrt_ph(__mmask8 __U, __m128h __A) {`。

### Lines 457-480

````c
  return (__m128h)__builtin_ia32_rsqrtph128_mask(
      (__v8hf)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_getexp_ph(__m128h __A) {
  return (__m128h)__builtin_ia32_getexpph128_mask(
      (__v8hf)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_getexp_ph(__m128h __W, __mmask8 __U, __m128h __A) {
  return (__m128h)__builtin_ia32_getexpph128_mask((__v8hf)__A, (__v8hf)__W,
                                                  (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_getexp_ph(__mmask8 __U, __m128h __A) {
  return (__m128h)__builtin_ia32_getexpph128_mask(
      (__v8hf)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_getexp_ph(__m256h __A) {
  return (__m256h)__builtin_ia32_getexpph256_mask(
      (__v16hf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1);
````
- **L457 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rsqrtph128_mask(`.
  **L457 CN**: 以 `(__m128h)__builtin_ia32_rsqrtph128_mask(` 从当前函数返回。
- **L458 EN**: Executes a call or declaration centered on `statement`.
  **L458 CN**: 执行以 `statement` 为核心的调用或声明。
- **L459 EN**: Closes the current lexical scope or compound statement.
  **L459 CN**: 结束当前词法作用域或复合语句块。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L461 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_getexp_ph(__m128h __A) {`.
  **L461 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_getexp_ph(__m128h __A) {`。
- **L462 EN**: Returns from the current function with `(__m128h)__builtin_ia32_getexpph128_mask(`.
  **L462 CN**: 以 `(__m128h)__builtin_ia32_getexpph128_mask(` 从当前函数返回。
- **L463 EN**: Executes a call or declaration centered on `statement`.
  **L463 CN**: 执行以 `statement` 为核心的调用或声明。
- **L464 EN**: Closes the current lexical scope or compound statement.
  **L464 CN**: 结束当前词法作用域或复合语句块。
- **L465 EN**: Blank line separating nearby declarations or logic blocks.
  **L465 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L466 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L466 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L467 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_getexp_ph(__m128h __W, __mmask8 __U, __m128h __A) {`.
  **L467 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_getexp_ph(__m128h __W, __mmask8 __U, __m128h __A) {`。
- **L468 EN**: Returns from the current function with `(__m128h)__builtin_ia32_getexpph128_mask((__v8hf)__A, (__v8hf)__W,`.
  **L468 CN**: 以 `(__m128h)__builtin_ia32_getexpph128_mask((__v8hf)__A, (__v8hf)__W,` 从当前函数返回。
- **L469 EN**: Executes a call or declaration centered on `statement`.
  **L469 CN**: 执行以 `statement` 为核心的调用或声明。
- **L470 EN**: Closes the current lexical scope or compound statement.
  **L470 CN**: 结束当前词法作用域或复合语句块。
- **L471 EN**: Blank line separating nearby declarations or logic blocks.
  **L471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L472 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L472 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L473 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_getexp_ph(__mmask8 __U, __m128h __A) {`.
  **L473 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_getexp_ph(__mmask8 __U, __m128h __A) {`。
- **L474 EN**: Returns from the current function with `(__m128h)__builtin_ia32_getexpph128_mask(`.
  **L474 CN**: 以 `(__m128h)__builtin_ia32_getexpph128_mask(` 从当前函数返回。
- **L475 EN**: Executes a call or declaration centered on `statement`.
  **L475 CN**: 执行以 `statement` 为核心的调用或声明。
- **L476 EN**: Closes the current lexical scope or compound statement.
  **L476 CN**: 结束当前词法作用域或复合语句块。
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L478 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_getexp_ph(__m256h __A) {`.
  **L478 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_getexp_ph(__m256h __A) {`。
- **L479 EN**: Returns from the current function with `(__m256h)__builtin_ia32_getexpph256_mask(`.
  **L479 CN**: 以 `(__m256h)__builtin_ia32_getexpph256_mask(` 从当前函数返回。
- **L480 EN**: Executes a call or declaration centered on `statement`.
  **L480 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 481-504

````c
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_getexp_ph(__m256h __W, __mmask16 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_getexpph256_mask((__v16hf)__A, (__v16hf)__W,
                                                  (__mmask16)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_getexp_ph(__mmask16 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_getexpph256_mask(
      (__v16hf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U);
}

#define _mm_getmant_ph(A, B, C)                                                \
  ((__m128h)__builtin_ia32_getmantph128_mask(                                  \
      (__v8hf)(__m128h)(A), (int)(((C) << 2) | (B)), (__v8hf)_mm_setzero_ph(), \
      (__mmask8)-1))

#define _mm_mask_getmant_ph(W, U, A, B, C)                                     \
  ((__m128h)__builtin_ia32_getmantph128_mask(                                  \
      (__v8hf)(__m128h)(A), (int)(((C) << 2) | (B)), (__v8hf)(__m128h)(W),     \
      (__mmask8)(U)))

````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L483 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L483 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L484 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_getexp_ph(__m256h __W, __mmask16 __U, __m256h __A) {`.
  **L484 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_getexp_ph(__m256h __W, __mmask16 __U, __m256h __A) {`。
- **L485 EN**: Returns from the current function with `(__m256h)__builtin_ia32_getexpph256_mask((__v16hf)__A, (__v16hf)__W,`.
  **L485 CN**: 以 `(__m256h)__builtin_ia32_getexpph256_mask((__v16hf)__A, (__v16hf)__W,` 从当前函数返回。
- **L486 EN**: Executes a call or declaration centered on `statement`.
  **L486 CN**: 执行以 `statement` 为核心的调用或声明。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L489 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L489 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L490 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_getexp_ph(__mmask16 __U, __m256h __A) {`.
  **L490 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_getexp_ph(__mmask16 __U, __m256h __A) {`。
- **L491 EN**: Returns from the current function with `(__m256h)__builtin_ia32_getexpph256_mask(`.
  **L491 CN**: 以 `(__m256h)__builtin_ia32_getexpph256_mask(` 从当前函数返回。
- **L492 EN**: Executes a call or declaration centered on `statement`.
  **L492 CN**: 执行以 `statement` 为核心的调用或声明。
- **L493 EN**: Closes the current lexical scope or compound statement.
  **L493 CN**: 结束当前词法作用域或复合语句块。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L495 EN**: Defines macro `_mm_getmant_ph(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L495 CN**: 定义宏 `_mm_getmant_ph(A, B, C)`，用于条件编译、简写或 API 生成。
- **L496 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph128_mask`.
  **L496 CN**: 继续与可调用符号 `__builtin_ia32_getmantph128_mask` 相关的逻辑。
- **L497 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L497 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L498 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L498 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L500 EN**: Defines macro `_mm_mask_getmant_ph(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L500 CN**: 定义宏 `_mm_mask_getmant_ph(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L501 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph128_mask`.
  **L501 CN**: 继续与可调用符号 `__builtin_ia32_getmantph128_mask` 相关的逻辑。
- **L502 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (int)(((C) << 2) | (B)), (__v8hf)(__m128h)(W),     \`.
  **L502 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (int)(((C) << 2) | (B)), (__v8hf)(__m128h)(W),     \`。
- **L503 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L503 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L504 EN**: Blank line separating nearby declarations or logic blocks.
  **L504 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 505-528

````c
#define _mm_maskz_getmant_ph(U, A, B, C)                                       \
  ((__m128h)__builtin_ia32_getmantph128_mask(                                  \
      (__v8hf)(__m128h)(A), (int)(((C) << 2) | (B)), (__v8hf)_mm_setzero_ph(), \
      (__mmask8)(U)))

#define _mm256_getmant_ph(A, B, C)                                             \
  ((__m256h)__builtin_ia32_getmantph256_mask(                                  \
      (__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)),                          \
      (__v16hf)_mm256_setzero_ph(), (__mmask16)-1))

#define _mm256_mask_getmant_ph(W, U, A, B, C)                                  \
  ((__m256h)__builtin_ia32_getmantph256_mask(                                  \
      (__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)), (__v16hf)(__m256h)(W),   \
      (__mmask16)(U)))

#define _mm256_maskz_getmant_ph(U, A, B, C)                                    \
  ((__m256h)__builtin_ia32_getmantph256_mask(                                  \
      (__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)),                          \
      (__v16hf)_mm256_setzero_ph(), (__mmask16)(U)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_scalef_ph(__m128h __A,
                                                              __m128h __B) {
  return (__m128h)__builtin_ia32_scalefph128_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1);
````
- **L505 EN**: Defines macro `_mm_maskz_getmant_ph(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L505 CN**: 定义宏 `_mm_maskz_getmant_ph(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L506 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph128_mask`.
  **L506 CN**: 继续与可调用符号 `__builtin_ia32_getmantph128_mask` 相关的逻辑。
- **L507 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L507 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L508 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L508 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L510 EN**: Defines macro `_mm256_getmant_ph(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L510 CN**: 定义宏 `_mm256_getmant_ph(A, B, C)`，用于条件编译、简写或 API 生成。
- **L511 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph256_mask`.
  **L511 CN**: 继续与可调用符号 `__builtin_ia32_getmantph256_mask` 相关的逻辑。
- **L512 EN**: Continues the surrounding expression or declaration: `(__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)),                          \`.
  **L512 CN**: 继续构造周围的表达式或声明：`(__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)),                          \`。
- **L513 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L513 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L514 EN**: Blank line separating nearby declarations or logic blocks.
  **L514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L515 EN**: Defines macro `_mm256_mask_getmant_ph(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L515 CN**: 定义宏 `_mm256_mask_getmant_ph(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L516 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph256_mask`.
  **L516 CN**: 继续与可调用符号 `__builtin_ia32_getmantph256_mask` 相关的逻辑。
- **L517 EN**: Continues the surrounding expression or declaration: `(__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)), (__v16hf)(__m256h)(W),   \`.
  **L517 CN**: 继续构造周围的表达式或声明：`(__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)), (__v16hf)(__m256h)(W),   \`。
- **L518 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U)))`.
  **L518 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U)))`。
- **L519 EN**: Blank line separating nearby declarations or logic blocks.
  **L519 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L520 EN**: Defines macro `_mm256_maskz_getmant_ph(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L520 CN**: 定义宏 `_mm256_maskz_getmant_ph(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L521 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph256_mask`.
  **L521 CN**: 继续与可调用符号 `__builtin_ia32_getmantph256_mask` 相关的逻辑。
- **L522 EN**: Continues the surrounding expression or declaration: `(__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)),                          \`.
  **L522 CN**: 继续构造周围的表达式或声明：`(__v16hf)(__m256h)(A), (int)(((C) << 2) | (B)),                          \`。
- **L523 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L523 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_scalef_ph(__m128h __A,`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_scalef_ph(__m128h __A,`。
- **L526 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L526 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L527 EN**: Returns from the current function with `(__m128h)__builtin_ia32_scalefph128_mask(`.
  **L527 CN**: 以 `(__m128h)__builtin_ia32_scalefph128_mask(` 从当前函数返回。
- **L528 EN**: Executes a call or declaration centered on `statement`.
  **L528 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 529-552

````c
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_scalef_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_scalefph128_mask((__v8hf)__A, (__v8hf)__B,
                                                  (__v8hf)__W, (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_scalef_ph(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_scalefph128_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_scalef_ph(__m256h __A,
                                                                 __m256h __B) {
  return (__m256h)__builtin_ia32_scalefph256_mask(
      (__v16hf)__A, (__v16hf)__B, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_scalef_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_scalefph256_mask((__v16hf)__A, (__v16hf)__B,
                                                  (__v16hf)__W, (__mmask16)__U);
````
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L531 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L531 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L532 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_scalef_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L532 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_scalef_ph(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L533 EN**: Returns from the current function with `(__m128h)__builtin_ia32_scalefph128_mask((__v8hf)__A, (__v8hf)__B,`.
  **L533 CN**: 以 `(__m128h)__builtin_ia32_scalefph128_mask((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L534 EN**: Executes a call or declaration centered on `statement`.
  **L534 CN**: 执行以 `statement` 为核心的调用或声明。
- **L535 EN**: Closes the current lexical scope or compound statement.
  **L535 CN**: 结束当前词法作用域或复合语句块。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L537 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L538 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_scalef_ph(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L538 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_scalef_ph(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L539 EN**: Returns from the current function with `(__m128h)__builtin_ia32_scalefph128_mask(`.
  **L539 CN**: 以 `(__m128h)__builtin_ia32_scalefph128_mask(` 从当前函数返回。
- **L540 EN**: Executes a call or declaration centered on `statement`.
  **L540 CN**: 执行以 `statement` 为核心的调用或声明。
- **L541 EN**: Closes the current lexical scope or compound statement.
  **L541 CN**: 结束当前词法作用域或复合语句块。
- **L542 EN**: Blank line separating nearby declarations or logic blocks.
  **L542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_scalef_ph(__m256h __A,`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_scalef_ph(__m256h __A,`。
- **L544 EN**: Continues the surrounding expression or declaration: `__m256h __B) {`.
  **L544 CN**: 继续构造周围的表达式或声明：`__m256h __B) {`。
- **L545 EN**: Returns from the current function with `(__m256h)__builtin_ia32_scalefph256_mask(`.
  **L545 CN**: 以 `(__m256h)__builtin_ia32_scalefph256_mask(` 从当前函数返回。
- **L546 EN**: Executes a call or declaration centered on `statement`.
  **L546 CN**: 执行以 `statement` 为核心的调用或声明。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Blank line separating nearby declarations or logic blocks.
  **L548 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L549 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L549 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L550 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_scalef_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`.
  **L550 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_scalef_ph(__m256h __W, __mmask16 __U, __m256h __A, __m256h __B) {`。
- **L551 EN**: Returns from the current function with `(__m256h)__builtin_ia32_scalefph256_mask((__v16hf)__A, (__v16hf)__B,`.
  **L551 CN**: 以 `(__m256h)__builtin_ia32_scalefph256_mask((__v16hf)__A, (__v16hf)__B,` 从当前函数返回。
- **L552 EN**: Executes a call or declaration centered on `statement`.
  **L552 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 553-576

````c
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_scalef_ph(__mmask16 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_scalefph256_mask(
      (__v16hf)__A, (__v16hf)__B, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U);
}

#define _mm_roundscale_ph(A, imm)                                              \
  ((__m128h)__builtin_ia32_rndscaleph_128_mask(                                \
      (__v8hf)(__m128h)(A), (int)(imm), (__v8hf)_mm_setzero_ph(),              \
      (__mmask8)-1))

#define _mm_mask_roundscale_ph(W, U, A, imm)                                   \
  ((__m128h)__builtin_ia32_rndscaleph_128_mask(                                \
      (__v8hf)(__m128h)(A), (int)(imm), (__v8hf)(__m128h)(W), (__mmask8)(U)))

#define _mm_maskz_roundscale_ph(U, A, imm)                                     \
  ((__m128h)__builtin_ia32_rndscaleph_128_mask(                                \
      (__v8hf)(__m128h)(A), (int)(imm), (__v8hf)_mm_setzero_ph(),              \
      (__mmask8)(U)))

#define _mm256_roundscale_ph(A, imm)                                           \
  ((__m256h)__builtin_ia32_rndscaleph_256_mask(                                \
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L555 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L556 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_scalef_ph(__mmask16 __U, __m256h __A, __m256h __B) {`.
  **L556 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_scalef_ph(__mmask16 __U, __m256h __A, __m256h __B) {`。
- **L557 EN**: Returns from the current function with `(__m256h)__builtin_ia32_scalefph256_mask(`.
  **L557 CN**: 以 `(__m256h)__builtin_ia32_scalefph256_mask(` 从当前函数返回。
- **L558 EN**: Executes a call or declaration centered on `statement`.
  **L558 CN**: 执行以 `statement` 为核心的调用或声明。
- **L559 EN**: Closes the current lexical scope or compound statement.
  **L559 CN**: 结束当前词法作用域或复合语句块。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L561 EN**: Defines macro `_mm_roundscale_ph(A, imm)` for conditional compilation, shorthand, or API generation.
  **L561 CN**: 定义宏 `_mm_roundscale_ph(A, imm)`，用于条件编译、简写或 API 生成。
- **L562 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_128_mask`.
  **L562 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_128_mask` 相关的逻辑。
- **L563 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L563 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L564 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L564 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L566 EN**: Defines macro `_mm_mask_roundscale_ph(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L566 CN**: 定义宏 `_mm_mask_roundscale_ph(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L567 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_128_mask`.
  **L567 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_128_mask` 相关的逻辑。
- **L568 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (int)(imm), (__v8hf)(__m128h)(W), (__mmask8)(U)))`.
  **L568 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (int)(imm), (__v8hf)(__m128h)(W), (__mmask8)(U)))`。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L570 EN**: Defines macro `_mm_maskz_roundscale_ph(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L570 CN**: 定义宏 `_mm_maskz_roundscale_ph(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L571 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_128_mask`.
  **L571 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_128_mask` 相关的逻辑。
- **L572 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L572 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L573 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L573 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Defines macro `_mm256_roundscale_ph(A, imm)` for conditional compilation, shorthand, or API generation.
  **L575 CN**: 定义宏 `_mm256_roundscale_ph(A, imm)`，用于条件编译、简写或 API 生成。
- **L576 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_256_mask`.
  **L576 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_256_mask` 相关的逻辑。

### Lines 577-600

````c
      (__v16hf)(__m256h)(A), (int)(imm), (__v16hf)_mm256_setzero_ph(),         \
      (__mmask16)-1))

#define _mm256_mask_roundscale_ph(W, U, A, imm)                                \
  ((__m256h)__builtin_ia32_rndscaleph_256_mask(                                \
      (__v16hf)(__m256h)(A), (int)(imm), (__v16hf)(__m256h)(W),                \
      (__mmask16)(U)))

#define _mm256_maskz_roundscale_ph(U, A, imm)                                  \
  ((__m256h)__builtin_ia32_rndscaleph_256_mask(                                \
      (__v16hf)(__m256h)(A), (int)(imm), (__v16hf)_mm256_setzero_ph(),         \
      (__mmask16)(U)))

#define _mm_reduce_ph(A, imm)                                                  \
  ((__m128h)__builtin_ia32_reduceph128_mask((__v8hf)(__m128h)(A), (int)(imm),  \
                                            (__v8hf)_mm_setzero_ph(),          \
                                            (__mmask8)-1))

#define _mm_mask_reduce_ph(W, U, A, imm)                                       \
  ((__m128h)__builtin_ia32_reduceph128_mask(                                   \
      (__v8hf)(__m128h)(A), (int)(imm), (__v8hf)(__m128h)(W), (__mmask8)(U)))

#define _mm_maskz_reduce_ph(U, A, imm)                                         \
  ((__m128h)__builtin_ia32_reduceph128_mask((__v8hf)(__m128h)(A), (int)(imm),  \
````
- **L577 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L577 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L578 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1))`.
  **L578 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1))`。
- **L579 EN**: Blank line separating nearby declarations or logic blocks.
  **L579 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L580 EN**: Defines macro `_mm256_mask_roundscale_ph(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L580 CN**: 定义宏 `_mm256_mask_roundscale_ph(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L581 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_256_mask`.
  **L581 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_256_mask` 相关的逻辑。
- **L582 EN**: Continues the surrounding expression or declaration: `(__v16hf)(__m256h)(A), (int)(imm), (__v16hf)(__m256h)(W),                \`.
  **L582 CN**: 继续构造周围的表达式或声明：`(__v16hf)(__m256h)(A), (int)(imm), (__v16hf)(__m256h)(W),                \`。
- **L583 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U)))`.
  **L583 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U)))`。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L585 EN**: Defines macro `_mm256_maskz_roundscale_ph(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L585 CN**: 定义宏 `_mm256_maskz_roundscale_ph(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L586 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_256_mask`.
  **L586 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_256_mask` 相关的逻辑。
- **L587 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L587 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L588 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U)))`.
  **L588 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U)))`。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L590 EN**: Defines macro `_mm_reduce_ph(A, imm)` for conditional compilation, shorthand, or API generation.
  **L590 CN**: 定义宏 `_mm_reduce_ph(A, imm)`，用于条件编译、简写或 API 生成。
- **L591 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph128_mask`.
  **L591 CN**: 继续与可调用符号 `__builtin_ia32_reduceph128_mask` 相关的逻辑。
- **L592 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L592 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L593 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L593 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L594 EN**: Blank line separating nearby declarations or logic blocks.
  **L594 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L595 EN**: Defines macro `_mm_mask_reduce_ph(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L595 CN**: 定义宏 `_mm_mask_reduce_ph(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L596 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph128_mask`.
  **L596 CN**: 继续与可调用符号 `__builtin_ia32_reduceph128_mask` 相关的逻辑。
- **L597 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (int)(imm), (__v8hf)(__m128h)(W), (__mmask8)(U)))`.
  **L597 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (int)(imm), (__v8hf)(__m128h)(W), (__mmask8)(U)))`。
- **L598 EN**: Blank line separating nearby declarations or logic blocks.
  **L598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L599 EN**: Defines macro `_mm_maskz_reduce_ph(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L599 CN**: 定义宏 `_mm_maskz_reduce_ph(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L600 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph128_mask`.
  **L600 CN**: 继续与可调用符号 `__builtin_ia32_reduceph128_mask` 相关的逻辑。

### Lines 601-624

````c
                                            (__v8hf)_mm_setzero_ph(),          \
                                            (__mmask8)(U)))

#define _mm256_reduce_ph(A, imm)                                               \
  ((__m256h)__builtin_ia32_reduceph256_mask((__v16hf)(__m256h)(A), (int)(imm), \
                                            (__v16hf)_mm256_setzero_ph(),      \
                                            (__mmask16)-1))

#define _mm256_mask_reduce_ph(W, U, A, imm)                                    \
  ((__m256h)__builtin_ia32_reduceph256_mask((__v16hf)(__m256h)(A), (int)(imm), \
                                            (__v16hf)(__m256h)(W),             \
                                            (__mmask16)(U)))

#define _mm256_maskz_reduce_ph(U, A, imm)                                      \
  ((__m256h)__builtin_ia32_reduceph256_mask((__v16hf)(__m256h)(A), (int)(imm), \
                                            (__v16hf)_mm256_setzero_ph(),      \
                                            (__mmask16)(U)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sqrt_ph(__m128h __a) {
  return __builtin_elementwise_sqrt(__a);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sqrt_ph(__m128h __W,
                                                                 __mmask8 __U,
````
- **L601 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L601 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L602 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L602 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L604 EN**: Defines macro `_mm256_reduce_ph(A, imm)` for conditional compilation, shorthand, or API generation.
  **L604 CN**: 定义宏 `_mm256_reduce_ph(A, imm)`，用于条件编译、简写或 API 生成。
- **L605 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph256_mask`.
  **L605 CN**: 继续与可调用符号 `__builtin_ia32_reduceph256_mask` 相关的逻辑。
- **L606 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L606 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L607 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1))`.
  **L607 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1))`。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Defines macro `_mm256_mask_reduce_ph(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L609 CN**: 定义宏 `_mm256_mask_reduce_ph(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L610 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph256_mask`.
  **L610 CN**: 继续与可调用符号 `__builtin_ia32_reduceph256_mask` 相关的逻辑。
- **L611 EN**: Continues the surrounding expression or declaration: `(__v16hf)(__m256h)(W),             \`.
  **L611 CN**: 继续构造周围的表达式或声明：`(__v16hf)(__m256h)(W),             \`。
- **L612 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U)))`.
  **L612 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U)))`。
- **L613 EN**: Blank line separating nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L614 EN**: Defines macro `_mm256_maskz_reduce_ph(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L614 CN**: 定义宏 `_mm256_maskz_reduce_ph(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L615 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph256_mask`.
  **L615 CN**: 继续与可调用符号 `__builtin_ia32_reduceph256_mask` 相关的逻辑。
- **L616 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L616 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L617 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U)))`.
  **L617 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U)))`。
- **L618 EN**: Blank line separating nearby declarations or logic blocks.
  **L618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L619 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sqrt_ph(__m128h __a) {`.
  **L619 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sqrt_ph(__m128h __a) {`。
- **L620 EN**: Returns from the current function with `__builtin_elementwise_sqrt(__a)`.
  **L620 CN**: 以 `__builtin_elementwise_sqrt(__a)` 从当前函数返回。
- **L621 EN**: Closes the current lexical scope or compound statement.
  **L621 CN**: 结束当前词法作用域或复合语句块。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sqrt_ph(__m128h __W,`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sqrt_ph(__m128h __W,`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。

### Lines 625-648

````c
                                                                 __m128h __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm_sqrt_ph(__A), (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sqrt_ph(__mmask8 __U,
                                                                  __m128h __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm_sqrt_ph(__A), (__v8hf)_mm_setzero_ph());
}

static __inline __m256h __DEFAULT_FN_ATTRS256 _mm256_sqrt_ph(__m256h __a) {
  return __builtin_elementwise_sqrt(__a);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_sqrt_ph(__m256h __W, __mmask16 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U, (__v16hf)_mm256_sqrt_ph(__A), (__v16hf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_sqrt_ph(__mmask16 __U, __m256h __A) {
  return (__m256h)__builtin_ia32_selectph_256((__mmask16)__U,
````
- **L625 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L625 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L626 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L626 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L627 EN**: Executes a call or declaration centered on `statement`.
  **L627 CN**: 执行以 `statement` 为核心的调用或声明。
- **L628 EN**: Closes the current lexical scope or compound statement.
  **L628 CN**: 结束当前词法作用域或复合语句块。
- **L629 EN**: Blank line separating nearby declarations or logic blocks.
  **L629 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sqrt_ph(__mmask8 __U,`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sqrt_ph(__mmask8 __U,`。
- **L631 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L631 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L632 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L632 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L633 EN**: Executes a call or declaration centered on `statement`.
  **L633 CN**: 执行以 `statement` 为核心的调用或声明。
- **L634 EN**: Closes the current lexical scope or compound statement.
  **L634 CN**: 结束当前词法作用域或复合语句块。
- **L635 EN**: Blank line separating nearby declarations or logic blocks.
  **L635 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L636 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline __m256h __DEFAULT_FN_ATTRS256 _mm256_sqrt_ph(__m256h __a) {`.
  **L636 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline __m256h __DEFAULT_FN_ATTRS256 _mm256_sqrt_ph(__m256h __a) {`。
- **L637 EN**: Returns from the current function with `__builtin_elementwise_sqrt(__a)`.
  **L637 CN**: 以 `__builtin_elementwise_sqrt(__a)` 从当前函数返回。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Blank line separating nearby declarations or logic blocks.
  **L639 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L640 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L640 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L641 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_sqrt_ph(__m256h __W, __mmask16 __U, __m256h __A) {`.
  **L641 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_sqrt_ph(__m256h __W, __mmask16 __U, __m256h __A) {`。
- **L642 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L642 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L643 EN**: Executes a call or declaration centered on `statement`.
  **L643 CN**: 执行以 `statement` 为核心的调用或声明。
- **L644 EN**: Closes the current lexical scope or compound statement.
  **L644 CN**: 结束当前词法作用域或复合语句块。
- **L645 EN**: Blank line separating nearby declarations or logic blocks.
  **L645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L646 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L646 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L647 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_sqrt_ph(__mmask16 __U, __m256h __A) {`.
  **L647 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_sqrt_ph(__mmask16 __U, __m256h __A) {`。
- **L648 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256((__mmask16)__U,`.
  **L648 CN**: 以 `(__m256h)__builtin_ia32_selectph_256((__mmask16)__U,` 从当前函数返回。

### Lines 649-672

````c
                                              (__v16hf)_mm256_sqrt_ph(__A),
                                              (__v16hf)_mm256_setzero_ph());
}

#define _mm_mask_fpclass_ph_mask(U, A, imm)                                    \
  ((__mmask8)__builtin_ia32_fpclassph128_mask((__v8hf)(__m128h)(A),            \
                                              (int)(imm), (__mmask8)(U)))

#define _mm_fpclass_ph_mask(A, imm)                                            \
  ((__mmask8)__builtin_ia32_fpclassph128_mask((__v8hf)(__m128h)(A),            \
                                              (int)(imm), (__mmask8)-1))

#define _mm256_mask_fpclass_ph_mask(U, A, imm)                                 \
  ((__mmask16)__builtin_ia32_fpclassph256_mask((__v16hf)(__m256h)(A),          \
                                               (int)(imm), (__mmask16)(U)))

#define _mm256_fpclass_ph_mask(A, imm)                                         \
  ((__mmask16)__builtin_ia32_fpclassph256_mask((__v16hf)(__m256h)(A),          \
                                               (int)(imm), (__mmask16)-1))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtpd_ph(__m128d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph128_mask(
      (__v2df)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}
````
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)_mm256_sqrt_ph(__A),`.
  **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)_mm256_sqrt_ph(__A),`。
- **L650 EN**: Executes a call or declaration centered on `statement`.
  **L650 CN**: 执行以 `statement` 为核心的调用或声明。
- **L651 EN**: Closes the current lexical scope or compound statement.
  **L651 CN**: 结束当前词法作用域或复合语句块。
- **L652 EN**: Blank line separating nearby declarations or logic blocks.
  **L652 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L653 EN**: Defines macro `_mm_mask_fpclass_ph_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L653 CN**: 定义宏 `_mm_mask_fpclass_ph_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L654 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassph128_mask`.
  **L654 CN**: 继续与可调用符号 `__builtin_ia32_fpclassph128_mask` 相关的逻辑。
- **L655 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask8)(U)))`.
  **L655 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask8)(U)))`。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Defines macro `_mm_fpclass_ph_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L657 CN**: 定义宏 `_mm_fpclass_ph_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L658 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassph128_mask`.
  **L658 CN**: 继续与可调用符号 `__builtin_ia32_fpclassph128_mask` 相关的逻辑。
- **L659 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask8)-1))`.
  **L659 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask8)-1))`。
- **L660 EN**: Blank line separating nearby declarations or logic blocks.
  **L660 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L661 EN**: Defines macro `_mm256_mask_fpclass_ph_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L661 CN**: 定义宏 `_mm256_mask_fpclass_ph_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L662 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassph256_mask`.
  **L662 CN**: 继续与可调用符号 `__builtin_ia32_fpclassph256_mask` 相关的逻辑。
- **L663 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask16)(U)))`.
  **L663 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask16)(U)))`。
- **L664 EN**: Blank line separating nearby declarations or logic blocks.
  **L664 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L665 EN**: Defines macro `_mm256_fpclass_ph_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L665 CN**: 定义宏 `_mm256_fpclass_ph_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L666 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassph256_mask`.
  **L666 CN**: 继续与可调用符号 `__builtin_ia32_fpclassph256_mask` 相关的逻辑。
- **L667 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask16)-1))`.
  **L667 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask16)-1))`。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtpd_ph(__m128d __A) {`.
  **L669 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtpd_ph(__m128d __A) {`。
- **L670 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph128_mask(`.
  **L670 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph128_mask(` 从当前函数返回。
- **L671 EN**: Executes a call or declaration centered on `statement`.
  **L671 CN**: 执行以 `statement` 为核心的调用或声明。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtpd_ph(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph128_mask((__v2df)__A, (__v8hf)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvtpd_ph(__mmask8 __U, __m128d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph128_mask(
      (__v2df)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256 _mm256_cvtpd_ph(__m256d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph256_mask(
      (__v4df)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_mask_cvtpd_ph(__m128h __W, __mmask8 __U, __m256d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph256_mask((__v4df)__A, (__v8hf)__W,
                                                   (__mmask8)__U);
}
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtpd_ph(__m128h __W,`.
  **L674 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtpd_ph(__m128h __W,`。
- **L675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L675 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L676 EN**: Continues the surrounding expression or declaration: `__m128d __A) {`.
  **L676 CN**: 继续构造周围的表达式或声明：`__m128d __A) {`。
- **L677 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph128_mask((__v2df)__A, (__v8hf)__W,`.
  **L677 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph128_mask((__v2df)__A, (__v8hf)__W,` 从当前函数返回。
- **L678 EN**: Executes a call or declaration centered on `statement`.
  **L678 CN**: 执行以 `statement` 为核心的调用或声明。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L681 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L681 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L682 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtpd_ph(__mmask8 __U, __m128d __A) {`.
  **L682 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtpd_ph(__mmask8 __U, __m128d __A) {`。
- **L683 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph128_mask(`.
  **L683 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph128_mask(` 从当前函数返回。
- **L684 EN**: Executes a call or declaration centered on `statement`.
  **L684 CN**: 执行以 `statement` 为核心的调用或声明。
- **L685 EN**: Closes the current lexical scope or compound statement.
  **L685 CN**: 结束当前词法作用域或复合语句块。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L687 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS256 _mm256_cvtpd_ph(__m256d __A) {`.
  **L687 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS256 _mm256_cvtpd_ph(__m256d __A) {`。
- **L688 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph256_mask(`.
  **L688 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph256_mask(` 从当前函数返回。
- **L689 EN**: Executes a call or declaration centered on `statement`.
  **L689 CN**: 执行以 `statement` 为核心的调用或声明。
- **L690 EN**: Closes the current lexical scope or compound statement.
  **L690 CN**: 结束当前词法作用域或复合语句块。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L692 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L692 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L693 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtpd_ph(__m128h __W, __mmask8 __U, __m256d __A) {`.
  **L693 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtpd_ph(__m128h __W, __mmask8 __U, __m256d __A) {`。
- **L694 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph256_mask((__v4df)__A, (__v8hf)__W,`.
  **L694 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph256_mask((__v4df)__A, (__v8hf)__W,` 从当前函数返回。
- **L695 EN**: Executes a call or declaration centered on `statement`.
  **L695 CN**: 执行以 `statement` 为核心的调用或声明。
- **L696 EN**: Closes the current lexical scope or compound statement.
  **L696 CN**: 结束当前词法作用域或复合语句块。

### Lines 697-720

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtpd_ph(__mmask8 __U, __m256d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph256_mask(
      (__v4df)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_cvtph_pd(__m128h __A) {
  return (__m128d)__builtin_ia32_vcvtph2pd128_mask(
      (__v8hf)__A, (__v2df)_mm_undefined_pd(), (__mmask8)-1);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_mask_cvtph_pd(__m128d __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A) {
  return (__m128d)__builtin_ia32_vcvtph2pd128_mask((__v8hf)__A, (__v2df)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {
  return (__m128d)__builtin_ia32_vcvtph2pd128_mask(
      (__v8hf)__A, (__v2df)_mm_setzero_pd(), (__mmask8)__U);
}
````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L698 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L698 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L699 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtpd_ph(__mmask8 __U, __m256d __A) {`.
  **L699 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtpd_ph(__mmask8 __U, __m256d __A) {`。
- **L700 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph256_mask(`.
  **L700 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph256_mask(` 从当前函数返回。
- **L701 EN**: Executes a call or declaration centered on `statement`.
  **L701 CN**: 执行以 `statement` 为核心的调用或声明。
- **L702 EN**: Closes the current lexical scope or compound statement.
  **L702 CN**: 结束当前词法作用域或复合语句块。
- **L703 EN**: Blank line separating nearby declarations or logic blocks.
  **L703 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L704 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_cvtph_pd(__m128h __A) {`.
  **L704 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_cvtph_pd(__m128h __A) {`。
- **L705 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vcvtph2pd128_mask(`.
  **L705 CN**: 以 `(__m128d)__builtin_ia32_vcvtph2pd128_mask(` 从当前函数返回。
- **L706 EN**: Executes a call or declaration centered on `statement`.
  **L706 CN**: 执行以 `statement` 为核心的调用或声明。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_mask_cvtph_pd(__m128d __W,`.
  **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_mask_cvtph_pd(__m128d __W,`。
- **L710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L710 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L711 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L711 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L712 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vcvtph2pd128_mask((__v8hf)__A, (__v2df)__W,`.
  **L712 CN**: 以 `(__m128d)__builtin_ia32_vcvtph2pd128_mask((__v8hf)__A, (__v2df)__W,` 从当前函数返回。
- **L713 EN**: Executes a call or declaration centered on `statement`.
  **L713 CN**: 执行以 `statement` 为核心的调用或声明。
- **L714 EN**: Closes the current lexical scope or compound statement.
  **L714 CN**: 结束当前词法作用域或复合语句块。
- **L715 EN**: Blank line separating nearby declarations or logic blocks.
  **L715 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L716 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L716 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L717 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {`.
  **L717 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {`。
- **L718 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vcvtph2pd128_mask(`.
  **L718 CN**: 以 `(__m128d)__builtin_ia32_vcvtph2pd128_mask(` 从当前函数返回。
- **L719 EN**: Executes a call or declaration centered on `statement`.
  **L719 CN**: 执行以 `statement` 为核心的调用或声明。
- **L720 EN**: Closes the current lexical scope or compound statement.
  **L720 CN**: 结束当前词法作用域或复合语句块。

### Lines 721-744

````c

static __inline__ __m256d __DEFAULT_FN_ATTRS256 _mm256_cvtph_pd(__m128h __A) {
  return (__m256d)__builtin_ia32_vcvtph2pd256_mask(
      (__v8hf)__A, (__v4df)_mm256_undefined_pd(), (__mmask8)-1);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_pd(__m256d __W, __mmask8 __U, __m128h __A) {
  return (__m256d)__builtin_ia32_vcvtph2pd256_mask((__v8hf)__A, (__v4df)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {
  return (__m256d)__builtin_ia32_vcvtph2pd256_mask(
      (__v8hf)__A, (__v4df)_mm256_setzero_pd(), (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi16(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2w128_mask(
      (__v8hf)__A, (__v8hi)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
````
- **L721 EN**: Blank line separating nearby declarations or logic blocks.
  **L721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L722 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256d __DEFAULT_FN_ATTRS256 _mm256_cvtph_pd(__m128h __A) {`.
  **L722 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256d __DEFAULT_FN_ATTRS256 _mm256_cvtph_pd(__m128h __A) {`。
- **L723 EN**: Returns from the current function with `(__m256d)__builtin_ia32_vcvtph2pd256_mask(`.
  **L723 CN**: 以 `(__m256d)__builtin_ia32_vcvtph2pd256_mask(` 从当前函数返回。
- **L724 EN**: Executes a call or declaration centered on `statement`.
  **L724 CN**: 执行以 `statement` 为核心的调用或声明。
- **L725 EN**: Closes the current lexical scope or compound statement.
  **L725 CN**: 结束当前词法作用域或复合语句块。
- **L726 EN**: Blank line separating nearby declarations or logic blocks.
  **L726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L727 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L727 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L728 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_pd(__m256d __W, __mmask8 __U, __m128h __A) {`.
  **L728 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_pd(__m256d __W, __mmask8 __U, __m128h __A) {`。
- **L729 EN**: Returns from the current function with `(__m256d)__builtin_ia32_vcvtph2pd256_mask((__v8hf)__A, (__v4df)__W,`.
  **L729 CN**: 以 `(__m256d)__builtin_ia32_vcvtph2pd256_mask((__v8hf)__A, (__v4df)__W,` 从当前函数返回。
- **L730 EN**: Executes a call or declaration centered on `statement`.
  **L730 CN**: 执行以 `statement` 为核心的调用或声明。
- **L731 EN**: Closes the current lexical scope or compound statement.
  **L731 CN**: 结束当前词法作用域或复合语句块。
- **L732 EN**: Blank line separating nearby declarations or logic blocks.
  **L732 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L733 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L733 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L734 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {`.
  **L734 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {`。
- **L735 EN**: Returns from the current function with `(__m256d)__builtin_ia32_vcvtph2pd256_mask(`.
  **L735 CN**: 以 `(__m256d)__builtin_ia32_vcvtph2pd256_mask(` 从当前函数返回。
- **L736 EN**: Executes a call or declaration centered on `statement`.
  **L736 CN**: 执行以 `statement` 为核心的调用或声明。
- **L737 EN**: Closes the current lexical scope or compound statement.
  **L737 CN**: 结束当前词法作用域或复合语句块。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L739 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi16(__m128h __A) {`.
  **L739 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi16(__m128h __A) {`。
- **L740 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2w128_mask(`.
  **L740 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2w128_mask(` 从当前函数返回。
- **L741 EN**: Executes a call or declaration centered on `statement`.
  **L741 CN**: 执行以 `statement` 为核心的调用或声明。
- **L742 EN**: Closes the current lexical scope or compound statement.
  **L742 CN**: 结束当前词法作用域或复合语句块。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L744 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L744 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。

### Lines 745-768

````c
_mm_mask_cvtph_epi16(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2w128_mask((__v8hf)__A, (__v8hi)__W,
                                                  (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtph_epi16(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2w128_mask(
      (__v8hf)__A, (__v8hi)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtph_epi16(__m256h __A) {
  return (__m256i)__builtin_ia32_vcvtph2w256_mask(
      (__v16hf)__A, (__v16hi)_mm256_undefined_si256(), (__mmask16)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_epi16(__m256i __W, __mmask16 __U, __m256h __A) {
  return (__m256i)__builtin_ia32_vcvtph2w256_mask((__v16hf)__A, (__v16hi)__W,
                                                  (__mmask16)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
````
- **L745 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtph_epi16(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L745 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtph_epi16(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L746 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2w128_mask((__v8hf)__A, (__v8hi)__W,`.
  **L746 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2w128_mask((__v8hf)__A, (__v8hi)__W,` 从当前函数返回。
- **L747 EN**: Executes a call or declaration centered on `statement`.
  **L747 CN**: 执行以 `statement` 为核心的调用或声明。
- **L748 EN**: Closes the current lexical scope or compound statement.
  **L748 CN**: 结束当前词法作用域或复合语句块。
- **L749 EN**: Blank line separating nearby declarations or logic blocks.
  **L749 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L750 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L750 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L751 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_epi16(__mmask8 __U, __m128h __A) {`.
  **L751 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_epi16(__mmask8 __U, __m128h __A) {`。
- **L752 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2w128_mask(`.
  **L752 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2w128_mask(` 从当前函数返回。
- **L753 EN**: Executes a call or declaration centered on `statement`.
  **L753 CN**: 执行以 `statement` 为核心的调用或声明。
- **L754 EN**: Closes the current lexical scope or compound statement.
  **L754 CN**: 结束当前词法作用域或复合语句块。
- **L755 EN**: Blank line separating nearby declarations or logic blocks.
  **L755 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L756 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L756 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L757 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtph_epi16(__m256h __A) {`.
  **L757 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtph_epi16(__m256h __A) {`。
- **L758 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2w256_mask(`.
  **L758 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2w256_mask(` 从当前函数返回。
- **L759 EN**: Executes a call or declaration centered on `statement`.
  **L759 CN**: 执行以 `statement` 为核心的调用或声明。
- **L760 EN**: Closes the current lexical scope or compound statement.
  **L760 CN**: 结束当前词法作用域或复合语句块。
- **L761 EN**: Blank line separating nearby declarations or logic blocks.
  **L761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L762 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L762 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L763 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_epi16(__m256i __W, __mmask16 __U, __m256h __A) {`.
  **L763 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_epi16(__m256i __W, __mmask16 __U, __m256h __A) {`。
- **L764 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2w256_mask((__v16hf)__A, (__v16hi)__W,`.
  **L764 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2w256_mask((__v16hf)__A, (__v16hi)__W,` 从当前函数返回。
- **L765 EN**: Executes a call or declaration centered on `statement`.
  **L765 CN**: 执行以 `statement` 为核心的调用或声明。
- **L766 EN**: Closes the current lexical scope or compound statement.
  **L766 CN**: 结束当前词法作用域或复合语句块。
- **L767 EN**: Blank line separating nearby declarations or logic blocks.
  **L767 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L768 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L768 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。

### Lines 769-792

````c
_mm256_maskz_cvtph_epi16(__mmask16 __U, __m256h __A) {
  return (__m256i)__builtin_ia32_vcvtph2w256_mask(
      (__v16hf)__A, (__v16hi)_mm256_setzero_si256(), (__mmask16)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi16(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2w128_mask(
      (__v8hf)__A, (__v8hi)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttph_epi16(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2w128_mask((__v8hf)__A, (__v8hi)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttph_epi16(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2w128_mask(
      (__v8hf)__A, (__v8hi)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttph_epi16(__m256h __A) {
````
- **L769 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_epi16(__mmask16 __U, __m256h __A) {`.
  **L769 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_epi16(__mmask16 __U, __m256h __A) {`。
- **L770 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2w256_mask(`.
  **L770 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2w256_mask(` 从当前函数返回。
- **L771 EN**: Executes a call or declaration centered on `statement`.
  **L771 CN**: 执行以 `statement` 为核心的调用或声明。
- **L772 EN**: Closes the current lexical scope or compound statement.
  **L772 CN**: 结束当前词法作用域或复合语句块。
- **L773 EN**: Blank line separating nearby declarations or logic blocks.
  **L773 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L774 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi16(__m128h __A) {`.
  **L774 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi16(__m128h __A) {`。
- **L775 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2w128_mask(`.
  **L775 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2w128_mask(` 从当前函数返回。
- **L776 EN**: Executes a call or declaration centered on `statement`.
  **L776 CN**: 执行以 `statement` 为核心的调用或声明。
- **L777 EN**: Closes the current lexical scope or compound statement.
  **L777 CN**: 结束当前词法作用域或复合语句块。
- **L778 EN**: Blank line separating nearby declarations or logic blocks.
  **L778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L779 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L779 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L780 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttph_epi16(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L780 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttph_epi16(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L781 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2w128_mask((__v8hf)__A, (__v8hi)__W,`.
  **L781 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2w128_mask((__v8hf)__A, (__v8hi)__W,` 从当前函数返回。
- **L782 EN**: Executes a call or declaration centered on `statement`.
  **L782 CN**: 执行以 `statement` 为核心的调用或声明。
- **L783 EN**: Closes the current lexical scope or compound statement.
  **L783 CN**: 结束当前词法作用域或复合语句块。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L785 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L785 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L786 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttph_epi16(__mmask8 __U, __m128h __A) {`.
  **L786 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttph_epi16(__mmask8 __U, __m128h __A) {`。
- **L787 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2w128_mask(`.
  **L787 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2w128_mask(` 从当前函数返回。
- **L788 EN**: Executes a call or declaration centered on `statement`.
  **L788 CN**: 执行以 `statement` 为核心的调用或声明。
- **L789 EN**: Closes the current lexical scope or compound statement.
  **L789 CN**: 结束当前词法作用域或复合语句块。
- **L790 EN**: Blank line separating nearby declarations or logic blocks.
  **L790 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L791 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L791 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L792 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttph_epi16(__m256h __A) {`.
  **L792 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttph_epi16(__m256h __A) {`。

### Lines 793-816

````c
  return (__m256i)__builtin_ia32_vcvttph2w256_mask(
      (__v16hf)__A, (__v16hi)_mm256_undefined_si256(), (__mmask16)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttph_epi16(__m256i __W, __mmask16 __U, __m256h __A) {
  return (__m256i)__builtin_ia32_vcvttph2w256_mask((__v16hf)__A, (__v16hi)__W,
                                                   (__mmask16)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttph_epi16(__mmask16 __U, __m256h __A) {
  return (__m256i)__builtin_ia32_vcvttph2w256_mask(
      (__v16hf)__A, (__v16hi)_mm256_setzero_si256(), (__mmask16)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_cvtepi16_ph(__m128i __A) {
  return (__m128h) __builtin_convertvector((__v8hi)__A, __v8hf);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_cvtepi16_ph(__m128h __W, __mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_selectph_128(
````
- **L793 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2w256_mask(`.
  **L793 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2w256_mask(` 从当前函数返回。
- **L794 EN**: Executes a call or declaration centered on `statement`.
  **L794 CN**: 执行以 `statement` 为核心的调用或声明。
- **L795 EN**: Closes the current lexical scope or compound statement.
  **L795 CN**: 结束当前词法作用域或复合语句块。
- **L796 EN**: Blank line separating nearby declarations or logic blocks.
  **L796 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L797 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L797 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L798 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttph_epi16(__m256i __W, __mmask16 __U, __m256h __A) {`.
  **L798 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttph_epi16(__m256i __W, __mmask16 __U, __m256h __A) {`。
- **L799 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2w256_mask((__v16hf)__A, (__v16hi)__W,`.
  **L799 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2w256_mask((__v16hf)__A, (__v16hi)__W,` 从当前函数返回。
- **L800 EN**: Executes a call or declaration centered on `statement`.
  **L800 CN**: 执行以 `statement` 为核心的调用或声明。
- **L801 EN**: Closes the current lexical scope or compound statement.
  **L801 CN**: 结束当前词法作用域或复合语句块。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L803 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L803 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L804 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttph_epi16(__mmask16 __U, __m256h __A) {`.
  **L804 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttph_epi16(__mmask16 __U, __m256h __A) {`。
- **L805 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2w256_mask(`.
  **L805 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2w256_mask(` 从当前函数返回。
- **L806 EN**: Executes a call or declaration centered on `statement`.
  **L806 CN**: 执行以 `statement` 为核心的调用或声明。
- **L807 EN**: Closes the current lexical scope or compound statement.
  **L807 CN**: 结束当前词法作用域或复合语句块。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L809 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L809 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L810 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi16_ph(__m128i __A) {`.
  **L810 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi16_ph(__m128i __A) {`。
- **L811 EN**: Returns from the current function with `(__m128h) __builtin_convertvector((__v8hi)__A, __v8hf)`.
  **L811 CN**: 以 `(__m128h) __builtin_convertvector((__v8hi)__A, __v8hf)` 从当前函数返回。
- **L812 EN**: Closes the current lexical scope or compound statement.
  **L812 CN**: 结束当前词法作用域或复合语句块。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L814 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L814 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L815 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepi16_ph(__m128h __W, __mmask8 __U, __m128i __A) {`.
  **L815 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepi16_ph(__m128h __W, __mmask8 __U, __m128i __A) {`。
- **L816 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L816 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。

### Lines 817-840

````c
      (__mmask8)__U, (__v8hf)_mm_cvtepi16_ph(__A), (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_cvtepi16_ph(__mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm_cvtepi16_ph(__A), (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtepi16_ph(__m256i __A) {
  return (__m256h) __builtin_convertvector((__v16hi)__A, __v16hf);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepi16_ph(__m256h __W, __mmask16 __U, __m256i __A) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U, (__v16hf)_mm256_cvtepi16_ph(__A), (__v16hf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepi16_ph(__mmask16 __U, __m256i __A) {
  return (__m256h)__builtin_ia32_selectph_256((__mmask16)__U,
                                              (__v16hf)_mm256_cvtepi16_ph(__A),
````
- **L817 EN**: Executes a call or declaration centered on `statement`.
  **L817 CN**: 执行以 `statement` 为核心的调用或声明。
- **L818 EN**: Closes the current lexical scope or compound statement.
  **L818 CN**: 结束当前词法作用域或复合语句块。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L820 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L820 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L821 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepi16_ph(__mmask8 __U, __m128i __A) {`.
  **L821 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepi16_ph(__mmask8 __U, __m128i __A) {`。
- **L822 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L822 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L823 EN**: Executes a call or declaration centered on `statement`.
  **L823 CN**: 执行以 `statement` 为核心的调用或声明。
- **L824 EN**: Closes the current lexical scope or compound statement.
  **L824 CN**: 结束当前词法作用域或复合语句块。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L826 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L827 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepi16_ph(__m256i __A) {`.
  **L827 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepi16_ph(__m256i __A) {`。
- **L828 EN**: Returns from the current function with `(__m256h) __builtin_convertvector((__v16hi)__A, __v16hf)`.
  **L828 CN**: 以 `(__m256h) __builtin_convertvector((__v16hi)__A, __v16hf)` 从当前函数返回。
- **L829 EN**: Closes the current lexical scope or compound statement.
  **L829 CN**: 结束当前词法作用域或复合语句块。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L831 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L831 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L832 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepi16_ph(__m256h __W, __mmask16 __U, __m256i __A) {`.
  **L832 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepi16_ph(__m256h __W, __mmask16 __U, __m256i __A) {`。
- **L833 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L833 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L834 EN**: Executes a call or declaration centered on `statement`.
  **L834 CN**: 执行以 `statement` 为核心的调用或声明。
- **L835 EN**: Closes the current lexical scope or compound statement.
  **L835 CN**: 结束当前词法作用域或复合语句块。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L837 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L837 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L838 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepi16_ph(__mmask16 __U, __m256i __A) {`.
  **L838 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepi16_ph(__mmask16 __U, __m256i __A) {`。
- **L839 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256((__mmask16)__U,`.
  **L839 CN**: 以 `(__m256h)__builtin_ia32_selectph_256((__mmask16)__U,` 从当前函数返回。
- **L840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)_mm256_cvtepi16_ph(__A),`.
  **L840 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)_mm256_cvtepi16_ph(__A),`。

### Lines 841-864

````c
                                              (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu16(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2uw128_mask(
      (__v8hf)__A, (__v8hu)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtph_epu16(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2uw128_mask((__v8hf)__A, (__v8hu)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtph_epu16(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2uw128_mask(
      (__v8hf)__A, (__v8hu)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtph_epu16(__m256h __A) {
  return (__m256i)__builtin_ia32_vcvtph2uw256_mask(
      (__v16hf)__A, (__v16hu)_mm256_undefined_si256(), (__mmask16)-1);
````
- **L841 EN**: Executes a call or declaration centered on `statement`.
  **L841 CN**: 执行以 `statement` 为核心的调用或声明。
- **L842 EN**: Closes the current lexical scope or compound statement.
  **L842 CN**: 结束当前词法作用域或复合语句块。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L844 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu16(__m128h __A) {`.
  **L844 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu16(__m128h __A) {`。
- **L845 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2uw128_mask(`.
  **L845 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2uw128_mask(` 从当前函数返回。
- **L846 EN**: Executes a call or declaration centered on `statement`.
  **L846 CN**: 执行以 `statement` 为核心的调用或声明。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L849 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L850 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtph_epu16(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L850 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtph_epu16(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L851 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2uw128_mask((__v8hf)__A, (__v8hu)__W,`.
  **L851 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2uw128_mask((__v8hf)__A, (__v8hu)__W,` 从当前函数返回。
- **L852 EN**: Executes a call or declaration centered on `statement`.
  **L852 CN**: 执行以 `statement` 为核心的调用或声明。
- **L853 EN**: Closes the current lexical scope or compound statement.
  **L853 CN**: 结束当前词法作用域或复合语句块。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L855 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L855 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L856 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_epu16(__mmask8 __U, __m128h __A) {`.
  **L856 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_epu16(__mmask8 __U, __m128h __A) {`。
- **L857 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2uw128_mask(`.
  **L857 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2uw128_mask(` 从当前函数返回。
- **L858 EN**: Executes a call or declaration centered on `statement`.
  **L858 CN**: 执行以 `statement` 为核心的调用或声明。
- **L859 EN**: Closes the current lexical scope or compound statement.
  **L859 CN**: 结束当前词法作用域或复合语句块。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L861 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L861 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L862 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtph_epu16(__m256h __A) {`.
  **L862 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtph_epu16(__m256h __A) {`。
- **L863 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2uw256_mask(`.
  **L863 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2uw256_mask(` 从当前函数返回。
- **L864 EN**: Executes a call or declaration centered on `statement`.
  **L864 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 865-888

````c
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_epu16(__m256i __W, __mmask16 __U, __m256h __A) {
  return (__m256i)__builtin_ia32_vcvtph2uw256_mask((__v16hf)__A, (__v16hu)__W,
                                                   (__mmask16)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtph_epu16(__mmask16 __U, __m256h __A) {
  return (__m256i)__builtin_ia32_vcvtph2uw256_mask(
      (__v16hf)__A, (__v16hu)_mm256_setzero_si256(), (__mmask16)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu16(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2uw128_mask(
      (__v8hf)__A, (__v8hu)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttph_epu16(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2uw128_mask((__v8hf)__A, (__v8hu)__W,
                                                    (__mmask8)__U);
}
````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L867 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L868 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_epu16(__m256i __W, __mmask16 __U, __m256h __A) {`.
  **L868 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_epu16(__m256i __W, __mmask16 __U, __m256h __A) {`。
- **L869 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2uw256_mask((__v16hf)__A, (__v16hu)__W,`.
  **L869 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2uw256_mask((__v16hf)__A, (__v16hu)__W,` 从当前函数返回。
- **L870 EN**: Executes a call or declaration centered on `statement`.
  **L870 CN**: 执行以 `statement` 为核心的调用或声明。
- **L871 EN**: Closes the current lexical scope or compound statement.
  **L871 CN**: 结束当前词法作用域或复合语句块。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L873 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L873 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L874 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_epu16(__mmask16 __U, __m256h __A) {`.
  **L874 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_epu16(__mmask16 __U, __m256h __A) {`。
- **L875 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2uw256_mask(`.
  **L875 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2uw256_mask(` 从当前函数返回。
- **L876 EN**: Executes a call or declaration centered on `statement`.
  **L876 CN**: 执行以 `statement` 为核心的调用或声明。
- **L877 EN**: Closes the current lexical scope or compound statement.
  **L877 CN**: 结束当前词法作用域或复合语句块。
- **L878 EN**: Blank line separating nearby declarations or logic blocks.
  **L878 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L879 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu16(__m128h __A) {`.
  **L879 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu16(__m128h __A) {`。
- **L880 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2uw128_mask(`.
  **L880 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2uw128_mask(` 从当前函数返回。
- **L881 EN**: Executes a call or declaration centered on `statement`.
  **L881 CN**: 执行以 `statement` 为核心的调用或声明。
- **L882 EN**: Closes the current lexical scope or compound statement.
  **L882 CN**: 结束当前词法作用域或复合语句块。
- **L883 EN**: Blank line separating nearby declarations or logic blocks.
  **L883 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L884 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L884 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L885 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttph_epu16(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L885 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttph_epu16(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L886 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2uw128_mask((__v8hf)__A, (__v8hu)__W,`.
  **L886 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2uw128_mask((__v8hf)__A, (__v8hu)__W,` 从当前函数返回。
- **L887 EN**: Executes a call or declaration centered on `statement`.
  **L887 CN**: 执行以 `statement` 为核心的调用或声明。
- **L888 EN**: Closes the current lexical scope or compound statement.
  **L888 CN**: 结束当前词法作用域或复合语句块。

### Lines 889-912

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttph_epu16(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2uw128_mask(
      (__v8hf)__A, (__v8hu)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttph_epu16(__m256h __A) {
  return (__m256i)__builtin_ia32_vcvttph2uw256_mask(
      (__v16hf)__A, (__v16hu)_mm256_undefined_si256(), (__mmask16)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttph_epu16(__m256i __W, __mmask16 __U, __m256h __A) {
  return (__m256i)__builtin_ia32_vcvttph2uw256_mask((__v16hf)__A, (__v16hu)__W,
                                                    (__mmask16)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttph_epu16(__mmask16 __U, __m256h __A) {
  return (__m256i)__builtin_ia32_vcvttph2uw256_mask(
      (__v16hf)__A, (__v16hu)_mm256_setzero_si256(), (__mmask16)__U);
}
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L890 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L890 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L891 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttph_epu16(__mmask8 __U, __m128h __A) {`.
  **L891 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttph_epu16(__mmask8 __U, __m128h __A) {`。
- **L892 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2uw128_mask(`.
  **L892 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2uw128_mask(` 从当前函数返回。
- **L893 EN**: Executes a call or declaration centered on `statement`.
  **L893 CN**: 执行以 `statement` 为核心的调用或声明。
- **L894 EN**: Closes the current lexical scope or compound statement.
  **L894 CN**: 结束当前词法作用域或复合语句块。
- **L895 EN**: Blank line separating nearby declarations or logic blocks.
  **L895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L896 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L896 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L897 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttph_epu16(__m256h __A) {`.
  **L897 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttph_epu16(__m256h __A) {`。
- **L898 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2uw256_mask(`.
  **L898 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2uw256_mask(` 从当前函数返回。
- **L899 EN**: Executes a call or declaration centered on `statement`.
  **L899 CN**: 执行以 `statement` 为核心的调用或声明。
- **L900 EN**: Closes the current lexical scope or compound statement.
  **L900 CN**: 结束当前词法作用域或复合语句块。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L902 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L902 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L903 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttph_epu16(__m256i __W, __mmask16 __U, __m256h __A) {`.
  **L903 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttph_epu16(__m256i __W, __mmask16 __U, __m256h __A) {`。
- **L904 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2uw256_mask((__v16hf)__A, (__v16hu)__W,`.
  **L904 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2uw256_mask((__v16hf)__A, (__v16hu)__W,` 从当前函数返回。
- **L905 EN**: Executes a call or declaration centered on `statement`.
  **L905 CN**: 执行以 `statement` 为核心的调用或声明。
- **L906 EN**: Closes the current lexical scope or compound statement.
  **L906 CN**: 结束当前词法作用域或复合语句块。
- **L907 EN**: Blank line separating nearby declarations or logic blocks.
  **L907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L908 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L908 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L909 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttph_epu16(__mmask16 __U, __m256h __A) {`.
  **L909 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttph_epu16(__mmask16 __U, __m256h __A) {`。
- **L910 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2uw256_mask(`.
  **L910 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2uw256_mask(` 从当前函数返回。
- **L911 EN**: Executes a call or declaration centered on `statement`.
  **L911 CN**: 执行以 `statement` 为核心的调用或声明。
- **L912 EN**: Closes the current lexical scope or compound statement.
  **L912 CN**: 结束当前词法作用域或复合语句块。

### Lines 913-936

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_cvtepu16_ph(__m128i __A) {
  return (__m128h) __builtin_convertvector((__v8hu)__A, __v8hf);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_cvtepu16_ph(__m128h __W, __mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm_cvtepu16_ph(__A), (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_cvtepu16_ph(__mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm_cvtepu16_ph(__A), (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtepu16_ph(__m256i __A) {
  return (__m256h) __builtin_convertvector((__v16hu)__A, __v16hf);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L914 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L914 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L915 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu16_ph(__m128i __A) {`.
  **L915 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu16_ph(__m128i __A) {`。
- **L916 EN**: Returns from the current function with `(__m128h) __builtin_convertvector((__v8hu)__A, __v8hf)`.
  **L916 CN**: 以 `(__m128h) __builtin_convertvector((__v8hu)__A, __v8hf)` 从当前函数返回。
- **L917 EN**: Closes the current lexical scope or compound statement.
  **L917 CN**: 结束当前词法作用域或复合语句块。
- **L918 EN**: Blank line separating nearby declarations or logic blocks.
  **L918 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L919 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L919 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L920 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepu16_ph(__m128h __W, __mmask8 __U, __m128i __A) {`.
  **L920 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepu16_ph(__m128h __W, __mmask8 __U, __m128i __A) {`。
- **L921 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L921 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L922 EN**: Executes a call or declaration centered on `statement`.
  **L922 CN**: 执行以 `statement` 为核心的调用或声明。
- **L923 EN**: Closes the current lexical scope or compound statement.
  **L923 CN**: 结束当前词法作用域或复合语句块。
- **L924 EN**: Blank line separating nearby declarations or logic blocks.
  **L924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L925 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L925 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L926 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepu16_ph(__mmask8 __U, __m128i __A) {`.
  **L926 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepu16_ph(__mmask8 __U, __m128i __A) {`。
- **L927 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L927 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L928 EN**: Executes a call or declaration centered on `statement`.
  **L928 CN**: 执行以 `statement` 为核心的调用或声明。
- **L929 EN**: Closes the current lexical scope or compound statement.
  **L929 CN**: 结束当前词法作用域或复合语句块。
- **L930 EN**: Blank line separating nearby declarations or logic blocks.
  **L930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L931 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L931 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L932 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepu16_ph(__m256i __A) {`.
  **L932 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepu16_ph(__m256i __A) {`。
- **L933 EN**: Returns from the current function with `(__m256h) __builtin_convertvector((__v16hu)__A, __v16hf)`.
  **L933 CN**: 以 `(__m256h) __builtin_convertvector((__v16hu)__A, __v16hf)` 从当前函数返回。
- **L934 EN**: Closes the current lexical scope or compound statement.
  **L934 CN**: 结束当前词法作用域或复合语句块。
- **L935 EN**: Blank line separating nearby declarations or logic blocks.
  **L935 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L936 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L936 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 937-960

````c
_mm256_mask_cvtepu16_ph(__m256h __W, __mmask16 __U, __m256i __A) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U, (__v16hf)_mm256_cvtepu16_ph(__A), (__v16hf)__W);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepu16_ph(__mmask16 __U, __m256i __A) {
  return (__m256h)__builtin_ia32_selectph_256((__mmask16)__U,
                                              (__v16hf)_mm256_cvtepu16_ph(__A),
                                              (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi32(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2dq128_mask(
      (__v8hf)__A, (__v4si)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtph_epi32(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2dq128_mask((__v8hf)__A, (__v4si)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
````
- **L937 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepu16_ph(__m256h __W, __mmask16 __U, __m256i __A) {`.
  **L937 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepu16_ph(__m256h __W, __mmask16 __U, __m256i __A) {`。
- **L938 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L938 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L939 EN**: Executes a call or declaration centered on `statement`.
  **L939 CN**: 执行以 `statement` 为核心的调用或声明。
- **L940 EN**: Closes the current lexical scope or compound statement.
  **L940 CN**: 结束当前词法作用域或复合语句块。
- **L941 EN**: Blank line separating nearby declarations or logic blocks.
  **L941 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L942 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L942 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L943 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepu16_ph(__mmask16 __U, __m256i __A) {`.
  **L943 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepu16_ph(__mmask16 __U, __m256i __A) {`。
- **L944 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256((__mmask16)__U,`.
  **L944 CN**: 以 `(__m256h)__builtin_ia32_selectph_256((__mmask16)__U,` 从当前函数返回。
- **L945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)_mm256_cvtepu16_ph(__A),`.
  **L945 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)_mm256_cvtepu16_ph(__A),`。
- **L946 EN**: Executes a call or declaration centered on `statement`.
  **L946 CN**: 执行以 `statement` 为核心的调用或声明。
- **L947 EN**: Closes the current lexical scope or compound statement.
  **L947 CN**: 结束当前词法作用域或复合语句块。
- **L948 EN**: Blank line separating nearby declarations or logic blocks.
  **L948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L949 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi32(__m128h __A) {`.
  **L949 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi32(__m128h __A) {`。
- **L950 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2dq128_mask(`.
  **L950 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2dq128_mask(` 从当前函数返回。
- **L951 EN**: Executes a call or declaration centered on `statement`.
  **L951 CN**: 执行以 `statement` 为核心的调用或声明。
- **L952 EN**: Closes the current lexical scope or compound statement.
  **L952 CN**: 结束当前词法作用域或复合语句块。
- **L953 EN**: Blank line separating nearby declarations or logic blocks.
  **L953 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L954 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L954 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L955 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtph_epi32(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L955 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtph_epi32(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L956 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2dq128_mask((__v8hf)__A, (__v4si)__W,`.
  **L956 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2dq128_mask((__v8hf)__A, (__v4si)__W,` 从当前函数返回。
- **L957 EN**: Executes a call or declaration centered on `statement`.
  **L957 CN**: 执行以 `statement` 为核心的调用或声明。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L960 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L960 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。

### Lines 961-984

````c
_mm_maskz_cvtph_epi32(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2dq128_mask(
      (__v8hf)__A, (__v4si)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtph_epi32(__m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2dq256_mask(
      (__v8hf)__A, (__v8si)_mm256_undefined_si256(), (__mmask8)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_epi32(__m256i __W, __mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2dq256_mask((__v8hf)__A, (__v8si)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtph_epi32(__mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2dq256_mask(
      (__v8hf)__A, (__v8si)_mm256_setzero_si256(), (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu32(__m128h __A) {
````
- **L961 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_epi32(__mmask8 __U, __m128h __A) {`.
  **L961 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_epi32(__mmask8 __U, __m128h __A) {`。
- **L962 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2dq128_mask(`.
  **L962 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2dq128_mask(` 从当前函数返回。
- **L963 EN**: Executes a call or declaration centered on `statement`.
  **L963 CN**: 执行以 `statement` 为核心的调用或声明。
- **L964 EN**: Closes the current lexical scope or compound statement.
  **L964 CN**: 结束当前词法作用域或复合语句块。
- **L965 EN**: Blank line separating nearby declarations or logic blocks.
  **L965 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L966 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L966 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L967 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtph_epi32(__m128h __A) {`.
  **L967 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtph_epi32(__m128h __A) {`。
- **L968 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2dq256_mask(`.
  **L968 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2dq256_mask(` 从当前函数返回。
- **L969 EN**: Executes a call or declaration centered on `statement`.
  **L969 CN**: 执行以 `statement` 为核心的调用或声明。
- **L970 EN**: Closes the current lexical scope or compound statement.
  **L970 CN**: 结束当前词法作用域或复合语句块。
- **L971 EN**: Blank line separating nearby declarations or logic blocks.
  **L971 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L972 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L972 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L973 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_epi32(__m256i __W, __mmask8 __U, __m128h __A) {`.
  **L973 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_epi32(__m256i __W, __mmask8 __U, __m128h __A) {`。
- **L974 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2dq256_mask((__v8hf)__A, (__v8si)__W,`.
  **L974 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2dq256_mask((__v8hf)__A, (__v8si)__W,` 从当前函数返回。
- **L975 EN**: Executes a call or declaration centered on `statement`.
  **L975 CN**: 执行以 `statement` 为核心的调用或声明。
- **L976 EN**: Closes the current lexical scope or compound statement.
  **L976 CN**: 结束当前词法作用域或复合语句块。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L978 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L978 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L979 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_epi32(__mmask8 __U, __m128h __A) {`.
  **L979 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_epi32(__mmask8 __U, __m128h __A) {`。
- **L980 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2dq256_mask(`.
  **L980 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2dq256_mask(` 从当前函数返回。
- **L981 EN**: Executes a call or declaration centered on `statement`.
  **L981 CN**: 执行以 `statement` 为核心的调用或声明。
- **L982 EN**: Closes the current lexical scope or compound statement.
  **L982 CN**: 结束当前词法作用域或复合语句块。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L984 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu32(__m128h __A) {`.
  **L984 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu32(__m128h __A) {`。

### Lines 985-1008

````c
  return (__m128i)__builtin_ia32_vcvtph2udq128_mask(
      (__v8hf)__A, (__v4su)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtph_epu32(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2udq128_mask((__v8hf)__A, (__v4su)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtph_epu32(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2udq128_mask(
      (__v8hf)__A, (__v4su)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtph_epu32(__m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2udq256_mask(
      (__v8hf)__A, (__v8su)_mm256_undefined_si256(), (__mmask8)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_epu32(__m256i __W, __mmask8 __U, __m128h __A) {
````
- **L985 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2udq128_mask(`.
  **L985 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2udq128_mask(` 从当前函数返回。
- **L986 EN**: Executes a call or declaration centered on `statement`.
  **L986 CN**: 执行以 `statement` 为核心的调用或声明。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Blank line separating nearby declarations or logic blocks.
  **L988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L989 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L989 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L990 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtph_epu32(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L990 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtph_epu32(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L991 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2udq128_mask((__v8hf)__A, (__v4su)__W,`.
  **L991 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2udq128_mask((__v8hf)__A, (__v4su)__W,` 从当前函数返回。
- **L992 EN**: Executes a call or declaration centered on `statement`.
  **L992 CN**: 执行以 `statement` 为核心的调用或声明。
- **L993 EN**: Closes the current lexical scope or compound statement.
  **L993 CN**: 结束当前词法作用域或复合语句块。
- **L994 EN**: Blank line separating nearby declarations or logic blocks.
  **L994 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L995 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L995 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L996 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_epu32(__mmask8 __U, __m128h __A) {`.
  **L996 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_epu32(__mmask8 __U, __m128h __A) {`。
- **L997 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2udq128_mask(`.
  **L997 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2udq128_mask(` 从当前函数返回。
- **L998 EN**: Executes a call or declaration centered on `statement`.
  **L998 CN**: 执行以 `statement` 为核心的调用或声明。
- **L999 EN**: Closes the current lexical scope or compound statement.
  **L999 CN**: 结束当前词法作用域或复合语句块。
- **L1000 EN**: Blank line separating nearby declarations or logic blocks.
  **L1000 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1001 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1001 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1002 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtph_epu32(__m128h __A) {`.
  **L1002 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtph_epu32(__m128h __A) {`。
- **L1003 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2udq256_mask(`.
  **L1003 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2udq256_mask(` 从当前函数返回。
- **L1004 EN**: Executes a call or declaration centered on `statement`.
  **L1004 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1005 EN**: Closes the current lexical scope or compound statement.
  **L1005 CN**: 结束当前词法作用域或复合语句块。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1007 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1007 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1008 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_epu32(__m256i __W, __mmask8 __U, __m128h __A) {`.
  **L1008 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_epu32(__m256i __W, __mmask8 __U, __m128h __A) {`。

### Lines 1009-1032

````c
  return (__m256i)__builtin_ia32_vcvtph2udq256_mask((__v8hf)__A, (__v8su)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtph_epu32(__mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2udq256_mask(
      (__v8hf)__A, (__v8su)_mm256_setzero_si256(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepi32_ph(__m128i __A) {
  return (__m128h)__builtin_ia32_vcvtdq2ph128_mask(
      (__v4si)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_cvtepi32_ph(__m128h __W, __mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvtdq2ph128_mask((__v4si)__A, (__v8hf)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvtepi32_ph(__mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvtdq2ph128_mask(
````
- **L1009 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2udq256_mask((__v8hf)__A, (__v8su)__W,`.
  **L1009 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2udq256_mask((__v8hf)__A, (__v8su)__W,` 从当前函数返回。
- **L1010 EN**: Executes a call or declaration centered on `statement`.
  **L1010 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1011 EN**: Closes the current lexical scope or compound statement.
  **L1011 CN**: 结束当前词法作用域或复合语句块。
- **L1012 EN**: Blank line separating nearby declarations or logic blocks.
  **L1012 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1013 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1013 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1014 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_epu32(__mmask8 __U, __m128h __A) {`.
  **L1014 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_epu32(__mmask8 __U, __m128h __A) {`。
- **L1015 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2udq256_mask(`.
  **L1015 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2udq256_mask(` 从当前函数返回。
- **L1016 EN**: Executes a call or declaration centered on `statement`.
  **L1016 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1017 EN**: Closes the current lexical scope or compound statement.
  **L1017 CN**: 结束当前词法作用域或复合语句块。
- **L1018 EN**: Blank line separating nearby declarations or logic blocks.
  **L1018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1019 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepi32_ph(__m128i __A) {`.
  **L1019 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepi32_ph(__m128i __A) {`。
- **L1020 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtdq2ph128_mask(`.
  **L1020 CN**: 以 `(__m128h)__builtin_ia32_vcvtdq2ph128_mask(` 从当前函数返回。
- **L1021 EN**: Executes a call or declaration centered on `statement`.
  **L1021 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Blank line separating nearby declarations or logic blocks.
  **L1023 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1024 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1024 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1025 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepi32_ph(__m128h __W, __mmask8 __U, __m128i __A) {`.
  **L1025 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepi32_ph(__m128h __W, __mmask8 __U, __m128i __A) {`。
- **L1026 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtdq2ph128_mask((__v4si)__A, (__v8hf)__W,`.
  **L1026 CN**: 以 `(__m128h)__builtin_ia32_vcvtdq2ph128_mask((__v4si)__A, (__v8hf)__W,` 从当前函数返回。
- **L1027 EN**: Executes a call or declaration centered on `statement`.
  **L1027 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1028 EN**: Closes the current lexical scope or compound statement.
  **L1028 CN**: 结束当前词法作用域或复合语句块。
- **L1029 EN**: Blank line separating nearby declarations or logic blocks.
  **L1029 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1030 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1030 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1031 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepi32_ph(__mmask8 __U, __m128i __A) {`.
  **L1031 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepi32_ph(__mmask8 __U, __m128i __A) {`。
- **L1032 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtdq2ph128_mask(`.
  **L1032 CN**: 以 `(__m128h)__builtin_ia32_vcvtdq2ph128_mask(` 从当前函数返回。

### Lines 1033-1056

````c
      (__v4si)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtepi32_ph(__m256i __A) {
  return (__m128h) __builtin_convertvector((__v8si)__A, __v8hf);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepi32_ph(__m128h __W, __mmask8 __U, __m256i __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm256_cvtepi32_ph(__A), (__v8hf)__W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepi32_ph(__mmask8 __U, __m256i __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm256_cvtepi32_ph(__A), (__v8hf)_mm_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepu32_ph(__m128i __A) {
  return (__m128h)__builtin_ia32_vcvtudq2ph128_mask(
      (__v4su)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}
````
- **L1033 EN**: Executes a call or declaration centered on `statement`.
  **L1033 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1034 EN**: Closes the current lexical scope or compound statement.
  **L1034 CN**: 结束当前词法作用域或复合语句块。
- **L1035 EN**: Blank line separating nearby declarations or logic blocks.
  **L1035 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1036 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1036 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1037 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepi32_ph(__m256i __A) {`.
  **L1037 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepi32_ph(__m256i __A) {`。
- **L1038 EN**: Returns from the current function with `(__m128h) __builtin_convertvector((__v8si)__A, __v8hf)`.
  **L1038 CN**: 以 `(__m128h) __builtin_convertvector((__v8si)__A, __v8hf)` 从当前函数返回。
- **L1039 EN**: Closes the current lexical scope or compound statement.
  **L1039 CN**: 结束当前词法作用域或复合语句块。
- **L1040 EN**: Blank line separating nearby declarations or logic blocks.
  **L1040 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1041 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1041 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1042 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepi32_ph(__m128h __W, __mmask8 __U, __m256i __A) {`.
  **L1042 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepi32_ph(__m128h __W, __mmask8 __U, __m256i __A) {`。
- **L1043 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1043 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1044 EN**: Executes a call or declaration centered on `statement`.
  **L1044 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1045 EN**: Closes the current lexical scope or compound statement.
  **L1045 CN**: 结束当前词法作用域或复合语句块。
- **L1046 EN**: Blank line separating nearby declarations or logic blocks.
  **L1046 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1047 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1047 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1048 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepi32_ph(__mmask8 __U, __m256i __A) {`.
  **L1048 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepi32_ph(__mmask8 __U, __m256i __A) {`。
- **L1049 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1049 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1050 EN**: Executes a call or declaration centered on `statement`.
  **L1050 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1051 EN**: Closes the current lexical scope or compound statement.
  **L1051 CN**: 结束当前词法作用域或复合语句块。
- **L1052 EN**: Blank line separating nearby declarations or logic blocks.
  **L1052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1053 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepu32_ph(__m128i __A) {`.
  **L1053 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepu32_ph(__m128i __A) {`。
- **L1054 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtudq2ph128_mask(`.
  **L1054 CN**: 以 `(__m128h)__builtin_ia32_vcvtudq2ph128_mask(` 从当前函数返回。
- **L1055 EN**: Executes a call or declaration centered on `statement`.
  **L1055 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1056 EN**: Closes the current lexical scope or compound statement.
  **L1056 CN**: 结束当前词法作用域或复合语句块。

### Lines 1057-1080

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_cvtepu32_ph(__m128h __W, __mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvtudq2ph128_mask((__v4su)__A, (__v8hf)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvtepu32_ph(__mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvtudq2ph128_mask(
      (__v4su)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtepu32_ph(__m256i __A) {
  return (__m128h) __builtin_convertvector((__v8su)__A, __v8hf);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepu32_ph(__m128h __W, __mmask8 __U, __m256i __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm256_cvtepu32_ph(__A), (__v8hf)__W);
}

````
- **L1057 EN**: Blank line separating nearby declarations or logic blocks.
  **L1057 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1058 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1058 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1059 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepu32_ph(__m128h __W, __mmask8 __U, __m128i __A) {`.
  **L1059 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepu32_ph(__m128h __W, __mmask8 __U, __m128i __A) {`。
- **L1060 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtudq2ph128_mask((__v4su)__A, (__v8hf)__W,`.
  **L1060 CN**: 以 `(__m128h)__builtin_ia32_vcvtudq2ph128_mask((__v4su)__A, (__v8hf)__W,` 从当前函数返回。
- **L1061 EN**: Executes a call or declaration centered on `statement`.
  **L1061 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1064 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1064 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1065 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepu32_ph(__mmask8 __U, __m128i __A) {`.
  **L1065 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepu32_ph(__mmask8 __U, __m128i __A) {`。
- **L1066 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtudq2ph128_mask(`.
  **L1066 CN**: 以 `(__m128h)__builtin_ia32_vcvtudq2ph128_mask(` 从当前函数返回。
- **L1067 EN**: Executes a call or declaration centered on `statement`.
  **L1067 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1068 EN**: Closes the current lexical scope or compound statement.
  **L1068 CN**: 结束当前词法作用域或复合语句块。
- **L1069 EN**: Blank line separating nearby declarations or logic blocks.
  **L1069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1070 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1070 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1071 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepu32_ph(__m256i __A) {`.
  **L1071 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepu32_ph(__m256i __A) {`。
- **L1072 EN**: Returns from the current function with `(__m128h) __builtin_convertvector((__v8su)__A, __v8hf)`.
  **L1072 CN**: 以 `(__m128h) __builtin_convertvector((__v8su)__A, __v8hf)` 从当前函数返回。
- **L1073 EN**: Closes the current lexical scope or compound statement.
  **L1073 CN**: 结束当前词法作用域或复合语句块。
- **L1074 EN**: Blank line separating nearby declarations or logic blocks.
  **L1074 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1075 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1075 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1076 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepu32_ph(__m128h __W, __mmask8 __U, __m256i __A) {`.
  **L1076 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepu32_ph(__m128h __W, __mmask8 __U, __m256i __A) {`。
- **L1077 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1077 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1078 EN**: Executes a call or declaration centered on `statement`.
  **L1078 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1079 EN**: Closes the current lexical scope or compound statement.
  **L1079 CN**: 结束当前词法作用域或复合语句块。
- **L1080 EN**: Blank line separating nearby declarations or logic blocks.
  **L1080 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1081-1104

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepu32_ph(__mmask8 __U, __m256i __A) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, (__v8hf)_mm256_cvtepu32_ph(__A), (__v8hf)_mm_setzero_ph());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi32(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2dq128_mask(
      (__v8hf)__A, (__v4si)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttph_epi32(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2dq128_mask((__v8hf)__A, (__v4si)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttph_epi32(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2dq128_mask(
      (__v8hf)__A, (__v4si)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
````
- **L1081 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1081 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1082 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepu32_ph(__mmask8 __U, __m256i __A) {`.
  **L1082 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepu32_ph(__mmask8 __U, __m256i __A) {`。
- **L1083 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1083 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1084 EN**: Executes a call or declaration centered on `statement`.
  **L1084 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1085 EN**: Closes the current lexical scope or compound statement.
  **L1085 CN**: 结束当前词法作用域或复合语句块。
- **L1086 EN**: Blank line separating nearby declarations or logic blocks.
  **L1086 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1087 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi32(__m128h __A) {`.
  **L1087 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi32(__m128h __A) {`。
- **L1088 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2dq128_mask(`.
  **L1088 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2dq128_mask(` 从当前函数返回。
- **L1089 EN**: Executes a call or declaration centered on `statement`.
  **L1089 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1090 EN**: Closes the current lexical scope or compound statement.
  **L1090 CN**: 结束当前词法作用域或复合语句块。
- **L1091 EN**: Blank line separating nearby declarations or logic blocks.
  **L1091 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1092 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1092 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1093 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttph_epi32(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L1093 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttph_epi32(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L1094 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2dq128_mask((__v8hf)__A, (__v4si)__W,`.
  **L1094 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2dq128_mask((__v8hf)__A, (__v4si)__W,` 从当前函数返回。
- **L1095 EN**: Executes a call or declaration centered on `statement`.
  **L1095 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1096 EN**: Closes the current lexical scope or compound statement.
  **L1096 CN**: 结束当前词法作用域或复合语句块。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1098 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1098 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1099 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttph_epi32(__mmask8 __U, __m128h __A) {`.
  **L1099 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttph_epi32(__mmask8 __U, __m128h __A) {`。
- **L1100 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2dq128_mask(`.
  **L1100 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2dq128_mask(` 从当前函数返回。
- **L1101 EN**: Executes a call or declaration centered on `statement`.
  **L1101 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1102 EN**: Closes the current lexical scope or compound statement.
  **L1102 CN**: 结束当前词法作用域或复合语句块。
- **L1103 EN**: Blank line separating nearby declarations or logic blocks.
  **L1103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1104 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1104 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。

### Lines 1105-1128

````c
_mm256_cvttph_epi32(__m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2dq256_mask(
      (__v8hf)__A, (__v8si)_mm256_undefined_si256(), (__mmask8)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttph_epi32(__m256i __W, __mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2dq256_mask((__v8hf)__A, (__v8si)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttph_epi32(__mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2dq256_mask(
      (__v8hf)__A, (__v8si)_mm256_setzero_si256(), (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu32(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2udq128_mask(
      (__v8hf)__A, (__v4su)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttph_epu32(__m128i __W, __mmask8 __U, __m128h __A) {
````
- **L1105 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttph_epi32(__m128h __A) {`.
  **L1105 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttph_epi32(__m128h __A) {`。
- **L1106 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2dq256_mask(`.
  **L1106 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2dq256_mask(` 从当前函数返回。
- **L1107 EN**: Executes a call or declaration centered on `statement`.
  **L1107 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1108 EN**: Closes the current lexical scope or compound statement.
  **L1108 CN**: 结束当前词法作用域或复合语句块。
- **L1109 EN**: Blank line separating nearby declarations or logic blocks.
  **L1109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1110 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1110 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1111 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttph_epi32(__m256i __W, __mmask8 __U, __m128h __A) {`.
  **L1111 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttph_epi32(__m256i __W, __mmask8 __U, __m128h __A) {`。
- **L1112 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2dq256_mask((__v8hf)__A, (__v8si)__W,`.
  **L1112 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2dq256_mask((__v8hf)__A, (__v8si)__W,` 从当前函数返回。
- **L1113 EN**: Executes a call or declaration centered on `statement`.
  **L1113 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1114 EN**: Closes the current lexical scope or compound statement.
  **L1114 CN**: 结束当前词法作用域或复合语句块。
- **L1115 EN**: Blank line separating nearby declarations or logic blocks.
  **L1115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1116 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1116 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1117 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttph_epi32(__mmask8 __U, __m128h __A) {`.
  **L1117 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttph_epi32(__mmask8 __U, __m128h __A) {`。
- **L1118 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2dq256_mask(`.
  **L1118 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2dq256_mask(` 从当前函数返回。
- **L1119 EN**: Executes a call or declaration centered on `statement`.
  **L1119 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1120 EN**: Closes the current lexical scope or compound statement.
  **L1120 CN**: 结束当前词法作用域或复合语句块。
- **L1121 EN**: Blank line separating nearby declarations or logic blocks.
  **L1121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1122 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu32(__m128h __A) {`.
  **L1122 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu32(__m128h __A) {`。
- **L1123 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2udq128_mask(`.
  **L1123 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2udq128_mask(` 从当前函数返回。
- **L1124 EN**: Executes a call or declaration centered on `statement`.
  **L1124 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1125 EN**: Closes the current lexical scope or compound statement.
  **L1125 CN**: 结束当前词法作用域或复合语句块。
- **L1126 EN**: Blank line separating nearby declarations or logic blocks.
  **L1126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1127 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1127 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1128 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttph_epu32(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L1128 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttph_epu32(__m128i __W, __mmask8 __U, __m128h __A) {`。

### Lines 1129-1152

````c
  return (__m128i)__builtin_ia32_vcvttph2udq128_mask((__v8hf)__A, (__v4su)__W,
                                                     (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttph_epu32(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2udq128_mask(
      (__v8hf)__A, (__v4su)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttph_epu32(__m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2udq256_mask(
      (__v8hf)__A, (__v8su)_mm256_undefined_si256(), (__mmask8)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttph_epu32(__m256i __W, __mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2udq256_mask((__v8hf)__A, (__v8su)__W,
                                                     (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttph_epu32(__mmask8 __U, __m128h __A) {
````
- **L1129 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2udq128_mask((__v8hf)__A, (__v4su)__W,`.
  **L1129 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2udq128_mask((__v8hf)__A, (__v4su)__W,` 从当前函数返回。
- **L1130 EN**: Executes a call or declaration centered on `statement`.
  **L1130 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1131 EN**: Closes the current lexical scope or compound statement.
  **L1131 CN**: 结束当前词法作用域或复合语句块。
- **L1132 EN**: Blank line separating nearby declarations or logic blocks.
  **L1132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1133 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1133 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1134 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttph_epu32(__mmask8 __U, __m128h __A) {`.
  **L1134 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttph_epu32(__mmask8 __U, __m128h __A) {`。
- **L1135 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2udq128_mask(`.
  **L1135 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2udq128_mask(` 从当前函数返回。
- **L1136 EN**: Executes a call or declaration centered on `statement`.
  **L1136 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1137 EN**: Closes the current lexical scope or compound statement.
  **L1137 CN**: 结束当前词法作用域或复合语句块。
- **L1138 EN**: Blank line separating nearby declarations or logic blocks.
  **L1138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1139 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1139 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1140 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttph_epu32(__m128h __A) {`.
  **L1140 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttph_epu32(__m128h __A) {`。
- **L1141 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2udq256_mask(`.
  **L1141 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2udq256_mask(` 从当前函数返回。
- **L1142 EN**: Executes a call or declaration centered on `statement`.
  **L1142 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1143 EN**: Closes the current lexical scope or compound statement.
  **L1143 CN**: 结束当前词法作用域或复合语句块。
- **L1144 EN**: Blank line separating nearby declarations or logic blocks.
  **L1144 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1145 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1145 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1146 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttph_epu32(__m256i __W, __mmask8 __U, __m128h __A) {`.
  **L1146 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttph_epu32(__m256i __W, __mmask8 __U, __m128h __A) {`。
- **L1147 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2udq256_mask((__v8hf)__A, (__v8su)__W,`.
  **L1147 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2udq256_mask((__v8hf)__A, (__v8su)__W,` 从当前函数返回。
- **L1148 EN**: Executes a call or declaration centered on `statement`.
  **L1148 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1149 EN**: Closes the current lexical scope or compound statement.
  **L1149 CN**: 结束当前词法作用域或复合语句块。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1151 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1151 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1152 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttph_epu32(__mmask8 __U, __m128h __A) {`.
  **L1152 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttph_epu32(__mmask8 __U, __m128h __A) {`。

### Lines 1153-1176

````c
  return (__m256i)__builtin_ia32_vcvttph2udq256_mask(
      (__v8hf)__A, (__v8su)_mm256_setzero_si256(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepi64_ph(__m128i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph128_mask(
      (__v2di)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph128_mask((__v2di)__A, (__v8hf)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvtepi64_ph(__mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph128_mask(
      (__v2di)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_cvtepi64_ph(__m256i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph256_mask(
````
- **L1153 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2udq256_mask(`.
  **L1153 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2udq256_mask(` 从当前函数返回。
- **L1154 EN**: Executes a call or declaration centered on `statement`.
  **L1154 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1155 EN**: Closes the current lexical scope or compound statement.
  **L1155 CN**: 结束当前词法作用域或复合语句块。
- **L1156 EN**: Blank line separating nearby declarations or logic blocks.
  **L1156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1157 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepi64_ph(__m128i __A) {`.
  **L1157 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepi64_ph(__m128i __A) {`。
- **L1158 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph128_mask(`.
  **L1158 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph128_mask(` 从当前函数返回。
- **L1159 EN**: Executes a call or declaration centered on `statement`.
  **L1159 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1160 EN**: Closes the current lexical scope or compound statement.
  **L1160 CN**: 结束当前词法作用域或复合语句块。
- **L1161 EN**: Blank line separating nearby declarations or logic blocks.
  **L1161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1162 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1162 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m128i __A) {`.
  **L1163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m128i __A) {`。
- **L1164 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph128_mask((__v2di)__A, (__v8hf)__W,`.
  **L1164 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph128_mask((__v2di)__A, (__v8hf)__W,` 从当前函数返回。
- **L1165 EN**: Executes a call or declaration centered on `statement`.
  **L1165 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1166 EN**: Closes the current lexical scope or compound statement.
  **L1166 CN**: 结束当前词法作用域或复合语句块。
- **L1167 EN**: Blank line separating nearby declarations or logic blocks.
  **L1167 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1168 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1168 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1169 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepi64_ph(__mmask8 __U, __m128i __A) {`.
  **L1169 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepi64_ph(__mmask8 __U, __m128i __A) {`。
- **L1170 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph128_mask(`.
  **L1170 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph128_mask(` 从当前函数返回。
- **L1171 EN**: Executes a call or declaration centered on `statement`.
  **L1171 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1172 EN**: Closes the current lexical scope or compound statement.
  **L1172 CN**: 结束当前词法作用域或复合语句块。
- **L1173 EN**: Blank line separating nearby declarations or logic blocks.
  **L1173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1174 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L1174 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L1175 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepi64_ph(__m256i __A) {`.
  **L1175 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepi64_ph(__m256i __A) {`。
- **L1176 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph256_mask(`.
  **L1176 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph256_mask(` 从当前函数返回。

### Lines 1177-1200

````c
      (__v4di)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m256i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph256_mask((__v4di)__A, (__v8hf)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtepi64_ph(__mmask8 __U, __m256i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph256_mask(
      (__v4di)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi64(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2qq128_mask(
      (__v8hf)__A, (__v2di)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtph_epi64(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2qq128_mask((__v8hf)__A, (__v2di)__W,
                                                   (__mmask8)__U);
````
- **L1177 EN**: Executes a call or declaration centered on `statement`.
  **L1177 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1178 EN**: Closes the current lexical scope or compound statement.
  **L1178 CN**: 结束当前词法作用域或复合语句块。
- **L1179 EN**: Blank line separating nearby declarations or logic blocks.
  **L1179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1180 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L1180 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L1181 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m256i __A) {`.
  **L1181 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m256i __A) {`。
- **L1182 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph256_mask((__v4di)__A, (__v8hf)__W,`.
  **L1182 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph256_mask((__v4di)__A, (__v8hf)__W,` 从当前函数返回。
- **L1183 EN**: Executes a call or declaration centered on `statement`.
  **L1183 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1184 EN**: Closes the current lexical scope or compound statement.
  **L1184 CN**: 结束当前词法作用域或复合语句块。
- **L1185 EN**: Blank line separating nearby declarations or logic blocks.
  **L1185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1186 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L1186 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L1187 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepi64_ph(__mmask8 __U, __m256i __A) {`.
  **L1187 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepi64_ph(__mmask8 __U, __m256i __A) {`。
- **L1188 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph256_mask(`.
  **L1188 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph256_mask(` 从当前函数返回。
- **L1189 EN**: Executes a call or declaration centered on `statement`.
  **L1189 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1190 EN**: Closes the current lexical scope or compound statement.
  **L1190 CN**: 结束当前词法作用域或复合语句块。
- **L1191 EN**: Blank line separating nearby declarations or logic blocks.
  **L1191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1192 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi64(__m128h __A) {`.
  **L1192 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epi64(__m128h __A) {`。
- **L1193 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2qq128_mask(`.
  **L1193 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2qq128_mask(` 从当前函数返回。
- **L1194 EN**: Executes a call or declaration centered on `statement`.
  **L1194 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1195 EN**: Closes the current lexical scope or compound statement.
  **L1195 CN**: 结束当前词法作用域或复合语句块。
- **L1196 EN**: Blank line separating nearby declarations or logic blocks.
  **L1196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1197 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1197 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1198 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtph_epi64(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L1198 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtph_epi64(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L1199 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2qq128_mask((__v8hf)__A, (__v2di)__W,`.
  **L1199 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2qq128_mask((__v8hf)__A, (__v2di)__W,` 从当前函数返回。
- **L1200 EN**: Executes a call or declaration centered on `statement`.
  **L1200 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1201-1224

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2qq128_mask(
      (__v8hf)__A, (__v2di)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtph_epi64(__m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2qq256_mask(
      (__v8hf)__A, (__v4di)_mm256_undefined_si256(), (__mmask8)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_epi64(__m256i __W, __mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2qq256_mask((__v8hf)__A, (__v4di)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2qq256_mask(
      (__v8hf)__A, (__v4di)_mm256_setzero_si256(), (__mmask8)__U);
````
- **L1201 EN**: Closes the current lexical scope or compound statement.
  **L1201 CN**: 结束当前词法作用域或复合语句块。
- **L1202 EN**: Blank line separating nearby declarations or logic blocks.
  **L1202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1203 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1203 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1204 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {`.
  **L1204 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {`。
- **L1205 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2qq128_mask(`.
  **L1205 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2qq128_mask(` 从当前函数返回。
- **L1206 EN**: Executes a call or declaration centered on `statement`.
  **L1206 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1209 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1209 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1210 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtph_epi64(__m128h __A) {`.
  **L1210 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtph_epi64(__m128h __A) {`。
- **L1211 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2qq256_mask(`.
  **L1211 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2qq256_mask(` 从当前函数返回。
- **L1212 EN**: Executes a call or declaration centered on `statement`.
  **L1212 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1213 EN**: Closes the current lexical scope or compound statement.
  **L1213 CN**: 结束当前词法作用域或复合语句块。
- **L1214 EN**: Blank line separating nearby declarations or logic blocks.
  **L1214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1215 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1215 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1216 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_epi64(__m256i __W, __mmask8 __U, __m128h __A) {`.
  **L1216 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_epi64(__m256i __W, __mmask8 __U, __m128h __A) {`。
- **L1217 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2qq256_mask((__v8hf)__A, (__v4di)__W,`.
  **L1217 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2qq256_mask((__v8hf)__A, (__v4di)__W,` 从当前函数返回。
- **L1218 EN**: Executes a call or declaration centered on `statement`.
  **L1218 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1219 EN**: Closes the current lexical scope or compound statement.
  **L1219 CN**: 结束当前词法作用域或复合语句块。
- **L1220 EN**: Blank line separating nearby declarations or logic blocks.
  **L1220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1221 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1221 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1222 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {`.
  **L1222 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {`。
- **L1223 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2qq256_mask(`.
  **L1223 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2qq256_mask(` 从当前函数返回。
- **L1224 EN**: Executes a call or declaration centered on `statement`.
  **L1224 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1225-1248

````c
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepu64_ph(__m128i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph128_mask(
      (__v2du)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph128_mask((__v2du)__A, (__v8hf)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvtepu64_ph(__mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph128_mask(
      (__v2du)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_cvtepu64_ph(__m256i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph256_mask(
      (__v4du)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}
````
- **L1225 EN**: Closes the current lexical scope or compound statement.
  **L1225 CN**: 结束当前词法作用域或复合语句块。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepu64_ph(__m128i __A) {`.
  **L1227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtepu64_ph(__m128i __A) {`。
- **L1228 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph128_mask(`.
  **L1228 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph128_mask(` 从当前函数返回。
- **L1229 EN**: Executes a call or declaration centered on `statement`.
  **L1229 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1230 EN**: Closes the current lexical scope or compound statement.
  **L1230 CN**: 结束当前词法作用域或复合语句块。
- **L1231 EN**: Blank line separating nearby declarations or logic blocks.
  **L1231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1232 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1232 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1233 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m128i __A) {`.
  **L1233 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m128i __A) {`。
- **L1234 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph128_mask((__v2du)__A, (__v8hf)__W,`.
  **L1234 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph128_mask((__v2du)__A, (__v8hf)__W,` 从当前函数返回。
- **L1235 EN**: Executes a call or declaration centered on `statement`.
  **L1235 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1236 EN**: Closes the current lexical scope or compound statement.
  **L1236 CN**: 结束当前词法作用域或复合语句块。
- **L1237 EN**: Blank line separating nearby declarations or logic blocks.
  **L1237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1238 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1238 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1239 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepu64_ph(__mmask8 __U, __m128i __A) {`.
  **L1239 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepu64_ph(__mmask8 __U, __m128i __A) {`。
- **L1240 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph128_mask(`.
  **L1240 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph128_mask(` 从当前函数返回。
- **L1241 EN**: Executes a call or declaration centered on `statement`.
  **L1241 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1242 EN**: Closes the current lexical scope or compound statement.
  **L1242 CN**: 结束当前词法作用域或复合语句块。
- **L1243 EN**: Blank line separating nearby declarations or logic blocks.
  **L1243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1244 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L1244 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L1245 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepu64_ph(__m256i __A) {`.
  **L1245 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepu64_ph(__m256i __A) {`。
- **L1246 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph256_mask(`.
  **L1246 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph256_mask(` 从当前函数返回。
- **L1247 EN**: Executes a call or declaration centered on `statement`.
  **L1247 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1248 EN**: Closes the current lexical scope or compound statement.
  **L1248 CN**: 结束当前词法作用域或复合语句块。

### Lines 1249-1272

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m256i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph256_mask((__v4du)__A, (__v8hf)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtepu64_ph(__mmask8 __U, __m256i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph256_mask(
      (__v4du)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu64(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2uqq128_mask(
      (__v8hf)__A, (__v2du)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtph_epu64(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2uqq128_mask((__v8hf)__A, (__v2du)__W,
                                                    (__mmask8)__U);
}

````
- **L1249 EN**: Blank line separating nearby declarations or logic blocks.
  **L1249 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1250 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L1250 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L1251 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m256i __A) {`.
  **L1251 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m256i __A) {`。
- **L1252 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph256_mask((__v4du)__A, (__v8hf)__W,`.
  **L1252 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph256_mask((__v4du)__A, (__v8hf)__W,` 从当前函数返回。
- **L1253 EN**: Executes a call or declaration centered on `statement`.
  **L1253 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1254 EN**: Closes the current lexical scope or compound statement.
  **L1254 CN**: 结束当前词法作用域或复合语句块。
- **L1255 EN**: Blank line separating nearby declarations or logic blocks.
  **L1255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1256 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L1256 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L1257 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepu64_ph(__mmask8 __U, __m256i __A) {`.
  **L1257 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepu64_ph(__mmask8 __U, __m256i __A) {`。
- **L1258 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph256_mask(`.
  **L1258 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph256_mask(` 从当前函数返回。
- **L1259 EN**: Executes a call or declaration centered on `statement`.
  **L1259 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1260 EN**: Closes the current lexical scope or compound statement.
  **L1260 CN**: 结束当前词法作用域或复合语句块。
- **L1261 EN**: Blank line separating nearby declarations or logic blocks.
  **L1261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1262 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu64(__m128h __A) {`.
  **L1262 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_epu64(__m128h __A) {`。
- **L1263 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2uqq128_mask(`.
  **L1263 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2uqq128_mask(` 从当前函数返回。
- **L1264 EN**: Executes a call or declaration centered on `statement`.
  **L1264 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1265 EN**: Closes the current lexical scope or compound statement.
  **L1265 CN**: 结束当前词法作用域或复合语句块。
- **L1266 EN**: Blank line separating nearby declarations or logic blocks.
  **L1266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1267 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1267 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1268 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtph_epu64(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L1268 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtph_epu64(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L1269 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2uqq128_mask((__v8hf)__A, (__v2du)__W,`.
  **L1269 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2uqq128_mask((__v8hf)__A, (__v2du)__W,` 从当前函数返回。
- **L1270 EN**: Executes a call or declaration centered on `statement`.
  **L1270 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1271 EN**: Closes the current lexical scope or compound statement.
  **L1271 CN**: 结束当前词法作用域或复合语句块。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1273-1296

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2uqq128_mask(
      (__v8hf)__A, (__v2du)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtph_epu64(__m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2uqq256_mask(
      (__v8hf)__A, (__v4du)_mm256_undefined_si256(), (__mmask8)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_epu64(__m256i __W, __mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2uqq256_mask((__v8hf)__A, (__v4du)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvtph2uqq256_mask(
      (__v8hf)__A, (__v4du)_mm256_setzero_si256(), (__mmask8)__U);
}

````
- **L1273 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1273 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1274 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {`.
  **L1274 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {`。
- **L1275 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2uqq128_mask(`.
  **L1275 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2uqq128_mask(` 从当前函数返回。
- **L1276 EN**: Executes a call or declaration centered on `statement`.
  **L1276 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1277 EN**: Closes the current lexical scope or compound statement.
  **L1277 CN**: 结束当前词法作用域或复合语句块。
- **L1278 EN**: Blank line separating nearby declarations or logic blocks.
  **L1278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1279 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1279 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1280 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtph_epu64(__m128h __A) {`.
  **L1280 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtph_epu64(__m128h __A) {`。
- **L1281 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2uqq256_mask(`.
  **L1281 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2uqq256_mask(` 从当前函数返回。
- **L1282 EN**: Executes a call or declaration centered on `statement`.
  **L1282 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1283 EN**: Closes the current lexical scope or compound statement.
  **L1283 CN**: 结束当前词法作用域或复合语句块。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1285 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1285 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1286 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_epu64(__m256i __W, __mmask8 __U, __m128h __A) {`.
  **L1286 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_epu64(__m256i __W, __mmask8 __U, __m128h __A) {`。
- **L1287 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2uqq256_mask((__v8hf)__A, (__v4du)__W,`.
  **L1287 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2uqq256_mask((__v8hf)__A, (__v4du)__W,` 从当前函数返回。
- **L1288 EN**: Executes a call or declaration centered on `statement`.
  **L1288 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1289 EN**: Closes the current lexical scope or compound statement.
  **L1289 CN**: 结束当前词法作用域或复合语句块。
- **L1290 EN**: Blank line separating nearby declarations or logic blocks.
  **L1290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1291 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1291 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1292 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {`.
  **L1292 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {`。
- **L1293 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvtph2uqq256_mask(`.
  **L1293 CN**: 以 `(__m256i)__builtin_ia32_vcvtph2uqq256_mask(` 从当前函数返回。
- **L1294 EN**: Executes a call or declaration centered on `statement`.
  **L1294 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1295 EN**: Closes the current lexical scope or compound statement.
  **L1295 CN**: 结束当前词法作用域或复合语句块。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1297-1320

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi64(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2qq128_mask(
      (__v8hf)__A, (__v2di)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttph_epi64(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2qq128_mask((__v8hf)__A, (__v2di)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2qq128_mask(
      (__v8hf)__A, (__v2di)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttph_epi64(__m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2qq256_mask(
      (__v8hf)__A, (__v4di)_mm256_undefined_si256(), (__mmask8)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
````
- **L1297 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi64(__m128h __A) {`.
  **L1297 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epi64(__m128h __A) {`。
- **L1298 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2qq128_mask(`.
  **L1298 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2qq128_mask(` 从当前函数返回。
- **L1299 EN**: Executes a call or declaration centered on `statement`.
  **L1299 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1300 EN**: Closes the current lexical scope or compound statement.
  **L1300 CN**: 结束当前词法作用域或复合语句块。
- **L1301 EN**: Blank line separating nearby declarations or logic blocks.
  **L1301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1302 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1302 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1303 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttph_epi64(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L1303 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttph_epi64(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L1304 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2qq128_mask((__v8hf)__A, (__v2di)__W,`.
  **L1304 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2qq128_mask((__v8hf)__A, (__v2di)__W,` 从当前函数返回。
- **L1305 EN**: Executes a call or declaration centered on `statement`.
  **L1305 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1306 EN**: Closes the current lexical scope or compound statement.
  **L1306 CN**: 结束当前词法作用域或复合语句块。
- **L1307 EN**: Blank line separating nearby declarations or logic blocks.
  **L1307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1308 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1308 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1309 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {`.
  **L1309 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {`。
- **L1310 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2qq128_mask(`.
  **L1310 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2qq128_mask(` 从当前函数返回。
- **L1311 EN**: Executes a call or declaration centered on `statement`.
  **L1311 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1312 EN**: Closes the current lexical scope or compound statement.
  **L1312 CN**: 结束当前词法作用域或复合语句块。
- **L1313 EN**: Blank line separating nearby declarations or logic blocks.
  **L1313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1314 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1314 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1315 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttph_epi64(__m128h __A) {`.
  **L1315 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttph_epi64(__m128h __A) {`。
- **L1316 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2qq256_mask(`.
  **L1316 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2qq256_mask(` 从当前函数返回。
- **L1317 EN**: Executes a call or declaration centered on `statement`.
  **L1317 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1318 EN**: Closes the current lexical scope or compound statement.
  **L1318 CN**: 结束当前词法作用域或复合语句块。
- **L1319 EN**: Blank line separating nearby declarations or logic blocks.
  **L1319 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1320 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1320 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。

### Lines 1321-1344

````c
_mm256_mask_cvttph_epi64(__m256i __W, __mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2qq256_mask((__v8hf)__A, (__v4di)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2qq256_mask(
      (__v8hf)__A, (__v4di)_mm256_setzero_si256(), (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu64(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2uqq128_mask(
      (__v8hf)__A, (__v2du)_mm_undefined_si128(), (__mmask8)-1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttph_epu64(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvttph2uqq128_mask((__v8hf)__A, (__v2du)__W,
                                                     (__mmask8)__U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {
````
- **L1321 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttph_epi64(__m256i __W, __mmask8 __U, __m128h __A) {`.
  **L1321 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttph_epi64(__m256i __W, __mmask8 __U, __m128h __A) {`。
- **L1322 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2qq256_mask((__v8hf)__A, (__v4di)__W,`.
  **L1322 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2qq256_mask((__v8hf)__A, (__v4di)__W,` 从当前函数返回。
- **L1323 EN**: Executes a call or declaration centered on `statement`.
  **L1323 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1324 EN**: Closes the current lexical scope or compound statement.
  **L1324 CN**: 结束当前词法作用域或复合语句块。
- **L1325 EN**: Blank line separating nearby declarations or logic blocks.
  **L1325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1326 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1326 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1327 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {`.
  **L1327 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {`。
- **L1328 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2qq256_mask(`.
  **L1328 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2qq256_mask(` 从当前函数返回。
- **L1329 EN**: Executes a call or declaration centered on `statement`.
  **L1329 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1330 EN**: Closes the current lexical scope or compound statement.
  **L1330 CN**: 结束当前词法作用域或复合语句块。
- **L1331 EN**: Blank line separating nearby declarations or logic blocks.
  **L1331 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1332 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu64(__m128h __A) {`.
  **L1332 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvttph_epu64(__m128h __A) {`。
- **L1333 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2uqq128_mask(`.
  **L1333 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2uqq128_mask(` 从当前函数返回。
- **L1334 EN**: Executes a call or declaration centered on `statement`.
  **L1334 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1335 EN**: Closes the current lexical scope or compound statement.
  **L1335 CN**: 结束当前词法作用域或复合语句块。
- **L1336 EN**: Blank line separating nearby declarations or logic blocks.
  **L1336 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1337 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1337 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1338 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttph_epu64(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L1338 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttph_epu64(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L1339 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2uqq128_mask((__v8hf)__A, (__v2du)__W,`.
  **L1339 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2uqq128_mask((__v8hf)__A, (__v2du)__W,` 从当前函数返回。
- **L1340 EN**: Executes a call or declaration centered on `statement`.
  **L1340 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1341 EN**: Closes the current lexical scope or compound statement.
  **L1341 CN**: 结束当前词法作用域或复合语句块。
- **L1342 EN**: Blank line separating nearby declarations or logic blocks.
  **L1342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1343 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1343 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1344 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {`.
  **L1344 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {`。

### Lines 1345-1368

````c
  return (__m128i)__builtin_ia32_vcvttph2uqq128_mask(
      (__v8hf)__A, (__v2du)_mm_setzero_si128(), (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttph_epu64(__m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2uqq256_mask(
      (__v8hf)__A, (__v4du)_mm256_undefined_si256(), (__mmask8)-1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttph_epu64(__m256i __W, __mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2uqq256_mask((__v8hf)__A, (__v4du)__W,
                                                     (__mmask8)__U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {
  return (__m256i)__builtin_ia32_vcvttph2uqq256_mask(
      (__v8hf)__A, (__v4du)_mm256_setzero_si256(), (__mmask8)__U);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_cvtxph_ps(__m128h __A) {
  return (__m128)__builtin_ia32_vcvtph2psx128_mask(
````
- **L1345 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvttph2uqq128_mask(`.
  **L1345 CN**: 以 `(__m128i)__builtin_ia32_vcvttph2uqq128_mask(` 从当前函数返回。
- **L1346 EN**: Executes a call or declaration centered on `statement`.
  **L1346 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1347 EN**: Closes the current lexical scope or compound statement.
  **L1347 CN**: 结束当前词法作用域或复合语句块。
- **L1348 EN**: Blank line separating nearby declarations or logic blocks.
  **L1348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1349 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1349 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1350 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttph_epu64(__m128h __A) {`.
  **L1350 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttph_epu64(__m128h __A) {`。
- **L1351 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2uqq256_mask(`.
  **L1351 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2uqq256_mask(` 从当前函数返回。
- **L1352 EN**: Executes a call or declaration centered on `statement`.
  **L1352 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1353 EN**: Closes the current lexical scope or compound statement.
  **L1353 CN**: 结束当前词法作用域或复合语句块。
- **L1354 EN**: Blank line separating nearby declarations or logic blocks.
  **L1354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1355 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1355 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1356 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttph_epu64(__m256i __W, __mmask8 __U, __m128h __A) {`.
  **L1356 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttph_epu64(__m256i __W, __mmask8 __U, __m128h __A) {`。
- **L1357 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2uqq256_mask((__v8hf)__A, (__v4du)__W,`.
  **L1357 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2uqq256_mask((__v8hf)__A, (__v4du)__W,` 从当前函数返回。
- **L1358 EN**: Executes a call or declaration centered on `statement`.
  **L1358 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1359 EN**: Closes the current lexical scope or compound statement.
  **L1359 CN**: 结束当前词法作用域或复合语句块。
- **L1360 EN**: Blank line separating nearby declarations or logic blocks.
  **L1360 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1361 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1361 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1362 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {`.
  **L1362 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {`。
- **L1363 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvttph2uqq256_mask(`.
  **L1363 CN**: 以 `(__m256i)__builtin_ia32_vcvttph2uqq256_mask(` 从当前函数返回。
- **L1364 EN**: Executes a call or declaration centered on `statement`.
  **L1364 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1365 EN**: Closes the current lexical scope or compound statement.
  **L1365 CN**: 结束当前词法作用域或复合语句块。
- **L1366 EN**: Blank line separating nearby declarations or logic blocks.
  **L1366 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1367 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_cvtxph_ps(__m128h __A) {`.
  **L1367 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_cvtxph_ps(__m128h __A) {`。
- **L1368 EN**: Returns from the current function with `(__m128)__builtin_ia32_vcvtph2psx128_mask(`.
  **L1368 CN**: 以 `(__m128)__builtin_ia32_vcvtph2psx128_mask(` 从当前函数返回。

### Lines 1369-1392

````c
      (__v8hf)__A, (__v4sf)_mm_undefined_ps(), (__mmask8)-1);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_mask_cvtxph_ps(__m128 __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A) {
  return (__m128)__builtin_ia32_vcvtph2psx128_mask((__v8hf)__A, (__v4sf)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_maskz_cvtxph_ps(__mmask8 __U, __m128h __A) {
  return (__m128)__builtin_ia32_vcvtph2psx128_mask(
      (__v8hf)__A, (__v4sf)_mm_setzero_ps(), (__mmask8)__U);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256 _mm256_cvtxph_ps(__m128h __A) {
  return (__m256)__builtin_ia32_vcvtph2psx256_mask(
      (__v8hf)__A, (__v8sf)_mm256_undefined_ps(), (__mmask8)-1);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_mask_cvtxph_ps(__m256 __W, __mmask8 __U, __m128h __A) {
  return (__m256)__builtin_ia32_vcvtph2psx256_mask((__v8hf)__A, (__v8sf)__W,
````
- **L1369 EN**: Executes a call or declaration centered on `statement`.
  **L1369 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1370 EN**: Closes the current lexical scope or compound statement.
  **L1370 CN**: 结束当前词法作用域或复合语句块。
- **L1371 EN**: Blank line separating nearby declarations or logic blocks.
  **L1371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_mask_cvtxph_ps(__m128 __W,`.
  **L1372 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_mask_cvtxph_ps(__m128 __W,`。
- **L1373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1373 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1374 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L1374 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L1375 EN**: Returns from the current function with `(__m128)__builtin_ia32_vcvtph2psx128_mask((__v8hf)__A, (__v4sf)__W,`.
  **L1375 CN**: 以 `(__m128)__builtin_ia32_vcvtph2psx128_mask((__v8hf)__A, (__v4sf)__W,` 从当前函数返回。
- **L1376 EN**: Executes a call or declaration centered on `statement`.
  **L1376 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1377 EN**: Closes the current lexical scope or compound statement.
  **L1377 CN**: 结束当前词法作用域或复合语句块。
- **L1378 EN**: Blank line separating nearby declarations or logic blocks.
  **L1378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1379 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L1379 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L1380 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtxph_ps(__mmask8 __U, __m128h __A) {`.
  **L1380 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtxph_ps(__mmask8 __U, __m128h __A) {`。
- **L1381 EN**: Returns from the current function with `(__m128)__builtin_ia32_vcvtph2psx128_mask(`.
  **L1381 CN**: 以 `(__m128)__builtin_ia32_vcvtph2psx128_mask(` 从当前函数返回。
- **L1382 EN**: Executes a call or declaration centered on `statement`.
  **L1382 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1383 EN**: Closes the current lexical scope or compound statement.
  **L1383 CN**: 结束当前词法作用域或复合语句块。
- **L1384 EN**: Blank line separating nearby declarations or logic blocks.
  **L1384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256 __DEFAULT_FN_ATTRS256 _mm256_cvtxph_ps(__m128h __A) {`.
  **L1385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256 __DEFAULT_FN_ATTRS256 _mm256_cvtxph_ps(__m128h __A) {`。
- **L1386 EN**: Returns from the current function with `(__m256)__builtin_ia32_vcvtph2psx256_mask(`.
  **L1386 CN**: 以 `(__m256)__builtin_ia32_vcvtph2psx256_mask(` 从当前函数返回。
- **L1387 EN**: Executes a call or declaration centered on `statement`.
  **L1387 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1388 EN**: Closes the current lexical scope or compound statement.
  **L1388 CN**: 结束当前词法作用域或复合语句块。
- **L1389 EN**: Blank line separating nearby declarations or logic blocks.
  **L1389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1390 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L1390 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L1391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtxph_ps(__m256 __W, __mmask8 __U, __m128h __A) {`.
  **L1391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtxph_ps(__m256 __W, __mmask8 __U, __m128h __A) {`。
- **L1392 EN**: Returns from the current function with `(__m256)__builtin_ia32_vcvtph2psx256_mask((__v8hf)__A, (__v8sf)__W,`.
  **L1392 CN**: 以 `(__m256)__builtin_ia32_vcvtph2psx256_mask((__v8hf)__A, (__v8sf)__W,` 从当前函数返回。

### Lines 1393-1416

````c
                                                   (__mmask8)__U);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtxph_ps(__mmask8 __U, __m128h __A) {
  return (__m256)__builtin_ia32_vcvtph2psx256_mask(
      (__v8hf)__A, (__v8sf)_mm256_setzero_ps(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtxps_ph(__m128 __A) {
  return (__m128h)__builtin_ia32_vcvtps2phx128_mask(
      (__v4sf)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtxps_ph(__m128h __W,
                                                                   __mmask8 __U,
                                                                   __m128 __A) {
  return (__m128h)__builtin_ia32_vcvtps2phx128_mask((__v4sf)__A, (__v8hf)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvtxps_ph(__mmask8 __U, __m128 __A) {
  return (__m128h)__builtin_ia32_vcvtps2phx128_mask(
````
- **L1393 EN**: Executes a call or declaration centered on `statement`.
  **L1393 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1394 EN**: Closes the current lexical scope or compound statement.
  **L1394 CN**: 结束当前词法作用域或复合语句块。
- **L1395 EN**: Blank line separating nearby declarations or logic blocks.
  **L1395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1396 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L1396 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L1397 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtxph_ps(__mmask8 __U, __m128h __A) {`.
  **L1397 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtxph_ps(__mmask8 __U, __m128h __A) {`。
- **L1398 EN**: Returns from the current function with `(__m256)__builtin_ia32_vcvtph2psx256_mask(`.
  **L1398 CN**: 以 `(__m256)__builtin_ia32_vcvtph2psx256_mask(` 从当前函数返回。
- **L1399 EN**: Executes a call or declaration centered on `statement`.
  **L1399 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1400 EN**: Closes the current lexical scope or compound statement.
  **L1400 CN**: 结束当前词法作用域或复合语句块。
- **L1401 EN**: Blank line separating nearby declarations or logic blocks.
  **L1401 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1402 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtxps_ph(__m128 __A) {`.
  **L1402 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtxps_ph(__m128 __A) {`。
- **L1403 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtps2phx128_mask(`.
  **L1403 CN**: 以 `(__m128h)__builtin_ia32_vcvtps2phx128_mask(` 从当前函数返回。
- **L1404 EN**: Executes a call or declaration centered on `statement`.
  **L1404 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1405 EN**: Closes the current lexical scope or compound statement.
  **L1405 CN**: 结束当前词法作用域或复合语句块。
- **L1406 EN**: Blank line separating nearby declarations or logic blocks.
  **L1406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtxps_ph(__m128h __W,`.
  **L1407 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtxps_ph(__m128h __W,`。
- **L1408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1408 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1409 EN**: Continues the surrounding expression or declaration: `__m128 __A) {`.
  **L1409 CN**: 继续构造周围的表达式或声明：`__m128 __A) {`。
- **L1410 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtps2phx128_mask((__v4sf)__A, (__v8hf)__W,`.
  **L1410 CN**: 以 `(__m128h)__builtin_ia32_vcvtps2phx128_mask((__v4sf)__A, (__v8hf)__W,` 从当前函数返回。
- **L1411 EN**: Executes a call or declaration centered on `statement`.
  **L1411 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1412 EN**: Closes the current lexical scope or compound statement.
  **L1412 CN**: 结束当前词法作用域或复合语句块。
- **L1413 EN**: Blank line separating nearby declarations or logic blocks.
  **L1413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1414 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1414 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1415 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtxps_ph(__mmask8 __U, __m128 __A) {`.
  **L1415 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtxps_ph(__mmask8 __U, __m128 __A) {`。
- **L1416 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtps2phx128_mask(`.
  **L1416 CN**: 以 `(__m128h)__builtin_ia32_vcvtps2phx128_mask(` 从当前函数返回。

### Lines 1417-1440

````c
      (__v4sf)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256 _mm256_cvtxps_ph(__m256 __A) {
  return (__m128h)__builtin_ia32_vcvtps2phx256_mask(
      (__v8sf)__A, (__v8hf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_mask_cvtxps_ph(__m128h __W, __mmask8 __U, __m256 __A) {
  return (__m128h)__builtin_ia32_vcvtps2phx256_mask((__v8sf)__A, (__v8hf)__W,
                                                    (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtxps_ph(__mmask8 __U, __m256 __A) {
  return (__m128h)__builtin_ia32_vcvtps2phx256_mask(
      (__v8sf)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_ph(__m128h __A,
                                                             __m128h __B,
                                                             __m128h __C) {
  return (__m128h)__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B,
````
- **L1417 EN**: Executes a call or declaration centered on `statement`.
  **L1417 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1418 EN**: Closes the current lexical scope or compound statement.
  **L1418 CN**: 结束当前词法作用域或复合语句块。
- **L1419 EN**: Blank line separating nearby declarations or logic blocks.
  **L1419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1420 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS256 _mm256_cvtxps_ph(__m256 __A) {`.
  **L1420 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS256 _mm256_cvtxps_ph(__m256 __A) {`。
- **L1421 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtps2phx256_mask(`.
  **L1421 CN**: 以 `(__m128h)__builtin_ia32_vcvtps2phx256_mask(` 从当前函数返回。
- **L1422 EN**: Executes a call or declaration centered on `statement`.
  **L1422 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1423 EN**: Closes the current lexical scope or compound statement.
  **L1423 CN**: 结束当前词法作用域或复合语句块。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1425 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L1425 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L1426 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtxps_ph(__m128h __W, __mmask8 __U, __m256 __A) {`.
  **L1426 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtxps_ph(__m128h __W, __mmask8 __U, __m256 __A) {`。
- **L1427 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtps2phx256_mask((__v8sf)__A, (__v8hf)__W,`.
  **L1427 CN**: 以 `(__m128h)__builtin_ia32_vcvtps2phx256_mask((__v8sf)__A, (__v8hf)__W,` 从当前函数返回。
- **L1428 EN**: Executes a call or declaration centered on `statement`.
  **L1428 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1429 EN**: Closes the current lexical scope or compound statement.
  **L1429 CN**: 结束当前词法作用域或复合语句块。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1431 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256`.
  **L1431 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256`。
- **L1432 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtxps_ph(__mmask8 __U, __m256 __A) {`.
  **L1432 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtxps_ph(__mmask8 __U, __m256 __A) {`。
- **L1433 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtps2phx256_mask(`.
  **L1433 CN**: 以 `(__m128h)__builtin_ia32_vcvtps2phx256_mask(` 从当前函数返回。
- **L1434 EN**: Executes a call or declaration centered on `statement`.
  **L1434 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Blank line separating nearby declarations or logic blocks.
  **L1436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_ph(__m128h __A,`.
  **L1437 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_ph(__m128h __A,`。
- **L1438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1438 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1439 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1439 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1440 EN**: Returns from the current function with `(__m128h)__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B,`.
  **L1440 CN**: 以 `(__m128h)__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。

### Lines 1441-1464

````c
                                            (__v8hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmadd_ph(__m128h __A,
                                                                  __mmask8 __U,
                                                                  __m128h __B,
                                                                  __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)__A);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fmadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
````
- **L1441 EN**: Executes a call or declaration centered on `statement`.
  **L1441 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1442 EN**: Closes the current lexical scope or compound statement.
  **L1442 CN**: 结束当前词法作用域或复合语句块。
- **L1443 EN**: Blank line separating nearby declarations or logic blocks.
  **L1443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmadd_ph(__m128h __A,`.
  **L1444 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmadd_ph(__m128h __A,`。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1446 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1447 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1447 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1448 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1448 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1449 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1450 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1451 EN**: Executes a call or declaration centered on `statement`.
  **L1451 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1452 EN**: Closes the current lexical scope or compound statement.
  **L1452 CN**: 结束当前词法作用域或复合语句块。
- **L1453 EN**: Blank line separating nearby declarations or logic blocks.
  **L1453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1454 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1454 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1455 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L1455 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L1456 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1456 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1457 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1458 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1459 EN**: Executes a call or declaration centered on `statement`.
  **L1459 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1460 EN**: Closes the current lexical scope or compound statement.
  **L1460 CN**: 结束当前词法作用域或复合语句块。
- **L1461 EN**: Blank line separating nearby declarations or logic blocks.
  **L1461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1462 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1462 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1463 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L1463 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L1464 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1464 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。

### Lines 1465-1488

````c
      (__mmask8)__U,
      __builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)_mm_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsub_ph(__m128h __A,
                                                             __m128h __B,
                                                             __m128h __C) {
  return (__m128h)__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B,
                                            -(__v8hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmsub_ph(__m128h __A,
                                                                  __mmask8 __U,
                                                                  __m128h __B,
                                                                  __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U, _mm_fmsub_ph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)__A);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
````
- **L1465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1465 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1466 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1467 EN**: Executes a call or declaration centered on `statement`.
  **L1467 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1468 EN**: Closes the current lexical scope or compound statement.
  **L1468 CN**: 结束当前词法作用域或复合语句块。
- **L1469 EN**: Blank line separating nearby declarations or logic blocks.
  **L1469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsub_ph(__m128h __A,`.
  **L1470 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsub_ph(__m128h __A,`。
- **L1471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1471 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1472 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1472 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1473 EN**: Returns from the current function with `(__m128h)__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B,`.
  **L1473 CN**: 以 `(__m128h)__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L1474 EN**: Executes a call or declaration centered on `-`.
  **L1474 CN**: 执行以 `-` 为核心的调用或声明。
- **L1475 EN**: Closes the current lexical scope or compound statement.
  **L1475 CN**: 结束当前词法作用域或复合语句块。
- **L1476 EN**: Blank line separating nearby declarations or logic blocks.
  **L1476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmsub_ph(__m128h __A,`.
  **L1477 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmsub_ph(__m128h __A,`。
- **L1478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1478 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1479 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1480 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1480 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1481 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1481 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fmsub_ph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1482 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fmsub_ph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1483 EN**: Executes a call or declaration centered on `statement`.
  **L1483 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1484 EN**: Closes the current lexical scope or compound statement.
  **L1484 CN**: 结束当前词法作用域或复合语句块。
- **L1485 EN**: Blank line separating nearby declarations or logic blocks.
  **L1485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1486 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1486 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1487 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L1487 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L1488 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1488 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。

### Lines 1489-1512

````c
      (__mmask8)__U, _mm_fmsub_ph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)_mm_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fnmadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fnmadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)_mm_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fnmsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
````
- **L1489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fmsub_ph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1489 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fmsub_ph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1490 EN**: Executes a call or declaration centered on `statement`.
  **L1490 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1491 EN**: Closes the current lexical scope or compound statement.
  **L1491 CN**: 结束当前词法作用域或复合语句块。
- **L1492 EN**: Blank line separating nearby declarations or logic blocks.
  **L1492 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1493 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1493 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1494 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fnmadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L1494 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fnmadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L1495 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1495 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1496 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1497 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1498 EN**: Executes a call or declaration centered on `statement`.
  **L1498 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1499 EN**: Closes the current lexical scope or compound statement.
  **L1499 CN**: 结束当前词法作用域或复合语句块。
- **L1500 EN**: Blank line separating nearby declarations or logic blocks.
  **L1500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1501 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1501 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1502 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fnmadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L1502 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fnmadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L1503 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1503 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1504 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1505 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1506 EN**: Executes a call or declaration centered on `statement`.
  **L1506 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1507 EN**: Closes the current lexical scope or compound statement.
  **L1507 CN**: 结束当前词法作用域或复合语句块。
- **L1508 EN**: Blank line separating nearby declarations or logic blocks.
  **L1508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1509 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1509 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1510 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fnmsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L1510 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fnmsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L1511 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1511 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1512 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。

### Lines 1513-1536

````c
      __builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),
      (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmadd_ph(__m256h __A,
                                                                __m256h __B,
                                                                __m256h __C) {
  return (__m256h)__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B,
                                            (__v16hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fmadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),
      (__v16hf)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask3_fmadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),
````
- **L1513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`.
  **L1513 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma(-(__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`。
- **L1514 EN**: Executes a call or declaration centered on `statement`.
  **L1514 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1515 EN**: Closes the current lexical scope or compound statement.
  **L1515 CN**: 结束当前词法作用域或复合语句块。
- **L1516 EN**: Blank line separating nearby declarations or logic blocks.
  **L1516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmadd_ph(__m256h __A,`.
  **L1517 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmadd_ph(__m256h __A,`。
- **L1518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m256h __B,`.
  **L1518 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m256h __B,`。
- **L1519 EN**: Continues the surrounding expression or declaration: `__m256h __C) {`.
  **L1519 CN**: 继续构造周围的表达式或声明：`__m256h __C) {`。
- **L1520 EN**: Returns from the current function with `(__m256h)__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B,`.
  **L1520 CN**: 以 `(__m256h)__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B,` 从当前函数返回。
- **L1521 EN**: Executes a call or declaration centered on `statement`.
  **L1521 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1522 EN**: Closes the current lexical scope or compound statement.
  **L1522 CN**: 结束当前词法作用域或复合语句块。
- **L1523 EN**: Blank line separating nearby declarations or logic blocks.
  **L1523 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1524 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1524 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1525 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fmadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`.
  **L1525 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fmadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`。
- **L1526 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1526 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1527 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`.
  **L1528 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`。
- **L1529 EN**: Executes a call or declaration centered on `statement`.
  **L1529 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1530 EN**: Closes the current lexical scope or compound statement.
  **L1530 CN**: 结束当前词法作用域或复合语句块。
- **L1531 EN**: Blank line separating nearby declarations or logic blocks.
  **L1531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1532 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1532 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1533 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask3_fmadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`.
  **L1533 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask3_fmadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`。
- **L1534 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1534 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1535 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`.
  **L1536 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`。

### Lines 1537-1560

````c
      (__v16hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fmadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),
      (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmsub_ph(__m256h __A,
                                                                __m256h __B,
                                                                __m256h __C) {
  return (__m256h)__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B,
                                            -(__v16hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fmsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),
      (__v16hf)__A);
````
- **L1537 EN**: Executes a call or declaration centered on `statement`.
  **L1537 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1538 EN**: Closes the current lexical scope or compound statement.
  **L1538 CN**: 结束当前词法作用域或复合语句块。
- **L1539 EN**: Blank line separating nearby declarations or logic blocks.
  **L1539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1540 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1540 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1541 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fmadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`.
  **L1541 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fmadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`。
- **L1542 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1542 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1543 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`.
  **L1544 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`。
- **L1545 EN**: Executes a call or declaration centered on `statement`.
  **L1545 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1546 EN**: Closes the current lexical scope or compound statement.
  **L1546 CN**: 结束当前词法作用域或复合语句块。
- **L1547 EN**: Blank line separating nearby declarations or logic blocks.
  **L1547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmsub_ph(__m256h __A,`.
  **L1548 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmsub_ph(__m256h __A,`。
- **L1549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m256h __B,`.
  **L1549 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m256h __B,`。
- **L1550 EN**: Continues the surrounding expression or declaration: `__m256h __C) {`.
  **L1550 CN**: 继续构造周围的表达式或声明：`__m256h __C) {`。
- **L1551 EN**: Returns from the current function with `(__m256h)__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B,`.
  **L1551 CN**: 以 `(__m256h)__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B,` 从当前函数返回。
- **L1552 EN**: Executes a call or declaration centered on `-`.
  **L1552 CN**: 执行以 `-` 为核心的调用或声明。
- **L1553 EN**: Closes the current lexical scope or compound statement.
  **L1553 CN**: 结束当前词法作用域或复合语句块。
- **L1554 EN**: Blank line separating nearby declarations or logic blocks.
  **L1554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1555 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1555 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1556 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fmsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`.
  **L1556 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fmsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`。
- **L1557 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1557 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1558 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`.
  **L1559 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`。
- **L1560 EN**: Executes a call or declaration centered on `statement`.
  **L1560 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1561-1584

````c
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fmsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),
      (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask3_fnmadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, (__v16hf)__C),
      (__v16hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fnmadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, (__v16hf)__C),
      (__v16hf)_mm256_setzero_ph());
````
- **L1561 EN**: Closes the current lexical scope or compound statement.
  **L1561 CN**: 结束当前词法作用域或复合语句块。
- **L1562 EN**: Blank line separating nearby declarations or logic blocks.
  **L1562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1563 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1563 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1564 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fmsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`.
  **L1564 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fmsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`。
- **L1565 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1565 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1566 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`.
  **L1567 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`。
- **L1568 EN**: Executes a call or declaration centered on `statement`.
  **L1568 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1569 EN**: Closes the current lexical scope or compound statement.
  **L1569 CN**: 结束当前词法作用域或复合语句块。
- **L1570 EN**: Blank line separating nearby declarations or logic blocks.
  **L1570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1571 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1571 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1572 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask3_fnmadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`.
  **L1572 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask3_fnmadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`。
- **L1573 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1573 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1574 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`.
  **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`。
- **L1576 EN**: Executes a call or declaration centered on `statement`.
  **L1576 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1577 EN**: Closes the current lexical scope or compound statement.
  **L1577 CN**: 结束当前词法作用域或复合语句块。
- **L1578 EN**: Blank line separating nearby declarations or logic blocks.
  **L1578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1579 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1579 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1580 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fnmadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`.
  **L1580 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fnmadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`。
- **L1581 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1581 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1582 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`.
  **L1583 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`。
- **L1584 EN**: Executes a call or declaration centered on `statement`.
  **L1584 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1585-1608

````c
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fnmsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),
      (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmaddsub_ph(__m128h __A,
                                                                __m128h __B,
                                                                __m128h __C) {
  return (__m128h)__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B,
                                             (__v8hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fmaddsub_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)__A);
}
````
- **L1585 EN**: Closes the current lexical scope or compound statement.
  **L1585 CN**: 结束当前词法作用域或复合语句块。
- **L1586 EN**: Blank line separating nearby declarations or logic blocks.
  **L1586 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1587 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1587 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1588 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fnmsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`.
  **L1588 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fnmsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`。
- **L1589 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1589 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1590 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`.
  **L1591 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma(-(__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`。
- **L1592 EN**: Executes a call or declaration centered on `statement`.
  **L1592 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1593 EN**: Closes the current lexical scope or compound statement.
  **L1593 CN**: 结束当前词法作用域或复合语句块。
- **L1594 EN**: Blank line separating nearby declarations or logic blocks.
  **L1594 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmaddsub_ph(__m128h __A,`.
  **L1595 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmaddsub_ph(__m128h __A,`。
- **L1596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1596 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1597 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1597 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1598 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B,`.
  **L1598 CN**: 以 `(__m128h)__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L1599 EN**: Executes a call or declaration centered on `statement`.
  **L1599 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1600 EN**: Closes the current lexical scope or compound statement.
  **L1600 CN**: 结束当前词法作用域或复合语句块。
- **L1601 EN**: Blank line separating nearby declarations or logic blocks.
  **L1601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1602 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1602 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1603 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fmaddsub_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`.
  **L1603 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fmaddsub_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`。
- **L1604 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1604 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1605 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1606 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1607 EN**: Executes a call or declaration centered on `statement`.
  **L1607 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1608 EN**: Closes the current lexical scope or compound statement.
  **L1608 CN**: 结束当前词法作用域或复合语句块。

### Lines 1609-1632

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fmaddsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmaddsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),
      (__v8hf)_mm_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsubadd_ph(__m128h __A,
                                                                __m128h __B,
                                                                __m128h __C) {
  return (__m128h)__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B,
                                             -(__v8hf)__C);
}

````
- **L1609 EN**: Blank line separating nearby declarations or logic blocks.
  **L1609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1610 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1610 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1611 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmaddsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L1611 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmaddsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L1612 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1612 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1613 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1614 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1615 EN**: Executes a call or declaration centered on `statement`.
  **L1615 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1616 EN**: Closes the current lexical scope or compound statement.
  **L1616 CN**: 结束当前词法作用域或复合语句块。
- **L1617 EN**: Blank line separating nearby declarations or logic blocks.
  **L1617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1618 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1618 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1619 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmaddsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L1619 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmaddsub_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L1620 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1620 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1621 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`.
  **L1622 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, (__v8hf)__C),`。
- **L1623 EN**: Executes a call or declaration centered on `statement`.
  **L1623 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1624 EN**: Closes the current lexical scope or compound statement.
  **L1624 CN**: 结束当前词法作用域或复合语句块。
- **L1625 EN**: Blank line separating nearby declarations or logic blocks.
  **L1625 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsubadd_ph(__m128h __A,`.
  **L1626 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsubadd_ph(__m128h __A,`。
- **L1627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1627 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1628 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1628 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1629 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B,`.
  **L1629 CN**: 以 `(__m128h)__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L1630 EN**: Executes a call or declaration centered on `-`.
  **L1630 CN**: 执行以 `-` 为核心的调用或声明。
- **L1631 EN**: Closes the current lexical scope or compound statement.
  **L1631 CN**: 结束当前词法作用域或复合语句块。
- **L1632 EN**: Blank line separating nearby declarations or logic blocks.
  **L1632 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1633-1656

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fmsubadd_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),
      (__v8hf)__A);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmsubadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),
      (__v8hf)_mm_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_fmaddsub_ph(__m256h __A, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B,
                                                (__v16hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fmaddsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {
````
- **L1633 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1633 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1634 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fmsubadd_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`.
  **L1634 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fmsubadd_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`。
- **L1635 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1635 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1636 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`.
  **L1637 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`。
- **L1638 EN**: Executes a call or declaration centered on `statement`.
  **L1638 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1639 EN**: Closes the current lexical scope or compound statement.
  **L1639 CN**: 结束当前词法作用域或复合语句块。
- **L1640 EN**: Blank line separating nearby declarations or logic blocks.
  **L1640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1641 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1641 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1642 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmsubadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L1642 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmsubadd_ph(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L1643 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1643 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1644 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`.
  **L1645 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`。
- **L1646 EN**: Executes a call or declaration centered on `statement`.
  **L1646 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1647 EN**: Closes the current lexical scope or compound statement.
  **L1647 CN**: 结束当前词法作用域或复合语句块。
- **L1648 EN**: Blank line separating nearby declarations or logic blocks.
  **L1648 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1649 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1649 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1650 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_fmaddsub_ph(__m256h __A, __m256h __B, __m256h __C) {`.
  **L1650 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_fmaddsub_ph(__m256h __A, __m256h __B, __m256h __C) {`。
- **L1651 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B,`.
  **L1651 CN**: 以 `(__m256h)__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B,` 从当前函数返回。
- **L1652 EN**: Executes a call or declaration centered on `statement`.
  **L1652 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1653 EN**: Closes the current lexical scope or compound statement.
  **L1653 CN**: 结束当前词法作用域或复合语句块。
- **L1654 EN**: Blank line separating nearby declarations or logic blocks.
  **L1654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1655 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1655 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1656 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fmaddsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`.
  **L1656 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fmaddsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`。

### Lines 1657-1680

````c
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),
      (__v16hf)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask3_fmaddsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),
      (__v16hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fmaddsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),
      (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_fmsubadd_ph(__m256h __A, __m256h __B, __m256h __C) {
````
- **L1657 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1657 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1658 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`.
  **L1659 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`。
- **L1660 EN**: Executes a call or declaration centered on `statement`.
  **L1660 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1661 EN**: Closes the current lexical scope or compound statement.
  **L1661 CN**: 结束当前词法作用域或复合语句块。
- **L1662 EN**: Blank line separating nearby declarations or logic blocks.
  **L1662 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1663 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1663 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1664 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask3_fmaddsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`.
  **L1664 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask3_fmaddsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`。
- **L1665 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1665 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1666 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`.
  **L1667 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`。
- **L1668 EN**: Executes a call or declaration centered on `statement`.
  **L1668 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1669 EN**: Closes the current lexical scope or compound statement.
  **L1669 CN**: 结束当前词法作用域或复合语句块。
- **L1670 EN**: Blank line separating nearby declarations or logic blocks.
  **L1670 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1671 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1671 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1672 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fmaddsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`.
  **L1672 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fmaddsub_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`。
- **L1673 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1673 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1674 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`.
  **L1675 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, (__v16hf)__C),`。
- **L1676 EN**: Executes a call or declaration centered on `statement`.
  **L1676 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1677 EN**: Closes the current lexical scope or compound statement.
  **L1677 CN**: 结束当前词法作用域或复合语句块。
- **L1678 EN**: Blank line separating nearby declarations or logic blocks.
  **L1678 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1679 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1679 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1680 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_fmsubadd_ph(__m256h __A, __m256h __B, __m256h __C) {`.
  **L1680 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_fmsubadd_ph(__m256h __A, __m256h __B, __m256h __C) {`。

### Lines 1681-1704

````c
  return (__m256h)__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B,
                                                -(__v16hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fmsubadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),
      (__v16hf)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fmsubadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),
      (__v16hf)_mm256_setzero_ph());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fmsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
````
- **L1681 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B,`.
  **L1681 CN**: 以 `(__m256h)__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B,` 从当前函数返回。
- **L1682 EN**: Executes a call or declaration centered on `-`.
  **L1682 CN**: 执行以 `-` 为核心的调用或声明。
- **L1683 EN**: Closes the current lexical scope or compound statement.
  **L1683 CN**: 结束当前词法作用域或复合语句块。
- **L1684 EN**: Blank line separating nearby declarations or logic blocks.
  **L1684 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1685 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1685 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1686 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fmsubadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`.
  **L1686 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fmsubadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`。
- **L1687 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1687 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1688 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`.
  **L1689 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`。
- **L1690 EN**: Executes a call or declaration centered on `statement`.
  **L1690 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1691 EN**: Closes the current lexical scope or compound statement.
  **L1691 CN**: 结束当前词法作用域或复合语句块。
- **L1692 EN**: Blank line separating nearby declarations or logic blocks.
  **L1692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1693 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1693 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1694 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fmsubadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`.
  **L1694 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fmsubadd_ph(__mmask16 __U, __m256h __A, __m256h __B, __m256h __C) {`。
- **L1695 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1695 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1696 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`.
  **L1697 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`。
- **L1698 EN**: Executes a call or declaration centered on `statement`.
  **L1698 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1699 EN**: Closes the current lexical scope or compound statement.
  **L1699 CN**: 结束当前词法作用域或复合语句块。
- **L1700 EN**: Blank line separating nearby declarations or logic blocks.
  **L1700 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1701 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1701 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1702 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L1702 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L1703 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1703 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1704 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。

### Lines 1705-1728

````c
      __builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),
      (__v8hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask3_fmsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),
      (__v16hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fmsubadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),
      (__v8hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask3_fmsubadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
````
- **L1705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`.
  **L1705 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`。
- **L1706 EN**: Executes a call or declaration centered on `statement`.
  **L1706 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1707 EN**: Closes the current lexical scope or compound statement.
  **L1707 CN**: 结束当前词法作用域或复合语句块。
- **L1708 EN**: Blank line separating nearby declarations or logic blocks.
  **L1708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1709 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1709 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1710 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask3_fmsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`.
  **L1710 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask3_fmsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`。
- **L1711 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1711 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1712 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`.
  **L1713 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`。
- **L1714 EN**: Executes a call or declaration centered on `statement`.
  **L1714 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1715 EN**: Closes the current lexical scope or compound statement.
  **L1715 CN**: 结束当前词法作用域或复合语句块。
- **L1716 EN**: Blank line separating nearby declarations or logic blocks.
  **L1716 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1717 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1717 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1718 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmsubadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L1718 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmsubadd_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L1719 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1719 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1720 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`.
  **L1721 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph((__v8hf)__A, (__v8hf)__B, -(__v8hf)__C),`。
- **L1722 EN**: Executes a call or declaration centered on `statement`.
  **L1722 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1723 EN**: Closes the current lexical scope or compound statement.
  **L1723 CN**: 结束当前词法作用域或复合语句块。
- **L1724 EN**: Blank line separating nearby declarations or logic blocks.
  **L1724 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1725 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1725 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1726 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask3_fmsubadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`.
  **L1726 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask3_fmsubadd_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`。
- **L1727 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1727 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1728 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。

### Lines 1729-1752

````c
      __builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),
      (__v16hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmadd_ph(__m128h __A,
                                                              __m128h __B,
                                                              __m128h __C) {
  return (__m128h)__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B,
                                            (__v8hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fnmadd_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, (__v8hf)__C),
      (__v8hf)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fnmadd_ph(__m256h __A,
                                                                 __m256h __B,
                                                                 __m256h __C) {
  return (__m256h)__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B,
                                            (__v16hf)__C);
````
- **L1729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`.
  **L1729 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddsubph256((__v16hf)__A, (__v16hf)__B, -(__v16hf)__C),`。
- **L1730 EN**: Executes a call or declaration centered on `statement`.
  **L1730 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1731 EN**: Closes the current lexical scope or compound statement.
  **L1731 CN**: 结束当前词法作用域或复合语句块。
- **L1732 EN**: Blank line separating nearby declarations or logic blocks.
  **L1732 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmadd_ph(__m128h __A,`.
  **L1733 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmadd_ph(__m128h __A,`。
- **L1734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1734 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1735 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1735 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1736 EN**: Returns from the current function with `(__m128h)__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B,`.
  **L1736 CN**: 以 `(__m128h)__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B,` 从当前函数返回。
- **L1737 EN**: Executes a call or declaration centered on `statement`.
  **L1737 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1738 EN**: Closes the current lexical scope or compound statement.
  **L1738 CN**: 结束当前词法作用域或复合语句块。
- **L1739 EN**: Blank line separating nearby declarations or logic blocks.
  **L1739 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1740 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1740 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1741 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fnmadd_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`.
  **L1741 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fnmadd_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`。
- **L1742 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1742 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1743 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, (__v8hf)__C),`.
  **L1744 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, (__v8hf)__C),`。
- **L1745 EN**: Executes a call or declaration centered on `statement`.
  **L1745 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1746 EN**: Closes the current lexical scope or compound statement.
  **L1746 CN**: 结束当前词法作用域或复合语句块。
- **L1747 EN**: Blank line separating nearby declarations or logic blocks.
  **L1747 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fnmadd_ph(__m256h __A,`.
  **L1748 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fnmadd_ph(__m256h __A,`。
- **L1749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m256h __B,`.
  **L1749 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m256h __B,`。
- **L1750 EN**: Continues the surrounding expression or declaration: `__m256h __C) {`.
  **L1750 CN**: 继续构造周围的表达式或声明：`__m256h __C) {`。
- **L1751 EN**: Returns from the current function with `(__m256h)__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B,`.
  **L1751 CN**: 以 `(__m256h)__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B,` 从当前函数返回。
- **L1752 EN**: Executes a call or declaration centered on `statement`.
  **L1752 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1753-1776

````c
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fnmadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, (__v16hf)__C),
      (__v16hf)__A);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmsub_ph(__m128h __A,
                                                              __m128h __B,
                                                              __m128h __C) {
  return (__m128h)__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B,
                                            -(__v8hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fnmsub_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C),
      (__v8hf)__A);
}
````
- **L1753 EN**: Closes the current lexical scope or compound statement.
  **L1753 CN**: 结束当前词法作用域或复合语句块。
- **L1754 EN**: Blank line separating nearby declarations or logic blocks.
  **L1754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1755 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1755 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1756 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fnmadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`.
  **L1756 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fnmadd_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`。
- **L1757 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1757 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1758 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, (__v16hf)__C),`.
  **L1759 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, (__v16hf)__C),`。
- **L1760 EN**: Executes a call or declaration centered on `statement`.
  **L1760 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1761 EN**: Closes the current lexical scope or compound statement.
  **L1761 CN**: 结束当前词法作用域或复合语句块。
- **L1762 EN**: Blank line separating nearby declarations or logic blocks.
  **L1762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmsub_ph(__m128h __A,`.
  **L1763 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmsub_ph(__m128h __A,`。
- **L1764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1764 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1765 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1765 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1766 EN**: Returns from the current function with `(__m128h)__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B,`.
  **L1766 CN**: 以 `(__m128h)__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B,` 从当前函数返回。
- **L1767 EN**: Executes a call or declaration centered on `-`.
  **L1767 CN**: 执行以 `-` 为核心的调用或声明。
- **L1768 EN**: Closes the current lexical scope or compound statement.
  **L1768 CN**: 结束当前词法作用域或复合语句块。
- **L1769 EN**: Blank line separating nearby declarations or logic blocks.
  **L1769 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1770 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1770 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1771 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fnmsub_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`.
  **L1771 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fnmsub_ph(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`。
- **L1772 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1772 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1773 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C),`.
  **L1774 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C),`。
- **L1775 EN**: Executes a call or declaration centered on `statement`.
  **L1775 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1776 EN**: Closes the current lexical scope or compound statement.
  **L1776 CN**: 结束当前词法作用域或复合语句块。

### Lines 1777-1800

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fnmsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_selectph_128(
      (__mmask8)__U,
      __builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C),
      (__v8hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fnmsub_ph(__m256h __A,
                                                                 __m256h __B,
                                                                 __m256h __C) {
  return (__m256h)__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B,
                                            -(__v16hf)__C);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fnmsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, -(__v16hf)__C),
      (__v16hf)__A);
}

````
- **L1777 EN**: Blank line separating nearby declarations or logic blocks.
  **L1777 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1778 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1778 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1779 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fnmsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L1779 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fnmsub_ph(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L1780 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128(`.
  **L1780 CN**: 以 `(__m128h)__builtin_ia32_selectph_128(` 从当前函数返回。
- **L1781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L1781 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L1782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C),`.
  **L1782 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C),`。
- **L1783 EN**: Executes a call or declaration centered on `statement`.
  **L1783 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1784 EN**: Closes the current lexical scope or compound statement.
  **L1784 CN**: 结束当前词法作用域或复合语句块。
- **L1785 EN**: Blank line separating nearby declarations or logic blocks.
  **L1785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fnmsub_ph(__m256h __A,`.
  **L1786 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fnmsub_ph(__m256h __A,`。
- **L1787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m256h __B,`.
  **L1787 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m256h __B,`。
- **L1788 EN**: Continues the surrounding expression or declaration: `__m256h __C) {`.
  **L1788 CN**: 继续构造周围的表达式或声明：`__m256h __C) {`。
- **L1789 EN**: Returns from the current function with `(__m256h)__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B,`.
  **L1789 CN**: 以 `(__m256h)__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B,` 从当前函数返回。
- **L1790 EN**: Executes a call or declaration centered on `-`.
  **L1790 CN**: 执行以 `-` 为核心的调用或声明。
- **L1791 EN**: Closes the current lexical scope or compound statement.
  **L1791 CN**: 结束当前词法作用域或复合语句块。
- **L1792 EN**: Blank line separating nearby declarations or logic blocks.
  **L1792 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1793 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1793 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1794 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fnmsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`.
  **L1794 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fnmsub_ph(__m256h __A, __mmask16 __U, __m256h __B, __m256h __C) {`。
- **L1795 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1795 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1796 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, -(__v16hf)__C),`.
  **L1797 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, -(__v16hf)__C),`。
- **L1798 EN**: Executes a call or declaration centered on `statement`.
  **L1798 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1799 EN**: Closes the current lexical scope or compound statement.
  **L1799 CN**: 结束当前词法作用域或复合语句块。
- **L1800 EN**: Blank line separating nearby declarations or logic blocks.
  **L1800 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1801-1824

````c
static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask3_fnmsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {
  return (__m256h)__builtin_ia32_selectph_256(
      (__mmask16)__U,
      __builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, -(__v16hf)__C),
      (__v16hf)__C);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmul_pch(__m128h __A,
                                                              __m128h __B) {
  return (__m128h)__builtin_ia32_vfcmulcph128_mask(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fcmul_pch(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_vfcmulcph128_mask((__v4sf)__A, (__v4sf)__B,
                                                   (__v4sf)__W, (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fcmul_pch(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_vfcmulcph128_mask(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_setzero_ph(), (__mmask8)__U);
````
- **L1801 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1801 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1802 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask3_fnmsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`.
  **L1802 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask3_fnmsub_ph(__m256h __A, __m256h __B, __m256h __C, __mmask16 __U) {`。
- **L1803 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256(`.
  **L1803 CN**: 以 `(__m256h)__builtin_ia32_selectph_256(` 从当前函数返回。
- **L1804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L1804 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L1805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, -(__v16hf)__C),`.
  **L1805 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_elementwise_fma((__v16hf)__A, -(__v16hf)__B, -(__v16hf)__C),`。
- **L1806 EN**: Executes a call or declaration centered on `statement`.
  **L1806 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1807 EN**: Closes the current lexical scope or compound statement.
  **L1807 CN**: 结束当前词法作用域或复合语句块。
- **L1808 EN**: Blank line separating nearby declarations or logic blocks.
  **L1808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmul_pch(__m128h __A,`.
  **L1809 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmul_pch(__m128h __A,`。
- **L1810 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1810 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1811 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmulcph128_mask(`.
  **L1811 CN**: 以 `(__m128h)__builtin_ia32_vfcmulcph128_mask(` 从当前函数返回。
- **L1812 EN**: Executes a call or declaration centered on `statement`.
  **L1812 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1813 EN**: Closes the current lexical scope or compound statement.
  **L1813 CN**: 结束当前词法作用域或复合语句块。
- **L1814 EN**: Blank line separating nearby declarations or logic blocks.
  **L1814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1815 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1815 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1816 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fcmul_pch(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L1816 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fcmul_pch(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L1817 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmulcph128_mask((__v4sf)__A, (__v4sf)__B,`.
  **L1817 CN**: 以 `(__m128h)__builtin_ia32_vfcmulcph128_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L1818 EN**: Executes a call or declaration centered on `statement`.
  **L1818 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1819 EN**: Closes the current lexical scope or compound statement.
  **L1819 CN**: 结束当前词法作用域或复合语句块。
- **L1820 EN**: Blank line separating nearby declarations or logic blocks.
  **L1820 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1821 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1821 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1822 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fcmul_pch(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L1822 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fcmul_pch(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L1823 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmulcph128_mask(`.
  **L1823 CN**: 以 `(__m128h)__builtin_ia32_vfcmulcph128_mask(` 从当前函数返回。
- **L1824 EN**: Executes a call or declaration centered on `statement`.
  **L1824 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1825-1848

````c
}

static __inline__ __m256h __DEFAULT_FN_ATTRS128 _mm256_fcmul_pch(__m256h __A,
                                                                 __m256h __B) {
  return (__m256h)__builtin_ia32_vfcmulcph256_mask(
      (__v8sf)__A, (__v8sf)__B, (__v8sf)_mm256_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fcmul_pch(__m256h __W, __mmask8 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_vfcmulcph256_mask((__v8sf)__A, (__v8sf)__B,
                                                   (__v8sf)__W, (__mmask8)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fcmul_pch(__mmask8 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_vfcmulcph256_mask(
      (__v8sf)__A, (__v8sf)__B, (__v8sf)_mm256_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmadd_pch(__m128h __A,
                                                               __m128h __B,
                                                               __m128h __C) {
  return (__m128h)__builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)__B,
````
- **L1825 EN**: Closes the current lexical scope or compound statement.
  **L1825 CN**: 结束当前词法作用域或复合语句块。
- **L1826 EN**: Blank line separating nearby declarations or logic blocks.
  **L1826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS128 _mm256_fcmul_pch(__m256h __A,`.
  **L1827 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS128 _mm256_fcmul_pch(__m256h __A,`。
- **L1828 EN**: Continues the surrounding expression or declaration: `__m256h __B) {`.
  **L1828 CN**: 继续构造周围的表达式或声明：`__m256h __B) {`。
- **L1829 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfcmulcph256_mask(`.
  **L1829 CN**: 以 `(__m256h)__builtin_ia32_vfcmulcph256_mask(` 从当前函数返回。
- **L1830 EN**: Executes a call or declaration centered on `statement`.
  **L1830 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1831 EN**: Closes the current lexical scope or compound statement.
  **L1831 CN**: 结束当前词法作用域或复合语句块。
- **L1832 EN**: Blank line separating nearby declarations or logic blocks.
  **L1832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1833 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1833 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1834 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fcmul_pch(__m256h __W, __mmask8 __U, __m256h __A, __m256h __B) {`.
  **L1834 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fcmul_pch(__m256h __W, __mmask8 __U, __m256h __A, __m256h __B) {`。
- **L1835 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfcmulcph256_mask((__v8sf)__A, (__v8sf)__B,`.
  **L1835 CN**: 以 `(__m256h)__builtin_ia32_vfcmulcph256_mask((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L1836 EN**: Executes a call or declaration centered on `statement`.
  **L1836 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1837 EN**: Closes the current lexical scope or compound statement.
  **L1837 CN**: 结束当前词法作用域或复合语句块。
- **L1838 EN**: Blank line separating nearby declarations or logic blocks.
  **L1838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1839 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1839 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1840 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fcmul_pch(__mmask8 __U, __m256h __A, __m256h __B) {`.
  **L1840 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fcmul_pch(__mmask8 __U, __m256h __A, __m256h __B) {`。
- **L1841 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfcmulcph256_mask(`.
  **L1841 CN**: 以 `(__m256h)__builtin_ia32_vfcmulcph256_mask(` 从当前函数返回。
- **L1842 EN**: Executes a call or declaration centered on `statement`.
  **L1842 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1843 EN**: Closes the current lexical scope or compound statement.
  **L1843 CN**: 结束当前词法作用域或复合语句块。
- **L1844 EN**: Blank line separating nearby declarations or logic blocks.
  **L1844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmadd_pch(__m128h __A,`.
  **L1845 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmadd_pch(__m128h __A,`。
- **L1846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1846 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1847 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1847 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1848 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)__B,`.
  **L1848 CN**: 以 `(__m128h)__builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。

### Lines 1849-1872

````c
                                                    (__v4sf)__C, (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fcmadd_pch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectps_128(
      __U,
      __builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)(__m128h)__B,
                                        (__v4sf)__C, (__mmask8)__U),
      (__v4sf)__A);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fcmadd_pch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)__B,
                                                    (__v4sf)__C, (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fcmadd_pch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_vfcmaddcph128_maskz(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)__C, (__mmask8)__U);
}

````
- **L1849 EN**: Executes a call or declaration centered on `statement`.
  **L1849 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1850 EN**: Closes the current lexical scope or compound statement.
  **L1850 CN**: 结束当前词法作用域或复合语句块。
- **L1851 EN**: Blank line separating nearby declarations or logic blocks.
  **L1851 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1852 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1852 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1853 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fcmadd_pch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`.
  **L1853 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fcmadd_pch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`。
- **L1854 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectps_128(`.
  **L1854 CN**: 以 `(__m128h)__builtin_ia32_selectps_128(` 从当前函数返回。
- **L1855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__U,`.
  **L1855 CN**: 继续一个多行参数列表、初始化器或聚合项：`__U,`。
- **L1856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)(__m128h)__B,`.
  **L1856 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)(__m128h)__B,`。
- **L1857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__C, (__mmask8)__U),`.
  **L1857 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__C, (__mmask8)__U),`。
- **L1858 EN**: Executes a call or declaration centered on `statement`.
  **L1858 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1859 EN**: Closes the current lexical scope or compound statement.
  **L1859 CN**: 结束当前词法作用域或复合语句块。
- **L1860 EN**: Blank line separating nearby declarations or logic blocks.
  **L1860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1861 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1861 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1862 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fcmadd_pch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L1862 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fcmadd_pch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L1863 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)__B,`.
  **L1863 CN**: 以 `(__m128h)__builtin_ia32_vfcmaddcph128_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L1864 EN**: Executes a call or declaration centered on `statement`.
  **L1864 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1865 EN**: Closes the current lexical scope or compound statement.
  **L1865 CN**: 结束当前词法作用域或复合语句块。
- **L1866 EN**: Blank line separating nearby declarations or logic blocks.
  **L1866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1867 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1867 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1868 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fcmadd_pch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L1868 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fcmadd_pch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L1869 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmaddcph128_maskz(`.
  **L1869 CN**: 以 `(__m128h)__builtin_ia32_vfcmaddcph128_maskz(` 从当前函数返回。
- **L1870 EN**: Executes a call or declaration centered on `statement`.
  **L1870 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1871 EN**: Closes the current lexical scope or compound statement.
  **L1871 CN**: 结束当前词法作用域或复合语句块。
- **L1872 EN**: Blank line separating nearby declarations or logic blocks.
  **L1872 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1873-1896

````c
static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fcmadd_pch(__m256h __A,
                                                                  __m256h __B,
                                                                  __m256h __C) {
  return (__m256h)__builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B,
                                                    (__v8sf)__C, (__mmask8)-1);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fcmadd_pch(__m256h __A, __mmask8 __U, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectps_256(
      __U,
      __builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B, (__v8sf)__C,
                                        (__mmask8)__U),
      (__v8sf)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask3_fcmadd_pch(__m256h __A, __m256h __B, __m256h __C, __mmask8 __U) {
  return (__m256h)__builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B,
                                                    (__v8sf)__C, (__mmask8)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fcmadd_pch(__mmask8 __U, __m256h __A, __m256h __B, __m256h __C) {
````
- **L1873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fcmadd_pch(__m256h __A,`.
  **L1873 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fcmadd_pch(__m256h __A,`。
- **L1874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m256h __B,`.
  **L1874 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m256h __B,`。
- **L1875 EN**: Continues the surrounding expression or declaration: `__m256h __C) {`.
  **L1875 CN**: 继续构造周围的表达式或声明：`__m256h __C) {`。
- **L1876 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B,`.
  **L1876 CN**: 以 `(__m256h)__builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L1877 EN**: Executes a call or declaration centered on `statement`.
  **L1877 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1878 EN**: Closes the current lexical scope or compound statement.
  **L1878 CN**: 结束当前词法作用域或复合语句块。
- **L1879 EN**: Blank line separating nearby declarations or logic blocks.
  **L1879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1880 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1880 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1881 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fcmadd_pch(__m256h __A, __mmask8 __U, __m256h __B, __m256h __C) {`.
  **L1881 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fcmadd_pch(__m256h __A, __mmask8 __U, __m256h __B, __m256h __C) {`。
- **L1882 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectps_256(`.
  **L1882 CN**: 以 `(__m256h)__builtin_ia32_selectps_256(` 从当前函数返回。
- **L1883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__U,`.
  **L1883 CN**: 继续一个多行参数列表、初始化器或聚合项：`__U,`。
- **L1884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B, (__v8sf)__C,`.
  **L1884 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B, (__v8sf)__C,`。
- **L1885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U),`.
  **L1885 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U),`。
- **L1886 EN**: Executes a call or declaration centered on `statement`.
  **L1886 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1887 EN**: Closes the current lexical scope or compound statement.
  **L1887 CN**: 结束当前词法作用域或复合语句块。
- **L1888 EN**: Blank line separating nearby declarations or logic blocks.
  **L1888 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1889 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1889 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1890 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask3_fcmadd_pch(__m256h __A, __m256h __B, __m256h __C, __mmask8 __U) {`.
  **L1890 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask3_fcmadd_pch(__m256h __A, __m256h __B, __m256h __C, __mmask8 __U) {`。
- **L1891 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B,`.
  **L1891 CN**: 以 `(__m256h)__builtin_ia32_vfcmaddcph256_mask((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L1892 EN**: Executes a call or declaration centered on `statement`.
  **L1892 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1893 EN**: Closes the current lexical scope or compound statement.
  **L1893 CN**: 结束当前词法作用域或复合语句块。
- **L1894 EN**: Blank line separating nearby declarations or logic blocks.
  **L1894 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1895 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1895 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1896 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fcmadd_pch(__mmask8 __U, __m256h __A, __m256h __B, __m256h __C) {`.
  **L1896 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fcmadd_pch(__mmask8 __U, __m256h __A, __m256h __B, __m256h __C) {`。

### Lines 1897-1920

````c
  return (__m256h)__builtin_ia32_vfcmaddcph256_maskz(
      (__v8sf)__A, (__v8sf)__B, (__v8sf)__C, (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmul_pch(__m128h __A,
                                                             __m128h __B) {
  return (__m128h)__builtin_ia32_vfmulcph128_mask(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmul_pch(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A,
                                                                  __m128h __B) {
  return (__m128h)__builtin_ia32_vfmulcph128_mask((__v4sf)__A, (__v4sf)__B,
                                                  (__v4sf)__W, (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmul_pch(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_vfmulcph128_mask(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_setzero_ph(), (__mmask8)__U);
}

````
- **L1897 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfcmaddcph256_maskz(`.
  **L1897 CN**: 以 `(__m256h)__builtin_ia32_vfcmaddcph256_maskz(` 从当前函数返回。
- **L1898 EN**: Executes a call or declaration centered on `statement`.
  **L1898 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1899 EN**: Closes the current lexical scope or compound statement.
  **L1899 CN**: 结束当前词法作用域或复合语句块。
- **L1900 EN**: Blank line separating nearby declarations or logic blocks.
  **L1900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmul_pch(__m128h __A,`.
  **L1901 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmul_pch(__m128h __A,`。
- **L1902 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1902 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1903 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmulcph128_mask(`.
  **L1903 CN**: 以 `(__m128h)__builtin_ia32_vfmulcph128_mask(` 从当前函数返回。
- **L1904 EN**: Executes a call or declaration centered on `statement`.
  **L1904 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1905 EN**: Closes the current lexical scope or compound statement.
  **L1905 CN**: 结束当前词法作用域或复合语句块。
- **L1906 EN**: Blank line separating nearby declarations or logic blocks.
  **L1906 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmul_pch(__m128h __W,`.
  **L1907 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmul_pch(__m128h __W,`。
- **L1908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1908 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1909 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1910 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1910 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1911 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmulcph128_mask((__v4sf)__A, (__v4sf)__B,`.
  **L1911 CN**: 以 `(__m128h)__builtin_ia32_vfmulcph128_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L1912 EN**: Executes a call or declaration centered on `statement`.
  **L1912 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1913 EN**: Closes the current lexical scope or compound statement.
  **L1913 CN**: 结束当前词法作用域或复合语句块。
- **L1914 EN**: Blank line separating nearby declarations or logic blocks.
  **L1914 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1915 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1915 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1916 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmul_pch(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L1916 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmul_pch(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L1917 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmulcph128_mask(`.
  **L1917 CN**: 以 `(__m128h)__builtin_ia32_vfmulcph128_mask(` 从当前函数返回。
- **L1918 EN**: Executes a call or declaration centered on `statement`.
  **L1918 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1919 EN**: Closes the current lexical scope or compound statement.
  **L1919 CN**: 结束当前词法作用域或复合语句块。
- **L1920 EN**: Blank line separating nearby declarations or logic blocks.
  **L1920 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1921-1944

````c
static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmul_pch(__m256h __A,
                                                                __m256h __B) {
  return (__m256h)__builtin_ia32_vfmulcph256_mask(
      (__v8sf)__A, (__v8sf)__B, (__v8sf)_mm256_undefined_ph(), (__mmask8)-1);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fmul_pch(__m256h __W, __mmask8 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_vfmulcph256_mask((__v8sf)__A, (__v8sf)__B,
                                                  (__v8sf)__W, (__mmask8)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fmul_pch(__mmask8 __U, __m256h __A, __m256h __B) {
  return (__m256h)__builtin_ia32_vfmulcph256_mask(
      (__v8sf)__A, (__v8sf)__B, (__v8sf)_mm256_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_pch(__m128h __A,
                                                              __m128h __B,
                                                              __m128h __C) {
  return (__m128h)__builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B,
                                                   (__v4sf)__C, (__mmask8)-1);
}
````
- **L1921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmul_pch(__m256h __A,`.
  **L1921 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmul_pch(__m256h __A,`。
- **L1922 EN**: Continues the surrounding expression or declaration: `__m256h __B) {`.
  **L1922 CN**: 继续构造周围的表达式或声明：`__m256h __B) {`。
- **L1923 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfmulcph256_mask(`.
  **L1923 CN**: 以 `(__m256h)__builtin_ia32_vfmulcph256_mask(` 从当前函数返回。
- **L1924 EN**: Executes a call or declaration centered on `statement`.
  **L1924 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1925 EN**: Closes the current lexical scope or compound statement.
  **L1925 CN**: 结束当前词法作用域或复合语句块。
- **L1926 EN**: Blank line separating nearby declarations or logic blocks.
  **L1926 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1927 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1927 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1928 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fmul_pch(__m256h __W, __mmask8 __U, __m256h __A, __m256h __B) {`.
  **L1928 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fmul_pch(__m256h __W, __mmask8 __U, __m256h __A, __m256h __B) {`。
- **L1929 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfmulcph256_mask((__v8sf)__A, (__v8sf)__B,`.
  **L1929 CN**: 以 `(__m256h)__builtin_ia32_vfmulcph256_mask((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L1930 EN**: Executes a call or declaration centered on `statement`.
  **L1930 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1931 EN**: Closes the current lexical scope or compound statement.
  **L1931 CN**: 结束当前词法作用域或复合语句块。
- **L1932 EN**: Blank line separating nearby declarations or logic blocks.
  **L1932 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1933 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1933 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1934 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fmul_pch(__mmask8 __U, __m256h __A, __m256h __B) {`.
  **L1934 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fmul_pch(__mmask8 __U, __m256h __A, __m256h __B) {`。
- **L1935 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfmulcph256_mask(`.
  **L1935 CN**: 以 `(__m256h)__builtin_ia32_vfmulcph256_mask(` 从当前函数返回。
- **L1936 EN**: Executes a call or declaration centered on `statement`.
  **L1936 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1937 EN**: Closes the current lexical scope or compound statement.
  **L1937 CN**: 结束当前词法作用域或复合语句块。
- **L1938 EN**: Blank line separating nearby declarations or logic blocks.
  **L1938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1939 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_pch(__m128h __A,`.
  **L1939 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_pch(__m128h __A,`。
- **L1940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L1940 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L1941 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L1941 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L1942 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B,`.
  **L1942 CN**: 以 `(__m128h)__builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L1943 EN**: Executes a call or declaration centered on `statement`.
  **L1943 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1944 EN**: Closes the current lexical scope or compound statement.
  **L1944 CN**: 结束当前词法作用域或复合语句块。

### Lines 1945-1968

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fmadd_pch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_selectps_128(
      __U,
      __builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B, (__v4sf)__C,
                                       (__mmask8)__U),
      (__v4sf)__A);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fmadd_pch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B,
                                                   (__v4sf)__C, (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmadd_pch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_vfmaddcph128_maskz((__v4sf)__A, (__v4sf)__B,
                                                    (__v4sf)__C, (__mmask8)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmadd_pch(__m256h __A,
                                                                 __m256h __B,
````
- **L1945 EN**: Blank line separating nearby declarations or logic blocks.
  **L1945 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1946 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1946 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1947 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fmadd_pch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`.
  **L1947 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fmadd_pch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`。
- **L1948 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectps_128(`.
  **L1948 CN**: 以 `(__m128h)__builtin_ia32_selectps_128(` 从当前函数返回。
- **L1949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__U,`.
  **L1949 CN**: 继续一个多行参数列表、初始化器或聚合项：`__U,`。
- **L1950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B, (__v4sf)__C,`.
  **L1950 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B, (__v4sf)__C,`。
- **L1951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U),`.
  **L1951 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U),`。
- **L1952 EN**: Executes a call or declaration centered on `statement`.
  **L1952 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1953 EN**: Closes the current lexical scope or compound statement.
  **L1953 CN**: 结束当前词法作用域或复合语句块。
- **L1954 EN**: Blank line separating nearby declarations or logic blocks.
  **L1954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1955 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1955 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1956 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmadd_pch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L1956 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmadd_pch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L1957 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B,`.
  **L1957 CN**: 以 `(__m128h)__builtin_ia32_vfmaddcph128_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L1958 EN**: Executes a call or declaration centered on `statement`.
  **L1958 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1959 EN**: Closes the current lexical scope or compound statement.
  **L1959 CN**: 结束当前词法作用域或复合语句块。
- **L1960 EN**: Blank line separating nearby declarations or logic blocks.
  **L1960 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1961 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1961 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1962 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmadd_pch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L1962 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmadd_pch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L1963 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddcph128_maskz((__v4sf)__A, (__v4sf)__B,`.
  **L1963 CN**: 以 `(__m128h)__builtin_ia32_vfmaddcph128_maskz((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L1964 EN**: Executes a call or declaration centered on `statement`.
  **L1964 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1965 EN**: Closes the current lexical scope or compound statement.
  **L1965 CN**: 结束当前词法作用域或复合语句块。
- **L1966 EN**: Blank line separating nearby declarations or logic blocks.
  **L1966 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1967 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmadd_pch(__m256h __A,`.
  **L1967 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_fmadd_pch(__m256h __A,`。
- **L1968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m256h __B,`.
  **L1968 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m256h __B,`。

### Lines 1969-1992

````c
                                                                 __m256h __C) {
  return (__m256h)__builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B,
                                                   (__v8sf)__C, (__mmask8)-1);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_fmadd_pch(__m256h __A, __mmask8 __U, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_selectps_256(
      __U,
      __builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B, (__v8sf)__C,
                                       (__mmask8)__U),
      (__v8sf)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask3_fmadd_pch(__m256h __A, __m256h __B, __m256h __C, __mmask8 __U) {
  return (__m256h)__builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B,
                                                   (__v8sf)__C, (__mmask8)__U);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_fmadd_pch(__mmask8 __U, __m256h __A, __m256h __B, __m256h __C) {
  return (__m256h)__builtin_ia32_vfmaddcph256_maskz((__v8sf)__A, (__v8sf)__B,
                                                    (__v8sf)__C, (__mmask8)__U);
````
- **L1969 EN**: Continues the surrounding expression or declaration: `__m256h __C) {`.
  **L1969 CN**: 继续构造周围的表达式或声明：`__m256h __C) {`。
- **L1970 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B,`.
  **L1970 CN**: 以 `(__m256h)__builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L1971 EN**: Executes a call or declaration centered on `statement`.
  **L1971 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1972 EN**: Closes the current lexical scope or compound statement.
  **L1972 CN**: 结束当前词法作用域或复合语句块。
- **L1973 EN**: Blank line separating nearby declarations or logic blocks.
  **L1973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1974 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1974 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1975 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fmadd_pch(__m256h __A, __mmask8 __U, __m256h __B, __m256h __C) {`.
  **L1975 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fmadd_pch(__m256h __A, __mmask8 __U, __m256h __B, __m256h __C) {`。
- **L1976 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectps_256(`.
  **L1976 CN**: 以 `(__m256h)__builtin_ia32_selectps_256(` 从当前函数返回。
- **L1977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__U,`.
  **L1977 CN**: 继续一个多行参数列表、初始化器或聚合项：`__U,`。
- **L1978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B, (__v8sf)__C,`.
  **L1978 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B, (__v8sf)__C,`。
- **L1979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U),`.
  **L1979 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U),`。
- **L1980 EN**: Executes a call or declaration centered on `statement`.
  **L1980 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1981 EN**: Closes the current lexical scope or compound statement.
  **L1981 CN**: 结束当前词法作用域或复合语句块。
- **L1982 EN**: Blank line separating nearby declarations or logic blocks.
  **L1982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1983 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1983 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1984 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask3_fmadd_pch(__m256h __A, __m256h __B, __m256h __C, __mmask8 __U) {`.
  **L1984 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask3_fmadd_pch(__m256h __A, __m256h __B, __m256h __C, __mmask8 __U) {`。
- **L1985 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B,`.
  **L1985 CN**: 以 `(__m256h)__builtin_ia32_vfmaddcph256_mask((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L1986 EN**: Executes a call or declaration centered on `statement`.
  **L1986 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1987 EN**: Closes the current lexical scope or compound statement.
  **L1987 CN**: 结束当前词法作用域或复合语句块。
- **L1988 EN**: Blank line separating nearby declarations or logic blocks.
  **L1988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1989 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L1989 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L1990 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_fmadd_pch(__mmask8 __U, __m256h __A, __m256h __B, __m256h __C) {`.
  **L1990 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_fmadd_pch(__mmask8 __U, __m256h __A, __m256h __B, __m256h __C) {`。
- **L1991 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vfmaddcph256_maskz((__v8sf)__A, (__v8sf)__B,`.
  **L1991 CN**: 以 `(__m256h)__builtin_ia32_vfmaddcph256_maskz((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L1992 EN**: Executes a call or declaration centered on `statement`.
  **L1992 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1993-2016

````c
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_blend_ph(__mmask8 __U, __m128h __A, __m128h __W) {
  return (__m128h)__builtin_ia32_selectph_128((__mmask8)__U, (__v8hf)__W,
                                              (__v8hf)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_blend_ph(__mmask16 __U, __m256h __A, __m256h __W) {
  return (__m256h)__builtin_ia32_selectph_256((__mmask16)__U, (__v16hf)__W,
                                              (__v16hf)__A);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_permutex2var_ph(__m128h __A, __m128i __I, __m128h __B) {
  return (__m128h)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,
                                                 (__v8hi)__B);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_permutex2var_ph(__m256h __A, __m256i __I, __m256h __B) {
  return (__m256h)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,
                                                 (__v16hi)__B);
````
- **L1993 EN**: Closes the current lexical scope or compound statement.
  **L1993 CN**: 结束当前词法作用域或复合语句块。
- **L1994 EN**: Blank line separating nearby declarations or logic blocks.
  **L1994 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1995 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1995 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1996 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_blend_ph(__mmask8 __U, __m128h __A, __m128h __W) {`.
  **L1996 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_blend_ph(__mmask8 __U, __m128h __A, __m128h __W) {`。
- **L1997 EN**: Returns from the current function with `(__m128h)__builtin_ia32_selectph_128((__mmask8)__U, (__v8hf)__W,`.
  **L1997 CN**: 以 `(__m128h)__builtin_ia32_selectph_128((__mmask8)__U, (__v8hf)__W,` 从当前函数返回。
- **L1998 EN**: Executes a call or declaration centered on `statement`.
  **L1998 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1999 EN**: Closes the current lexical scope or compound statement.
  **L1999 CN**: 结束当前词法作用域或复合语句块。
- **L2000 EN**: Blank line separating nearby declarations or logic blocks.
  **L2000 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2001 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2001 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2002 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_blend_ph(__mmask16 __U, __m256h __A, __m256h __W) {`.
  **L2002 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_blend_ph(__mmask16 __U, __m256h __A, __m256h __W) {`。
- **L2003 EN**: Returns from the current function with `(__m256h)__builtin_ia32_selectph_256((__mmask16)__U, (__v16hf)__W,`.
  **L2003 CN**: 以 `(__m256h)__builtin_ia32_selectph_256((__mmask16)__U, (__v16hf)__W,` 从当前函数返回。
- **L2004 EN**: Executes a call or declaration centered on `statement`.
  **L2004 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2005 EN**: Closes the current lexical scope or compound statement.
  **L2005 CN**: 结束当前词法作用域或复合语句块。
- **L2006 EN**: Blank line separating nearby declarations or logic blocks.
  **L2006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2007 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2007 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2008 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_permutex2var_ph(__m128h __A, __m128i __I, __m128h __B) {`.
  **L2008 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_permutex2var_ph(__m128h __A, __m128i __I, __m128h __B) {`。
- **L2009 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,`.
  **L2009 CN**: 以 `(__m128h)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,` 从当前函数返回。
- **L2010 EN**: Executes a call or declaration centered on `statement`.
  **L2010 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2011 EN**: Closes the current lexical scope or compound statement.
  **L2011 CN**: 结束当前词法作用域或复合语句块。
- **L2012 EN**: Blank line separating nearby declarations or logic blocks.
  **L2012 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2013 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2013 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2014 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_permutex2var_ph(__m256h __A, __m256i __I, __m256h __B) {`.
  **L2014 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_permutex2var_ph(__m256h __A, __m256i __I, __m256h __B) {`。
- **L2015 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,`.
  **L2015 CN**: 以 `(__m256h)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,` 从当前函数返回。
- **L2016 EN**: Executes a call or declaration centered on `statement`.
  **L2016 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 2017-2040

````c
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_permutexvar_ph(__m128i __A, __m128h __B) {
  return (__m128h)__builtin_ia32_permvarhi128((__v8hi)__B, (__v8hi)__A);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_permutexvar_ph(__m256i __A, __m256h __B) {
  return (__m256h)__builtin_ia32_permvarhi256((__v16hi)__B, (__v16hi)__A);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS256
_mm256_reduce_add_ph(__m256h __W) {
  return __builtin_ia32_reduce_fadd_ph256(-0.0f16, __W);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS256
_mm256_reduce_mul_ph(__m256h __W) {
  return __builtin_ia32_reduce_fmul_ph256(1.0f16, __W);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS256
_mm256_reduce_max_ph(__m256h __V) {
````
- **L2017 EN**: Closes the current lexical scope or compound statement.
  **L2017 CN**: 结束当前词法作用域或复合语句块。
- **L2018 EN**: Blank line separating nearby declarations or logic blocks.
  **L2018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2019 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2019 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2020 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_permutexvar_ph(__m128i __A, __m128h __B) {`.
  **L2020 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_permutexvar_ph(__m128i __A, __m128h __B) {`。
- **L2021 EN**: Returns from the current function with `(__m128h)__builtin_ia32_permvarhi128((__v8hi)__B, (__v8hi)__A)`.
  **L2021 CN**: 以 `(__m128h)__builtin_ia32_permvarhi128((__v8hi)__B, (__v8hi)__A)` 从当前函数返回。
- **L2022 EN**: Closes the current lexical scope or compound statement.
  **L2022 CN**: 结束当前词法作用域或复合语句块。
- **L2023 EN**: Blank line separating nearby declarations or logic blocks.
  **L2023 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2024 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2024 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2025 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_permutexvar_ph(__m256i __A, __m256h __B) {`.
  **L2025 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_permutexvar_ph(__m256i __A, __m256h __B) {`。
- **L2026 EN**: Returns from the current function with `(__m256h)__builtin_ia32_permvarhi256((__v16hi)__B, (__v16hi)__A)`.
  **L2026 CN**: 以 `(__m256h)__builtin_ia32_permvarhi256((__v16hi)__B, (__v16hi)__A)` 从当前函数返回。
- **L2027 EN**: Closes the current lexical scope or compound statement.
  **L2027 CN**: 结束当前词法作用域或复合语句块。
- **L2028 EN**: Blank line separating nearby declarations or logic blocks.
  **L2028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2029 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS256`.
  **L2029 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS256`。
- **L2030 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_add_ph(__m256h __W) {`.
  **L2030 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_add_ph(__m256h __W) {`。
- **L2031 EN**: Returns from the current function with `__builtin_ia32_reduce_fadd_ph256(-0.0f16, __W)`.
  **L2031 CN**: 以 `__builtin_ia32_reduce_fadd_ph256(-0.0f16, __W)` 从当前函数返回。
- **L2032 EN**: Closes the current lexical scope or compound statement.
  **L2032 CN**: 结束当前词法作用域或复合语句块。
- **L2033 EN**: Blank line separating nearby declarations or logic blocks.
  **L2033 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2034 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS256`.
  **L2034 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS256`。
- **L2035 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_mul_ph(__m256h __W) {`.
  **L2035 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_mul_ph(__m256h __W) {`。
- **L2036 EN**: Returns from the current function with `__builtin_ia32_reduce_fmul_ph256(1.0f16, __W)`.
  **L2036 CN**: 以 `__builtin_ia32_reduce_fmul_ph256(1.0f16, __W)` 从当前函数返回。
- **L2037 EN**: Closes the current lexical scope or compound statement.
  **L2037 CN**: 结束当前词法作用域或复合语句块。
- **L2038 EN**: Blank line separating nearby declarations or logic blocks.
  **L2038 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2039 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS256`.
  **L2039 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS256`。
- **L2040 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_max_ph(__m256h __V) {`.
  **L2040 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_max_ph(__m256h __V) {`。

### Lines 2041-2064

````c
  return __builtin_ia32_reduce_fmax_ph256(__V);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS256
_mm256_reduce_min_ph(__m256h __V) {
  return __builtin_ia32_reduce_fmin_ph256(__V);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS128
_mm_reduce_add_ph(__m128h __W) {
  return __builtin_ia32_reduce_fadd_ph128(-0.0f16, __W);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS128
_mm_reduce_mul_ph(__m128h __W) {
  return __builtin_ia32_reduce_fmul_ph128(1.0f16, __W);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS128
_mm_reduce_max_ph(__m128h __V) {
  return __builtin_ia32_reduce_fmax_ph128(__V);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS128
````
- **L2041 EN**: Returns from the current function with `__builtin_ia32_reduce_fmax_ph256(__V)`.
  **L2041 CN**: 以 `__builtin_ia32_reduce_fmax_ph256(__V)` 从当前函数返回。
- **L2042 EN**: Closes the current lexical scope or compound statement.
  **L2042 CN**: 结束当前词法作用域或复合语句块。
- **L2043 EN**: Blank line separating nearby declarations or logic blocks.
  **L2043 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2044 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS256`.
  **L2044 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS256`。
- **L2045 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_min_ph(__m256h __V) {`.
  **L2045 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_min_ph(__m256h __V) {`。
- **L2046 EN**: Returns from the current function with `__builtin_ia32_reduce_fmin_ph256(__V)`.
  **L2046 CN**: 以 `__builtin_ia32_reduce_fmin_ph256(__V)` 从当前函数返回。
- **L2047 EN**: Closes the current lexical scope or compound statement.
  **L2047 CN**: 结束当前词法作用域或复合语句块。
- **L2048 EN**: Blank line separating nearby declarations or logic blocks.
  **L2048 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2049 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS128`.
  **L2049 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS128`。
- **L2050 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_add_ph(__m128h __W) {`.
  **L2050 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_add_ph(__m128h __W) {`。
- **L2051 EN**: Returns from the current function with `__builtin_ia32_reduce_fadd_ph128(-0.0f16, __W)`.
  **L2051 CN**: 以 `__builtin_ia32_reduce_fadd_ph128(-0.0f16, __W)` 从当前函数返回。
- **L2052 EN**: Closes the current lexical scope or compound statement.
  **L2052 CN**: 结束当前词法作用域或复合语句块。
- **L2053 EN**: Blank line separating nearby declarations or logic blocks.
  **L2053 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2054 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS128`.
  **L2054 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS128`。
- **L2055 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_mul_ph(__m128h __W) {`.
  **L2055 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_mul_ph(__m128h __W) {`。
- **L2056 EN**: Returns from the current function with `__builtin_ia32_reduce_fmul_ph128(1.0f16, __W)`.
  **L2056 CN**: 以 `__builtin_ia32_reduce_fmul_ph128(1.0f16, __W)` 从当前函数返回。
- **L2057 EN**: Closes the current lexical scope or compound statement.
  **L2057 CN**: 结束当前词法作用域或复合语句块。
- **L2058 EN**: Blank line separating nearby declarations or logic blocks.
  **L2058 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2059 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS128`.
  **L2059 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS128`。
- **L2060 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_max_ph(__m128h __V) {`.
  **L2060 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_max_ph(__m128h __V) {`。
- **L2061 EN**: Returns from the current function with `__builtin_ia32_reduce_fmax_ph128(__V)`.
  **L2061 CN**: 以 `__builtin_ia32_reduce_fmax_ph128(__V)` 从当前函数返回。
- **L2062 EN**: Closes the current lexical scope or compound statement.
  **L2062 CN**: 结束当前词法作用域或复合语句块。
- **L2063 EN**: Blank line separating nearby declarations or logic blocks.
  **L2063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2064 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS128`.
  **L2064 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS128`。

### Lines 2065-2088

````c
_mm_reduce_min_ph(__m128h __V) {
  return __builtin_ia32_reduce_fmin_ph128(__V);
}

// intrinsics below are alias for f*mul_*ch
#define _mm_mul_pch(A, B) _mm_fmul_pch(A, B)
#define _mm_mask_mul_pch(W, U, A, B) _mm_mask_fmul_pch(W, U, A, B)
#define _mm_maskz_mul_pch(U, A, B) _mm_maskz_fmul_pch(U, A, B)
#define _mm256_mul_pch(A, B) _mm256_fmul_pch(A, B)
#define _mm256_mask_mul_pch(W, U, A, B) _mm256_mask_fmul_pch(W, U, A, B)
#define _mm256_maskz_mul_pch(U, A, B) _mm256_maskz_fmul_pch(U, A, B)

#define _mm_cmul_pch(A, B) _mm_fcmul_pch(A, B)
#define _mm_mask_cmul_pch(W, U, A, B) _mm_mask_fcmul_pch(W, U, A, B)
#define _mm_maskz_cmul_pch(U, A, B) _mm_maskz_fcmul_pch(U, A, B)
#define _mm256_cmul_pch(A, B) _mm256_fcmul_pch(A, B)
#define _mm256_mask_cmul_pch(W, U, A, B) _mm256_mask_fcmul_pch(W, U, A, B)
#define _mm256_maskz_cmul_pch(U, A, B) _mm256_maskz_fcmul_pch(U, A, B)

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256
#undef __DEFAULT_FN_ATTRS256_CONSTEXPR
#undef __DEFAULT_FN_ATTRS128_CONSTEXPR

````
- **L2065 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_min_ph(__m128h __V) {`.
  **L2065 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_min_ph(__m128h __V) {`。
- **L2066 EN**: Returns from the current function with `__builtin_ia32_reduce_fmin_ph128(__V)`.
  **L2066 CN**: 以 `__builtin_ia32_reduce_fmin_ph128(__V)` 从当前函数返回。
- **L2067 EN**: Closes the current lexical scope or compound statement.
  **L2067 CN**: 结束当前词法作用域或复合语句块。
- **L2068 EN**: Blank line separating nearby declarations or logic blocks.
  **L2068 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2069 EN**: Comment explains nearby logic, constraints, or intent: `intrinsics below are alias for f*mul_*ch`.
  **L2069 CN**: 注释解释附近代码的逻辑、约束或设计意图：`intrinsics below are alias for f*mul_*ch`。
- **L2070 EN**: Defines macro `_mm_mul_pch(A, B)` for conditional compilation, shorthand, or API generation.
  **L2070 CN**: 定义宏 `_mm_mul_pch(A, B)`，用于条件编译、简写或 API 生成。
- **L2071 EN**: Defines macro `_mm_mask_mul_pch(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L2071 CN**: 定义宏 `_mm_mask_mul_pch(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L2072 EN**: Defines macro `_mm_maskz_mul_pch(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L2072 CN**: 定义宏 `_mm_maskz_mul_pch(U, A, B)`，用于条件编译、简写或 API 生成。
- **L2073 EN**: Defines macro `_mm256_mul_pch(A, B)` for conditional compilation, shorthand, or API generation.
  **L2073 CN**: 定义宏 `_mm256_mul_pch(A, B)`，用于条件编译、简写或 API 生成。
- **L2074 EN**: Defines macro `_mm256_mask_mul_pch(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L2074 CN**: 定义宏 `_mm256_mask_mul_pch(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L2075 EN**: Defines macro `_mm256_maskz_mul_pch(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L2075 CN**: 定义宏 `_mm256_maskz_mul_pch(U, A, B)`，用于条件编译、简写或 API 生成。
- **L2076 EN**: Blank line separating nearby declarations or logic blocks.
  **L2076 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2077 EN**: Defines macro `_mm_cmul_pch(A, B)` for conditional compilation, shorthand, or API generation.
  **L2077 CN**: 定义宏 `_mm_cmul_pch(A, B)`，用于条件编译、简写或 API 生成。
- **L2078 EN**: Defines macro `_mm_mask_cmul_pch(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L2078 CN**: 定义宏 `_mm_mask_cmul_pch(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L2079 EN**: Defines macro `_mm_maskz_cmul_pch(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L2079 CN**: 定义宏 `_mm_maskz_cmul_pch(U, A, B)`，用于条件编译、简写或 API 生成。
- **L2080 EN**: Defines macro `_mm256_cmul_pch(A, B)` for conditional compilation, shorthand, or API generation.
  **L2080 CN**: 定义宏 `_mm256_cmul_pch(A, B)`，用于条件编译、简写或 API 生成。
- **L2081 EN**: Defines macro `_mm256_mask_cmul_pch(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L2081 CN**: 定义宏 `_mm256_mask_cmul_pch(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L2082 EN**: Defines macro `_mm256_maskz_cmul_pch(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L2082 CN**: 定义宏 `_mm256_maskz_cmul_pch(U, A, B)`，用于条件编译、简写或 API 生成。
- **L2083 EN**: Blank line separating nearby declarations or logic blocks.
  **L2083 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2084 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L2084 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L2085 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L2085 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。
- **L2086 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2086 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2087 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2087 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2088 EN**: Blank line separating nearby declarations or logic blocks.
  **L2088 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2089-2090

````c
#endif
#endif
````
- **L2089 EN**: Closes the current preprocessor conditional block.
  **L2089 CN**: 结束当前预处理条件块。
- **L2090 EN**: Closes the current preprocessor conditional block.
  **L2090 CN**: 结束当前预处理条件块。

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
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__SSE2__`, `__AVX512VLFP16INTRIN_H`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_bit_cast`, `__builtin_ia32_selectph_256`, `__builtin_ia32_selectph_128`, `__builtin_ia32_minph256`, `__builtin_ia32_minph128`, `__builtin_ia32_maxph256`, `__builtin_ia32_maxph128`, `__builtin_ia32_selectps_256`, `__builtin_ia32_selectps_128`, `__builtin_ia32_cmpph256_mask`, `__builtin_ia32_cmpph128_mask`, `__builtin_ia32_rcpph256_mask`
