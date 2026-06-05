# avx512dqintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx512dqintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: AVX512DQ intrinsics.
- **Purpose (CN)**: 提供 AVX512DQ intrinsic 接口。
- **Line Count / 行数**: 1350

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- avx512dqintrin.h - AVX512DQ intrinsics ---------------------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __IMMINTRIN_H
#error "Never use <avx512dqintrin.h> directly; include <immintrin.h> instead."
#endif

#ifndef __AVX512DQINTRIN_H
#define __AVX512DQINTRIN_H

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS512                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("avx512dq"),       \
                 __min_vector_width__(512)))
#define __DEFAULT_FN_ATTRS                                                     \
  __attribute__((__always_inline__, __nodebug__, __target__("avx512dq")))

#if defined(__cplusplus) && (__cplusplus >= 201103L)
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
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <avx512dqintrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error "Never use <avx512dqintrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifndef __AVX512DQINTRIN_H`.
  **L14 CN**: 开始一个预处理条件块：`#ifndef __AVX512DQINTRIN_H`。
- **L15 EN**: Defines macro `__AVX512DQINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `__AVX512DQINTRIN_H`，用于条件编译、简写或 API 生成。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L18 EN**: Defines macro `__DEFAULT_FN_ATTRS512` for conditional compilation, shorthand, or API generation.
  **L18 CN**: 定义宏 `__DEFAULT_FN_ATTRS512`，用于条件编译、简写或 API 生成。
- **L19 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx512dq"),       \`.
  **L19 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx512dq"),       \`。
- **L20 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L20 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L21 EN**: Defines macro `__DEFAULT_FN_ATTRS` for conditional compilation, shorthand, or API generation.
  **L21 CN**: 定义宏 `__DEFAULT_FN_ATTRS`，用于条件编译、简写或 API 生成。
- **L22 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx512dq")))`.
  **L22 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx512dq")))`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L24 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。

### Lines 25-48

````c
#define __DEFAULT_FN_ATTRS512_CONSTEXPR __DEFAULT_FN_ATTRS512 constexpr
#define __DEFAULT_FN_ATTRS_CONSTEXPR __DEFAULT_FN_ATTRS constexpr
#else
#define __DEFAULT_FN_ATTRS512_CONSTEXPR __DEFAULT_FN_ATTRS512
#define __DEFAULT_FN_ATTRS_CONSTEXPR __DEFAULT_FN_ATTRS
#endif

static __inline __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR
_knot_mask8(__mmask8 __M) {
  return __builtin_ia32_knotqi(__M);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR
_kand_mask8(__mmask8 __A, __mmask8 __B) {
  return (__mmask8)__builtin_ia32_kandqi((__mmask8)__A, (__mmask8)__B);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR
_kandn_mask8(__mmask8 __A, __mmask8 __B) {
  return (__mmask8)__builtin_ia32_kandnqi((__mmask8)__A, (__mmask8)__B);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR
_kor_mask8(__mmask8 __A, __mmask8 __B) {
````
- **L25 EN**: Defines macro `__DEFAULT_FN_ATTRS512_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L25 CN**: 定义宏 `__DEFAULT_FN_ATTRS512_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L26 EN**: Defines macro `__DEFAULT_FN_ATTRS_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L26 CN**: 定义宏 `__DEFAULT_FN_ATTRS_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L27 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L27 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L28 EN**: Defines macro `__DEFAULT_FN_ATTRS512_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L28 CN**: 定义宏 `__DEFAULT_FN_ATTRS512_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L29 EN**: Defines macro `__DEFAULT_FN_ATTRS_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L29 CN**: 定义宏 `__DEFAULT_FN_ATTRS_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L30 EN**: Closes the current preprocessor conditional block.
  **L30 CN**: 结束当前预处理条件块。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L32 EN**: Continues the surrounding expression or declaration: `static __inline __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L32 CN**: 继续构造周围的表达式或声明：`static __inline __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L33 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_knot_mask8(__mmask8 __M) {`.
  **L33 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_knot_mask8(__mmask8 __M) {`。
- **L34 EN**: Returns from the current function with `__builtin_ia32_knotqi(__M)`.
  **L34 CN**: 以 `__builtin_ia32_knotqi(__M)` 从当前函数返回。
- **L35 EN**: Closes the current lexical scope or compound statement.
  **L35 CN**: 结束当前词法作用域或复合语句块。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L37 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L37 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L38 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kand_mask8(__mmask8 __A, __mmask8 __B) {`.
  **L38 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kand_mask8(__mmask8 __A, __mmask8 __B) {`。
- **L39 EN**: Returns from the current function with `(__mmask8)__builtin_ia32_kandqi((__mmask8)__A, (__mmask8)__B)`.
  **L39 CN**: 以 `(__mmask8)__builtin_ia32_kandqi((__mmask8)__A, (__mmask8)__B)` 从当前函数返回。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L42 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L43 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kandn_mask8(__mmask8 __A, __mmask8 __B) {`.
  **L43 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kandn_mask8(__mmask8 __A, __mmask8 __B) {`。
- **L44 EN**: Returns from the current function with `(__mmask8)__builtin_ia32_kandnqi((__mmask8)__A, (__mmask8)__B)`.
  **L44 CN**: 以 `(__mmask8)__builtin_ia32_kandnqi((__mmask8)__A, (__mmask8)__B)` 从当前函数返回。
- **L45 EN**: Closes the current lexical scope or compound statement.
  **L45 CN**: 结束当前词法作用域或复合语句块。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L47 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L48 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kor_mask8(__mmask8 __A, __mmask8 __B) {`.
  **L48 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kor_mask8(__mmask8 __A, __mmask8 __B) {`。

### Lines 49-72

````c
  return (__mmask8)__builtin_ia32_korqi((__mmask8)__A, (__mmask8)__B);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR
_kxnor_mask8(__mmask8 __A, __mmask8 __B) {
  return (__mmask8)__builtin_ia32_kxnorqi((__mmask8)__A, (__mmask8)__B);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR
_kxor_mask8(__mmask8 __A, __mmask8 __B) {
  return (__mmask8)__builtin_ia32_kxorqi((__mmask8)__A, (__mmask8)__B);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
_kortestc_mask8_u8(__mmask8 __A, __mmask8 __B) {
  return (unsigned char)__builtin_ia32_kortestcqi(__A, __B);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
_kortestz_mask8_u8(__mmask8 __A, __mmask8 __B) {
  return (unsigned char)__builtin_ia32_kortestzqi(__A, __B);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
````
- **L49 EN**: Returns from the current function with `(__mmask8)__builtin_ia32_korqi((__mmask8)__A, (__mmask8)__B)`.
  **L49 CN**: 以 `(__mmask8)__builtin_ia32_korqi((__mmask8)__A, (__mmask8)__B)` 从当前函数返回。
- **L50 EN**: Closes the current lexical scope or compound statement.
  **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L52 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L53 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kxnor_mask8(__mmask8 __A, __mmask8 __B) {`.
  **L53 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kxnor_mask8(__mmask8 __A, __mmask8 __B) {`。
- **L54 EN**: Returns from the current function with `(__mmask8)__builtin_ia32_kxnorqi((__mmask8)__A, (__mmask8)__B)`.
  **L54 CN**: 以 `(__mmask8)__builtin_ia32_kxnorqi((__mmask8)__A, (__mmask8)__B)` 从当前函数返回。
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L57 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L57 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L58 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kxor_mask8(__mmask8 __A, __mmask8 __B) {`.
  **L58 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kxor_mask8(__mmask8 __A, __mmask8 __B) {`。
- **L59 EN**: Returns from the current function with `(__mmask8)__builtin_ia32_kxorqi((__mmask8)__A, (__mmask8)__B)`.
  **L59 CN**: 以 `(__mmask8)__builtin_ia32_kxorqi((__mmask8)__A, (__mmask8)__B)` 从当前函数返回。
- **L60 EN**: Closes the current lexical scope or compound statement.
  **L60 CN**: 结束当前词法作用域或复合语句块。
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L62 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L62 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L63 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kortestc_mask8_u8(__mmask8 __A, __mmask8 __B) {`.
  **L63 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kortestc_mask8_u8(__mmask8 __A, __mmask8 __B) {`。
- **L64 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_kortestcqi(__A, __B)`.
  **L64 CN**: 以 `(unsigned char)__builtin_ia32_kortestcqi(__A, __B)` 从当前函数返回。
- **L65 EN**: Closes the current lexical scope or compound statement.
  **L65 CN**: 结束当前词法作用域或复合语句块。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L67 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L67 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L68 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kortestz_mask8_u8(__mmask8 __A, __mmask8 __B) {`.
  **L68 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kortestz_mask8_u8(__mmask8 __A, __mmask8 __B) {`。
- **L69 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_kortestzqi(__A, __B)`.
  **L69 CN**: 以 `(unsigned char)__builtin_ia32_kortestzqi(__A, __B)` 从当前函数返回。
- **L70 EN**: Closes the current lexical scope or compound statement.
  **L70 CN**: 结束当前词法作用域或复合语句块。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L72 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。

### Lines 73-96

````c
_kortest_mask8_u8(__mmask8 __A, __mmask8 __B, unsigned char *__C) {
  *__C = (unsigned char)__builtin_ia32_kortestcqi(__A, __B);
  return (unsigned char)__builtin_ia32_kortestzqi(__A, __B);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
_ktestc_mask8_u8(__mmask8 __A, __mmask8 __B) {
  return (unsigned char)__builtin_ia32_ktestcqi(__A, __B);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
_ktestz_mask8_u8(__mmask8 __A, __mmask8 __B) {
  return (unsigned char)__builtin_ia32_ktestzqi(__A, __B);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
_ktest_mask8_u8(__mmask8 __A, __mmask8 __B, unsigned char *__C) {
  *__C = (unsigned char)__builtin_ia32_ktestcqi(__A, __B);
  return (unsigned char)__builtin_ia32_ktestzqi(__A, __B);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
_ktestc_mask16_u8(__mmask16 __A, __mmask16 __B) {
  return (unsigned char)__builtin_ia32_ktestchi(__A, __B);
````
- **L73 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kortest_mask8_u8(__mmask8 __A, __mmask8 __B, unsigned char *__C) {`.
  **L73 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kortest_mask8_u8(__mmask8 __A, __mmask8 __B, unsigned char *__C) {`。
- **L74 EN**: Comment explains nearby logic, constraints, or intent: `__C (unsigned char)__builtin_ia32_kortestcqi(__A, __B);`.
  **L74 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__C (unsigned char)__builtin_ia32_kortestcqi(__A, __B);`。
- **L75 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_kortestzqi(__A, __B)`.
  **L75 CN**: 以 `(unsigned char)__builtin_ia32_kortestzqi(__A, __B)` 从当前函数返回。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L78 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L78 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L79 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_ktestc_mask8_u8(__mmask8 __A, __mmask8 __B) {`.
  **L79 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_ktestc_mask8_u8(__mmask8 __A, __mmask8 __B) {`。
- **L80 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_ktestcqi(__A, __B)`.
  **L80 CN**: 以 `(unsigned char)__builtin_ia32_ktestcqi(__A, __B)` 从当前函数返回。
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L83 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L83 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L84 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_ktestz_mask8_u8(__mmask8 __A, __mmask8 __B) {`.
  **L84 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_ktestz_mask8_u8(__mmask8 __A, __mmask8 __B) {`。
- **L85 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_ktestzqi(__A, __B)`.
  **L85 CN**: 以 `(unsigned char)__builtin_ia32_ktestzqi(__A, __B)` 从当前函数返回。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L88 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L88 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L89 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_ktest_mask8_u8(__mmask8 __A, __mmask8 __B, unsigned char *__C) {`.
  **L89 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_ktest_mask8_u8(__mmask8 __A, __mmask8 __B, unsigned char *__C) {`。
- **L90 EN**: Comment explains nearby logic, constraints, or intent: `__C (unsigned char)__builtin_ia32_ktestcqi(__A, __B);`.
  **L90 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__C (unsigned char)__builtin_ia32_ktestcqi(__A, __B);`。
- **L91 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_ktestzqi(__A, __B)`.
  **L91 CN**: 以 `(unsigned char)__builtin_ia32_ktestzqi(__A, __B)` 从当前函数返回。
- **L92 EN**: Closes the current lexical scope or compound statement.
  **L92 CN**: 结束当前词法作用域或复合语句块。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L94 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L94 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L95 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_ktestc_mask16_u8(__mmask16 __A, __mmask16 __B) {`.
  **L95 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_ktestc_mask16_u8(__mmask16 __A, __mmask16 __B) {`。
- **L96 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_ktestchi(__A, __B)`.
  **L96 CN**: 以 `(unsigned char)__builtin_ia32_ktestchi(__A, __B)` 从当前函数返回。

### Lines 97-120

````c
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
_ktestz_mask16_u8(__mmask16 __A, __mmask16 __B) {
  return (unsigned char)__builtin_ia32_ktestzhi(__A, __B);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR
_ktest_mask16_u8(__mmask16 __A, __mmask16 __B, unsigned char *__C) {
  *__C = (unsigned char)__builtin_ia32_ktestchi(__A, __B);
  return (unsigned char)__builtin_ia32_ktestzhi(__A, __B);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR
_kadd_mask8(__mmask8 __A, __mmask8 __B) {
  return (__mmask8)__builtin_ia32_kaddqi((__mmask8)__A, (__mmask8)__B);
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS_CONSTEXPR
_kadd_mask16(__mmask16 __A, __mmask16 __B) {
  return (__mmask16)__builtin_ia32_kaddhi((__mmask16)__A, (__mmask16)__B);
}

#define _kshiftli_mask8(A, I) \
````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L99 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L99 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L100 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_ktestz_mask16_u8(__mmask16 __A, __mmask16 __B) {`.
  **L100 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_ktestz_mask16_u8(__mmask16 __A, __mmask16 __B) {`。
- **L101 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_ktestzhi(__A, __B)`.
  **L101 CN**: 以 `(unsigned char)__builtin_ia32_ktestzhi(__A, __B)` 从当前函数返回。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L104 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L104 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L105 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_ktest_mask16_u8(__mmask16 __A, __mmask16 __B, unsigned char *__C) {`.
  **L105 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_ktest_mask16_u8(__mmask16 __A, __mmask16 __B, unsigned char *__C) {`。
- **L106 EN**: Comment explains nearby logic, constraints, or intent: `__C (unsigned char)__builtin_ia32_ktestchi(__A, __B);`.
  **L106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__C (unsigned char)__builtin_ia32_ktestchi(__A, __B);`。
- **L107 EN**: Returns from the current function with `(unsigned char)__builtin_ia32_ktestzhi(__A, __B)`.
  **L107 CN**: 以 `(unsigned char)__builtin_ia32_ktestzhi(__A, __B)` 从当前函数返回。
- **L108 EN**: Closes the current lexical scope or compound statement.
  **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Blank line separating nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L110 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L110 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L111 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kadd_mask8(__mmask8 __A, __mmask8 __B) {`.
  **L111 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kadd_mask8(__mmask8 __A, __mmask8 __B) {`。
- **L112 EN**: Returns from the current function with `(__mmask8)__builtin_ia32_kaddqi((__mmask8)__A, (__mmask8)__B)`.
  **L112 CN**: 以 `(__mmask8)__builtin_ia32_kaddqi((__mmask8)__A, (__mmask8)__B)` 从当前函数返回。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L115 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L116 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_kadd_mask16(__mmask16 __A, __mmask16 __B) {`.
  **L116 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_kadd_mask16(__mmask16 __A, __mmask16 __B) {`。
- **L117 EN**: Returns from the current function with `(__mmask16)__builtin_ia32_kaddhi((__mmask16)__A, (__mmask16)__B)`.
  **L117 CN**: 以 `(__mmask16)__builtin_ia32_kaddhi((__mmask16)__A, (__mmask16)__B)` 从当前函数返回。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L120 EN**: Defines macro `_kshiftli_mask8(A, I)` for conditional compilation, shorthand, or API generation.
  **L120 CN**: 定义宏 `_kshiftli_mask8(A, I)`，用于条件编译、简写或 API 生成。

### Lines 121-144

````c
  ((__mmask8)__builtin_ia32_kshiftliqi((__mmask8)(A), (unsigned int)(I)))

#define _kshiftri_mask8(A, I) \
  ((__mmask8)__builtin_ia32_kshiftriqi((__mmask8)(A), (unsigned int)(I)))

static __inline__ unsigned int
    __DEFAULT_FN_ATTRS_CONSTEXPR _cvtmask8_u32(__mmask8 __A) {
  return (unsigned int)__builtin_ia32_kmovb((__mmask8)__A);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR
_cvtu32_mask8(unsigned int __A) {
  return (__mmask8)__builtin_ia32_kmovb((__mmask8)__A);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS
_load_mask8(__mmask8 *__A) {
  return (__mmask8)__builtin_ia32_kmovb(*(__mmask8 *)__A);
}

static __inline__ void __DEFAULT_FN_ATTRS
_store_mask8(__mmask8 *__A, __mmask8 __B) {
  *(__mmask8 *)__A = __builtin_ia32_kmovb((__mmask8)__B);
}
````
- **L121 EN**: Continues logic associated with callable symbol `__builtin_ia32_kshiftliqi`.
  **L121 CN**: 继续与可调用符号 `__builtin_ia32_kshiftliqi` 相关的逻辑。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L123 EN**: Defines macro `_kshiftri_mask8(A, I)` for conditional compilation, shorthand, or API generation.
  **L123 CN**: 定义宏 `_kshiftri_mask8(A, I)`，用于条件编译、简写或 API 生成。
- **L124 EN**: Continues logic associated with callable symbol `__builtin_ia32_kshiftriqi`.
  **L124 CN**: 继续与可调用符号 `__builtin_ia32_kshiftriqi` 相关的逻辑。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L126 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned int`.
  **L126 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned int`。
- **L127 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS_CONSTEXPR _cvtmask8_u32(__mmask8 __A) {`.
  **L127 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS_CONSTEXPR _cvtmask8_u32(__mmask8 __A) {`。
- **L128 EN**: Returns from the current function with `(unsigned int)__builtin_ia32_kmovb((__mmask8)__A)`.
  **L128 CN**: 以 `(unsigned int)__builtin_ia32_kmovb((__mmask8)__A)` 从当前函数返回。
- **L129 EN**: Closes the current lexical scope or compound statement.
  **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L131 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L131 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L132 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_cvtu32_mask8(unsigned int __A) {`.
  **L132 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_cvtu32_mask8(unsigned int __A) {`。
- **L133 EN**: Returns from the current function with `(__mmask8)__builtin_ia32_kmovb((__mmask8)__A)`.
  **L133 CN**: 以 `(__mmask8)__builtin_ia32_kmovb((__mmask8)__A)` 从当前函数返回。
- **L134 EN**: Closes the current lexical scope or compound statement.
  **L134 CN**: 结束当前词法作用域或复合语句块。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L136 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS`.
  **L136 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS`。
- **L137 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_load_mask8(__mmask8 *__A) {`.
  **L137 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_load_mask8(__mmask8 *__A) {`。
- **L138 EN**: Returns from the current function with `(__mmask8)__builtin_ia32_kmovb(*(__mmask8 *)__A)`.
  **L138 CN**: 以 `(__mmask8)__builtin_ia32_kmovb(*(__mmask8 *)__A)` 从当前函数返回。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L141 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS`.
  **L141 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS`。
- **L142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_store_mask8(__mmask8 *__A, __mmask8 __B) {`.
  **L142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_store_mask8(__mmask8 *__A, __mmask8 __B) {`。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `(__mmask8 *)__A __builtin_ia32_kmovb((__mmask8)__B);`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__mmask8 *)__A __builtin_ia32_kmovb((__mmask8)__B);`。
- **L144 EN**: Closes the current lexical scope or compound statement.
  **L144 CN**: 结束当前词法作用域或复合语句块。

### Lines 145-168

````c

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mullo_epi64(__m512i __A, __m512i __B) {
  return (__m512i) ((__v8du) __A * (__v8du) __B);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_mullo_epi64(__m512i __W, __mmask8 __U, __m512i __A, __m512i __B) {
  return (__m512i)__builtin_ia32_selectq_512((__mmask8)__U,
                                             (__v8di)_mm512_mullo_epi64(__A, __B),
                                             (__v8di)__W);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_mullo_epi64(__mmask8 __U, __m512i __A, __m512i __B) {
  return (__m512i)__builtin_ia32_selectq_512((__mmask8)__U,
                                             (__v8di)_mm512_mullo_epi64(__A, __B),
                                             (__v8di)_mm512_setzero_si512());
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_xor_pd(__m512d __A, __m512d __B) {
  return (__m512d)((__v8du)__A ^ (__v8du)__B);
}
````
- **L145 EN**: Blank line separating nearby declarations or logic blocks.
  **L145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L146 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L146 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L147 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mullo_epi64(__m512i __A, __m512i __B) {`.
  **L147 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mullo_epi64(__m512i __A, __m512i __B) {`。
- **L148 EN**: Returns from the current function with `(__m512i) ((__v8du) __A * (__v8du) __B)`.
  **L148 CN**: 以 `(__m512i) ((__v8du) __A * (__v8du) __B)` 从当前函数返回。
- **L149 EN**: Closes the current lexical scope or compound statement.
  **L149 CN**: 结束当前词法作用域或复合语句块。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L151 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L151 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L152 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_mullo_epi64(__m512i __W, __mmask8 __U, __m512i __A, __m512i __B) {`.
  **L152 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_mullo_epi64(__m512i __W, __mmask8 __U, __m512i __A, __m512i __B) {`。
- **L153 EN**: Returns from the current function with `(__m512i)__builtin_ia32_selectq_512((__mmask8)__U,`.
  **L153 CN**: 以 `(__m512i)__builtin_ia32_selectq_512((__mmask8)__U,` 从当前函数返回。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di)_mm512_mullo_epi64(__A, __B),`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di)_mm512_mullo_epi64(__A, __B),`。
- **L155 EN**: Executes a call or declaration centered on `statement`.
  **L155 CN**: 执行以 `statement` 为核心的调用或声明。
- **L156 EN**: Closes the current lexical scope or compound statement.
  **L156 CN**: 结束当前词法作用域或复合语句块。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L158 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L158 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L159 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_mullo_epi64(__mmask8 __U, __m512i __A, __m512i __B) {`.
  **L159 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_mullo_epi64(__mmask8 __U, __m512i __A, __m512i __B) {`。
- **L160 EN**: Returns from the current function with `(__m512i)__builtin_ia32_selectq_512((__mmask8)__U,`.
  **L160 CN**: 以 `(__m512i)__builtin_ia32_selectq_512((__mmask8)__U,` 从当前函数返回。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di)_mm512_mullo_epi64(__A, __B),`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di)_mm512_mullo_epi64(__A, __B),`。
- **L162 EN**: Executes a call or declaration centered on `statement`.
  **L162 CN**: 执行以 `statement` 为核心的调用或声明。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L165 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L166 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_xor_pd(__m512d __A, __m512d __B) {`.
  **L166 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_xor_pd(__m512d __A, __m512d __B) {`。
- **L167 EN**: Returns from the current function with `(__m512d)((__v8du)__A ^ (__v8du)__B)`.
  **L167 CN**: 以 `(__m512d)((__v8du)__A ^ (__v8du)__B)` 从当前函数返回。
- **L168 EN**: Closes the current lexical scope or compound statement.
  **L168 CN**: 结束当前词法作用域或复合语句块。

### Lines 169-192

````c

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_mask_xor_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_xor_pd(__A, __B),
                                              (__v8df)__W);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_maskz_xor_pd(__mmask8 __U, __m512d __A, __m512d __B) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_xor_pd(__A, __B),
                                              (__v8df)_mm512_setzero_pd());
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_xor_ps (__m512 __A, __m512 __B) {
  return (__m512)((__v16su)__A ^ (__v16su)__B);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_mask_xor_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__U,
                                             (__v16sf)_mm512_xor_ps(__A, __B),
````
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L170 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L170 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L171 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_xor_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {`.
  **L171 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_xor_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {`。
- **L172 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L172 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_xor_pd(__A, __B),`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_xor_pd(__A, __B),`。
- **L174 EN**: Executes a call or declaration centered on `statement`.
  **L174 CN**: 执行以 `statement` 为核心的调用或声明。
- **L175 EN**: Closes the current lexical scope or compound statement.
  **L175 CN**: 结束当前词法作用域或复合语句块。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L177 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L177 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L178 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_xor_pd(__mmask8 __U, __m512d __A, __m512d __B) {`.
  **L178 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_xor_pd(__mmask8 __U, __m512d __A, __m512d __B) {`。
- **L179 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L179 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_xor_pd(__A, __B),`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_xor_pd(__A, __B),`。
- **L181 EN**: Executes a call or declaration centered on `statement`.
  **L181 CN**: 执行以 `statement` 为核心的调用或声明。
- **L182 EN**: Closes the current lexical scope or compound statement.
  **L182 CN**: 结束当前词法作用域或复合语句块。
- **L183 EN**: Blank line separating nearby declarations or logic blocks.
  **L183 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L184 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L184 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L185 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_xor_ps (__m512 __A, __m512 __B) {`.
  **L185 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_xor_ps (__m512 __A, __m512 __B) {`。
- **L186 EN**: Returns from the current function with `(__m512)((__v16su)__A ^ (__v16su)__B)`.
  **L186 CN**: 以 `(__m512)((__v16su)__A ^ (__v16su)__B)` 从当前函数返回。
- **L187 EN**: Closes the current lexical scope or compound statement.
  **L187 CN**: 结束当前词法作用域或复合语句块。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L189 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L189 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L190 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_xor_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {`.
  **L190 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_xor_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {`。
- **L191 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L191 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_xor_ps(__A, __B),`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_xor_ps(__A, __B),`。

### Lines 193-216

````c
                                             (__v16sf)__W);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_maskz_xor_ps(__mmask16 __U, __m512 __A, __m512 __B) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__U,
                                             (__v16sf)_mm512_xor_ps(__A, __B),
                                             (__v16sf)_mm512_setzero_ps());
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_or_pd(__m512d __A, __m512d __B) {
  return (__m512d)((__v8du)__A | (__v8du)__B);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_mask_or_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_or_pd(__A, __B),
                                              (__v8df)__W);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_maskz_or_pd(__mmask8 __U, __m512d __A, __m512d __B) {
````
- **L193 EN**: Executes a call or declaration centered on `statement`.
  **L193 CN**: 执行以 `statement` 为核心的调用或声明。
- **L194 EN**: Closes the current lexical scope or compound statement.
  **L194 CN**: 结束当前词法作用域或复合语句块。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L196 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L196 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L197 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_xor_ps(__mmask16 __U, __m512 __A, __m512 __B) {`.
  **L197 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_xor_ps(__mmask16 __U, __m512 __A, __m512 __B) {`。
- **L198 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L198 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_xor_ps(__A, __B),`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_xor_ps(__A, __B),`。
- **L200 EN**: Executes a call or declaration centered on `statement`.
  **L200 CN**: 执行以 `statement` 为核心的调用或声明。
- **L201 EN**: Closes the current lexical scope or compound statement.
  **L201 CN**: 结束当前词法作用域或复合语句块。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L203 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L203 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L204 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_or_pd(__m512d __A, __m512d __B) {`.
  **L204 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_or_pd(__m512d __A, __m512d __B) {`。
- **L205 EN**: Returns from the current function with `(__m512d)((__v8du)__A | (__v8du)__B)`.
  **L205 CN**: 以 `(__m512d)((__v8du)__A | (__v8du)__B)` 从当前函数返回。
- **L206 EN**: Closes the current lexical scope or compound statement.
  **L206 CN**: 结束当前词法作用域或复合语句块。
- **L207 EN**: Blank line separating nearby declarations or logic blocks.
  **L207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L208 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L208 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L209 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_or_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {`.
  **L209 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_or_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {`。
- **L210 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L210 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_or_pd(__A, __B),`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_or_pd(__A, __B),`。
- **L212 EN**: Executes a call or declaration centered on `statement`.
  **L212 CN**: 执行以 `statement` 为核心的调用或声明。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L215 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L215 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L216 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_or_pd(__mmask8 __U, __m512d __A, __m512d __B) {`.
  **L216 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_or_pd(__mmask8 __U, __m512d __A, __m512d __B) {`。

### Lines 217-240

````c
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_or_pd(__A, __B),
                                              (__v8df)_mm512_setzero_pd());
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_or_ps(__m512 __A, __m512 __B) {
  return (__m512)((__v16su)__A | (__v16su)__B);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_mask_or_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__U,
                                             (__v16sf)_mm512_or_ps(__A, __B),
                                             (__v16sf)__W);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_maskz_or_ps(__mmask16 __U, __m512 __A, __m512 __B) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__U,
                                             (__v16sf)_mm512_or_ps(__A, __B),
                                             (__v16sf)_mm512_setzero_ps());
}

````
- **L217 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L217 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_or_pd(__A, __B),`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_or_pd(__A, __B),`。
- **L219 EN**: Executes a call or declaration centered on `statement`.
  **L219 CN**: 执行以 `statement` 为核心的调用或声明。
- **L220 EN**: Closes the current lexical scope or compound statement.
  **L220 CN**: 结束当前词法作用域或复合语句块。
- **L221 EN**: Blank line separating nearby declarations or logic blocks.
  **L221 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L222 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L222 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L223 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_or_ps(__m512 __A, __m512 __B) {`.
  **L223 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_or_ps(__m512 __A, __m512 __B) {`。
- **L224 EN**: Returns from the current function with `(__m512)((__v16su)__A | (__v16su)__B)`.
  **L224 CN**: 以 `(__m512)((__v16su)__A | (__v16su)__B)` 从当前函数返回。
- **L225 EN**: Closes the current lexical scope or compound statement.
  **L225 CN**: 结束当前词法作用域或复合语句块。
- **L226 EN**: Blank line separating nearby declarations or logic blocks.
  **L226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L227 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L227 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L228 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_or_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {`.
  **L228 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_or_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {`。
- **L229 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L229 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_or_ps(__A, __B),`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_or_ps(__A, __B),`。
- **L231 EN**: Executes a call or declaration centered on `statement`.
  **L231 CN**: 执行以 `statement` 为核心的调用或声明。
- **L232 EN**: Closes the current lexical scope or compound statement.
  **L232 CN**: 结束当前词法作用域或复合语句块。
- **L233 EN**: Blank line separating nearby declarations or logic blocks.
  **L233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L234 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L234 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L235 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_or_ps(__mmask16 __U, __m512 __A, __m512 __B) {`.
  **L235 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_or_ps(__mmask16 __U, __m512 __A, __m512 __B) {`。
- **L236 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L236 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_or_ps(__A, __B),`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_or_ps(__A, __B),`。
- **L238 EN**: Executes a call or declaration centered on `statement`.
  **L238 CN**: 执行以 `statement` 为核心的调用或声明。
- **L239 EN**: Closes the current lexical scope or compound statement.
  **L239 CN**: 结束当前词法作用域或复合语句块。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 241-264

````c
static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_and_pd(__m512d __A, __m512d __B) {
  return (__m512d)((__v8du)__A & (__v8du)__B);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_mask_and_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_and_pd(__A, __B),
                                              (__v8df)__W);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_maskz_and_pd(__mmask8 __U, __m512d __A, __m512d __B) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_and_pd(__A, __B),
                                              (__v8df)_mm512_setzero_pd());
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_and_ps(__m512 __A, __m512 __B) {
  return (__m512)((__v16su)__A & (__v16su)__B);
}

````
- **L241 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L241 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L242 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_and_pd(__m512d __A, __m512d __B) {`.
  **L242 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_and_pd(__m512d __A, __m512d __B) {`。
- **L243 EN**: Returns from the current function with `(__m512d)((__v8du)__A & (__v8du)__B)`.
  **L243 CN**: 以 `(__m512d)((__v8du)__A & (__v8du)__B)` 从当前函数返回。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L246 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L246 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L247 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_and_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {`.
  **L247 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_and_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {`。
- **L248 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L248 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_and_pd(__A, __B),`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_and_pd(__A, __B),`。
- **L250 EN**: Executes a call or declaration centered on `statement`.
  **L250 CN**: 执行以 `statement` 为核心的调用或声明。
- **L251 EN**: Closes the current lexical scope or compound statement.
  **L251 CN**: 结束当前词法作用域或复合语句块。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L253 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L253 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L254 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_and_pd(__mmask8 __U, __m512d __A, __m512d __B) {`.
  **L254 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_and_pd(__mmask8 __U, __m512d __A, __m512d __B) {`。
- **L255 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L255 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_and_pd(__A, __B),`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_and_pd(__A, __B),`。
- **L257 EN**: Executes a call or declaration centered on `statement`.
  **L257 CN**: 执行以 `statement` 为核心的调用或声明。
- **L258 EN**: Closes the current lexical scope or compound statement.
  **L258 CN**: 结束当前词法作用域或复合语句块。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L260 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L260 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L261 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_and_ps(__m512 __A, __m512 __B) {`.
  **L261 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_and_ps(__m512 __A, __m512 __B) {`。
- **L262 EN**: Returns from the current function with `(__m512)((__v16su)__A & (__v16su)__B)`.
  **L262 CN**: 以 `(__m512)((__v16su)__A & (__v16su)__B)` 从当前函数返回。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 265-288

````c
static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_mask_and_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__U,
                                             (__v16sf)_mm512_and_ps(__A, __B),
                                             (__v16sf)__W);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_maskz_and_ps(__mmask16 __U, __m512 __A, __m512 __B) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__U,
                                             (__v16sf)_mm512_and_ps(__A, __B),
                                             (__v16sf)_mm512_setzero_ps());
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_andnot_pd(__m512d __A, __m512d __B) {
  return (__m512d)(~(__v8du)__A & (__v8du)__B);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_mask_andnot_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_andnot_pd(__A, __B),
                                              (__v8df)__W);
````
- **L265 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L265 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L266 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_and_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {`.
  **L266 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_and_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {`。
- **L267 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L267 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_and_ps(__A, __B),`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_and_ps(__A, __B),`。
- **L269 EN**: Executes a call or declaration centered on `statement`.
  **L269 CN**: 执行以 `statement` 为核心的调用或声明。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Blank line separating nearby declarations or logic blocks.
  **L271 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L272 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L272 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L273 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_and_ps(__mmask16 __U, __m512 __A, __m512 __B) {`.
  **L273 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_and_ps(__mmask16 __U, __m512 __A, __m512 __B) {`。
- **L274 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L274 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_and_ps(__A, __B),`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_and_ps(__A, __B),`。
- **L276 EN**: Executes a call or declaration centered on `statement`.
  **L276 CN**: 执行以 `statement` 为核心的调用或声明。
- **L277 EN**: Closes the current lexical scope or compound statement.
  **L277 CN**: 结束当前词法作用域或复合语句块。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L279 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L279 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L280 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_andnot_pd(__m512d __A, __m512d __B) {`.
  **L280 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_andnot_pd(__m512d __A, __m512d __B) {`。
- **L281 EN**: Returns from the current function with `(__m512d)(~(__v8du)__A & (__v8du)__B)`.
  **L281 CN**: 以 `(__m512d)(~(__v8du)__A & (__v8du)__B)` 从当前函数返回。
- **L282 EN**: Closes the current lexical scope or compound statement.
  **L282 CN**: 结束当前词法作用域或复合语句块。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L284 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L284 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L285 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_andnot_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {`.
  **L285 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_andnot_pd(__m512d __W, __mmask8 __U, __m512d __A, __m512d __B) {`。
- **L286 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L286 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_andnot_pd(__A, __B),`.
  **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_andnot_pd(__A, __B),`。
- **L288 EN**: Executes a call or declaration centered on `statement`.
  **L288 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 289-312

````c
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_maskz_andnot_pd(__mmask8 __U, __m512d __A, __m512d __B) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_andnot_pd(__A, __B),
                                              (__v8df)_mm512_setzero_pd());
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_andnot_ps(__m512 __A, __m512 __B) {
  return (__m512)(~(__v16su)__A & (__v16su)__B);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_mask_andnot_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__U,
                                             (__v16sf)_mm512_andnot_ps(__A, __B),
                                             (__v16sf)__W);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_maskz_andnot_ps(__mmask16 __U, __m512 __A, __m512 __B) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__U,
````
- **L289 EN**: Closes the current lexical scope or compound statement.
  **L289 CN**: 结束当前词法作用域或复合语句块。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L291 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L292 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_andnot_pd(__mmask8 __U, __m512d __A, __m512d __B) {`.
  **L292 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_andnot_pd(__mmask8 __U, __m512d __A, __m512d __B) {`。
- **L293 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L293 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_andnot_pd(__A, __B),`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_andnot_pd(__A, __B),`。
- **L295 EN**: Executes a call or declaration centered on `statement`.
  **L295 CN**: 执行以 `statement` 为核心的调用或声明。
- **L296 EN**: Closes the current lexical scope or compound statement.
  **L296 CN**: 结束当前词法作用域或复合语句块。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L298 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L298 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L299 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_andnot_ps(__m512 __A, __m512 __B) {`.
  **L299 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_andnot_ps(__m512 __A, __m512 __B) {`。
- **L300 EN**: Returns from the current function with `(__m512)(~(__v16su)__A & (__v16su)__B)`.
  **L300 CN**: 以 `(__m512)(~(__v16su)__A & (__v16su)__B)` 从当前函数返回。
- **L301 EN**: Closes the current lexical scope or compound statement.
  **L301 CN**: 结束当前词法作用域或复合语句块。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L303 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L303 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L304 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_andnot_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {`.
  **L304 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_andnot_ps(__m512 __W, __mmask16 __U, __m512 __A, __m512 __B) {`。
- **L305 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L305 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_andnot_ps(__A, __B),`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_andnot_ps(__A, __B),`。
- **L307 EN**: Executes a call or declaration centered on `statement`.
  **L307 CN**: 执行以 `statement` 为核心的调用或声明。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L310 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L311 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_andnot_ps(__mmask16 __U, __m512 __A, __m512 __B) {`.
  **L311 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_andnot_ps(__mmask16 __U, __m512 __A, __m512 __B) {`。
- **L312 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L312 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。

### Lines 313-336

````c
                                             (__v16sf)_mm512_andnot_ps(__A, __B),
                                             (__v16sf)_mm512_setzero_ps());
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtpd_epi64 (__m512d __A) {
  return (__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,
                (__v8di) _mm512_setzero_si512(),
                (__mmask8) -1,
                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtpd_epi64 (__m512i __W, __mmask8 __U, __m512d __A) {
  return (__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,
                (__v8di) __W,
                (__mmask8) __U,
                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtpd_epi64 (__mmask8 __U, __m512d __A) {
  return (__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,
                (__v8di) _mm512_setzero_si512(),
````
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_andnot_ps(__A, __B),`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_andnot_ps(__A, __B),`。
- **L314 EN**: Executes a call or declaration centered on `statement`.
  **L314 CN**: 执行以 `statement` 为核心的调用或声明。
- **L315 EN**: Closes the current lexical scope or compound statement.
  **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L317 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L317 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L318 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtpd_epi64 (__m512d __A) {`.
  **L318 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtpd_epi64 (__m512d __A) {`。
- **L319 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,`.
  **L319 CN**: 以 `(__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,` 从当前函数返回。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L322 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L322 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L325 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L325 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L326 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtpd_epi64 (__m512i __W, __mmask8 __U, __m512d __A) {`.
  **L326 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtpd_epi64 (__m512i __W, __mmask8 __U, __m512d __A) {`。
- **L327 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,`.
  **L327 CN**: 以 `(__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,` 从当前函数返回。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) __W,`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) __W,`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L330 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L330 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L333 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L334 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtpd_epi64 (__mmask8 __U, __m512d __A) {`.
  **L334 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtpd_epi64 (__mmask8 __U, __m512d __A) {`。
- **L335 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,`.
  **L335 CN**: 以 `(__m512i) __builtin_ia32_cvtpd2qq512_mask ((__v8df) __A,` 从当前函数返回。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。

### Lines 337-360

````c
                (__mmask8) __U,
                _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundpd_epi64(A, R) \
  ((__m512i)__builtin_ia32_cvtpd2qq512_mask((__v8df)(__m512d)(A), \
                                            (__v8di)_mm512_setzero_si512(), \
                                            (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvt_roundpd_epi64(W, U, A, R) \
  ((__m512i)__builtin_ia32_cvtpd2qq512_mask((__v8df)(__m512d)(A), \
                                            (__v8di)(__m512i)(W), \
                                            (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundpd_epi64(U, A, R) \
  ((__m512i)__builtin_ia32_cvtpd2qq512_mask((__v8df)(__m512d)(A), \
                                            (__v8di)_mm512_setzero_si512(), \
                                            (__mmask8)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtpd_epu64 (__m512d __A) {
  return (__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,
                 (__v8di) _mm512_setzero_si512(),
                 (__mmask8) -1,
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L338 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L338 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L341 EN**: Defines macro `_mm512_cvt_roundpd_epi64(A, R)` for conditional compilation, shorthand, or API generation.
  **L341 CN**: 定义宏 `_mm512_cvt_roundpd_epi64(A, R)`，用于条件编译、简写或 API 生成。
- **L342 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtpd2qq512_mask`.
  **L342 CN**: 继续与可调用符号 `__builtin_ia32_cvtpd2qq512_mask` 相关的逻辑。
- **L343 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L343 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L344 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L344 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L346 EN**: Defines macro `_mm512_mask_cvt_roundpd_epi64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L346 CN**: 定义宏 `_mm512_mask_cvt_roundpd_epi64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L347 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtpd2qq512_mask`.
  **L347 CN**: 继续与可调用符号 `__builtin_ia32_cvtpd2qq512_mask` 相关的逻辑。
- **L348 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W), \`.
  **L348 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W), \`。
- **L349 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L349 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L351 EN**: Defines macro `_mm512_maskz_cvt_roundpd_epi64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L351 CN**: 定义宏 `_mm512_maskz_cvt_roundpd_epi64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L352 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtpd2qq512_mask`.
  **L352 CN**: 继续与可调用符号 `__builtin_ia32_cvtpd2qq512_mask` 相关的逻辑。
- **L353 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L353 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L354 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L354 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L356 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L356 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L357 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtpd_epu64 (__m512d __A) {`.
  **L357 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtpd_epu64 (__m512d __A) {`。
- **L358 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,`.
  **L358 CN**: 以 `(__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,` 从当前函数返回。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。

### Lines 361-384

````c
                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtpd_epu64 (__m512i __W, __mmask8 __U, __m512d __A) {
  return (__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,
                 (__v8di) __W,
                 (__mmask8) __U,
                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtpd_epu64 (__mmask8 __U, __m512d __A) {
  return (__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,
                 (__v8di) _mm512_setzero_si512(),
                 (__mmask8) __U,
                 _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundpd_epu64(A, R) \
  ((__m512i)__builtin_ia32_cvtpd2uqq512_mask((__v8df)(__m512d)(A), \
                                             (__v8di)_mm512_setzero_si512(), \
                                             (__mmask8)-1, (int)(R)))

````
- **L361 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L361 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L362 EN**: Closes the current lexical scope or compound statement.
  **L362 CN**: 结束当前词法作用域或复合语句块。
- **L363 EN**: Blank line separating nearby declarations or logic blocks.
  **L363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L364 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L364 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L365 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtpd_epu64 (__m512i __W, __mmask8 __U, __m512d __A) {`.
  **L365 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtpd_epu64 (__m512i __W, __mmask8 __U, __m512d __A) {`。
- **L366 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,`.
  **L366 CN**: 以 `(__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,` 从当前函数返回。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) __W,`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) __W,`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L369 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L369 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L370 EN**: Closes the current lexical scope or compound statement.
  **L370 CN**: 结束当前词法作用域或复合语句块。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L372 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L372 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L373 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtpd_epu64 (__mmask8 __U, __m512d __A) {`.
  **L373 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtpd_epu64 (__mmask8 __U, __m512d __A) {`。
- **L374 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,`.
  **L374 CN**: 以 `(__m512i) __builtin_ia32_cvtpd2uqq512_mask ((__v8df) __A,` 从当前函数返回。
- **L375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L375 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L376 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L377 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L377 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L378 EN**: Closes the current lexical scope or compound statement.
  **L378 CN**: 结束当前词法作用域或复合语句块。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L380 EN**: Defines macro `_mm512_cvt_roundpd_epu64(A, R)` for conditional compilation, shorthand, or API generation.
  **L380 CN**: 定义宏 `_mm512_cvt_roundpd_epu64(A, R)`，用于条件编译、简写或 API 生成。
- **L381 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtpd2uqq512_mask`.
  **L381 CN**: 继续与可调用符号 `__builtin_ia32_cvtpd2uqq512_mask` 相关的逻辑。
- **L382 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L382 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L383 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L383 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````c
#define _mm512_mask_cvt_roundpd_epu64(W, U, A, R) \
  ((__m512i)__builtin_ia32_cvtpd2uqq512_mask((__v8df)(__m512d)(A), \
                                             (__v8di)(__m512i)(W), \
                                             (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundpd_epu64(U, A, R) \
  ((__m512i)__builtin_ia32_cvtpd2uqq512_mask((__v8df)(__m512d)(A), \
                                             (__v8di)_mm512_setzero_si512(), \
                                             (__mmask8)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtps_epi64 (__m256 __A) {
  return (__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,
                (__v8di) _mm512_setzero_si512(),
                (__mmask8) -1,
                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtps_epi64 (__m512i __W, __mmask8 __U, __m256 __A) {
  return (__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,
                (__v8di) __W,
                (__mmask8) __U,
                _MM_FROUND_CUR_DIRECTION);
````
- **L385 EN**: Defines macro `_mm512_mask_cvt_roundpd_epu64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L385 CN**: 定义宏 `_mm512_mask_cvt_roundpd_epu64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L386 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtpd2uqq512_mask`.
  **L386 CN**: 继续与可调用符号 `__builtin_ia32_cvtpd2uqq512_mask` 相关的逻辑。
- **L387 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W), \`.
  **L387 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W), \`。
- **L388 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L388 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L390 EN**: Defines macro `_mm512_maskz_cvt_roundpd_epu64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L390 CN**: 定义宏 `_mm512_maskz_cvt_roundpd_epu64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L391 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtpd2uqq512_mask`.
  **L391 CN**: 继续与可调用符号 `__builtin_ia32_cvtpd2uqq512_mask` 相关的逻辑。
- **L392 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L392 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L393 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L393 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L394 EN**: Blank line separating nearby declarations or logic blocks.
  **L394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L395 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L395 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L396 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtps_epi64 (__m256 __A) {`.
  **L396 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtps_epi64 (__m256 __A) {`。
- **L397 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,`.
  **L397 CN**: 以 `(__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L400 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L400 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L401 EN**: Closes the current lexical scope or compound statement.
  **L401 CN**: 结束当前词法作用域或复合语句块。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L403 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L403 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L404 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtps_epi64 (__m512i __W, __mmask8 __U, __m256 __A) {`.
  **L404 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtps_epi64 (__m512i __W, __mmask8 __U, __m256 __A) {`。
- **L405 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,`.
  **L405 CN**: 以 `(__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) __W,`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) __W,`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L408 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L408 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。

### Lines 409-432

````c
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtps_epi64 (__mmask8 __U, __m256 __A) {
  return (__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,
                (__v8di) _mm512_setzero_si512(),
                (__mmask8) __U,
                _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundps_epi64(A, R) \
  ((__m512i)__builtin_ia32_cvtps2qq512_mask((__v8sf)(__m256)(A), \
                                            (__v8di)_mm512_setzero_si512(), \
                                            (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvt_roundps_epi64(W, U, A, R) \
  ((__m512i)__builtin_ia32_cvtps2qq512_mask((__v8sf)(__m256)(A), \
                                            (__v8di)(__m512i)(W), \
                                            (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundps_epi64(U, A, R) \
  ((__m512i)__builtin_ia32_cvtps2qq512_mask((__v8sf)(__m256)(A), \
                                            (__v8di)_mm512_setzero_si512(), \
                                            (__mmask8)(U), (int)(R)))
````
- **L409 EN**: Closes the current lexical scope or compound statement.
  **L409 CN**: 结束当前词法作用域或复合语句块。
- **L410 EN**: Blank line separating nearby declarations or logic blocks.
  **L410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L411 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L411 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L412 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtps_epi64 (__mmask8 __U, __m256 __A) {`.
  **L412 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtps_epi64 (__mmask8 __U, __m256 __A) {`。
- **L413 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,`.
  **L413 CN**: 以 `(__m512i) __builtin_ia32_cvtps2qq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L416 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L416 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L419 EN**: Defines macro `_mm512_cvt_roundps_epi64(A, R)` for conditional compilation, shorthand, or API generation.
  **L419 CN**: 定义宏 `_mm512_cvt_roundps_epi64(A, R)`，用于条件编译、简写或 API 生成。
- **L420 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtps2qq512_mask`.
  **L420 CN**: 继续与可调用符号 `__builtin_ia32_cvtps2qq512_mask` 相关的逻辑。
- **L421 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L421 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L422 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L422 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L423 EN**: Blank line separating nearby declarations or logic blocks.
  **L423 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L424 EN**: Defines macro `_mm512_mask_cvt_roundps_epi64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L424 CN**: 定义宏 `_mm512_mask_cvt_roundps_epi64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L425 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtps2qq512_mask`.
  **L425 CN**: 继续与可调用符号 `__builtin_ia32_cvtps2qq512_mask` 相关的逻辑。
- **L426 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W), \`.
  **L426 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W), \`。
- **L427 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L427 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L428 EN**: Blank line separating nearby declarations or logic blocks.
  **L428 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L429 EN**: Defines macro `_mm512_maskz_cvt_roundps_epi64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L429 CN**: 定义宏 `_mm512_maskz_cvt_roundps_epi64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L430 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtps2qq512_mask`.
  **L430 CN**: 继续与可调用符号 `__builtin_ia32_cvtps2qq512_mask` 相关的逻辑。
- **L431 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L431 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L432 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L432 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。

### Lines 433-456

````c

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtps_epu64 (__m256 __A) {
  return (__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,
                 (__v8di) _mm512_setzero_si512(),
                 (__mmask8) -1,
                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtps_epu64 (__m512i __W, __mmask8 __U, __m256 __A) {
  return (__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,
                 (__v8di) __W,
                 (__mmask8) __U,
                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtps_epu64 (__mmask8 __U, __m256 __A) {
  return (__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,
                 (__v8di) _mm512_setzero_si512(),
                 (__mmask8) __U,
                 _MM_FROUND_CUR_DIRECTION);
}
````
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L434 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L434 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L435 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtps_epu64 (__m256 __A) {`.
  **L435 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtps_epu64 (__m256 __A) {`。
- **L436 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,`.
  **L436 CN**: 以 `(__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L439 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L439 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L440 EN**: Closes the current lexical scope or compound statement.
  **L440 CN**: 结束当前词法作用域或复合语句块。
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L442 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L442 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L443 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtps_epu64 (__m512i __W, __mmask8 __U, __m256 __A) {`.
  **L443 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtps_epu64 (__m512i __W, __mmask8 __U, __m256 __A) {`。
- **L444 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,`.
  **L444 CN**: 以 `(__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) __W,`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) __W,`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L447 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L447 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L448 EN**: Closes the current lexical scope or compound statement.
  **L448 CN**: 结束当前词法作用域或复合语句块。
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L450 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L450 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L451 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtps_epu64 (__mmask8 __U, __m256 __A) {`.
  **L451 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtps_epu64 (__mmask8 __U, __m256 __A) {`。
- **L452 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,`.
  **L452 CN**: 以 `(__m512i) __builtin_ia32_cvtps2uqq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L454 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L455 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L455 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L456 EN**: Closes the current lexical scope or compound statement.
  **L456 CN**: 结束当前词法作用域或复合语句块。

### Lines 457-480

````c

#define _mm512_cvt_roundps_epu64(A, R) \
  ((__m512i)__builtin_ia32_cvtps2uqq512_mask((__v8sf)(__m256)(A), \
                                             (__v8di)_mm512_setzero_si512(), \
                                             (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvt_roundps_epu64(W, U, A, R) \
  ((__m512i)__builtin_ia32_cvtps2uqq512_mask((__v8sf)(__m256)(A), \
                                             (__v8di)(__m512i)(W), \
                                             (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundps_epu64(U, A, R) \
  ((__m512i)__builtin_ia32_cvtps2uqq512_mask((__v8sf)(__m256)(A), \
                                             (__v8di)_mm512_setzero_si512(), \
                                             (__mmask8)(U), (int)(R)))

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_cvtepi64_pd(__m512i __A) {
  return (__m512d)__builtin_convertvector((__v8di)__A, __v8df);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_cvtepi64_pd(__m512d __W, __mmask8 __U, __m512i __A) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
````
- **L457 EN**: Blank line separating nearby declarations or logic blocks.
  **L457 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L458 EN**: Defines macro `_mm512_cvt_roundps_epu64(A, R)` for conditional compilation, shorthand, or API generation.
  **L458 CN**: 定义宏 `_mm512_cvt_roundps_epu64(A, R)`，用于条件编译、简写或 API 生成。
- **L459 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtps2uqq512_mask`.
  **L459 CN**: 继续与可调用符号 `__builtin_ia32_cvtps2uqq512_mask` 相关的逻辑。
- **L460 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L460 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L461 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L461 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L463 EN**: Defines macro `_mm512_mask_cvt_roundps_epu64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L463 CN**: 定义宏 `_mm512_mask_cvt_roundps_epu64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L464 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtps2uqq512_mask`.
  **L464 CN**: 继续与可调用符号 `__builtin_ia32_cvtps2uqq512_mask` 相关的逻辑。
- **L465 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W), \`.
  **L465 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W), \`。
- **L466 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L466 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Defines macro `_mm512_maskz_cvt_roundps_epu64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L468 CN**: 定义宏 `_mm512_maskz_cvt_roundps_epu64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L469 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtps2uqq512_mask`.
  **L469 CN**: 继续与可调用符号 `__builtin_ia32_cvtps2uqq512_mask` 相关的逻辑。
- **L470 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L470 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L471 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L471 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L473 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L473 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L474 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepi64_pd(__m512i __A) {`.
  **L474 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepi64_pd(__m512i __A) {`。
- **L475 EN**: Returns from the current function with `(__m512d)__builtin_convertvector((__v8di)__A, __v8df)`.
  **L475 CN**: 以 `(__m512d)__builtin_convertvector((__v8di)__A, __v8df)` 从当前函数返回。
- **L476 EN**: Closes the current lexical scope or compound statement.
  **L476 CN**: 结束当前词法作用域或复合语句块。
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L478 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L478 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L479 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepi64_pd(__m512d __W, __mmask8 __U, __m512i __A) {`.
  **L479 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepi64_pd(__m512d __W, __mmask8 __U, __m512i __A) {`。
- **L480 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L480 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。

### Lines 481-504

````c
                                              (__v8df)_mm512_cvtepi64_pd(__A),
                                              (__v8df)__W);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_cvtepi64_pd(__mmask8 __U, __m512i __A) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_cvtepi64_pd(__A),
                                              (__v8df)_mm512_setzero_pd());
}

#define _mm512_cvt_roundepi64_pd(A, R) \
  ((__m512d)__builtin_ia32_cvtqq2pd512_mask((__v8di)(__m512i)(A), \
                                            (__v8df)_mm512_setzero_pd(), \
                                            (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvt_roundepi64_pd(W, U, A, R) \
  ((__m512d)__builtin_ia32_cvtqq2pd512_mask((__v8di)(__m512i)(A), \
                                            (__v8df)(__m512d)(W), \
                                            (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundepi64_pd(U, A, R) \
  ((__m512d)__builtin_ia32_cvtqq2pd512_mask((__v8di)(__m512i)(A), \
                                            (__v8df)_mm512_setzero_pd(), \
````
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_cvtepi64_pd(__A),`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_cvtepi64_pd(__A),`。
- **L482 EN**: Executes a call or declaration centered on `statement`.
  **L482 CN**: 执行以 `statement` 为核心的调用或声明。
- **L483 EN**: Closes the current lexical scope or compound statement.
  **L483 CN**: 结束当前词法作用域或复合语句块。
- **L484 EN**: Blank line separating nearby declarations or logic blocks.
  **L484 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L485 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L485 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L486 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepi64_pd(__mmask8 __U, __m512i __A) {`.
  **L486 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepi64_pd(__mmask8 __U, __m512i __A) {`。
- **L487 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L487 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_cvtepi64_pd(__A),`.
  **L488 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_cvtepi64_pd(__A),`。
- **L489 EN**: Executes a call or declaration centered on `statement`.
  **L489 CN**: 执行以 `statement` 为核心的调用或声明。
- **L490 EN**: Closes the current lexical scope or compound statement.
  **L490 CN**: 结束当前词法作用域或复合语句块。
- **L491 EN**: Blank line separating nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L492 EN**: Defines macro `_mm512_cvt_roundepi64_pd(A, R)` for conditional compilation, shorthand, or API generation.
  **L492 CN**: 定义宏 `_mm512_cvt_roundepi64_pd(A, R)`，用于条件编译、简写或 API 生成。
- **L493 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtqq2pd512_mask`.
  **L493 CN**: 继续与可调用符号 `__builtin_ia32_cvtqq2pd512_mask` 相关的逻辑。
- **L494 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L494 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L495 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L495 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L496 EN**: Blank line separating nearby declarations or logic blocks.
  **L496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L497 EN**: Defines macro `_mm512_mask_cvt_roundepi64_pd(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L497 CN**: 定义宏 `_mm512_mask_cvt_roundepi64_pd(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L498 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtqq2pd512_mask`.
  **L498 CN**: 继续与可调用符号 `__builtin_ia32_cvtqq2pd512_mask` 相关的逻辑。
- **L499 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(W), \`.
  **L499 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(W), \`。
- **L500 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L500 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L501 EN**: Blank line separating nearby declarations or logic blocks.
  **L501 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L502 EN**: Defines macro `_mm512_maskz_cvt_roundepi64_pd(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L502 CN**: 定义宏 `_mm512_maskz_cvt_roundepi64_pd(U, A, R)`，用于条件编译、简写或 API 生成。
- **L503 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtqq2pd512_mask`.
  **L503 CN**: 继续与可调用符号 `__builtin_ia32_cvtqq2pd512_mask` 相关的逻辑。
- **L504 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L504 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。

### Lines 505-528

````c
                                            (__mmask8)(U), (int)(R)))

static __inline__ __m256 __DEFAULT_FN_ATTRS512
_mm512_cvtepi64_ps (__m512i __A) {
  return (__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,
               (__v8sf) _mm256_setzero_ps(),
               (__mmask8) -1,
               _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS512
_mm512_mask_cvtepi64_ps (__m256 __W, __mmask8 __U, __m512i __A) {
  return (__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,
               (__v8sf) __W,
               (__mmask8) __U,
               _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtepi64_ps (__mmask8 __U, __m512i __A) {
  return (__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,
               (__v8sf) _mm256_setzero_ps(),
               (__mmask8) __U,
               _MM_FROUND_CUR_DIRECTION);
````
- **L505 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L505 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L507 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS512`.
  **L507 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS512`。
- **L508 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepi64_ps (__m512i __A) {`.
  **L508 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepi64_ps (__m512i __A) {`。
- **L509 EN**: Returns from the current function with `(__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,`.
  **L509 CN**: 以 `(__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,` 从当前函数返回。
- **L510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf) _mm256_setzero_ps(),`.
  **L510 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf) _mm256_setzero_ps(),`。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L512 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L512 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L513 EN**: Closes the current lexical scope or compound statement.
  **L513 CN**: 结束当前词法作用域或复合语句块。
- **L514 EN**: Blank line separating nearby declarations or logic blocks.
  **L514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L515 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS512`.
  **L515 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS512`。
- **L516 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepi64_ps (__m256 __W, __mmask8 __U, __m512i __A) {`.
  **L516 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepi64_ps (__m256 __W, __mmask8 __U, __m512i __A) {`。
- **L517 EN**: Returns from the current function with `(__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,`.
  **L517 CN**: 以 `(__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,` 从当前函数返回。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf) __W,`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf) __W,`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L520 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L520 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L521 EN**: Closes the current lexical scope or compound statement.
  **L521 CN**: 结束当前词法作用域或复合语句块。
- **L522 EN**: Blank line separating nearby declarations or logic blocks.
  **L522 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L523 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS512`.
  **L523 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS512`。
- **L524 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepi64_ps (__mmask8 __U, __m512i __A) {`.
  **L524 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepi64_ps (__mmask8 __U, __m512i __A) {`。
- **L525 EN**: Returns from the current function with `(__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,`.
  **L525 CN**: 以 `(__m256) __builtin_ia32_cvtqq2ps512_mask ((__v8di) __A,` 从当前函数返回。
- **L526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf) _mm256_setzero_ps(),`.
  **L526 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf) _mm256_setzero_ps(),`。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L528 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L528 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。

### Lines 529-552

````c
}

#define _mm512_cvt_roundepi64_ps(A, R) \
  ((__m256)__builtin_ia32_cvtqq2ps512_mask((__v8di)(__m512i)(A), \
                                           (__v8sf)_mm256_setzero_ps(), \
                                           (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvt_roundepi64_ps(W, U, A, R) \
  ((__m256)__builtin_ia32_cvtqq2ps512_mask((__v8di)(__m512i)(A), \
                                           (__v8sf)(__m256)(W), (__mmask8)(U), \
                                           (int)(R)))

#define _mm512_maskz_cvt_roundepi64_ps(U, A, R) \
  ((__m256)__builtin_ia32_cvtqq2ps512_mask((__v8di)(__m512i)(A), \
                                           (__v8sf)_mm256_setzero_ps(), \
                                           (__mmask8)(U), (int)(R)))


static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttpd_epi64 (__m512d __A) {
  return (__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,
                 (__v8di) _mm512_setzero_si512(),
                 (__mmask8) -1,
                 _MM_FROUND_CUR_DIRECTION);
````
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L531 EN**: Defines macro `_mm512_cvt_roundepi64_ps(A, R)` for conditional compilation, shorthand, or API generation.
  **L531 CN**: 定义宏 `_mm512_cvt_roundepi64_ps(A, R)`，用于条件编译、简写或 API 生成。
- **L532 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtqq2ps512_mask`.
  **L532 CN**: 继续与可调用符号 `__builtin_ia32_cvtqq2ps512_mask` 相关的逻辑。
- **L533 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L533 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L534 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L534 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L535 EN**: Blank line separating nearby declarations or logic blocks.
  **L535 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L536 EN**: Defines macro `_mm512_mask_cvt_roundepi64_ps(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L536 CN**: 定义宏 `_mm512_mask_cvt_roundepi64_ps(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L537 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtqq2ps512_mask`.
  **L537 CN**: 继续与可调用符号 `__builtin_ia32_cvtqq2ps512_mask` 相关的逻辑。
- **L538 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(W), (__mmask8)(U), \`.
  **L538 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(W), (__mmask8)(U), \`。
- **L539 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L539 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L540 EN**: Blank line separating nearby declarations or logic blocks.
  **L540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L541 EN**: Defines macro `_mm512_maskz_cvt_roundepi64_ps(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L541 CN**: 定义宏 `_mm512_maskz_cvt_roundepi64_ps(U, A, R)`，用于条件编译、简写或 API 生成。
- **L542 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtqq2ps512_mask`.
  **L542 CN**: 继续与可调用符号 `__builtin_ia32_cvtqq2ps512_mask` 相关的逻辑。
- **L543 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L543 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L544 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L544 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L546 EN**: Blank line separating nearby declarations or logic blocks.
  **L546 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L547 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L547 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L548 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttpd_epi64 (__m512d __A) {`.
  **L548 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttpd_epi64 (__m512d __A) {`。
- **L549 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,`.
  **L549 CN**: 以 `(__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,` 从当前函数返回。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L552 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L552 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。

### Lines 553-576

````c
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttpd_epi64 (__m512i __W, __mmask8 __U, __m512d __A) {
  return (__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,
                 (__v8di) __W,
                 (__mmask8) __U,
                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttpd_epi64 (__mmask8 __U, __m512d __A) {
  return (__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,
                 (__v8di) _mm512_setzero_si512(),
                 (__mmask8) __U,
                 _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvtt_roundpd_epi64(A, R) \
  ((__m512i)__builtin_ia32_cvttpd2qq512_mask((__v8df)(__m512d)(A), \
                                             (__v8di)_mm512_setzero_si512(), \
                                             (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvtt_roundpd_epi64(W, U, A, R) \
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L555 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L556 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttpd_epi64 (__m512i __W, __mmask8 __U, __m512d __A) {`.
  **L556 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttpd_epi64 (__m512i __W, __mmask8 __U, __m512d __A) {`。
- **L557 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,`.
  **L557 CN**: 以 `(__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,` 从当前函数返回。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) __W,`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) __W,`。
- **L559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L559 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L560 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L560 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L561 EN**: Closes the current lexical scope or compound statement.
  **L561 CN**: 结束当前词法作用域或复合语句块。
- **L562 EN**: Blank line separating nearby declarations or logic blocks.
  **L562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L563 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L563 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L564 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttpd_epi64 (__mmask8 __U, __m512d __A) {`.
  **L564 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttpd_epi64 (__mmask8 __U, __m512d __A) {`。
- **L565 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,`.
  **L565 CN**: 以 `(__m512i) __builtin_ia32_cvttpd2qq512_mask ((__v8df) __A,` 从当前函数返回。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L567 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L568 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L568 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L569 EN**: Closes the current lexical scope or compound statement.
  **L569 CN**: 结束当前词法作用域或复合语句块。
- **L570 EN**: Blank line separating nearby declarations or logic blocks.
  **L570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L571 EN**: Defines macro `_mm512_cvtt_roundpd_epi64(A, R)` for conditional compilation, shorthand, or API generation.
  **L571 CN**: 定义宏 `_mm512_cvtt_roundpd_epi64(A, R)`，用于条件编译、简写或 API 生成。
- **L572 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttpd2qq512_mask`.
  **L572 CN**: 继续与可调用符号 `__builtin_ia32_cvttpd2qq512_mask` 相关的逻辑。
- **L573 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L573 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L574 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L574 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L575 EN**: Blank line separating nearby declarations or logic blocks.
  **L575 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L576 EN**: Defines macro `_mm512_mask_cvtt_roundpd_epi64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L576 CN**: 定义宏 `_mm512_mask_cvtt_roundpd_epi64(W, U, A, R)`，用于条件编译、简写或 API 生成。

### Lines 577-600

````c
  ((__m512i)__builtin_ia32_cvttpd2qq512_mask((__v8df)(__m512d)(A), \
                                             (__v8di)(__m512i)(W), \
                                             (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvtt_roundpd_epi64(U, A, R) \
  ((__m512i)__builtin_ia32_cvttpd2qq512_mask((__v8df)(__m512d)(A), \
                                             (__v8di)_mm512_setzero_si512(), \
                                             (__mmask8)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttpd_epu64 (__m512d __A) {
  return (__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,
                  (__v8di) _mm512_setzero_si512(),
                  (__mmask8) -1,
                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttpd_epu64 (__m512i __W, __mmask8 __U, __m512d __A) {
  return (__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,
                  (__v8di) __W,
                  (__mmask8) __U,
                  _MM_FROUND_CUR_DIRECTION);
}
````
- **L577 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttpd2qq512_mask`.
  **L577 CN**: 继续与可调用符号 `__builtin_ia32_cvttpd2qq512_mask` 相关的逻辑。
- **L578 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W), \`.
  **L578 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W), \`。
- **L579 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L579 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L580 EN**: Blank line separating nearby declarations or logic blocks.
  **L580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L581 EN**: Defines macro `_mm512_maskz_cvtt_roundpd_epi64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L581 CN**: 定义宏 `_mm512_maskz_cvtt_roundpd_epi64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L582 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttpd2qq512_mask`.
  **L582 CN**: 继续与可调用符号 `__builtin_ia32_cvttpd2qq512_mask` 相关的逻辑。
- **L583 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L583 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L584 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L584 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L585 EN**: Blank line separating nearby declarations or logic blocks.
  **L585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L586 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L586 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L587 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttpd_epu64 (__m512d __A) {`.
  **L587 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttpd_epu64 (__m512d __A) {`。
- **L588 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,`.
  **L588 CN**: 以 `(__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,` 从当前函数返回。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L591 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L591 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L592 EN**: Closes the current lexical scope or compound statement.
  **L592 CN**: 结束当前词法作用域或复合语句块。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L594 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L595 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttpd_epu64 (__m512i __W, __mmask8 __U, __m512d __A) {`.
  **L595 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttpd_epu64 (__m512i __W, __mmask8 __U, __m512d __A) {`。
- **L596 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,`.
  **L596 CN**: 以 `(__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,` 从当前函数返回。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) __W,`.
  **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) __W,`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L599 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L599 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L600 EN**: Closes the current lexical scope or compound statement.
  **L600 CN**: 结束当前词法作用域或复合语句块。

### Lines 601-624

````c

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttpd_epu64 (__mmask8 __U, __m512d __A) {
  return (__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,
                  (__v8di) _mm512_setzero_si512(),
                  (__mmask8) __U,
                  _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvtt_roundpd_epu64(A, R) \
  ((__m512i)__builtin_ia32_cvttpd2uqq512_mask((__v8df)(__m512d)(A), \
                                              (__v8di)_mm512_setzero_si512(), \
                                              (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvtt_roundpd_epu64(W, U, A, R) \
  ((__m512i)__builtin_ia32_cvttpd2uqq512_mask((__v8df)(__m512d)(A), \
                                              (__v8di)(__m512i)(W), \
                                              (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvtt_roundpd_epu64(U, A, R) \
  ((__m512i)__builtin_ia32_cvttpd2uqq512_mask((__v8df)(__m512d)(A), \
                                              (__v8di)_mm512_setzero_si512(), \
                                              (__mmask8)(U), (int)(R)))

````
- **L601 EN**: Blank line separating nearby declarations or logic blocks.
  **L601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L602 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L602 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L603 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttpd_epu64 (__mmask8 __U, __m512d __A) {`.
  **L603 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttpd_epu64 (__mmask8 __U, __m512d __A) {`。
- **L604 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,`.
  **L604 CN**: 以 `(__m512i) __builtin_ia32_cvttpd2uqq512_mask ((__v8df) __A,` 从当前函数返回。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L607 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L607 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L608 EN**: Closes the current lexical scope or compound statement.
  **L608 CN**: 结束当前词法作用域或复合语句块。
- **L609 EN**: Blank line separating nearby declarations or logic blocks.
  **L609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L610 EN**: Defines macro `_mm512_cvtt_roundpd_epu64(A, R)` for conditional compilation, shorthand, or API generation.
  **L610 CN**: 定义宏 `_mm512_cvtt_roundpd_epu64(A, R)`，用于条件编译、简写或 API 生成。
- **L611 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttpd2uqq512_mask`.
  **L611 CN**: 继续与可调用符号 `__builtin_ia32_cvttpd2uqq512_mask` 相关的逻辑。
- **L612 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L612 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L613 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L613 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L615 EN**: Defines macro `_mm512_mask_cvtt_roundpd_epu64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L615 CN**: 定义宏 `_mm512_mask_cvtt_roundpd_epu64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L616 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttpd2uqq512_mask`.
  **L616 CN**: 继续与可调用符号 `__builtin_ia32_cvttpd2uqq512_mask` 相关的逻辑。
- **L617 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W), \`.
  **L617 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W), \`。
- **L618 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L618 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L619 EN**: Blank line separating nearby declarations or logic blocks.
  **L619 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L620 EN**: Defines macro `_mm512_maskz_cvtt_roundpd_epu64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L620 CN**: 定义宏 `_mm512_maskz_cvtt_roundpd_epu64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L621 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttpd2uqq512_mask`.
  **L621 CN**: 继续与可调用符号 `__builtin_ia32_cvttpd2uqq512_mask` 相关的逻辑。
- **L622 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L622 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L623 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L623 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 625-648

````c
static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttps_epi64 (__m256 __A) {
  return (__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,
                 (__v8di) _mm512_setzero_si512(),
                 (__mmask8) -1,
                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttps_epi64 (__m512i __W, __mmask8 __U, __m256 __A) {
  return (__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,
                 (__v8di) __W,
                 (__mmask8) __U,
                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttps_epi64 (__mmask8 __U, __m256 __A) {
  return (__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,
                 (__v8di) _mm512_setzero_si512(),
                 (__mmask8) __U,
                 _MM_FROUND_CUR_DIRECTION);
}

````
- **L625 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L625 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L626 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttps_epi64 (__m256 __A) {`.
  **L626 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttps_epi64 (__m256 __A) {`。
- **L627 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,`.
  **L627 CN**: 以 `(__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L630 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L630 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L633 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L634 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttps_epi64 (__m512i __W, __mmask8 __U, __m256 __A) {`.
  **L634 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttps_epi64 (__m512i __W, __mmask8 __U, __m256 __A) {`。
- **L635 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,`.
  **L635 CN**: 以 `(__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) __W,`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) __W,`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L638 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L638 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L639 EN**: Closes the current lexical scope or compound statement.
  **L639 CN**: 结束当前词法作用域或复合语句块。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L641 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L641 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L642 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttps_epi64 (__mmask8 __U, __m256 __A) {`.
  **L642 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttps_epi64 (__mmask8 __U, __m256 __A) {`。
- **L643 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,`.
  **L643 CN**: 以 `(__m512i) __builtin_ia32_cvttps2qq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L646 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L646 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L647 EN**: Closes the current lexical scope or compound statement.
  **L647 CN**: 结束当前词法作用域或复合语句块。
- **L648 EN**: Blank line separating nearby declarations or logic blocks.
  **L648 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 649-672

````c
#define _mm512_cvtt_roundps_epi64(A, R) \
  ((__m512i)__builtin_ia32_cvttps2qq512_mask((__v8sf)(__m256)(A), \
                                             (__v8di)_mm512_setzero_si512(), \
                                             (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvtt_roundps_epi64(W, U, A, R) \
  ((__m512i)__builtin_ia32_cvttps2qq512_mask((__v8sf)(__m256)(A), \
                                             (__v8di)(__m512i)(W), \
                                             (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvtt_roundps_epi64(U, A, R) \
  ((__m512i)__builtin_ia32_cvttps2qq512_mask((__v8sf)(__m256)(A), \
                                             (__v8di)_mm512_setzero_si512(), \
                                             (__mmask8)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttps_epu64 (__m256 __A) {
  return (__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,
                  (__v8di) _mm512_setzero_si512(),
                  (__mmask8) -1,
                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
````
- **L649 EN**: Defines macro `_mm512_cvtt_roundps_epi64(A, R)` for conditional compilation, shorthand, or API generation.
  **L649 CN**: 定义宏 `_mm512_cvtt_roundps_epi64(A, R)`，用于条件编译、简写或 API 生成。
- **L650 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttps2qq512_mask`.
  **L650 CN**: 继续与可调用符号 `__builtin_ia32_cvttps2qq512_mask` 相关的逻辑。
- **L651 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L651 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L652 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L652 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Defines macro `_mm512_mask_cvtt_roundps_epi64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L654 CN**: 定义宏 `_mm512_mask_cvtt_roundps_epi64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L655 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttps2qq512_mask`.
  **L655 CN**: 继续与可调用符号 `__builtin_ia32_cvttps2qq512_mask` 相关的逻辑。
- **L656 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W), \`.
  **L656 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W), \`。
- **L657 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L657 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L658 EN**: Blank line separating nearby declarations or logic blocks.
  **L658 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L659 EN**: Defines macro `_mm512_maskz_cvtt_roundps_epi64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L659 CN**: 定义宏 `_mm512_maskz_cvtt_roundps_epi64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L660 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttps2qq512_mask`.
  **L660 CN**: 继续与可调用符号 `__builtin_ia32_cvttps2qq512_mask` 相关的逻辑。
- **L661 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L661 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L662 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L662 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L664 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L664 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L665 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttps_epu64 (__m256 __A) {`.
  **L665 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttps_epu64 (__m256 __A) {`。
- **L666 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,`.
  **L666 CN**: 以 `(__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L667 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L669 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L669 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L670 EN**: Closes the current lexical scope or compound statement.
  **L670 CN**: 结束当前词法作用域或复合语句块。
- **L671 EN**: Blank line separating nearby declarations or logic blocks.
  **L671 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L672 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L672 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。

### Lines 673-696

````c
_mm512_mask_cvttps_epu64 (__m512i __W, __mmask8 __U, __m256 __A) {
  return (__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,
                  (__v8di) __W,
                  (__mmask8) __U,
                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttps_epu64 (__mmask8 __U, __m256 __A) {
  return (__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,
                  (__v8di) _mm512_setzero_si512(),
                  (__mmask8) __U,
                  _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvtt_roundps_epu64(A, R) \
  ((__m512i)__builtin_ia32_cvttps2uqq512_mask((__v8sf)(__m256)(A), \
                                              (__v8di)_mm512_setzero_si512(), \
                                              (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvtt_roundps_epu64(W, U, A, R) \
  ((__m512i)__builtin_ia32_cvttps2uqq512_mask((__v8sf)(__m256)(A), \
                                              (__v8di)(__m512i)(W), \
                                              (__mmask8)(U), (int)(R)))
````
- **L673 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttps_epu64 (__m512i __W, __mmask8 __U, __m256 __A) {`.
  **L673 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttps_epu64 (__m512i __W, __mmask8 __U, __m256 __A) {`。
- **L674 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,`.
  **L674 CN**: 以 `(__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) __W,`.
  **L675 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) __W,`。
- **L676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L676 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L677 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L677 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L678 EN**: Closes the current lexical scope or compound statement.
  **L678 CN**: 结束当前词法作用域或复合语句块。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L680 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L680 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L681 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttps_epu64 (__mmask8 __U, __m256 __A) {`.
  **L681 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttps_epu64 (__mmask8 __U, __m256 __A) {`。
- **L682 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,`.
  **L682 CN**: 以 `(__m512i) __builtin_ia32_cvttps2uqq512_mask ((__v8sf) __A,` 从当前函数返回。
- **L683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di) _mm512_setzero_si512(),`.
  **L683 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di) _mm512_setzero_si512(),`。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L685 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L685 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L686 EN**: Closes the current lexical scope or compound statement.
  **L686 CN**: 结束当前词法作用域或复合语句块。
- **L687 EN**: Blank line separating nearby declarations or logic blocks.
  **L687 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L688 EN**: Defines macro `_mm512_cvtt_roundps_epu64(A, R)` for conditional compilation, shorthand, or API generation.
  **L688 CN**: 定义宏 `_mm512_cvtt_roundps_epu64(A, R)`，用于条件编译、简写或 API 生成。
- **L689 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttps2uqq512_mask`.
  **L689 CN**: 继续与可调用符号 `__builtin_ia32_cvttps2uqq512_mask` 相关的逻辑。
- **L690 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L690 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L691 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L691 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L692 EN**: Blank line separating nearby declarations or logic blocks.
  **L692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L693 EN**: Defines macro `_mm512_mask_cvtt_roundps_epu64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L693 CN**: 定义宏 `_mm512_mask_cvtt_roundps_epu64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L694 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttps2uqq512_mask`.
  **L694 CN**: 继续与可调用符号 `__builtin_ia32_cvttps2uqq512_mask` 相关的逻辑。
- **L695 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W), \`.
  **L695 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W), \`。
- **L696 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L696 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。

### Lines 697-720

````c

#define _mm512_maskz_cvtt_roundps_epu64(U, A, R) \
  ((__m512i)__builtin_ia32_cvttps2uqq512_mask((__v8sf)(__m256)(A), \
                                              (__v8di)_mm512_setzero_si512(), \
                                              (__mmask8)(U), (int)(R)))

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_cvtepu64_pd(__m512i __A) {
  return (__m512d)__builtin_convertvector((__v8du)__A, __v8df);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_cvtepu64_pd(__m512d __W, __mmask8 __U, __m512i __A) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_cvtepu64_pd(__A),
                                              (__v8df)__W);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_cvtepu64_pd(__mmask8 __U, __m512i __A) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,
                                              (__v8df)_mm512_cvtepu64_pd(__A),
                                              (__v8df)_mm512_setzero_pd());
}
````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L698 EN**: Defines macro `_mm512_maskz_cvtt_roundps_epu64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L698 CN**: 定义宏 `_mm512_maskz_cvtt_roundps_epu64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L699 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvttps2uqq512_mask`.
  **L699 CN**: 继续与可调用符号 `__builtin_ia32_cvttps2uqq512_mask` 相关的逻辑。
- **L700 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L700 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L701 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L701 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L702 EN**: Blank line separating nearby declarations or logic blocks.
  **L702 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L703 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L703 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L704 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepu64_pd(__m512i __A) {`.
  **L704 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepu64_pd(__m512i __A) {`。
- **L705 EN**: Returns from the current function with `(__m512d)__builtin_convertvector((__v8du)__A, __v8df)`.
  **L705 CN**: 以 `(__m512d)__builtin_convertvector((__v8du)__A, __v8df)` 从当前函数返回。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L708 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L708 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L709 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepu64_pd(__m512d __W, __mmask8 __U, __m512i __A) {`.
  **L709 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepu64_pd(__m512d __W, __mmask8 __U, __m512i __A) {`。
- **L710 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L710 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_cvtepu64_pd(__A),`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_cvtepu64_pd(__A),`。
- **L712 EN**: Executes a call or declaration centered on `statement`.
  **L712 CN**: 执行以 `statement` 为核心的调用或声明。
- **L713 EN**: Closes the current lexical scope or compound statement.
  **L713 CN**: 结束当前词法作用域或复合语句块。
- **L714 EN**: Blank line separating nearby declarations or logic blocks.
  **L714 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L715 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L715 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L716 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepu64_pd(__mmask8 __U, __m512i __A) {`.
  **L716 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepu64_pd(__mmask8 __U, __m512i __A) {`。
- **L717 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,`.
  **L717 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__U,` 从当前函数返回。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_cvtepu64_pd(__A),`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_cvtepu64_pd(__A),`。
- **L719 EN**: Executes a call or declaration centered on `statement`.
  **L719 CN**: 执行以 `statement` 为核心的调用或声明。
- **L720 EN**: Closes the current lexical scope or compound statement.
  **L720 CN**: 结束当前词法作用域或复合语句块。

### Lines 721-744

````c

#define _mm512_cvt_roundepu64_pd(A, R) \
  ((__m512d)__builtin_ia32_cvtuqq2pd512_mask((__v8di)(__m512i)(A), \
                                             (__v8df)_mm512_setzero_pd(), \
                                             (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvt_roundepu64_pd(W, U, A, R) \
  ((__m512d)__builtin_ia32_cvtuqq2pd512_mask((__v8di)(__m512i)(A), \
                                             (__v8df)(__m512d)(W), \
                                             (__mmask8)(U), (int)(R)))


#define _mm512_maskz_cvt_roundepu64_pd(U, A, R) \
  ((__m512d)__builtin_ia32_cvtuqq2pd512_mask((__v8di)(__m512i)(A), \
                                             (__v8df)_mm512_setzero_pd(), \
                                             (__mmask8)(U), (int)(R)))


static __inline__ __m256 __DEFAULT_FN_ATTRS512
_mm512_cvtepu64_ps (__m512i __A) {
  return (__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,
                (__v8sf) _mm256_setzero_ps(),
                (__mmask8) -1,
                _MM_FROUND_CUR_DIRECTION);
````
- **L721 EN**: Blank line separating nearby declarations or logic blocks.
  **L721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L722 EN**: Defines macro `_mm512_cvt_roundepu64_pd(A, R)` for conditional compilation, shorthand, or API generation.
  **L722 CN**: 定义宏 `_mm512_cvt_roundepu64_pd(A, R)`，用于条件编译、简写或 API 生成。
- **L723 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtuqq2pd512_mask`.
  **L723 CN**: 继续与可调用符号 `__builtin_ia32_cvtuqq2pd512_mask` 相关的逻辑。
- **L724 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L724 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L725 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L725 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L726 EN**: Blank line separating nearby declarations or logic blocks.
  **L726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L727 EN**: Defines macro `_mm512_mask_cvt_roundepu64_pd(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L727 CN**: 定义宏 `_mm512_mask_cvt_roundepu64_pd(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L728 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtuqq2pd512_mask`.
  **L728 CN**: 继续与可调用符号 `__builtin_ia32_cvtuqq2pd512_mask` 相关的逻辑。
- **L729 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(W), \`.
  **L729 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(W), \`。
- **L730 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L730 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L731 EN**: Blank line separating nearby declarations or logic blocks.
  **L731 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L732 EN**: Blank line separating nearby declarations or logic blocks.
  **L732 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L733 EN**: Defines macro `_mm512_maskz_cvt_roundepu64_pd(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L733 CN**: 定义宏 `_mm512_maskz_cvt_roundepu64_pd(U, A, R)`，用于条件编译、简写或 API 生成。
- **L734 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtuqq2pd512_mask`.
  **L734 CN**: 继续与可调用符号 `__builtin_ia32_cvtuqq2pd512_mask` 相关的逻辑。
- **L735 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L735 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L736 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L736 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L739 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS512`.
  **L739 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS512`。
- **L740 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepu64_ps (__m512i __A) {`.
  **L740 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepu64_ps (__m512i __A) {`。
- **L741 EN**: Returns from the current function with `(__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,`.
  **L741 CN**: 以 `(__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,` 从当前函数返回。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf) _mm256_setzero_ps(),`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf) _mm256_setzero_ps(),`。
- **L743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) -1,`.
  **L743 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) -1,`。
- **L744 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L744 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。

### Lines 745-768

````c
}

static __inline__ __m256 __DEFAULT_FN_ATTRS512
_mm512_mask_cvtepu64_ps (__m256 __W, __mmask8 __U, __m512i __A) {
  return (__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,
                (__v8sf) __W,
                (__mmask8) __U,
                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtepu64_ps (__mmask8 __U, __m512i __A) {
  return (__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,
                (__v8sf) _mm256_setzero_ps(),
                (__mmask8) __U,
                _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundepu64_ps(A, R) \
  ((__m256)__builtin_ia32_cvtuqq2ps512_mask((__v8di)(__m512i)(A), \
                                            (__v8sf)_mm256_setzero_ps(), \
                                            (__mmask8)-1, (int)(R)))

#define _mm512_mask_cvt_roundepu64_ps(W, U, A, R) \
````
- **L745 EN**: Closes the current lexical scope or compound statement.
  **L745 CN**: 结束当前词法作用域或复合语句块。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L747 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS512`.
  **L747 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS512`。
- **L748 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepu64_ps (__m256 __W, __mmask8 __U, __m512i __A) {`.
  **L748 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepu64_ps (__m256 __W, __mmask8 __U, __m512i __A) {`。
- **L749 EN**: Returns from the current function with `(__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,`.
  **L749 CN**: 以 `(__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,` 从当前函数返回。
- **L750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf) __W,`.
  **L750 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf) __W,`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L752 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L752 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L753 EN**: Closes the current lexical scope or compound statement.
  **L753 CN**: 结束当前词法作用域或复合语句块。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L755 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS512`.
  **L755 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS512`。
- **L756 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepu64_ps (__mmask8 __U, __m512i __A) {`.
  **L756 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepu64_ps (__mmask8 __U, __m512i __A) {`。
- **L757 EN**: Returns from the current function with `(__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,`.
  **L757 CN**: 以 `(__m256) __builtin_ia32_cvtuqq2ps512_mask ((__v8di) __A,` 从当前函数返回。
- **L758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf) _mm256_setzero_ps(),`.
  **L758 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf) _mm256_setzero_ps(),`。
- **L759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8) __U,`.
  **L759 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8) __U,`。
- **L760 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L760 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L761 EN**: Closes the current lexical scope or compound statement.
  **L761 CN**: 结束当前词法作用域或复合语句块。
- **L762 EN**: Blank line separating nearby declarations or logic blocks.
  **L762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L763 EN**: Defines macro `_mm512_cvt_roundepu64_ps(A, R)` for conditional compilation, shorthand, or API generation.
  **L763 CN**: 定义宏 `_mm512_cvt_roundepu64_ps(A, R)`，用于条件编译、简写或 API 生成。
- **L764 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtuqq2ps512_mask`.
  **L764 CN**: 继续与可调用符号 `__builtin_ia32_cvtuqq2ps512_mask` 相关的逻辑。
- **L765 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L765 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L766 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L766 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L767 EN**: Blank line separating nearby declarations or logic blocks.
  **L767 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L768 EN**: Defines macro `_mm512_mask_cvt_roundepu64_ps(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L768 CN**: 定义宏 `_mm512_mask_cvt_roundepu64_ps(W, U, A, R)`，用于条件编译、简写或 API 生成。

### Lines 769-792

````c
  ((__m256)__builtin_ia32_cvtuqq2ps512_mask((__v8di)(__m512i)(A), \
                                            (__v8sf)(__m256)(W), (__mmask8)(U), \
                                            (int)(R)))

#define _mm512_maskz_cvt_roundepu64_ps(U, A, R) \
  ((__m256)__builtin_ia32_cvtuqq2ps512_mask((__v8di)(__m512i)(A), \
                                            (__v8sf)_mm256_setzero_ps(), \
                                            (__mmask8)(U), (int)(R)))

#define _mm512_range_pd(A, B, C) \
  ((__m512d)__builtin_ia32_rangepd512_mask((__v8df)(__m512d)(A), \
                                           (__v8df)(__m512d)(B), (int)(C), \
                                           (__v8df)_mm512_setzero_pd(), \
                                           (__mmask8)-1, \
                                           _MM_FROUND_CUR_DIRECTION))

#define _mm512_mask_range_pd(W, U, A, B, C) \
  ((__m512d)__builtin_ia32_rangepd512_mask((__v8df)(__m512d)(A), \
                                           (__v8df)(__m512d)(B), (int)(C), \
                                           (__v8df)(__m512d)(W), (__mmask8)(U), \
                                           _MM_FROUND_CUR_DIRECTION))

#define _mm512_maskz_range_pd(U, A, B, C) \
  ((__m512d)__builtin_ia32_rangepd512_mask((__v8df)(__m512d)(A), \
````
- **L769 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtuqq2ps512_mask`.
  **L769 CN**: 继续与可调用符号 `__builtin_ia32_cvtuqq2ps512_mask` 相关的逻辑。
- **L770 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(W), (__mmask8)(U), \`.
  **L770 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(W), (__mmask8)(U), \`。
- **L771 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L771 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L773 EN**: Defines macro `_mm512_maskz_cvt_roundepu64_ps(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L773 CN**: 定义宏 `_mm512_maskz_cvt_roundepu64_ps(U, A, R)`，用于条件编译、简写或 API 生成。
- **L774 EN**: Continues logic associated with callable symbol `__builtin_ia32_cvtuqq2ps512_mask`.
  **L774 CN**: 继续与可调用符号 `__builtin_ia32_cvtuqq2ps512_mask` 相关的逻辑。
- **L775 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L775 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L776 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L776 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L777 EN**: Blank line separating nearby declarations or logic blocks.
  **L777 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L778 EN**: Defines macro `_mm512_range_pd(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L778 CN**: 定义宏 `_mm512_range_pd(A, B, C)`，用于条件编译、简写或 API 生成。
- **L779 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd512_mask`.
  **L779 CN**: 继续与可调用符号 `__builtin_ia32_rangepd512_mask` 相关的逻辑。
- **L780 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(B), (int)(C), \`.
  **L780 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(B), (int)(C), \`。
- **L781 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L781 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L782 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, \`.
  **L782 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, \`。
- **L783 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L783 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L785 EN**: Defines macro `_mm512_mask_range_pd(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L785 CN**: 定义宏 `_mm512_mask_range_pd(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L786 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd512_mask`.
  **L786 CN**: 继续与可调用符号 `__builtin_ia32_rangepd512_mask` 相关的逻辑。
- **L787 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(B), (int)(C), \`.
  **L787 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(B), (int)(C), \`。
- **L788 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(W), (__mmask8)(U), \`.
  **L788 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(W), (__mmask8)(U), \`。
- **L789 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L789 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L790 EN**: Blank line separating nearby declarations or logic blocks.
  **L790 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L791 EN**: Defines macro `_mm512_maskz_range_pd(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L791 CN**: 定义宏 `_mm512_maskz_range_pd(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L792 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd512_mask`.
  **L792 CN**: 继续与可调用符号 `__builtin_ia32_rangepd512_mask` 相关的逻辑。

### Lines 793-816

````c
                                           (__v8df)(__m512d)(B), (int)(C), \
                                           (__v8df)_mm512_setzero_pd(), \
                                           (__mmask8)(U), \
                                           _MM_FROUND_CUR_DIRECTION))

#define _mm512_range_round_pd(A, B, C, R) \
  ((__m512d)__builtin_ia32_rangepd512_mask((__v8df)(__m512d)(A), \
                                           (__v8df)(__m512d)(B), (int)(C), \
                                           (__v8df)_mm512_setzero_pd(), \
                                           (__mmask8)-1, (int)(R)))

#define _mm512_mask_range_round_pd(W, U, A, B, C, R) \
  ((__m512d)__builtin_ia32_rangepd512_mask((__v8df)(__m512d)(A), \
                                           (__v8df)(__m512d)(B), (int)(C), \
                                           (__v8df)(__m512d)(W), (__mmask8)(U), \
                                           (int)(R)))

#define _mm512_maskz_range_round_pd(U, A, B, C, R) \
  ((__m512d)__builtin_ia32_rangepd512_mask((__v8df)(__m512d)(A), \
                                           (__v8df)(__m512d)(B), (int)(C), \
                                           (__v8df)_mm512_setzero_pd(), \
                                           (__mmask8)(U), (int)(R)))

#define _mm512_range_ps(A, B, C) \
````
- **L793 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(B), (int)(C), \`.
  **L793 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(B), (int)(C), \`。
- **L794 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L794 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L795 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), \`.
  **L795 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), \`。
- **L796 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L796 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L797 EN**: Blank line separating nearby declarations or logic blocks.
  **L797 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L798 EN**: Defines macro `_mm512_range_round_pd(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L798 CN**: 定义宏 `_mm512_range_round_pd(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L799 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd512_mask`.
  **L799 CN**: 继续与可调用符号 `__builtin_ia32_rangepd512_mask` 相关的逻辑。
- **L800 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(B), (int)(C), \`.
  **L800 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(B), (int)(C), \`。
- **L801 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L801 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L802 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L802 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L803 EN**: Blank line separating nearby declarations or logic blocks.
  **L803 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L804 EN**: Defines macro `_mm512_mask_range_round_pd(W, U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L804 CN**: 定义宏 `_mm512_mask_range_round_pd(W, U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L805 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd512_mask`.
  **L805 CN**: 继续与可调用符号 `__builtin_ia32_rangepd512_mask` 相关的逻辑。
- **L806 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(B), (int)(C), \`.
  **L806 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(B), (int)(C), \`。
- **L807 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(W), (__mmask8)(U), \`.
  **L807 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(W), (__mmask8)(U), \`。
- **L808 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L808 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L809 EN**: Blank line separating nearby declarations or logic blocks.
  **L809 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L810 EN**: Defines macro `_mm512_maskz_range_round_pd(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L810 CN**: 定义宏 `_mm512_maskz_range_round_pd(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L811 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd512_mask`.
  **L811 CN**: 继续与可调用符号 `__builtin_ia32_rangepd512_mask` 相关的逻辑。
- **L812 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(B), (int)(C), \`.
  **L812 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(B), (int)(C), \`。
- **L813 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L813 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L814 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L814 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L815 EN**: Blank line separating nearby declarations or logic blocks.
  **L815 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L816 EN**: Defines macro `_mm512_range_ps(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L816 CN**: 定义宏 `_mm512_range_ps(A, B, C)`，用于条件编译、简写或 API 生成。

### Lines 817-840

````c
  ((__m512)__builtin_ia32_rangeps512_mask((__v16sf)(__m512)(A), \
                                          (__v16sf)(__m512)(B), (int)(C), \
                                          (__v16sf)_mm512_setzero_ps(), \
                                          (__mmask16)-1, \
                                          _MM_FROUND_CUR_DIRECTION))

#define _mm512_mask_range_ps(W, U, A, B, C) \
  ((__m512)__builtin_ia32_rangeps512_mask((__v16sf)(__m512)(A), \
                                          (__v16sf)(__m512)(B), (int)(C), \
                                          (__v16sf)(__m512)(W), (__mmask16)(U), \
                                          _MM_FROUND_CUR_DIRECTION))

#define _mm512_maskz_range_ps(U, A, B, C) \
  ((__m512)__builtin_ia32_rangeps512_mask((__v16sf)(__m512)(A), \
                                          (__v16sf)(__m512)(B), (int)(C), \
                                          (__v16sf)_mm512_setzero_ps(), \
                                          (__mmask16)(U), \
                                          _MM_FROUND_CUR_DIRECTION))

#define _mm512_range_round_ps(A, B, C, R) \
  ((__m512)__builtin_ia32_rangeps512_mask((__v16sf)(__m512)(A), \
                                          (__v16sf)(__m512)(B), (int)(C), \
                                          (__v16sf)_mm512_setzero_ps(), \
                                          (__mmask16)-1, (int)(R)))
````
- **L817 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps512_mask`.
  **L817 CN**: 继续与可调用符号 `__builtin_ia32_rangeps512_mask` 相关的逻辑。
- **L818 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(B), (int)(C), \`.
  **L818 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(B), (int)(C), \`。
- **L819 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L819 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L820 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1, \`.
  **L820 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1, \`。
- **L821 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L821 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Defines macro `_mm512_mask_range_ps(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L823 CN**: 定义宏 `_mm512_mask_range_ps(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L824 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps512_mask`.
  **L824 CN**: 继续与可调用符号 `__builtin_ia32_rangeps512_mask` 相关的逻辑。
- **L825 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(B), (int)(C), \`.
  **L825 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(B), (int)(C), \`。
- **L826 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(W), (__mmask16)(U), \`.
  **L826 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(W), (__mmask16)(U), \`。
- **L827 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L827 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L829 EN**: Defines macro `_mm512_maskz_range_ps(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L829 CN**: 定义宏 `_mm512_maskz_range_ps(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L830 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps512_mask`.
  **L830 CN**: 继续与可调用符号 `__builtin_ia32_rangeps512_mask` 相关的逻辑。
- **L831 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(B), (int)(C), \`.
  **L831 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(B), (int)(C), \`。
- **L832 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L832 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L833 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), \`.
  **L833 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), \`。
- **L834 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L834 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L835 EN**: Blank line separating nearby declarations or logic blocks.
  **L835 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L836 EN**: Defines macro `_mm512_range_round_ps(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L836 CN**: 定义宏 `_mm512_range_round_ps(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L837 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps512_mask`.
  **L837 CN**: 继续与可调用符号 `__builtin_ia32_rangeps512_mask` 相关的逻辑。
- **L838 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(B), (int)(C), \`.
  **L838 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(B), (int)(C), \`。
- **L839 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L839 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L840 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1, (int)(R)))`.
  **L840 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1, (int)(R)))`。

### Lines 841-864

````c

#define _mm512_mask_range_round_ps(W, U, A, B, C, R) \
  ((__m512)__builtin_ia32_rangeps512_mask((__v16sf)(__m512)(A), \
                                          (__v16sf)(__m512)(B), (int)(C), \
                                          (__v16sf)(__m512)(W), (__mmask16)(U), \
                                          (int)(R)))

#define _mm512_maskz_range_round_ps(U, A, B, C, R) \
  ((__m512)__builtin_ia32_rangeps512_mask((__v16sf)(__m512)(A), \
                                          (__v16sf)(__m512)(B), (int)(C), \
                                          (__v16sf)_mm512_setzero_ps(), \
                                          (__mmask16)(U), (int)(R)))

#define _mm_range_round_ss(A, B, C, R) \
  ((__m128)__builtin_ia32_rangess128_round_mask((__v4sf)(__m128)(A), \
                                                (__v4sf)(__m128)(B), \
                                                (__v4sf)_mm_setzero_ps(), \
                                                (__mmask8) -1, (int)(C),\
                                                (int)(R)))

#define _mm_range_ss(A ,B , C) _mm_range_round_ss(A, B, C ,_MM_FROUND_CUR_DIRECTION)

#define _mm_mask_range_round_ss(W, U, A, B, C, R) \
  ((__m128)__builtin_ia32_rangess128_round_mask((__v4sf)(__m128)(A), \
````
- **L841 EN**: Blank line separating nearby declarations or logic blocks.
  **L841 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L842 EN**: Defines macro `_mm512_mask_range_round_ps(W, U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L842 CN**: 定义宏 `_mm512_mask_range_round_ps(W, U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L843 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps512_mask`.
  **L843 CN**: 继续与可调用符号 `__builtin_ia32_rangeps512_mask` 相关的逻辑。
- **L844 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(B), (int)(C), \`.
  **L844 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(B), (int)(C), \`。
- **L845 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(W), (__mmask16)(U), \`.
  **L845 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(W), (__mmask16)(U), \`。
- **L846 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L846 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L847 EN**: Blank line separating nearby declarations or logic blocks.
  **L847 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L848 EN**: Defines macro `_mm512_maskz_range_round_ps(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L848 CN**: 定义宏 `_mm512_maskz_range_round_ps(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L849 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps512_mask`.
  **L849 CN**: 继续与可调用符号 `__builtin_ia32_rangeps512_mask` 相关的逻辑。
- **L850 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(B), (int)(C), \`.
  **L850 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(B), (int)(C), \`。
- **L851 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L851 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L852 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L852 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L853 EN**: Blank line separating nearby declarations or logic blocks.
  **L853 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L854 EN**: Defines macro `_mm_range_round_ss(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L854 CN**: 定义宏 `_mm_range_round_ss(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L855 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangess128_round_mask`.
  **L855 CN**: 继续与可调用符号 `__builtin_ia32_rangess128_round_mask` 相关的逻辑。
- **L856 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L856 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L857 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L857 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L858 EN**: Continues the surrounding expression or declaration: `(__mmask8) -1, (int)(C),\`.
  **L858 CN**: 继续构造周围的表达式或声明：`(__mmask8) -1, (int)(C),\`。
- **L859 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L859 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L861 EN**: Defines macro `_mm_range_ss(A ,B , C)` for conditional compilation, shorthand, or API generation.
  **L861 CN**: 定义宏 `_mm_range_ss(A ,B , C)`，用于条件编译、简写或 API 生成。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L863 EN**: Defines macro `_mm_mask_range_round_ss(W, U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L863 CN**: 定义宏 `_mm_mask_range_round_ss(W, U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L864 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangess128_round_mask`.
  **L864 CN**: 继续与可调用符号 `__builtin_ia32_rangess128_round_mask` 相关的逻辑。

### Lines 865-888

````c
                                                (__v4sf)(__m128)(B), \
                                                (__v4sf)(__m128)(W),\
                                                (__mmask8)(U), (int)(C),\
                                                (int)(R)))

#define _mm_mask_range_ss(W , U, A, B, C) _mm_mask_range_round_ss(W, U, A, B, C , _MM_FROUND_CUR_DIRECTION)

#define _mm_maskz_range_round_ss(U, A, B, C, R) \
  ((__m128)__builtin_ia32_rangess128_round_mask((__v4sf)(__m128)(A), \
                                                (__v4sf)(__m128)(B), \
                                                (__v4sf)_mm_setzero_ps(), \
                                                (__mmask8)(U), (int)(C),\
                                                (int)(R)))

#define _mm_maskz_range_ss(U, A ,B , C) _mm_maskz_range_round_ss(U, A, B, C ,_MM_FROUND_CUR_DIRECTION)

#define _mm_range_round_sd(A, B, C, R) \
  ((__m128d)__builtin_ia32_rangesd128_round_mask((__v2df)(__m128d)(A), \
                                                 (__v2df)(__m128d)(B), \
                                                 (__v2df)_mm_setzero_pd(), \
                                                 (__mmask8) -1, (int)(C),\
                                                 (int)(R)))

#define _mm_range_sd(A ,B , C) _mm_range_round_sd(A, B, C ,_MM_FROUND_CUR_DIRECTION)
````
- **L865 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L865 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L866 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(W),\`.
  **L866 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(W),\`。
- **L867 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C),\`.
  **L867 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C),\`。
- **L868 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L868 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L869 EN**: Blank line separating nearby declarations or logic blocks.
  **L869 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L870 EN**: Defines macro `_mm_mask_range_ss(W , U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L870 CN**: 定义宏 `_mm_mask_range_ss(W , U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L871 EN**: Blank line separating nearby declarations or logic blocks.
  **L871 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L872 EN**: Defines macro `_mm_maskz_range_round_ss(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L872 CN**: 定义宏 `_mm_maskz_range_round_ss(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L873 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangess128_round_mask`.
  **L873 CN**: 继续与可调用符号 `__builtin_ia32_rangess128_round_mask` 相关的逻辑。
- **L874 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L874 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L875 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L875 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L876 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C),\`.
  **L876 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C),\`。
- **L877 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L877 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L878 EN**: Blank line separating nearby declarations or logic blocks.
  **L878 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L879 EN**: Defines macro `_mm_maskz_range_ss(U, A ,B , C)` for conditional compilation, shorthand, or API generation.
  **L879 CN**: 定义宏 `_mm_maskz_range_ss(U, A ,B , C)`，用于条件编译、简写或 API 生成。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L881 EN**: Defines macro `_mm_range_round_sd(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L881 CN**: 定义宏 `_mm_range_round_sd(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L882 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangesd128_round_mask`.
  **L882 CN**: 继续与可调用符号 `__builtin_ia32_rangesd128_round_mask` 相关的逻辑。
- **L883 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L883 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L884 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L884 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L885 EN**: Continues the surrounding expression or declaration: `(__mmask8) -1, (int)(C),\`.
  **L885 CN**: 继续构造周围的表达式或声明：`(__mmask8) -1, (int)(C),\`。
- **L886 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L886 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L887 EN**: Blank line separating nearby declarations or logic blocks.
  **L887 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L888 EN**: Defines macro `_mm_range_sd(A ,B , C)` for conditional compilation, shorthand, or API generation.
  **L888 CN**: 定义宏 `_mm_range_sd(A ,B , C)`，用于条件编译、简写或 API 生成。

### Lines 889-912

````c

#define _mm_mask_range_round_sd(W, U, A, B, C, R) \
  ((__m128d)__builtin_ia32_rangesd128_round_mask((__v2df)(__m128d)(A), \
                                                 (__v2df)(__m128d)(B), \
                                                 (__v2df)(__m128d)(W),\
                                                 (__mmask8)(U), (int)(C),\
                                                 (int)(R)))

#define _mm_mask_range_sd(W, U, A, B, C) _mm_mask_range_round_sd(W, U, A, B, C ,_MM_FROUND_CUR_DIRECTION)

#define _mm_maskz_range_round_sd(U, A, B, C, R) \
  ((__m128d)__builtin_ia32_rangesd128_round_mask((__v2df)(__m128d)(A), \
                                                 (__v2df)(__m128d)(B), \
                                                 (__v2df)_mm_setzero_pd(), \
                                                 (__mmask8)(U), (int)(C),\
                                                 (int)(R)))

#define _mm_maskz_range_sd(U, A, B, C) _mm_maskz_range_round_sd(U, A, B, C ,_MM_FROUND_CUR_DIRECTION)

#define _mm512_reduce_pd(A, B) \
  ((__m512d)__builtin_ia32_reducepd512_mask((__v8df)(__m512d)(A), (int)(B), \
                                            (__v8df)_mm512_setzero_pd(), \
                                            (__mmask8)-1, \
                                            _MM_FROUND_CUR_DIRECTION))
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L890 EN**: Defines macro `_mm_mask_range_round_sd(W, U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L890 CN**: 定义宏 `_mm_mask_range_round_sd(W, U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L891 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangesd128_round_mask`.
  **L891 CN**: 继续与可调用符号 `__builtin_ia32_rangesd128_round_mask` 相关的逻辑。
- **L892 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L892 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L893 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(W),\`.
  **L893 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(W),\`。
- **L894 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C),\`.
  **L894 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C),\`。
- **L895 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L895 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L897 EN**: Defines macro `_mm_mask_range_sd(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L897 CN**: 定义宏 `_mm_mask_range_sd(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L899 EN**: Defines macro `_mm_maskz_range_round_sd(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L899 CN**: 定义宏 `_mm_maskz_range_round_sd(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L900 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangesd128_round_mask`.
  **L900 CN**: 继续与可调用符号 `__builtin_ia32_rangesd128_round_mask` 相关的逻辑。
- **L901 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L901 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L902 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L902 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L903 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C),\`.
  **L903 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C),\`。
- **L904 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L904 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L905 EN**: Blank line separating nearby declarations or logic blocks.
  **L905 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L906 EN**: Defines macro `_mm_maskz_range_sd(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L906 CN**: 定义宏 `_mm_maskz_range_sd(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L907 EN**: Blank line separating nearby declarations or logic blocks.
  **L907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L908 EN**: Defines macro `_mm512_reduce_pd(A, B)` for conditional compilation, shorthand, or API generation.
  **L908 CN**: 定义宏 `_mm512_reduce_pd(A, B)`，用于条件编译、简写或 API 生成。
- **L909 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd512_mask`.
  **L909 CN**: 继续与可调用符号 `__builtin_ia32_reducepd512_mask` 相关的逻辑。
- **L910 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L910 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L911 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, \`.
  **L911 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, \`。
- **L912 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L912 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。

### Lines 913-936

````c

#define _mm512_mask_reduce_pd(W, U, A, B) \
  ((__m512d)__builtin_ia32_reducepd512_mask((__v8df)(__m512d)(A), (int)(B), \
                                            (__v8df)(__m512d)(W), \
                                            (__mmask8)(U), \
                                            _MM_FROUND_CUR_DIRECTION))

#define _mm512_maskz_reduce_pd(U, A, B) \
  ((__m512d)__builtin_ia32_reducepd512_mask((__v8df)(__m512d)(A), (int)(B), \
                                            (__v8df)_mm512_setzero_pd(), \
                                            (__mmask8)(U), \
                                            _MM_FROUND_CUR_DIRECTION))

#define _mm512_reduce_ps(A, B) \
  ((__m512)__builtin_ia32_reduceps512_mask((__v16sf)(__m512)(A), (int)(B), \
                                           (__v16sf)_mm512_setzero_ps(), \
                                           (__mmask16)-1, \
                                           _MM_FROUND_CUR_DIRECTION))

#define _mm512_mask_reduce_ps(W, U, A, B) \
  ((__m512)__builtin_ia32_reduceps512_mask((__v16sf)(__m512)(A), (int)(B), \
                                           (__v16sf)(__m512)(W), \
                                           (__mmask16)(U), \
                                           _MM_FROUND_CUR_DIRECTION))
````
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L914 EN**: Defines macro `_mm512_mask_reduce_pd(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L914 CN**: 定义宏 `_mm512_mask_reduce_pd(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L915 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd512_mask`.
  **L915 CN**: 继续与可调用符号 `__builtin_ia32_reducepd512_mask` 相关的逻辑。
- **L916 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(W), \`.
  **L916 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(W), \`。
- **L917 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), \`.
  **L917 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), \`。
- **L918 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L918 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L919 EN**: Blank line separating nearby declarations or logic blocks.
  **L919 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L920 EN**: Defines macro `_mm512_maskz_reduce_pd(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L920 CN**: 定义宏 `_mm512_maskz_reduce_pd(U, A, B)`，用于条件编译、简写或 API 生成。
- **L921 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd512_mask`.
  **L921 CN**: 继续与可调用符号 `__builtin_ia32_reducepd512_mask` 相关的逻辑。
- **L922 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L922 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L923 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), \`.
  **L923 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), \`。
- **L924 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L924 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L925 EN**: Blank line separating nearby declarations or logic blocks.
  **L925 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L926 EN**: Defines macro `_mm512_reduce_ps(A, B)` for conditional compilation, shorthand, or API generation.
  **L926 CN**: 定义宏 `_mm512_reduce_ps(A, B)`，用于条件编译、简写或 API 生成。
- **L927 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps512_mask`.
  **L927 CN**: 继续与可调用符号 `__builtin_ia32_reduceps512_mask` 相关的逻辑。
- **L928 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L928 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L929 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1, \`.
  **L929 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1, \`。
- **L930 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L930 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L931 EN**: Blank line separating nearby declarations or logic blocks.
  **L931 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L932 EN**: Defines macro `_mm512_mask_reduce_ps(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L932 CN**: 定义宏 `_mm512_mask_reduce_ps(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L933 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps512_mask`.
  **L933 CN**: 继续与可调用符号 `__builtin_ia32_reduceps512_mask` 相关的逻辑。
- **L934 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(W), \`.
  **L934 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(W), \`。
- **L935 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), \`.
  **L935 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), \`。
- **L936 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L936 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。

### Lines 937-960

````c

#define _mm512_maskz_reduce_ps(U, A, B) \
  ((__m512)__builtin_ia32_reduceps512_mask((__v16sf)(__m512)(A), (int)(B), \
                                           (__v16sf)_mm512_setzero_ps(), \
                                           (__mmask16)(U), \
                                           _MM_FROUND_CUR_DIRECTION))

#define _mm512_reduce_round_pd(A, B, R) \
  ((__m512d)__builtin_ia32_reducepd512_mask((__v8df)(__m512d)(A), (int)(B), \
                                            (__v8df)_mm512_setzero_pd(), \
                                            (__mmask8)-1, (int)(R)))

#define _mm512_mask_reduce_round_pd(W, U, A, B, R) \
  ((__m512d)__builtin_ia32_reducepd512_mask((__v8df)(__m512d)(A), (int)(B), \
                                            (__v8df)(__m512d)(W), \
                                            (__mmask8)(U), (int)(R)))

#define _mm512_maskz_reduce_round_pd(U, A, B, R) \
  ((__m512d)__builtin_ia32_reducepd512_mask((__v8df)(__m512d)(A), (int)(B), \
                                            (__v8df)_mm512_setzero_pd(), \
                                            (__mmask8)(U), (int)(R)))

#define _mm512_reduce_round_ps(A, B, R) \
  ((__m512)__builtin_ia32_reduceps512_mask((__v16sf)(__m512)(A), (int)(B), \
````
- **L937 EN**: Blank line separating nearby declarations or logic blocks.
  **L937 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L938 EN**: Defines macro `_mm512_maskz_reduce_ps(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L938 CN**: 定义宏 `_mm512_maskz_reduce_ps(U, A, B)`，用于条件编译、简写或 API 生成。
- **L939 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps512_mask`.
  **L939 CN**: 继续与可调用符号 `__builtin_ia32_reduceps512_mask` 相关的逻辑。
- **L940 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L940 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L941 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), \`.
  **L941 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), \`。
- **L942 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L942 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Defines macro `_mm512_reduce_round_pd(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L944 CN**: 定义宏 `_mm512_reduce_round_pd(A, B, R)`，用于条件编译、简写或 API 生成。
- **L945 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd512_mask`.
  **L945 CN**: 继续与可调用符号 `__builtin_ia32_reducepd512_mask` 相关的逻辑。
- **L946 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L946 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L947 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L947 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L948 EN**: Blank line separating nearby declarations or logic blocks.
  **L948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L949 EN**: Defines macro `_mm512_mask_reduce_round_pd(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L949 CN**: 定义宏 `_mm512_mask_reduce_round_pd(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L950 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd512_mask`.
  **L950 CN**: 继续与可调用符号 `__builtin_ia32_reducepd512_mask` 相关的逻辑。
- **L951 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(W), \`.
  **L951 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(W), \`。
- **L952 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L952 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L953 EN**: Blank line separating nearby declarations or logic blocks.
  **L953 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L954 EN**: Defines macro `_mm512_maskz_reduce_round_pd(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L954 CN**: 定义宏 `_mm512_maskz_reduce_round_pd(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L955 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd512_mask`.
  **L955 CN**: 继续与可调用符号 `__builtin_ia32_reducepd512_mask` 相关的逻辑。
- **L956 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L956 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L957 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L957 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L958 EN**: Blank line separating nearby declarations or logic blocks.
  **L958 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L959 EN**: Defines macro `_mm512_reduce_round_ps(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L959 CN**: 定义宏 `_mm512_reduce_round_ps(A, B, R)`，用于条件编译、简写或 API 生成。
- **L960 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps512_mask`.
  **L960 CN**: 继续与可调用符号 `__builtin_ia32_reduceps512_mask` 相关的逻辑。

### Lines 961-984

````c
                                           (__v16sf)_mm512_setzero_ps(), \
                                           (__mmask16)-1, (int)(R)))

#define _mm512_mask_reduce_round_ps(W, U, A, B, R) \
  ((__m512)__builtin_ia32_reduceps512_mask((__v16sf)(__m512)(A), (int)(B), \
                                           (__v16sf)(__m512)(W), \
                                           (__mmask16)(U), (int)(R)))

#define _mm512_maskz_reduce_round_ps(U, A, B, R) \
  ((__m512)__builtin_ia32_reduceps512_mask((__v16sf)(__m512)(A), (int)(B), \
                                           (__v16sf)_mm512_setzero_ps(), \
                                           (__mmask16)(U), (int)(R)))

#define _mm_reduce_ss(A, B, C) \
  ((__m128)__builtin_ia32_reducess_mask((__v4sf)(__m128)(A), \
                                        (__v4sf)(__m128)(B), \
                                        (__v4sf)_mm_setzero_ps(), (__mmask8)-1, \
                                        (int)(C), _MM_FROUND_CUR_DIRECTION))

#define _mm_mask_reduce_ss(W, U, A, B, C) \
  ((__m128)__builtin_ia32_reducess_mask((__v4sf)(__m128)(A), \
                                        (__v4sf)(__m128)(B), \
                                        (__v4sf)(__m128)(W), (__mmask8)(U), \
                                        (int)(C), _MM_FROUND_CUR_DIRECTION))
````
- **L961 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L961 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L962 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1, (int)(R)))`.
  **L962 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1, (int)(R)))`。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L964 EN**: Defines macro `_mm512_mask_reduce_round_ps(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L964 CN**: 定义宏 `_mm512_mask_reduce_round_ps(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L965 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps512_mask`.
  **L965 CN**: 继续与可调用符号 `__builtin_ia32_reduceps512_mask` 相关的逻辑。
- **L966 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(W), \`.
  **L966 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(W), \`。
- **L967 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L967 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Defines macro `_mm512_maskz_reduce_round_ps(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L969 CN**: 定义宏 `_mm512_maskz_reduce_round_ps(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L970 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps512_mask`.
  **L970 CN**: 继续与可调用符号 `__builtin_ia32_reduceps512_mask` 相关的逻辑。
- **L971 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L971 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L972 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L972 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L973 EN**: Blank line separating nearby declarations or logic blocks.
  **L973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L974 EN**: Defines macro `_mm_reduce_ss(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L974 CN**: 定义宏 `_mm_reduce_ss(A, B, C)`，用于条件编译、简写或 API 生成。
- **L975 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducess_mask`.
  **L975 CN**: 继续与可调用符号 `__builtin_ia32_reducess_mask` 相关的逻辑。
- **L976 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L976 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L977 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L977 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L978 EN**: Continues the surrounding expression or declaration: `(int)(C), _MM_FROUND_CUR_DIRECTION))`.
  **L978 CN**: 继续构造周围的表达式或声明：`(int)(C), _MM_FROUND_CUR_DIRECTION))`。
- **L979 EN**: Blank line separating nearby declarations or logic blocks.
  **L979 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L980 EN**: Defines macro `_mm_mask_reduce_ss(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L980 CN**: 定义宏 `_mm_mask_reduce_ss(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L981 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducess_mask`.
  **L981 CN**: 继续与可调用符号 `__builtin_ia32_reducess_mask` 相关的逻辑。
- **L982 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L982 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L983 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(W), (__mmask8)(U), \`.
  **L983 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(W), (__mmask8)(U), \`。
- **L984 EN**: Continues the surrounding expression or declaration: `(int)(C), _MM_FROUND_CUR_DIRECTION))`.
  **L984 CN**: 继续构造周围的表达式或声明：`(int)(C), _MM_FROUND_CUR_DIRECTION))`。

### Lines 985-1008

````c

#define _mm_maskz_reduce_ss(U, A, B, C) \
  ((__m128)__builtin_ia32_reducess_mask((__v4sf)(__m128)(A), \
                                        (__v4sf)(__m128)(B), \
                                        (__v4sf)_mm_setzero_ps(), \
                                        (__mmask8)(U), (int)(C), \
                                        _MM_FROUND_CUR_DIRECTION))

#define _mm_reduce_round_ss(A, B, C, R) \
  ((__m128)__builtin_ia32_reducess_mask((__v4sf)(__m128)(A), \
                                        (__v4sf)(__m128)(B), \
                                        (__v4sf)_mm_setzero_ps(), (__mmask8)-1, \
                                        (int)(C), (int)(R)))

#define _mm_mask_reduce_round_ss(W, U, A, B, C, R) \
  ((__m128)__builtin_ia32_reducess_mask((__v4sf)(__m128)(A), \
                                        (__v4sf)(__m128)(B), \
                                        (__v4sf)(__m128)(W), (__mmask8)(U), \
                                        (int)(C), (int)(R)))

#define _mm_maskz_reduce_round_ss(U, A, B, C, R) \
  ((__m128)__builtin_ia32_reducess_mask((__v4sf)(__m128)(A), \
                                        (__v4sf)(__m128)(B), \
                                        (__v4sf)_mm_setzero_ps(), \
````
- **L985 EN**: Blank line separating nearby declarations or logic blocks.
  **L985 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L986 EN**: Defines macro `_mm_maskz_reduce_ss(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L986 CN**: 定义宏 `_mm_maskz_reduce_ss(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L987 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducess_mask`.
  **L987 CN**: 继续与可调用符号 `__builtin_ia32_reducess_mask` 相关的逻辑。
- **L988 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L988 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L989 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L989 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L990 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C), \`.
  **L990 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C), \`。
- **L991 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L991 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L992 EN**: Blank line separating nearby declarations or logic blocks.
  **L992 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L993 EN**: Defines macro `_mm_reduce_round_ss(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L993 CN**: 定义宏 `_mm_reduce_round_ss(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L994 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducess_mask`.
  **L994 CN**: 继续与可调用符号 `__builtin_ia32_reducess_mask` 相关的逻辑。
- **L995 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L995 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L996 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L996 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L997 EN**: Continues the surrounding expression or declaration: `(int)(C), (int)(R)))`.
  **L997 CN**: 继续构造周围的表达式或声明：`(int)(C), (int)(R)))`。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L999 EN**: Defines macro `_mm_mask_reduce_round_ss(W, U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L999 CN**: 定义宏 `_mm_mask_reduce_round_ss(W, U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1000 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducess_mask`.
  **L1000 CN**: 继续与可调用符号 `__builtin_ia32_reducess_mask` 相关的逻辑。
- **L1001 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L1001 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L1002 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(W), (__mmask8)(U), \`.
  **L1002 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(W), (__mmask8)(U), \`。
- **L1003 EN**: Continues the surrounding expression or declaration: `(int)(C), (int)(R)))`.
  **L1003 CN**: 继续构造周围的表达式或声明：`(int)(C), (int)(R)))`。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1005 EN**: Defines macro `_mm_maskz_reduce_round_ss(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1005 CN**: 定义宏 `_mm_maskz_reduce_round_ss(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1006 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducess_mask`.
  **L1006 CN**: 继续与可调用符号 `__builtin_ia32_reducess_mask` 相关的逻辑。
- **L1007 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), \`.
  **L1007 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), \`。
- **L1008 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L1008 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。

### Lines 1009-1032

````c
                                        (__mmask8)(U), (int)(C), (int)(R)))

#define _mm_reduce_sd(A, B, C) \
  ((__m128d)__builtin_ia32_reducesd_mask((__v2df)(__m128d)(A), \
                                         (__v2df)(__m128d)(B), \
                                         (__v2df)_mm_setzero_pd(), \
                                         (__mmask8)-1, (int)(C), \
                                         _MM_FROUND_CUR_DIRECTION))

#define _mm_mask_reduce_sd(W, U, A, B, C) \
  ((__m128d)__builtin_ia32_reducesd_mask((__v2df)(__m128d)(A), \
                                         (__v2df)(__m128d)(B), \
                                         (__v2df)(__m128d)(W), (__mmask8)(U), \
                                         (int)(C), _MM_FROUND_CUR_DIRECTION))

#define _mm_maskz_reduce_sd(U, A, B, C) \
  ((__m128d)__builtin_ia32_reducesd_mask((__v2df)(__m128d)(A), \
                                         (__v2df)(__m128d)(B), \
                                         (__v2df)_mm_setzero_pd(), \
                                         (__mmask8)(U), (int)(C), \
                                         _MM_FROUND_CUR_DIRECTION))

#define _mm_reduce_round_sd(A, B, C, R) \
  ((__m128d)__builtin_ia32_reducesd_mask((__v2df)(__m128d)(A), \
````
- **L1009 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C), (int)(R)))`.
  **L1009 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C), (int)(R)))`。
- **L1010 EN**: Blank line separating nearby declarations or logic blocks.
  **L1010 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1011 EN**: Defines macro `_mm_reduce_sd(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1011 CN**: 定义宏 `_mm_reduce_sd(A, B, C)`，用于条件编译、简写或 API 生成。
- **L1012 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesd_mask`.
  **L1012 CN**: 继续与可调用符号 `__builtin_ia32_reducesd_mask` 相关的逻辑。
- **L1013 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L1013 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L1014 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1014 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1015 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(C), \`.
  **L1015 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(C), \`。
- **L1016 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L1016 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L1017 EN**: Blank line separating nearby declarations or logic blocks.
  **L1017 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1018 EN**: Defines macro `_mm_mask_reduce_sd(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1018 CN**: 定义宏 `_mm_mask_reduce_sd(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L1019 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesd_mask`.
  **L1019 CN**: 继续与可调用符号 `__builtin_ia32_reducesd_mask` 相关的逻辑。
- **L1020 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L1020 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L1021 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(W), (__mmask8)(U), \`.
  **L1021 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(W), (__mmask8)(U), \`。
- **L1022 EN**: Continues the surrounding expression or declaration: `(int)(C), _MM_FROUND_CUR_DIRECTION))`.
  **L1022 CN**: 继续构造周围的表达式或声明：`(int)(C), _MM_FROUND_CUR_DIRECTION))`。
- **L1023 EN**: Blank line separating nearby declarations or logic blocks.
  **L1023 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1024 EN**: Defines macro `_mm_maskz_reduce_sd(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1024 CN**: 定义宏 `_mm_maskz_reduce_sd(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L1025 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesd_mask`.
  **L1025 CN**: 继续与可调用符号 `__builtin_ia32_reducesd_mask` 相关的逻辑。
- **L1026 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L1026 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L1027 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1027 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1028 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C), \`.
  **L1028 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C), \`。
- **L1029 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L1029 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L1030 EN**: Blank line separating nearby declarations or logic blocks.
  **L1030 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1031 EN**: Defines macro `_mm_reduce_round_sd(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1031 CN**: 定义宏 `_mm_reduce_round_sd(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1032 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesd_mask`.
  **L1032 CN**: 继续与可调用符号 `__builtin_ia32_reducesd_mask` 相关的逻辑。

### Lines 1033-1056

````c
                                         (__v2df)(__m128d)(B), \
                                         (__v2df)_mm_setzero_pd(), \
                                         (__mmask8)-1, (int)(C), (int)(R)))

#define _mm_mask_reduce_round_sd(W, U, A, B, C, R) \
  ((__m128d)__builtin_ia32_reducesd_mask((__v2df)(__m128d)(A), \
                                         (__v2df)(__m128d)(B), \
                                         (__v2df)(__m128d)(W), (__mmask8)(U), \
                                         (int)(C), (int)(R)))

#define _mm_maskz_reduce_round_sd(U, A, B, C, R) \
  ((__m128d)__builtin_ia32_reducesd_mask((__v2df)(__m128d)(A), \
                                         (__v2df)(__m128d)(B), \
                                         (__v2df)_mm_setzero_pd(), \
                                         (__mmask8)(U), (int)(C), (int)(R)))

static __inline__ __mmask16
    __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_movepi32_mask(__m512i __A) {
  return (__mmask16) __builtin_ia32_cvtd2mask512 ((__v16si) __A);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_movm_epi32(__mmask16 __A) {
  return (__m512i) __builtin_ia32_cvtmask2d512 (__A);
````
- **L1033 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L1033 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L1034 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1034 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1035 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(C), (int)(R)))`.
  **L1035 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(C), (int)(R)))`。
- **L1036 EN**: Blank line separating nearby declarations or logic blocks.
  **L1036 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1037 EN**: Defines macro `_mm_mask_reduce_round_sd(W, U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1037 CN**: 定义宏 `_mm_mask_reduce_round_sd(W, U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1038 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesd_mask`.
  **L1038 CN**: 继续与可调用符号 `__builtin_ia32_reducesd_mask` 相关的逻辑。
- **L1039 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L1039 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L1040 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(W), (__mmask8)(U), \`.
  **L1040 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(W), (__mmask8)(U), \`。
- **L1041 EN**: Continues the surrounding expression or declaration: `(int)(C), (int)(R)))`.
  **L1041 CN**: 继续构造周围的表达式或声明：`(int)(C), (int)(R)))`。
- **L1042 EN**: Blank line separating nearby declarations or logic blocks.
  **L1042 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1043 EN**: Defines macro `_mm_maskz_reduce_round_sd(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1043 CN**: 定义宏 `_mm_maskz_reduce_round_sd(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1044 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesd_mask`.
  **L1044 CN**: 继续与可调用符号 `__builtin_ia32_reducesd_mask` 相关的逻辑。
- **L1045 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), \`.
  **L1045 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), \`。
- **L1046 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1046 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1047 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C), (int)(R)))`.
  **L1047 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C), (int)(R)))`。
- **L1048 EN**: Blank line separating nearby declarations or logic blocks.
  **L1048 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1049 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16`.
  **L1049 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16`。
- **L1050 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_movepi32_mask(__m512i __A) {`.
  **L1050 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_movepi32_mask(__m512i __A) {`。
- **L1051 EN**: Returns from the current function with `(__mmask16) __builtin_ia32_cvtd2mask512 ((__v16si) __A)`.
  **L1051 CN**: 以 `(__mmask16) __builtin_ia32_cvtd2mask512 ((__v16si) __A)` 从当前函数返回。
- **L1052 EN**: Closes the current lexical scope or compound statement.
  **L1052 CN**: 结束当前词法作用域或复合语句块。
- **L1053 EN**: Blank line separating nearby declarations or logic blocks.
  **L1053 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1054 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1054 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1055 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_movm_epi32(__mmask16 __A) {`.
  **L1055 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_movm_epi32(__mmask16 __A) {`。
- **L1056 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtmask2d512 (__A)`.
  **L1056 CN**: 以 `(__m512i) __builtin_ia32_cvtmask2d512 (__A)` 从当前函数返回。

### Lines 1057-1080

````c
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_movm_epi64(__mmask8 __A) {
  return (__m512i) __builtin_ia32_cvtmask2q512 (__A);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_movepi64_mask(__m512i __A) {
  return (__mmask8) __builtin_ia32_cvtq2mask512 ((__v8di) __A);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_broadcast_f32x2(__m128 __A) {
  return (__m512)__builtin_shufflevector((__v4sf)__A, (__v4sf)__A,
                                         0, 1, 0, 1, 0, 1, 0, 1,
                                         0, 1, 0, 1, 0, 1, 0, 1);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_broadcast_f32x2(__m512 __O, __mmask16 __M, __m128 __A) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__M,
                                             (__v16sf)_mm512_broadcast_f32x2(__A),
                                             (__v16sf)__O);
````
- **L1057 EN**: Closes the current lexical scope or compound statement.
  **L1057 CN**: 结束当前词法作用域或复合语句块。
- **L1058 EN**: Blank line separating nearby declarations or logic blocks.
  **L1058 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1059 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1059 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1060 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_movm_epi64(__mmask8 __A) {`.
  **L1060 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_movm_epi64(__mmask8 __A) {`。
- **L1061 EN**: Returns from the current function with `(__m512i) __builtin_ia32_cvtmask2q512 (__A)`.
  **L1061 CN**: 以 `(__m512i) __builtin_ia32_cvtmask2q512 (__A)` 从当前函数返回。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1064 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1064 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1065 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_movepi64_mask(__m512i __A) {`.
  **L1065 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_movepi64_mask(__m512i __A) {`。
- **L1066 EN**: Returns from the current function with `(__mmask8) __builtin_ia32_cvtq2mask512 ((__v8di) __A)`.
  **L1066 CN**: 以 `(__mmask8) __builtin_ia32_cvtq2mask512 ((__v8di) __A)` 从当前函数返回。
- **L1067 EN**: Closes the current lexical scope or compound statement.
  **L1067 CN**: 结束当前词法作用域或复合语句块。
- **L1068 EN**: Blank line separating nearby declarations or logic blocks.
  **L1068 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1069 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1069 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1070 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_broadcast_f32x2(__m128 __A) {`.
  **L1070 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_broadcast_f32x2(__m128 __A) {`。
- **L1071 EN**: Returns from the current function with `(__m512)__builtin_shufflevector((__v4sf)__A, (__v4sf)__A,`.
  **L1071 CN**: 以 `(__m512)__builtin_shufflevector((__v4sf)__A, (__v4sf)__A,` 从当前函数返回。
- **L1072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0, 1, 0, 1, 0, 1, 0, 1,`.
  **L1072 CN**: 继续一个多行参数列表、初始化器或聚合项：`0, 1, 0, 1, 0, 1, 0, 1,`。
- **L1073 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1, 0, 1, 0, 1);`.
  **L1073 CN**: 添加一条独立语句或声明：`0, 1, 0, 1, 0, 1, 0, 1);`。
- **L1074 EN**: Closes the current lexical scope or compound statement.
  **L1074 CN**: 结束当前词法作用域或复合语句块。
- **L1075 EN**: Blank line separating nearby declarations or logic blocks.
  **L1075 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1076 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1076 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1077 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_broadcast_f32x2(__m512 __O, __mmask16 __M, __m128 __A) {`.
  **L1077 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_broadcast_f32x2(__m512 __O, __mmask16 __M, __m128 __A) {`。
- **L1078 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__M,`.
  **L1078 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__M,` 从当前函数返回。
- **L1079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_broadcast_f32x2(__A),`.
  **L1079 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_broadcast_f32x2(__A),`。
- **L1080 EN**: Executes a call or declaration centered on `statement`.
  **L1080 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1081-1104

````c
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_broadcast_f32x2(__mmask16 __M, __m128 __A) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__M,
                                             (__v16sf)_mm512_broadcast_f32x2(__A),
                                             (__v16sf)_mm512_setzero_ps());
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_broadcast_f32x8(__m256 __A) {
  return (__m512)__builtin_shufflevector((__v8sf)__A, (__v8sf)__A,
                                         0, 1, 2, 3, 4, 5, 6, 7,
                                         0, 1, 2, 3, 4, 5, 6, 7);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_broadcast_f32x8(__m512 __O, __mmask16 __M, __m256 __A) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__M,
                                           (__v16sf)_mm512_broadcast_f32x8(__A),
                                           (__v16sf)__O);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR
````
- **L1081 EN**: Closes the current lexical scope or compound statement.
  **L1081 CN**: 结束当前词法作用域或复合语句块。
- **L1082 EN**: Blank line separating nearby declarations or logic blocks.
  **L1082 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1083 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1083 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1084 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_broadcast_f32x2(__mmask16 __M, __m128 __A) {`.
  **L1084 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_broadcast_f32x2(__mmask16 __M, __m128 __A) {`。
- **L1085 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__M,`.
  **L1085 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__M,` 从当前函数返回。
- **L1086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_broadcast_f32x2(__A),`.
  **L1086 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_broadcast_f32x2(__A),`。
- **L1087 EN**: Executes a call or declaration centered on `statement`.
  **L1087 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1088 EN**: Closes the current lexical scope or compound statement.
  **L1088 CN**: 结束当前词法作用域或复合语句块。
- **L1089 EN**: Blank line separating nearby declarations or logic blocks.
  **L1089 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1090 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1090 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1091 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_broadcast_f32x8(__m256 __A) {`.
  **L1091 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_broadcast_f32x8(__m256 __A) {`。
- **L1092 EN**: Returns from the current function with `(__m512)__builtin_shufflevector((__v8sf)__A, (__v8sf)__A,`.
  **L1092 CN**: 以 `(__m512)__builtin_shufflevector((__v8sf)__A, (__v8sf)__A,` 从当前函数返回。
- **L1093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0, 1, 2, 3, 4, 5, 6, 7,`.
  **L1093 CN**: 继续一个多行参数列表、初始化器或聚合项：`0, 1, 2, 3, 4, 5, 6, 7,`。
- **L1094 EN**: Adds a standalone statement or declaration: `0, 1, 2, 3, 4, 5, 6, 7);`.
  **L1094 CN**: 添加一条独立语句或声明：`0, 1, 2, 3, 4, 5, 6, 7);`。
- **L1095 EN**: Closes the current lexical scope or compound statement.
  **L1095 CN**: 结束当前词法作用域或复合语句块。
- **L1096 EN**: Blank line separating nearby declarations or logic blocks.
  **L1096 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1097 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1097 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1098 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_broadcast_f32x8(__m512 __O, __mmask16 __M, __m256 __A) {`.
  **L1098 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_broadcast_f32x8(__m512 __O, __mmask16 __M, __m256 __A) {`。
- **L1099 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__M,`.
  **L1099 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__M,` 从当前函数返回。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_broadcast_f32x8(__A),`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_broadcast_f32x8(__A),`。
- **L1101 EN**: Executes a call or declaration centered on `statement`.
  **L1101 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1102 EN**: Closes the current lexical scope or compound statement.
  **L1102 CN**: 结束当前词法作用域或复合语句块。
- **L1103 EN**: Blank line separating nearby declarations or logic blocks.
  **L1103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1104 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1104 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR`。

### Lines 1105-1128

````c
_mm512_maskz_broadcast_f32x8(__mmask16 __M, __m256 __A) {
  return (__m512)__builtin_ia32_selectps_512((__mmask16)__M,
                                           (__v16sf)_mm512_broadcast_f32x8(__A),
                                           (__v16sf)_mm512_setzero_ps());
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_broadcast_f64x2(__m128d __A) {
  return (__m512d)__builtin_shufflevector((__v2df)__A, (__v2df)__A,
                                          0, 1, 0, 1, 0, 1, 0, 1);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_broadcast_f64x2(__m512d __O, __mmask8 __M, __m128d __A) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__M,
                                            (__v8df)_mm512_broadcast_f64x2(__A),
                                            (__v8df)__O);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_broadcast_f64x2(__mmask8 __M, __m128d __A) {
  return (__m512d)__builtin_ia32_selectpd_512((__mmask8)__M,
                                            (__v8df)_mm512_broadcast_f64x2(__A),
                                            (__v8df)_mm512_setzero_pd());
````
- **L1105 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_broadcast_f32x8(__mmask16 __M, __m256 __A) {`.
  **L1105 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_broadcast_f32x8(__mmask16 __M, __m256 __A) {`。
- **L1106 EN**: Returns from the current function with `(__m512)__builtin_ia32_selectps_512((__mmask16)__M,`.
  **L1106 CN**: 以 `(__m512)__builtin_ia32_selectps_512((__mmask16)__M,` 从当前函数返回。
- **L1107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_broadcast_f32x8(__A),`.
  **L1107 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_broadcast_f32x8(__A),`。
- **L1108 EN**: Executes a call or declaration centered on `statement`.
  **L1108 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1109 EN**: Closes the current lexical scope or compound statement.
  **L1109 CN**: 结束当前词法作用域或复合语句块。
- **L1110 EN**: Blank line separating nearby declarations or logic blocks.
  **L1110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1111 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1111 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1112 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_broadcast_f64x2(__m128d __A) {`.
  **L1112 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_broadcast_f64x2(__m128d __A) {`。
- **L1113 EN**: Returns from the current function with `(__m512d)__builtin_shufflevector((__v2df)__A, (__v2df)__A,`.
  **L1113 CN**: 以 `(__m512d)__builtin_shufflevector((__v2df)__A, (__v2df)__A,` 从当前函数返回。
- **L1114 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1, 0, 1, 0, 1);`.
  **L1114 CN**: 添加一条独立语句或声明：`0, 1, 0, 1, 0, 1, 0, 1);`。
- **L1115 EN**: Closes the current lexical scope or compound statement.
  **L1115 CN**: 结束当前词法作用域或复合语句块。
- **L1116 EN**: Blank line separating nearby declarations or logic blocks.
  **L1116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1117 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1117 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1118 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_broadcast_f64x2(__m512d __O, __mmask8 __M, __m128d __A) {`.
  **L1118 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_broadcast_f64x2(__m512d __O, __mmask8 __M, __m128d __A) {`。
- **L1119 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__M,`.
  **L1119 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__M,` 从当前函数返回。
- **L1120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_broadcast_f64x2(__A),`.
  **L1120 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_broadcast_f64x2(__A),`。
- **L1121 EN**: Executes a call or declaration centered on `statement`.
  **L1121 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1122 EN**: Closes the current lexical scope or compound statement.
  **L1122 CN**: 结束当前词法作用域或复合语句块。
- **L1123 EN**: Blank line separating nearby declarations or logic blocks.
  **L1123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1124 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1124 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_broadcast_f64x2(__mmask8 __M, __m128d __A) {`.
  **L1125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_broadcast_f64x2(__mmask8 __M, __m128d __A) {`。
- **L1126 EN**: Returns from the current function with `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__M,`.
  **L1126 CN**: 以 `(__m512d)__builtin_ia32_selectpd_512((__mmask8)__M,` 从当前函数返回。
- **L1127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)_mm512_broadcast_f64x2(__A),`.
  **L1127 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)_mm512_broadcast_f64x2(__A),`。
- **L1128 EN**: Executes a call or declaration centered on `statement`.
  **L1128 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1129-1152

````c
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_broadcast_i32x2(__m128i __A) {
  return (__m512i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,
                                          0, 1, 0, 1, 0, 1, 0, 1,
                                          0, 1, 0, 1, 0, 1, 0, 1);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_broadcast_i32x2(__m512i __O, __mmask16 __M, __m128i __A) {
  return (__m512i)__builtin_ia32_selectd_512((__mmask16)__M,
                                             (__v16si)_mm512_broadcast_i32x2(__A),
                                             (__v16si)__O);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_broadcast_i32x2(__mmask16 __M, __m128i __A) {
  return (__m512i)__builtin_ia32_selectd_512((__mmask16)__M,
                                             (__v16si)_mm512_broadcast_i32x2(__A),
                                             (__v16si)_mm512_setzero_si512());
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
````
- **L1129 EN**: Closes the current lexical scope or compound statement.
  **L1129 CN**: 结束当前词法作用域或复合语句块。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1131 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1131 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1132 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_broadcast_i32x2(__m128i __A) {`.
  **L1132 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_broadcast_i32x2(__m128i __A) {`。
- **L1133 EN**: Returns from the current function with `(__m512i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,`.
  **L1133 CN**: 以 `(__m512i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,` 从当前函数返回。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0, 1, 0, 1, 0, 1, 0, 1,`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`0, 1, 0, 1, 0, 1, 0, 1,`。
- **L1135 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1, 0, 1, 0, 1);`.
  **L1135 CN**: 添加一条独立语句或声明：`0, 1, 0, 1, 0, 1, 0, 1);`。
- **L1136 EN**: Closes the current lexical scope or compound statement.
  **L1136 CN**: 结束当前词法作用域或复合语句块。
- **L1137 EN**: Blank line separating nearby declarations or logic blocks.
  **L1137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1138 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1138 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1139 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_broadcast_i32x2(__m512i __O, __mmask16 __M, __m128i __A) {`.
  **L1139 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_broadcast_i32x2(__m512i __O, __mmask16 __M, __m128i __A) {`。
- **L1140 EN**: Returns from the current function with `(__m512i)__builtin_ia32_selectd_512((__mmask16)__M,`.
  **L1140 CN**: 以 `(__m512i)__builtin_ia32_selectd_512((__mmask16)__M,` 从当前函数返回。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16si)_mm512_broadcast_i32x2(__A),`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16si)_mm512_broadcast_i32x2(__A),`。
- **L1142 EN**: Executes a call or declaration centered on `statement`.
  **L1142 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1143 EN**: Closes the current lexical scope or compound statement.
  **L1143 CN**: 结束当前词法作用域或复合语句块。
- **L1144 EN**: Blank line separating nearby declarations or logic blocks.
  **L1144 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1145 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1145 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1146 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_broadcast_i32x2(__mmask16 __M, __m128i __A) {`.
  **L1146 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_broadcast_i32x2(__mmask16 __M, __m128i __A) {`。
- **L1147 EN**: Returns from the current function with `(__m512i)__builtin_ia32_selectd_512((__mmask16)__M,`.
  **L1147 CN**: 以 `(__m512i)__builtin_ia32_selectd_512((__mmask16)__M,` 从当前函数返回。
- **L1148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16si)_mm512_broadcast_i32x2(__A),`.
  **L1148 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16si)_mm512_broadcast_i32x2(__A),`。
- **L1149 EN**: Executes a call or declaration centered on `statement`.
  **L1149 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1150 EN**: Closes the current lexical scope or compound statement.
  **L1150 CN**: 结束当前词法作用域或复合语句块。
- **L1151 EN**: Blank line separating nearby declarations or logic blocks.
  **L1151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1152 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1152 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。

### Lines 1153-1176

````c
_mm512_broadcast_i32x8(__m256i __A) {
  return (__m512i)__builtin_shufflevector((__v8si)__A, (__v8si)__A,
                                          0, 1, 2, 3, 4, 5, 6, 7,
                                          0, 1, 2, 3, 4, 5, 6, 7);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_broadcast_i32x8(__m512i __O, __mmask16 __M, __m256i __A) {
  return (__m512i)__builtin_ia32_selectd_512((__mmask16)__M,
                                           (__v16si)_mm512_broadcast_i32x8(__A),
                                           (__v16si)__O);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_broadcast_i32x8(__mmask16 __M, __m256i __A) {
  return (__m512i)__builtin_ia32_selectd_512((__mmask16)__M,
                                           (__v16si)_mm512_broadcast_i32x8(__A),
                                           (__v16si)_mm512_setzero_si512());
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_broadcast_i64x2(__m128i __A) {
  return (__m512i)__builtin_shufflevector((__v2di)__A, (__v2di)__A,
                                          0, 1, 0, 1, 0, 1, 0, 1);
````
- **L1153 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_broadcast_i32x8(__m256i __A) {`.
  **L1153 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_broadcast_i32x8(__m256i __A) {`。
- **L1154 EN**: Returns from the current function with `(__m512i)__builtin_shufflevector((__v8si)__A, (__v8si)__A,`.
  **L1154 CN**: 以 `(__m512i)__builtin_shufflevector((__v8si)__A, (__v8si)__A,` 从当前函数返回。
- **L1155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0, 1, 2, 3, 4, 5, 6, 7,`.
  **L1155 CN**: 继续一个多行参数列表、初始化器或聚合项：`0, 1, 2, 3, 4, 5, 6, 7,`。
- **L1156 EN**: Adds a standalone statement or declaration: `0, 1, 2, 3, 4, 5, 6, 7);`.
  **L1156 CN**: 添加一条独立语句或声明：`0, 1, 2, 3, 4, 5, 6, 7);`。
- **L1157 EN**: Closes the current lexical scope or compound statement.
  **L1157 CN**: 结束当前词法作用域或复合语句块。
- **L1158 EN**: Blank line separating nearby declarations or logic blocks.
  **L1158 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1159 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1159 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1160 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_broadcast_i32x8(__m512i __O, __mmask16 __M, __m256i __A) {`.
  **L1160 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_broadcast_i32x8(__m512i __O, __mmask16 __M, __m256i __A) {`。
- **L1161 EN**: Returns from the current function with `(__m512i)__builtin_ia32_selectd_512((__mmask16)__M,`.
  **L1161 CN**: 以 `(__m512i)__builtin_ia32_selectd_512((__mmask16)__M,` 从当前函数返回。
- **L1162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16si)_mm512_broadcast_i32x8(__A),`.
  **L1162 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16si)_mm512_broadcast_i32x8(__A),`。
- **L1163 EN**: Executes a call or declaration centered on `statement`.
  **L1163 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1164 EN**: Closes the current lexical scope or compound statement.
  **L1164 CN**: 结束当前词法作用域或复合语句块。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1166 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1166 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1167 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_broadcast_i32x8(__mmask16 __M, __m256i __A) {`.
  **L1167 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_broadcast_i32x8(__mmask16 __M, __m256i __A) {`。
- **L1168 EN**: Returns from the current function with `(__m512i)__builtin_ia32_selectd_512((__mmask16)__M,`.
  **L1168 CN**: 以 `(__m512i)__builtin_ia32_selectd_512((__mmask16)__M,` 从当前函数返回。
- **L1169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16si)_mm512_broadcast_i32x8(__A),`.
  **L1169 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16si)_mm512_broadcast_i32x8(__A),`。
- **L1170 EN**: Executes a call or declaration centered on `statement`.
  **L1170 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1171 EN**: Closes the current lexical scope or compound statement.
  **L1171 CN**: 结束当前词法作用域或复合语句块。
- **L1172 EN**: Blank line separating nearby declarations or logic blocks.
  **L1172 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1173 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1173 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1174 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_broadcast_i64x2(__m128i __A) {`.
  **L1174 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_broadcast_i64x2(__m128i __A) {`。
- **L1175 EN**: Returns from the current function with `(__m512i)__builtin_shufflevector((__v2di)__A, (__v2di)__A,`.
  **L1175 CN**: 以 `(__m512i)__builtin_shufflevector((__v2di)__A, (__v2di)__A,` 从当前函数返回。
- **L1176 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1, 0, 1, 0, 1);`.
  **L1176 CN**: 添加一条独立语句或声明：`0, 1, 0, 1, 0, 1, 0, 1);`。

### Lines 1177-1200

````c
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_broadcast_i64x2(__m512i __O, __mmask8 __M, __m128i __A) {
  return (__m512i)__builtin_ia32_selectq_512((__mmask8)__M,
                                            (__v8di)_mm512_broadcast_i64x2(__A),
                                            (__v8di)__O);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_broadcast_i64x2(__mmask8 __M, __m128i __A) {
  return (__m512i)__builtin_ia32_selectq_512((__mmask8)__M,
                                            (__v8di)_mm512_broadcast_i64x2(__A),
                                            (__v8di)_mm512_setzero_si512());
}

#define _mm512_extractf32x8_ps(A, imm)                                         \
  ((__m256)__builtin_ia32_extractf32x8_mask((__v16sf)(__m512)(A), (int)(imm),  \
                                            (__v8sf)_mm256_setzero_ps(),       \
                                            (__mmask8) - 1))

#define _mm512_mask_extractf32x8_ps(W, U, A, imm) \
  ((__m256)__builtin_ia32_extractf32x8_mask((__v16sf)(__m512)(A), (int)(imm), \
                                            (__v8sf)(__m256)(W), \
````
- **L1177 EN**: Closes the current lexical scope or compound statement.
  **L1177 CN**: 结束当前词法作用域或复合语句块。
- **L1178 EN**: Blank line separating nearby declarations or logic blocks.
  **L1178 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1179 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1179 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1180 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_broadcast_i64x2(__m512i __O, __mmask8 __M, __m128i __A) {`.
  **L1180 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_broadcast_i64x2(__m512i __O, __mmask8 __M, __m128i __A) {`。
- **L1181 EN**: Returns from the current function with `(__m512i)__builtin_ia32_selectq_512((__mmask8)__M,`.
  **L1181 CN**: 以 `(__m512i)__builtin_ia32_selectq_512((__mmask8)__M,` 从当前函数返回。
- **L1182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di)_mm512_broadcast_i64x2(__A),`.
  **L1182 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di)_mm512_broadcast_i64x2(__A),`。
- **L1183 EN**: Executes a call or declaration centered on `statement`.
  **L1183 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1184 EN**: Closes the current lexical scope or compound statement.
  **L1184 CN**: 结束当前词法作用域或复合语句块。
- **L1185 EN**: Blank line separating nearby declarations or logic blocks.
  **L1185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1186 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1186 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1187 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_broadcast_i64x2(__mmask8 __M, __m128i __A) {`.
  **L1187 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_broadcast_i64x2(__mmask8 __M, __m128i __A) {`。
- **L1188 EN**: Returns from the current function with `(__m512i)__builtin_ia32_selectq_512((__mmask8)__M,`.
  **L1188 CN**: 以 `(__m512i)__builtin_ia32_selectq_512((__mmask8)__M,` 从当前函数返回。
- **L1189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di)_mm512_broadcast_i64x2(__A),`.
  **L1189 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di)_mm512_broadcast_i64x2(__A),`。
- **L1190 EN**: Executes a call or declaration centered on `statement`.
  **L1190 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1191 EN**: Closes the current lexical scope or compound statement.
  **L1191 CN**: 结束当前词法作用域或复合语句块。
- **L1192 EN**: Blank line separating nearby declarations or logic blocks.
  **L1192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1193 EN**: Defines macro `_mm512_extractf32x8_ps(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1193 CN**: 定义宏 `_mm512_extractf32x8_ps(A, imm)`，用于条件编译、简写或 API 生成。
- **L1194 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf32x8_mask`.
  **L1194 CN**: 继续与可调用符号 `__builtin_ia32_extractf32x8_mask` 相关的逻辑。
- **L1195 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L1195 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L1196 EN**: Continues the surrounding expression or declaration: `(__mmask8) - 1))`.
  **L1196 CN**: 继续构造周围的表达式或声明：`(__mmask8) - 1))`。
- **L1197 EN**: Blank line separating nearby declarations or logic blocks.
  **L1197 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1198 EN**: Defines macro `_mm512_mask_extractf32x8_ps(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1198 CN**: 定义宏 `_mm512_mask_extractf32x8_ps(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1199 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf32x8_mask`.
  **L1199 CN**: 继续与可调用符号 `__builtin_ia32_extractf32x8_mask` 相关的逻辑。
- **L1200 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(W), \`.
  **L1200 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(W), \`。

### Lines 1201-1224

````c
                                            (__mmask8)(U)))

#define _mm512_maskz_extractf32x8_ps(U, A, imm) \
  ((__m256)__builtin_ia32_extractf32x8_mask((__v16sf)(__m512)(A), (int)(imm), \
                                            (__v8sf)_mm256_setzero_ps(), \
                                            (__mmask8)(U)))

#define _mm512_extractf64x2_pd(A, imm)                                         \
  ((__m128d)__builtin_ia32_extractf64x2_512_mask(                              \
      (__v8df)(__m512d)(A), (int)(imm), (__v2df)_mm_setzero_pd(),              \
      (__mmask8) - 1))

#define _mm512_mask_extractf64x2_pd(W, U, A, imm) \
  ((__m128d)__builtin_ia32_extractf64x2_512_mask((__v8df)(__m512d)(A), \
                                                 (int)(imm), \
                                                 (__v2df)(__m128d)(W), \
                                                 (__mmask8)(U)))

#define _mm512_maskz_extractf64x2_pd(U, A, imm) \
  ((__m128d)__builtin_ia32_extractf64x2_512_mask((__v8df)(__m512d)(A), \
                                                 (int)(imm), \
                                                 (__v2df)_mm_setzero_pd(), \
                                                 (__mmask8)(U)))

````
- **L1201 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1201 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1202 EN**: Blank line separating nearby declarations or logic blocks.
  **L1202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1203 EN**: Defines macro `_mm512_maskz_extractf32x8_ps(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1203 CN**: 定义宏 `_mm512_maskz_extractf32x8_ps(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1204 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf32x8_mask`.
  **L1204 CN**: 继续与可调用符号 `__builtin_ia32_extractf32x8_mask` 相关的逻辑。
- **L1205 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L1205 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L1206 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1206 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1207 EN**: Blank line separating nearby declarations or logic blocks.
  **L1207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1208 EN**: Defines macro `_mm512_extractf64x2_pd(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1208 CN**: 定义宏 `_mm512_extractf64x2_pd(A, imm)`，用于条件编译、简写或 API 生成。
- **L1209 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf64x2_512_mask`.
  **L1209 CN**: 继续与可调用符号 `__builtin_ia32_extractf64x2_512_mask` 相关的逻辑。
- **L1210 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1210 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1211 EN**: Continues the surrounding expression or declaration: `(__mmask8) - 1))`.
  **L1211 CN**: 继续构造周围的表达式或声明：`(__mmask8) - 1))`。
- **L1212 EN**: Blank line separating nearby declarations or logic blocks.
  **L1212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1213 EN**: Defines macro `_mm512_mask_extractf64x2_pd(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1213 CN**: 定义宏 `_mm512_mask_extractf64x2_pd(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1214 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf64x2_512_mask`.
  **L1214 CN**: 继续与可调用符号 `__builtin_ia32_extractf64x2_512_mask` 相关的逻辑。
- **L1215 EN**: Continues the surrounding expression or declaration: `(int)(imm), \`.
  **L1215 CN**: 继续构造周围的表达式或声明：`(int)(imm), \`。
- **L1216 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(W), \`.
  **L1216 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(W), \`。
- **L1217 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1217 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1218 EN**: Blank line separating nearby declarations or logic blocks.
  **L1218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1219 EN**: Defines macro `_mm512_maskz_extractf64x2_pd(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1219 CN**: 定义宏 `_mm512_maskz_extractf64x2_pd(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1220 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf64x2_512_mask`.
  **L1220 CN**: 继续与可调用符号 `__builtin_ia32_extractf64x2_512_mask` 相关的逻辑。
- **L1221 EN**: Continues the surrounding expression or declaration: `(int)(imm), \`.
  **L1221 CN**: 继续构造周围的表达式或声明：`(int)(imm), \`。
- **L1222 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1222 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1223 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1223 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1224 EN**: Blank line separating nearby declarations or logic blocks.
  **L1224 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1225-1248

````c
#define _mm512_extracti32x8_epi32(A, imm)                                      \
  ((__m256i)__builtin_ia32_extracti32x8_mask(                                  \
      (__v16si)(__m512i)(A), (int)(imm), (__v8si)_mm256_setzero_si256(),       \
      (__mmask8) - 1))

#define _mm512_mask_extracti32x8_epi32(W, U, A, imm) \
  ((__m256i)__builtin_ia32_extracti32x8_mask((__v16si)(__m512i)(A), (int)(imm), \
                                             (__v8si)(__m256i)(W), \
                                             (__mmask8)(U)))

#define _mm512_maskz_extracti32x8_epi32(U, A, imm) \
  ((__m256i)__builtin_ia32_extracti32x8_mask((__v16si)(__m512i)(A), (int)(imm), \
                                             (__v8si)_mm256_setzero_si256(), \
                                             (__mmask8)(U)))

#define _mm512_extracti64x2_epi64(A, imm)                                      \
  ((__m128i)__builtin_ia32_extracti64x2_512_mask(                              \
      (__v8di)(__m512i)(A), (int)(imm), (__v2di)_mm_setzero_si128(),           \
      (__mmask8) - 1))

#define _mm512_mask_extracti64x2_epi64(W, U, A, imm) \
  ((__m128i)__builtin_ia32_extracti64x2_512_mask((__v8di)(__m512i)(A), \
                                                 (int)(imm), \
                                                 (__v2di)(__m128i)(W), \
````
- **L1225 EN**: Defines macro `_mm512_extracti32x8_epi32(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1225 CN**: 定义宏 `_mm512_extracti32x8_epi32(A, imm)`，用于条件编译、简写或 API 生成。
- **L1226 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti32x8_mask`.
  **L1226 CN**: 继续与可调用符号 `__builtin_ia32_extracti32x8_mask` 相关的逻辑。
- **L1227 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L1227 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L1228 EN**: Continues the surrounding expression or declaration: `(__mmask8) - 1))`.
  **L1228 CN**: 继续构造周围的表达式或声明：`(__mmask8) - 1))`。
- **L1229 EN**: Blank line separating nearby declarations or logic blocks.
  **L1229 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1230 EN**: Defines macro `_mm512_mask_extracti32x8_epi32(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1230 CN**: 定义宏 `_mm512_mask_extracti32x8_epi32(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1231 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti32x8_mask`.
  **L1231 CN**: 继续与可调用符号 `__builtin_ia32_extracti32x8_mask` 相关的逻辑。
- **L1232 EN**: Continues the surrounding expression or declaration: `(__v8si)(__m256i)(W), \`.
  **L1232 CN**: 继续构造周围的表达式或声明：`(__v8si)(__m256i)(W), \`。
- **L1233 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1233 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1234 EN**: Blank line separating nearby declarations or logic blocks.
  **L1234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1235 EN**: Defines macro `_mm512_maskz_extracti32x8_epi32(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1235 CN**: 定义宏 `_mm512_maskz_extracti32x8_epi32(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1236 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti32x8_mask`.
  **L1236 CN**: 继续与可调用符号 `__builtin_ia32_extracti32x8_mask` 相关的逻辑。
- **L1237 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L1237 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L1238 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1238 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1239 EN**: Blank line separating nearby declarations or logic blocks.
  **L1239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1240 EN**: Defines macro `_mm512_extracti64x2_epi64(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1240 CN**: 定义宏 `_mm512_extracti64x2_epi64(A, imm)`，用于条件编译、简写或 API 生成。
- **L1241 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti64x2_512_mask`.
  **L1241 CN**: 继续与可调用符号 `__builtin_ia32_extracti64x2_512_mask` 相关的逻辑。
- **L1242 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L1242 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L1243 EN**: Continues the surrounding expression or declaration: `(__mmask8) - 1))`.
  **L1243 CN**: 继续构造周围的表达式或声明：`(__mmask8) - 1))`。
- **L1244 EN**: Blank line separating nearby declarations or logic blocks.
  **L1244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1245 EN**: Defines macro `_mm512_mask_extracti64x2_epi64(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1245 CN**: 定义宏 `_mm512_mask_extracti64x2_epi64(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1246 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti64x2_512_mask`.
  **L1246 CN**: 继续与可调用符号 `__builtin_ia32_extracti64x2_512_mask` 相关的逻辑。
- **L1247 EN**: Continues the surrounding expression or declaration: `(int)(imm), \`.
  **L1247 CN**: 继续构造周围的表达式或声明：`(int)(imm), \`。
- **L1248 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(W), \`.
  **L1248 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(W), \`。

### Lines 1249-1272

````c
                                                 (__mmask8)(U)))

#define _mm512_maskz_extracti64x2_epi64(U, A, imm) \
  ((__m128i)__builtin_ia32_extracti64x2_512_mask((__v8di)(__m512i)(A), \
                                                 (int)(imm), \
                                                 (__v2di)_mm_setzero_si128(), \
                                                 (__mmask8)(U)))

#define _mm512_insertf32x8(A, B, imm) \
  ((__m512)__builtin_ia32_insertf32x8((__v16sf)(__m512)(A), \
                                      (__v8sf)(__m256)(B), (int)(imm)))

#define _mm512_mask_insertf32x8(W, U, A, B, imm) \
  ((__m512)__builtin_ia32_selectps_512((__mmask16)(U), \
                                 (__v16sf)_mm512_insertf32x8((A), (B), (imm)), \
                                 (__v16sf)(__m512)(W)))

#define _mm512_maskz_insertf32x8(U, A, B, imm) \
  ((__m512)__builtin_ia32_selectps_512((__mmask16)(U), \
                                 (__v16sf)_mm512_insertf32x8((A), (B), (imm)), \
                                 (__v16sf)_mm512_setzero_ps()))

#define _mm512_insertf64x2(A, B, imm) \
  ((__m512d)__builtin_ia32_insertf64x2_512((__v8df)(__m512d)(A), \
````
- **L1249 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1249 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1250 EN**: Blank line separating nearby declarations or logic blocks.
  **L1250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1251 EN**: Defines macro `_mm512_maskz_extracti64x2_epi64(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1251 CN**: 定义宏 `_mm512_maskz_extracti64x2_epi64(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1252 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti64x2_512_mask`.
  **L1252 CN**: 继续与可调用符号 `__builtin_ia32_extracti64x2_512_mask` 相关的逻辑。
- **L1253 EN**: Continues the surrounding expression or declaration: `(int)(imm), \`.
  **L1253 CN**: 继续构造周围的表达式或声明：`(int)(imm), \`。
- **L1254 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L1254 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L1255 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1255 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1256 EN**: Blank line separating nearby declarations or logic blocks.
  **L1256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1257 EN**: Defines macro `_mm512_insertf32x8(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1257 CN**: 定义宏 `_mm512_insertf32x8(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1258 EN**: Continues logic associated with callable symbol `__builtin_ia32_insertf32x8`.
  **L1258 CN**: 继续与可调用符号 `__builtin_ia32_insertf32x8` 相关的逻辑。
- **L1259 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(B), (int)(imm)))`.
  **L1259 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(B), (int)(imm)))`。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1261 EN**: Defines macro `_mm512_mask_insertf32x8(W, U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1261 CN**: 定义宏 `_mm512_mask_insertf32x8(W, U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1262 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectps_512`.
  **L1262 CN**: 继续与可调用符号 `__builtin_ia32_selectps_512` 相关的逻辑。
- **L1263 EN**: Continues logic associated with callable symbol `_mm512_insertf32x8`.
  **L1263 CN**: 继续与可调用符号 `_mm512_insertf32x8` 相关的逻辑。
- **L1264 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512)(W)))`.
  **L1264 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512)(W)))`。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Defines macro `_mm512_maskz_insertf32x8(U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1266 CN**: 定义宏 `_mm512_maskz_insertf32x8(U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1267 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectps_512`.
  **L1267 CN**: 继续与可调用符号 `__builtin_ia32_selectps_512` 相关的逻辑。
- **L1268 EN**: Continues logic associated with callable symbol `_mm512_insertf32x8`.
  **L1268 CN**: 继续与可调用符号 `_mm512_insertf32x8` 相关的逻辑。
- **L1269 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L1269 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L1270 EN**: Blank line separating nearby declarations or logic blocks.
  **L1270 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1271 EN**: Defines macro `_mm512_insertf64x2(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1271 CN**: 定义宏 `_mm512_insertf64x2(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1272 EN**: Continues logic associated with callable symbol `__builtin_ia32_insertf64x2_512`.
  **L1272 CN**: 继续与可调用符号 `__builtin_ia32_insertf64x2_512` 相关的逻辑。

### Lines 1273-1296

````c
                                           (__v2df)(__m128d)(B), (int)(imm)))

#define _mm512_mask_insertf64x2(W, U, A, B, imm) \
  ((__m512d)__builtin_ia32_selectpd_512((__mmask8)(U), \
                                  (__v8df)_mm512_insertf64x2((A), (B), (imm)), \
                                  (__v8df)(__m512d)(W)))

#define _mm512_maskz_insertf64x2(U, A, B, imm) \
  ((__m512d)__builtin_ia32_selectpd_512((__mmask8)(U), \
                                  (__v8df)_mm512_insertf64x2((A), (B), (imm)), \
                                  (__v8df)_mm512_setzero_pd()))

#define _mm512_inserti32x8(A, B, imm) \
  ((__m512i)__builtin_ia32_inserti32x8((__v16si)(__m512i)(A), \
                                       (__v8si)(__m256i)(B), (int)(imm)))

#define _mm512_mask_inserti32x8(W, U, A, B, imm) \
  ((__m512i)__builtin_ia32_selectd_512((__mmask16)(U), \
                                 (__v16si)_mm512_inserti32x8((A), (B), (imm)), \
                                 (__v16si)(__m512i)(W)))

#define _mm512_maskz_inserti32x8(U, A, B, imm) \
  ((__m512i)__builtin_ia32_selectd_512((__mmask16)(U), \
                                 (__v16si)_mm512_inserti32x8((A), (B), (imm)), \
````
- **L1273 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), (int)(imm)))`.
  **L1273 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), (int)(imm)))`。
- **L1274 EN**: Blank line separating nearby declarations or logic blocks.
  **L1274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1275 EN**: Defines macro `_mm512_mask_insertf64x2(W, U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1275 CN**: 定义宏 `_mm512_mask_insertf64x2(W, U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1276 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectpd_512`.
  **L1276 CN**: 继续与可调用符号 `__builtin_ia32_selectpd_512` 相关的逻辑。
- **L1277 EN**: Continues logic associated with callable symbol `_mm512_insertf64x2`.
  **L1277 CN**: 继续与可调用符号 `_mm512_insertf64x2` 相关的逻辑。
- **L1278 EN**: Continues the surrounding expression or declaration: `(__v8df)(__m512d)(W)))`.
  **L1278 CN**: 继续构造周围的表达式或声明：`(__v8df)(__m512d)(W)))`。
- **L1279 EN**: Blank line separating nearby declarations or logic blocks.
  **L1279 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1280 EN**: Defines macro `_mm512_maskz_insertf64x2(U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1280 CN**: 定义宏 `_mm512_maskz_insertf64x2(U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1281 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectpd_512`.
  **L1281 CN**: 继续与可调用符号 `__builtin_ia32_selectpd_512` 相关的逻辑。
- **L1282 EN**: Continues logic associated with callable symbol `_mm512_insertf64x2`.
  **L1282 CN**: 继续与可调用符号 `_mm512_insertf64x2` 相关的逻辑。
- **L1283 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L1283 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1285 EN**: Defines macro `_mm512_inserti32x8(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1285 CN**: 定义宏 `_mm512_inserti32x8(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1286 EN**: Continues logic associated with callable symbol `__builtin_ia32_inserti32x8`.
  **L1286 CN**: 继续与可调用符号 `__builtin_ia32_inserti32x8` 相关的逻辑。
- **L1287 EN**: Continues the surrounding expression or declaration: `(__v8si)(__m256i)(B), (int)(imm)))`.
  **L1287 CN**: 继续构造周围的表达式或声明：`(__v8si)(__m256i)(B), (int)(imm)))`。
- **L1288 EN**: Blank line separating nearby declarations or logic blocks.
  **L1288 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1289 EN**: Defines macro `_mm512_mask_inserti32x8(W, U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1289 CN**: 定义宏 `_mm512_mask_inserti32x8(W, U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1290 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_512`.
  **L1290 CN**: 继续与可调用符号 `__builtin_ia32_selectd_512` 相关的逻辑。
- **L1291 EN**: Continues logic associated with callable symbol `_mm512_inserti32x8`.
  **L1291 CN**: 继续与可调用符号 `_mm512_inserti32x8` 相关的逻辑。
- **L1292 EN**: Continues the surrounding expression or declaration: `(__v16si)(__m512i)(W)))`.
  **L1292 CN**: 继续构造周围的表达式或声明：`(__v16si)(__m512i)(W)))`。
- **L1293 EN**: Blank line separating nearby declarations or logic blocks.
  **L1293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1294 EN**: Defines macro `_mm512_maskz_inserti32x8(U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1294 CN**: 定义宏 `_mm512_maskz_inserti32x8(U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1295 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_512`.
  **L1295 CN**: 继续与可调用符号 `__builtin_ia32_selectd_512` 相关的逻辑。
- **L1296 EN**: Continues logic associated with callable symbol `_mm512_inserti32x8`.
  **L1296 CN**: 继续与可调用符号 `_mm512_inserti32x8` 相关的逻辑。

### Lines 1297-1320

````c
                                 (__v16si)_mm512_setzero_si512()))

#define _mm512_inserti64x2(A, B, imm) \
  ((__m512i)__builtin_ia32_inserti64x2_512((__v8di)(__m512i)(A), \
                                           (__v2di)(__m128i)(B), (int)(imm)))

#define _mm512_mask_inserti64x2(W, U, A, B, imm) \
  ((__m512i)__builtin_ia32_selectq_512((__mmask8)(U), \
                                  (__v8di)_mm512_inserti64x2((A), (B), (imm)), \
                                  (__v8di)(__m512i)(W)))

#define _mm512_maskz_inserti64x2(U, A, B, imm) \
  ((__m512i)__builtin_ia32_selectq_512((__mmask8)(U), \
                                  (__v8di)_mm512_inserti64x2((A), (B), (imm)), \
                                  (__v8di)_mm512_setzero_si512()))

#define _mm512_mask_fpclass_ps_mask(U, A, imm) \
  ((__mmask16)__builtin_ia32_fpclassps512_mask((__v16sf)(__m512)(A), \
                                               (int)(imm), (__mmask16)(U)))

#define _mm512_fpclass_ps_mask(A, imm) \
  ((__mmask16)__builtin_ia32_fpclassps512_mask((__v16sf)(__m512)(A), \
                                               (int)(imm), (__mmask16)-1))

````
- **L1297 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L1297 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L1298 EN**: Blank line separating nearby declarations or logic blocks.
  **L1298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1299 EN**: Defines macro `_mm512_inserti64x2(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1299 CN**: 定义宏 `_mm512_inserti64x2(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1300 EN**: Continues logic associated with callable symbol `__builtin_ia32_inserti64x2_512`.
  **L1300 CN**: 继续与可调用符号 `__builtin_ia32_inserti64x2_512` 相关的逻辑。
- **L1301 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(B), (int)(imm)))`.
  **L1301 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(B), (int)(imm)))`。
- **L1302 EN**: Blank line separating nearby declarations or logic blocks.
  **L1302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1303 EN**: Defines macro `_mm512_mask_inserti64x2(W, U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1303 CN**: 定义宏 `_mm512_mask_inserti64x2(W, U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1304 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_512`.
  **L1304 CN**: 继续与可调用符号 `__builtin_ia32_selectq_512` 相关的逻辑。
- **L1305 EN**: Continues logic associated with callable symbol `_mm512_inserti64x2`.
  **L1305 CN**: 继续与可调用符号 `_mm512_inserti64x2` 相关的逻辑。
- **L1306 EN**: Continues the surrounding expression or declaration: `(__v8di)(__m512i)(W)))`.
  **L1306 CN**: 继续构造周围的表达式或声明：`(__v8di)(__m512i)(W)))`。
- **L1307 EN**: Blank line separating nearby declarations or logic blocks.
  **L1307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1308 EN**: Defines macro `_mm512_maskz_inserti64x2(U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1308 CN**: 定义宏 `_mm512_maskz_inserti64x2(U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1309 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_512`.
  **L1309 CN**: 继续与可调用符号 `__builtin_ia32_selectq_512` 相关的逻辑。
- **L1310 EN**: Continues logic associated with callable symbol `_mm512_inserti64x2`.
  **L1310 CN**: 继续与可调用符号 `_mm512_inserti64x2` 相关的逻辑。
- **L1311 EN**: Continues logic associated with callable symbol `_mm512_setzero_si512`.
  **L1311 CN**: 继续与可调用符号 `_mm512_setzero_si512` 相关的逻辑。
- **L1312 EN**: Blank line separating nearby declarations or logic blocks.
  **L1312 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1313 EN**: Defines macro `_mm512_mask_fpclass_ps_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1313 CN**: 定义宏 `_mm512_mask_fpclass_ps_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1314 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassps512_mask`.
  **L1314 CN**: 继续与可调用符号 `__builtin_ia32_fpclassps512_mask` 相关的逻辑。
- **L1315 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask16)(U)))`.
  **L1315 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask16)(U)))`。
- **L1316 EN**: Blank line separating nearby declarations or logic blocks.
  **L1316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1317 EN**: Defines macro `_mm512_fpclass_ps_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1317 CN**: 定义宏 `_mm512_fpclass_ps_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1318 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassps512_mask`.
  **L1318 CN**: 继续与可调用符号 `__builtin_ia32_fpclassps512_mask` 相关的逻辑。
- **L1319 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask16)-1))`.
  **L1319 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask16)-1))`。
- **L1320 EN**: Blank line separating nearby declarations or logic blocks.
  **L1320 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1321-1344

````c
#define _mm512_mask_fpclass_pd_mask(U, A, imm) \
  ((__mmask8)__builtin_ia32_fpclasspd512_mask((__v8df)(__m512d)(A), (int)(imm), \
                                              (__mmask8)(U)))

#define _mm512_fpclass_pd_mask(A, imm) \
  ((__mmask8)__builtin_ia32_fpclasspd512_mask((__v8df)(__m512d)(A), (int)(imm), \
                                              (__mmask8)-1))

#define _mm_fpclass_sd_mask(A, imm) \
  ((__mmask8)__builtin_ia32_fpclasssd_mask((__v2df)(__m128d)(A), (int)(imm), \
                                           (__mmask8)-1))

#define _mm_mask_fpclass_sd_mask(U, A, imm) \
  ((__mmask8)__builtin_ia32_fpclasssd_mask((__v2df)(__m128d)(A), (int)(imm), \
                                           (__mmask8)(U)))

#define _mm_fpclass_ss_mask(A, imm) \
  ((__mmask8)__builtin_ia32_fpclassss_mask((__v4sf)(__m128)(A), (int)(imm), \
                                           (__mmask8)-1))

#define _mm_mask_fpclass_ss_mask(U, A, imm) \
  ((__mmask8)__builtin_ia32_fpclassss_mask((__v4sf)(__m128)(A), (int)(imm), \
                                           (__mmask8)(U)))

````
- **L1321 EN**: Defines macro `_mm512_mask_fpclass_pd_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1321 CN**: 定义宏 `_mm512_mask_fpclass_pd_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1322 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasspd512_mask`.
  **L1322 CN**: 继续与可调用符号 `__builtin_ia32_fpclasspd512_mask` 相关的逻辑。
- **L1323 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1323 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1324 EN**: Blank line separating nearby declarations or logic blocks.
  **L1324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1325 EN**: Defines macro `_mm512_fpclass_pd_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1325 CN**: 定义宏 `_mm512_fpclass_pd_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1326 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasspd512_mask`.
  **L1326 CN**: 继续与可调用符号 `__builtin_ia32_fpclasspd512_mask` 相关的逻辑。
- **L1327 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L1327 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L1328 EN**: Blank line separating nearby declarations or logic blocks.
  **L1328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1329 EN**: Defines macro `_mm_fpclass_sd_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1329 CN**: 定义宏 `_mm_fpclass_sd_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1330 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasssd_mask`.
  **L1330 CN**: 继续与可调用符号 `__builtin_ia32_fpclasssd_mask` 相关的逻辑。
- **L1331 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L1331 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L1332 EN**: Blank line separating nearby declarations or logic blocks.
  **L1332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1333 EN**: Defines macro `_mm_mask_fpclass_sd_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1333 CN**: 定义宏 `_mm_mask_fpclass_sd_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1334 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasssd_mask`.
  **L1334 CN**: 继续与可调用符号 `__builtin_ia32_fpclasssd_mask` 相关的逻辑。
- **L1335 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1335 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1336 EN**: Blank line separating nearby declarations or logic blocks.
  **L1336 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1337 EN**: Defines macro `_mm_fpclass_ss_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1337 CN**: 定义宏 `_mm_fpclass_ss_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1338 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassss_mask`.
  **L1338 CN**: 继续与可调用符号 `__builtin_ia32_fpclassss_mask` 相关的逻辑。
- **L1339 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L1339 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L1340 EN**: Blank line separating nearby declarations or logic blocks.
  **L1340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1341 EN**: Defines macro `_mm_mask_fpclass_ss_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1341 CN**: 定义宏 `_mm_mask_fpclass_ss_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1342 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassss_mask`.
  **L1342 CN**: 继续与可调用符号 `__builtin_ia32_fpclassss_mask` 相关的逻辑。
- **L1343 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1343 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1344 EN**: Blank line separating nearby declarations or logic blocks.
  **L1344 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1345-1350

````c
#undef __DEFAULT_FN_ATTRS512
#undef __DEFAULT_FN_ATTRS
#undef __DEFAULT_FN_ATTRS512_CONSTEXPR
#undef __DEFAULT_FN_ATTRS_CONSTEXPR

#endif
````
- **L1345 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS512`.
  **L1345 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS512`。
- **L1346 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS`.
  **L1346 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS`。
- **L1347 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L1347 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L1348 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1348 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1349 EN**: Blank line separating nearby declarations or logic blocks.
  **L1349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1350 EN**: Closes the current preprocessor conditional block.
  **L1350 CN**: 结束当前预处理条件块。

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
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__AVX512DQINTRIN_H`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_ia32_knotqi`, `__builtin_ia32_kandqi`, `__builtin_ia32_kandnqi`, `__builtin_ia32_korqi`, `__builtin_ia32_kxnorqi`, `__builtin_ia32_kxorqi`, `__builtin_ia32_kortestcqi`, `__builtin_ia32_kortestzqi`, `__builtin_ia32_ktestcqi`, `__builtin_ia32_ktestzqi`, `__builtin_ia32_ktestchi`, `__builtin_ia32_ktestzhi`
