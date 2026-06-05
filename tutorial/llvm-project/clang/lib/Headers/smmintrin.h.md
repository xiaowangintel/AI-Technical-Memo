# smmintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/smmintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: SSE4 intrinsics.
- **Purpose (CN)**: 提供 SSE4 intrinsic 接口。
- **Line Count / 行数**: 2352

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- smmintrin.h - SSE4 intrinsics ------------------------------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __SMMINTRIN_H
#define __SMMINTRIN_H

#if !defined(__i386__) && !defined(__x86_64__)
#error "This header is only meant to be used on x86 and x64 architecture"
#endif

#include <tmmintrin.h>

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS                                                     \
  __attribute__((__always_inline__, __nodebug__, __target__("sse4.1"),         \
                 __min_vector_width__(128)))

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
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __SMMINTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __SMMINTRIN_H`。
- **L11 EN**: Defines macro `__SMMINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `__SMMINTRIN_H`，用于条件编译、简写或 API 生成。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#if !defined(__i386__) && !defined(__x86_64__)`.
  **L13 CN**: 开始一个预处理条件块：`#if !defined(__i386__) && !defined(__x86_64__)`。
- **L14 EN**: Emits a compilation error for an unsupported configuration: `#error "This header is only meant to be used on x86 and x64 architecture"`.
  **L14 CN**: 为不受支持的配置触发编译错误：`#error "This header is only meant to be used on x86 and x64 architecture"`。
- **L15 EN**: Closes the current preprocessor conditional block.
  **L15 CN**: 结束当前预处理条件块。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Includes <tmmintrin.h> to access related header declarations.
  **L17 CN**: 引入 <tmmintrin.h> 以使用相关头文件声明。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L20 EN**: Defines macro `__DEFAULT_FN_ATTRS` for conditional compilation, shorthand, or API generation.
  **L20 CN**: 定义宏 `__DEFAULT_FN_ATTRS`，用于条件编译、简写或 API 生成。
- **L21 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("sse4.1"),         \`.
  **L21 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("sse4.1"),         \`。
- **L22 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L22 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L24 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。

### Lines 25-48

````c
#define __DEFAULT_FN_ATTRS_CONSTEXPR __DEFAULT_FN_ATTRS constexpr
#else
#define __DEFAULT_FN_ATTRS_CONSTEXPR __DEFAULT_FN_ATTRS
#endif

/* SSE4 Rounding macros. */
#define _MM_FROUND_TO_NEAREST_INT 0x00
#define _MM_FROUND_TO_NEG_INF 0x01
#define _MM_FROUND_TO_POS_INF 0x02
#define _MM_FROUND_TO_ZERO 0x03
#define _MM_FROUND_CUR_DIRECTION 0x04

#define _MM_FROUND_RAISE_EXC 0x00
#define _MM_FROUND_NO_EXC 0x08

#define _MM_FROUND_NINT (_MM_FROUND_RAISE_EXC | _MM_FROUND_TO_NEAREST_INT)
#define _MM_FROUND_FLOOR (_MM_FROUND_RAISE_EXC | _MM_FROUND_TO_NEG_INF)
#define _MM_FROUND_CEIL (_MM_FROUND_RAISE_EXC | _MM_FROUND_TO_POS_INF)
#define _MM_FROUND_TRUNC (_MM_FROUND_RAISE_EXC | _MM_FROUND_TO_ZERO)
#define _MM_FROUND_RINT (_MM_FROUND_RAISE_EXC | _MM_FROUND_CUR_DIRECTION)
#define _MM_FROUND_NEARBYINT (_MM_FROUND_NO_EXC | _MM_FROUND_CUR_DIRECTION)

/// Rounds up each element of the 128-bit vector of [4 x float] to an
///    integer and returns the rounded values in a 128-bit vector of
````
- **L25 EN**: Defines macro `__DEFAULT_FN_ATTRS_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L25 CN**: 定义宏 `__DEFAULT_FN_ATTRS_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L26 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L26 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L27 EN**: Defines macro `__DEFAULT_FN_ATTRS_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L27 CN**: 定义宏 `__DEFAULT_FN_ATTRS_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L28 EN**: Closes the current preprocessor conditional block.
  **L28 CN**: 结束当前预处理条件块。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L30 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Rounding macros.`.
  **L30 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Rounding macros.`。
- **L31 EN**: Defines macro `_MM_FROUND_TO_NEAREST_INT` for conditional compilation, shorthand, or API generation.
  **L31 CN**: 定义宏 `_MM_FROUND_TO_NEAREST_INT`，用于条件编译、简写或 API 生成。
- **L32 EN**: Defines macro `_MM_FROUND_TO_NEG_INF` for conditional compilation, shorthand, or API generation.
  **L32 CN**: 定义宏 `_MM_FROUND_TO_NEG_INF`，用于条件编译、简写或 API 生成。
- **L33 EN**: Defines macro `_MM_FROUND_TO_POS_INF` for conditional compilation, shorthand, or API generation.
  **L33 CN**: 定义宏 `_MM_FROUND_TO_POS_INF`，用于条件编译、简写或 API 生成。
- **L34 EN**: Defines macro `_MM_FROUND_TO_ZERO` for conditional compilation, shorthand, or API generation.
  **L34 CN**: 定义宏 `_MM_FROUND_TO_ZERO`，用于条件编译、简写或 API 生成。
- **L35 EN**: Defines macro `_MM_FROUND_CUR_DIRECTION` for conditional compilation, shorthand, or API generation.
  **L35 CN**: 定义宏 `_MM_FROUND_CUR_DIRECTION`，用于条件编译、简写或 API 生成。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L37 EN**: Defines macro `_MM_FROUND_RAISE_EXC` for conditional compilation, shorthand, or API generation.
  **L37 CN**: 定义宏 `_MM_FROUND_RAISE_EXC`，用于条件编译、简写或 API 生成。
- **L38 EN**: Defines macro `_MM_FROUND_NO_EXC` for conditional compilation, shorthand, or API generation.
  **L38 CN**: 定义宏 `_MM_FROUND_NO_EXC`，用于条件编译、简写或 API 生成。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L40 EN**: Defines macro `_MM_FROUND_NINT` for conditional compilation, shorthand, or API generation.
  **L40 CN**: 定义宏 `_MM_FROUND_NINT`，用于条件编译、简写或 API 生成。
- **L41 EN**: Defines macro `_MM_FROUND_FLOOR` for conditional compilation, shorthand, or API generation.
  **L41 CN**: 定义宏 `_MM_FROUND_FLOOR`，用于条件编译、简写或 API 生成。
- **L42 EN**: Defines macro `_MM_FROUND_CEIL` for conditional compilation, shorthand, or API generation.
  **L42 CN**: 定义宏 `_MM_FROUND_CEIL`，用于条件编译、简写或 API 生成。
- **L43 EN**: Defines macro `_MM_FROUND_TRUNC` for conditional compilation, shorthand, or API generation.
  **L43 CN**: 定义宏 `_MM_FROUND_TRUNC`，用于条件编译、简写或 API 生成。
- **L44 EN**: Defines macro `_MM_FROUND_RINT` for conditional compilation, shorthand, or API generation.
  **L44 CN**: 定义宏 `_MM_FROUND_RINT`，用于条件编译、简写或 API 生成。
- **L45 EN**: Defines macro `_MM_FROUND_NEARBYINT` for conditional compilation, shorthand, or API generation.
  **L45 CN**: 定义宏 `_MM_FROUND_NEARBYINT`，用于条件编译、简写或 API 生成。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Comment explains nearby logic, constraints, or intent: `Rounds up each element of the 128-bit vector of [4 x float] to an`.
  **L47 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds up each element of the 128-bit vector of [4 x float] to an`。
- **L48 EN**: Comment explains nearby logic, constraints, or intent: `integer and returns the rounded values in a 128-bit vector of`.
  **L48 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer and returns the rounded values in a 128-bit vector of`。

### Lines 49-72

````c
///    [4 x float].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128 _mm_ceil_ps(__m128 X);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.
///
/// \param X
///    A 128-bit vector of [4 x float] values to be rounded up.
/// \returns A 128-bit vector of [4 x float] containing the rounded values.
#define _mm_ceil_ps(X) _mm_round_ps((X), _MM_FROUND_CEIL)

/// Rounds up each element of the 128-bit vector of [2 x double] to an
///    integer and returns the rounded values in a 128-bit vector of
///    [2 x double].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128d _mm_ceil_pd(__m128d X);
/// \endcode
````
- **L49 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float].`.
  **L49 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float].`。
- **L50 EN**: Separator comment used for visual grouping.
  **L50 CN**: 用于视觉分组的分隔注释。
- **L51 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L51 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L52 EN**: Separator comment used for visual grouping.
  **L52 CN**: 用于视觉分组的分隔注释。
- **L53 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L53 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L54 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_ceil_ps(__m128 X);`.
  **L54 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_ceil_ps(__m128 X);`。
- **L55 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L55 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L56 EN**: Separator comment used for visual grouping.
  **L56 CN**: 用于视觉分组的分隔注释。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.`。
- **L58 EN**: Separator comment used for visual grouping.
  **L58 CN**: 用于视觉分组的分隔注释。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L60 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] values to be rounded up.`.
  **L60 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] values to be rounded up.`。
- **L61 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the rounded values.`.
  **L61 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the rounded values.`。
- **L62 EN**: Defines macro `_mm_ceil_ps(X)` for conditional compilation, shorthand, or API generation.
  **L62 CN**: 定义宏 `_mm_ceil_ps(X)`，用于条件编译、简写或 API 生成。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `Rounds up each element of the 128-bit vector of [2 x double] to an`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds up each element of the 128-bit vector of [2 x double] to an`。
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `integer and returns the rounded values in a 128-bit vector of`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer and returns the rounded values in a 128-bit vector of`。
- **L66 EN**: Comment explains nearby logic, constraints, or intent: `[2 x double].`.
  **L66 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x double].`。
- **L67 EN**: Separator comment used for visual grouping.
  **L67 CN**: 用于视觉分组的分隔注释。
- **L68 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L68 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L69 EN**: Separator comment used for visual grouping.
  **L69 CN**: 用于视觉分组的分隔注释。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L71 EN**: Comment explains nearby logic, constraints, or intent: `__m128d _mm_ceil_pd(__m128d X);`.
  **L71 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128d _mm_ceil_pd(__m128d X);`。
- **L72 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L72 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。

### Lines 73-96

````c
///
/// This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.
///
/// \param X
///    A 128-bit vector of [2 x double] values to be rounded up.
/// \returns A 128-bit vector of [2 x double] containing the rounded values.
#define _mm_ceil_pd(X) _mm_round_pd((X), _MM_FROUND_CEIL)

/// Copies three upper elements of the first 128-bit vector operand to
///    the corresponding three upper elements of the 128-bit result vector of
///    [4 x float]. Rounds up the lowest element of the second 128-bit vector
///    operand to an integer and copies it to the lowest element of the 128-bit
///    result vector of [4 x float].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128 _mm_ceil_ss(__m128 X, __m128 Y);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.
///
/// \param X
///    A 128-bit vector of [4 x float]. The values stored in bits [127:32] are
````
- **L73 EN**: Separator comment used for visual grouping.
  **L73 CN**: 用于视觉分组的分隔注释。
- **L74 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.`.
  **L74 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.`。
- **L75 EN**: Separator comment used for visual grouping.
  **L75 CN**: 用于视觉分组的分隔注释。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L77 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] values to be rounded up.`.
  **L77 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] values to be rounded up.`。
- **L78 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the rounded values.`.
  **L78 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the rounded values.`。
- **L79 EN**: Defines macro `_mm_ceil_pd(X)` for conditional compilation, shorthand, or API generation.
  **L79 CN**: 定义宏 `_mm_ceil_pd(X)`，用于条件编译、简写或 API 生成。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L81 EN**: Comment explains nearby logic, constraints, or intent: `Copies three upper elements of the first 128-bit vector operand to`.
  **L81 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Copies three upper elements of the first 128-bit vector operand to`。
- **L82 EN**: Comment explains nearby logic, constraints, or intent: `the corresponding three upper elements of the 128-bit result vector of`.
  **L82 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the corresponding three upper elements of the 128-bit result vector of`。
- **L83 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float]. Rounds up the lowest element of the second 128-bit vector`.
  **L83 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float]. Rounds up the lowest element of the second 128-bit vector`。
- **L84 EN**: Comment explains nearby logic, constraints, or intent: `operand to an integer and copies it to the lowest element of the 128-bit`.
  **L84 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operand to an integer and copies it to the lowest element of the 128-bit`。
- **L85 EN**: Comment explains nearby logic, constraints, or intent: `result vector of [4 x float].`.
  **L85 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result vector of [4 x float].`。
- **L86 EN**: Separator comment used for visual grouping.
  **L86 CN**: 用于视觉分组的分隔注释。
- **L87 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L87 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L88 EN**: Separator comment used for visual grouping.
  **L88 CN**: 用于视觉分组的分隔注释。
- **L89 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L89 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L90 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_ceil_ss(__m128 X, __m128 Y);`.
  **L90 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_ceil_ss(__m128 X, __m128 Y);`。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L92 EN**: Separator comment used for visual grouping.
  **L92 CN**: 用于视觉分组的分隔注释。
- **L93 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.`.
  **L93 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.`。
- **L94 EN**: Separator comment used for visual grouping.
  **L94 CN**: 用于视觉分组的分隔注释。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float]. The values stored in bits [127:32] are`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float]. The values stored in bits [127:32] are`。

### Lines 97-120

````c
///    copied to the corresponding bits of the result.
/// \param Y
///    A 128-bit vector of [4 x float]. The value stored in bits [31:0] is
///    rounded up to the nearest integer and copied to the corresponding bits
///    of the result.
/// \returns A 128-bit vector of [4 x float] containing the copied and rounded
///    values.
#define _mm_ceil_ss(X, Y) _mm_round_ss((X), (Y), _MM_FROUND_CEIL)

/// Copies the upper element of the first 128-bit vector operand to the
///    corresponding upper element of the 128-bit result vector of [2 x double].
///    Rounds up the lower element of the second 128-bit vector operand to an
///    integer and copies it to the lower element of the 128-bit result vector
///    of [2 x double].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128d _mm_ceil_sd(__m128d X, __m128d Y);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.
///
/// \param X
````
- **L97 EN**: Comment explains nearby logic, constraints, or intent: `copied to the corresponding bits of the result.`.
  **L97 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copied to the corresponding bits of the result.`。
- **L98 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L98 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L99 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float]. The value stored in bits [31:0] is`.
  **L99 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float]. The value stored in bits [31:0] is`。
- **L100 EN**: Comment explains nearby logic, constraints, or intent: `rounded up to the nearest integer and copied to the corresponding bits`.
  **L100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`rounded up to the nearest integer and copied to the corresponding bits`。
- **L101 EN**: Comment explains nearby logic, constraints, or intent: `of the result.`.
  **L101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the result.`。
- **L102 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the copied and rounded`.
  **L102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the copied and rounded`。
- **L103 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L104 EN**: Defines macro `_mm_ceil_ss(X, Y)` for conditional compilation, shorthand, or API generation.
  **L104 CN**: 定义宏 `_mm_ceil_ss(X, Y)`，用于条件编译、简写或 API 生成。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L106 EN**: Comment explains nearby logic, constraints, or intent: `Copies the upper element of the first 128-bit vector operand to the`.
  **L106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Copies the upper element of the first 128-bit vector operand to the`。
- **L107 EN**: Comment explains nearby logic, constraints, or intent: `corresponding upper element of the 128-bit result vector of [2 x double].`.
  **L107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding upper element of the 128-bit result vector of [2 x double].`。
- **L108 EN**: Comment explains nearby logic, constraints, or intent: `Rounds up the lower element of the second 128-bit vector operand to an`.
  **L108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds up the lower element of the second 128-bit vector operand to an`。
- **L109 EN**: Comment explains nearby logic, constraints, or intent: `integer and copies it to the lower element of the 128-bit result vector`.
  **L109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer and copies it to the lower element of the 128-bit result vector`。
- **L110 EN**: Comment explains nearby logic, constraints, or intent: `of [2 x double].`.
  **L110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [2 x double].`。
- **L111 EN**: Separator comment used for visual grouping.
  **L111 CN**: 用于视觉分组的分隔注释。
- **L112 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L112 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L113 EN**: Separator comment used for visual grouping.
  **L113 CN**: 用于视觉分组的分隔注释。
- **L114 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L115 EN**: Comment explains nearby logic, constraints, or intent: `__m128d _mm_ceil_sd(__m128d X, __m128d Y);`.
  **L115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128d _mm_ceil_sd(__m128d X, __m128d Y);`。
- **L116 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L117 EN**: Separator comment used for visual grouping.
  **L117 CN**: 用于视觉分组的分隔注释。
- **L118 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.`.
  **L118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.`。
- **L119 EN**: Separator comment used for visual grouping.
  **L119 CN**: 用于视觉分组的分隔注释。
- **L120 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。

### Lines 121-144

````c
///    A 128-bit vector of [2 x double]. The value stored in bits [127:64] is
///    copied to the corresponding bits of the result.
/// \param Y
///    A 128-bit vector of [2 x double]. The value stored in bits [63:0] is
///    rounded up to the nearest integer and copied to the corresponding bits
///    of the result.
/// \returns A 128-bit vector of [2 x double] containing the copied and rounded
///    values.
#define _mm_ceil_sd(X, Y) _mm_round_sd((X), (Y), _MM_FROUND_CEIL)

/// Rounds down each element of the 128-bit vector of [4 x float] to an
///    an integer and returns the rounded values in a 128-bit vector of
///    [4 x float].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128 _mm_floor_ps(__m128 X);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.
///
/// \param X
///    A 128-bit vector of [4 x float] values to be rounded down.
````
- **L121 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double]. The value stored in bits [127:64] is`.
  **L121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double]. The value stored in bits [127:64] is`。
- **L122 EN**: Comment explains nearby logic, constraints, or intent: `copied to the corresponding bits of the result.`.
  **L122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copied to the corresponding bits of the result.`。
- **L123 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L124 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double]. The value stored in bits [63:0] is`.
  **L124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double]. The value stored in bits [63:0] is`。
- **L125 EN**: Comment explains nearby logic, constraints, or intent: `rounded up to the nearest integer and copied to the corresponding bits`.
  **L125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`rounded up to the nearest integer and copied to the corresponding bits`。
- **L126 EN**: Comment explains nearby logic, constraints, or intent: `of the result.`.
  **L126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the result.`。
- **L127 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the copied and rounded`.
  **L127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the copied and rounded`。
- **L128 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L129 EN**: Defines macro `_mm_ceil_sd(X, Y)` for conditional compilation, shorthand, or API generation.
  **L129 CN**: 定义宏 `_mm_ceil_sd(X, Y)`，用于条件编译、简写或 API 生成。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L131 EN**: Comment explains nearby logic, constraints, or intent: `Rounds down each element of the 128-bit vector of [4 x float] to an`.
  **L131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds down each element of the 128-bit vector of [4 x float] to an`。
- **L132 EN**: Comment explains nearby logic, constraints, or intent: `an integer and returns the rounded values in a 128-bit vector of`.
  **L132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`an integer and returns the rounded values in a 128-bit vector of`。
- **L133 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float].`.
  **L133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float].`。
- **L134 EN**: Separator comment used for visual grouping.
  **L134 CN**: 用于视觉分组的分隔注释。
- **L135 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L136 EN**: Separator comment used for visual grouping.
  **L136 CN**: 用于视觉分组的分隔注释。
- **L137 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L138 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_floor_ps(__m128 X);`.
  **L138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_floor_ps(__m128 X);`。
- **L139 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L140 EN**: Separator comment used for visual grouping.
  **L140 CN**: 用于视觉分组的分隔注释。
- **L141 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.`.
  **L141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.`。
- **L142 EN**: Separator comment used for visual grouping.
  **L142 CN**: 用于视觉分组的分隔注释。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L144 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] values to be rounded down.`.
  **L144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] values to be rounded down.`。

### Lines 145-168

````c
/// \returns A 128-bit vector of [4 x float] containing the rounded values.
#define _mm_floor_ps(X) _mm_round_ps((X), _MM_FROUND_FLOOR)

/// Rounds down each element of the 128-bit vector of [2 x double] to an
///    integer and returns the rounded values in a 128-bit vector of
///    [2 x double].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128d _mm_floor_pd(__m128d X);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.
///
/// \param X
///    A 128-bit vector of [2 x double].
/// \returns A 128-bit vector of [2 x double] containing the rounded values.
#define _mm_floor_pd(X) _mm_round_pd((X), _MM_FROUND_FLOOR)

/// Copies three upper elements of the first 128-bit vector operand to
///    the corresponding three upper elements of the 128-bit result vector of
///    [4 x float]. Rounds down the lowest element of the second 128-bit vector
///    operand to an integer and copies it to the lowest element of the 128-bit
````
- **L145 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the rounded values.`.
  **L145 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the rounded values.`。
- **L146 EN**: Defines macro `_mm_floor_ps(X)` for conditional compilation, shorthand, or API generation.
  **L146 CN**: 定义宏 `_mm_floor_ps(X)`，用于条件编译、简写或 API 生成。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Comment explains nearby logic, constraints, or intent: `Rounds down each element of the 128-bit vector of [2 x double] to an`.
  **L148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds down each element of the 128-bit vector of [2 x double] to an`。
- **L149 EN**: Comment explains nearby logic, constraints, or intent: `integer and returns the rounded values in a 128-bit vector of`.
  **L149 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer and returns the rounded values in a 128-bit vector of`。
- **L150 EN**: Comment explains nearby logic, constraints, or intent: `[2 x double].`.
  **L150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x double].`。
- **L151 EN**: Separator comment used for visual grouping.
  **L151 CN**: 用于视觉分组的分隔注释。
- **L152 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L153 EN**: Separator comment used for visual grouping.
  **L153 CN**: 用于视觉分组的分隔注释。
- **L154 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L155 EN**: Comment explains nearby logic, constraints, or intent: `__m128d _mm_floor_pd(__m128d X);`.
  **L155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128d _mm_floor_pd(__m128d X);`。
- **L156 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L156 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L157 EN**: Separator comment used for visual grouping.
  **L157 CN**: 用于视觉分组的分隔注释。
- **L158 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.`.
  **L158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.`。
- **L159 EN**: Separator comment used for visual grouping.
  **L159 CN**: 用于视觉分组的分隔注释。
- **L160 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L161 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double].`.
  **L161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double].`。
- **L162 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the rounded values.`.
  **L162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the rounded values.`。
- **L163 EN**: Defines macro `_mm_floor_pd(X)` for conditional compilation, shorthand, or API generation.
  **L163 CN**: 定义宏 `_mm_floor_pd(X)`，用于条件编译、简写或 API 生成。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `Copies three upper elements of the first 128-bit vector operand to`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Copies three upper elements of the first 128-bit vector operand to`。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `the corresponding three upper elements of the 128-bit result vector of`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the corresponding three upper elements of the 128-bit result vector of`。
- **L167 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float]. Rounds down the lowest element of the second 128-bit vector`.
  **L167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float]. Rounds down the lowest element of the second 128-bit vector`。
- **L168 EN**: Comment explains nearby logic, constraints, or intent: `operand to an integer and copies it to the lowest element of the 128-bit`.
  **L168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operand to an integer and copies it to the lowest element of the 128-bit`。

### Lines 169-192

````c
///    result vector of [4 x float].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128 _mm_floor_ss(__m128 X, __m128 Y);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.
///
/// \param X
///    A 128-bit vector of [4 x float]. The values stored in bits [127:32] are
///    copied to the corresponding bits of the result.
/// \param Y
///    A 128-bit vector of [4 x float]. The value stored in bits [31:0] is
///    rounded down to the nearest integer and copied to the corresponding bits
///    of the result.
/// \returns A 128-bit vector of [4 x float] containing the copied and rounded
///    values.
#define _mm_floor_ss(X, Y) _mm_round_ss((X), (Y), _MM_FROUND_FLOOR)

/// Copies the upper element of the first 128-bit vector operand to the
///    corresponding upper element of the 128-bit result vector of [2 x double].
///    Rounds down the lower element of the second 128-bit vector operand to an
````
- **L169 EN**: Comment explains nearby logic, constraints, or intent: `result vector of [4 x float].`.
  **L169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result vector of [4 x float].`。
- **L170 EN**: Separator comment used for visual grouping.
  **L170 CN**: 用于视觉分组的分隔注释。
- **L171 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L171 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L172 EN**: Separator comment used for visual grouping.
  **L172 CN**: 用于视觉分组的分隔注释。
- **L173 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L174 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_floor_ss(__m128 X, __m128 Y);`.
  **L174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_floor_ss(__m128 X, __m128 Y);`。
- **L175 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L176 EN**: Separator comment used for visual grouping.
  **L176 CN**: 用于视觉分组的分隔注释。
- **L177 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.`.
  **L177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.`。
- **L178 EN**: Separator comment used for visual grouping.
  **L178 CN**: 用于视觉分组的分隔注释。
- **L179 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L180 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float]. The values stored in bits [127:32] are`.
  **L180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float]. The values stored in bits [127:32] are`。
- **L181 EN**: Comment explains nearby logic, constraints, or intent: `copied to the corresponding bits of the result.`.
  **L181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copied to the corresponding bits of the result.`。
- **L182 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L183 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float]. The value stored in bits [31:0] is`.
  **L183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float]. The value stored in bits [31:0] is`。
- **L184 EN**: Comment explains nearby logic, constraints, or intent: `rounded down to the nearest integer and copied to the corresponding bits`.
  **L184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`rounded down to the nearest integer and copied to the corresponding bits`。
- **L185 EN**: Comment explains nearby logic, constraints, or intent: `of the result.`.
  **L185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the result.`。
- **L186 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the copied and rounded`.
  **L186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the copied and rounded`。
- **L187 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L188 EN**: Defines macro `_mm_floor_ss(X, Y)` for conditional compilation, shorthand, or API generation.
  **L188 CN**: 定义宏 `_mm_floor_ss(X, Y)`，用于条件编译、简写或 API 生成。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L190 EN**: Comment explains nearby logic, constraints, or intent: `Copies the upper element of the first 128-bit vector operand to the`.
  **L190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Copies the upper element of the first 128-bit vector operand to the`。
- **L191 EN**: Comment explains nearby logic, constraints, or intent: `corresponding upper element of the 128-bit result vector of [2 x double].`.
  **L191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding upper element of the 128-bit result vector of [2 x double].`。
- **L192 EN**: Comment explains nearby logic, constraints, or intent: `Rounds down the lower element of the second 128-bit vector operand to an`.
  **L192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds down the lower element of the second 128-bit vector operand to an`。

### Lines 193-216

````c
///    integer and copies it to the lower element of the 128-bit result vector
///    of [2 x double].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128d _mm_floor_sd(__m128d X, __m128d Y);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.
///
/// \param X
///    A 128-bit vector of [2 x double]. The value stored in bits [127:64] is
///    copied to the corresponding bits of the result.
/// \param Y
///    A 128-bit vector of [2 x double]. The value stored in bits [63:0] is
///    rounded down to the nearest integer and copied to the corresponding bits
///    of the result.
/// \returns A 128-bit vector of [2 x double] containing the copied and rounded
///    values.
#define _mm_floor_sd(X, Y) _mm_round_sd((X), (Y), _MM_FROUND_FLOOR)

/// Rounds each element of the 128-bit vector of [4 x float] to an
///    integer value according to the rounding control specified by the second
````
- **L193 EN**: Comment explains nearby logic, constraints, or intent: `integer and copies it to the lower element of the 128-bit result vector`.
  **L193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer and copies it to the lower element of the 128-bit result vector`。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `of [2 x double].`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [2 x double].`。
- **L195 EN**: Separator comment used for visual grouping.
  **L195 CN**: 用于视觉分组的分隔注释。
- **L196 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L197 EN**: Separator comment used for visual grouping.
  **L197 CN**: 用于视觉分组的分隔注释。
- **L198 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L199 EN**: Comment explains nearby logic, constraints, or intent: `__m128d _mm_floor_sd(__m128d X, __m128d Y);`.
  **L199 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128d _mm_floor_sd(__m128d X, __m128d Y);`。
- **L200 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L201 EN**: Separator comment used for visual grouping.
  **L201 CN**: 用于视觉分组的分隔注释。
- **L202 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.`.
  **L202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.`。
- **L203 EN**: Separator comment used for visual grouping.
  **L203 CN**: 用于视觉分组的分隔注释。
- **L204 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L205 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double]. The value stored in bits [127:64] is`.
  **L205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double]. The value stored in bits [127:64] is`。
- **L206 EN**: Comment explains nearby logic, constraints, or intent: `copied to the corresponding bits of the result.`.
  **L206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copied to the corresponding bits of the result.`。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L208 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double]. The value stored in bits [63:0] is`.
  **L208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double]. The value stored in bits [63:0] is`。
- **L209 EN**: Comment explains nearby logic, constraints, or intent: `rounded down to the nearest integer and copied to the corresponding bits`.
  **L209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`rounded down to the nearest integer and copied to the corresponding bits`。
- **L210 EN**: Comment explains nearby logic, constraints, or intent: `of the result.`.
  **L210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the result.`。
- **L211 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the copied and rounded`.
  **L211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the copied and rounded`。
- **L212 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L213 EN**: Defines macro `_mm_floor_sd(X, Y)` for conditional compilation, shorthand, or API generation.
  **L213 CN**: 定义宏 `_mm_floor_sd(X, Y)`，用于条件编译、简写或 API 生成。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L215 EN**: Comment explains nearby logic, constraints, or intent: `Rounds each element of the 128-bit vector of [4 x float] to an`.
  **L215 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds each element of the 128-bit vector of [4 x float] to an`。
- **L216 EN**: Comment explains nearby logic, constraints, or intent: `integer value according to the rounding control specified by the second`.
  **L216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer value according to the rounding control specified by the second`。

### Lines 217-240

````c
///    argument and returns the rounded values in a 128-bit vector of
///    [4 x float].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128 _mm_round_ps(__m128 X, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.
///
/// \param X
///    A 128-bit vector of [4 x float].
/// \param M
///    An integer value that specifies the rounding operation. \n
///    Bits [7:4] are reserved. \n
///    Bit [3] is a precision exception value: \n
///      0: A normal PE exception is used \n
///      1: The PE field is not updated \n
///    Bit [2] is the rounding control source: \n
///      0: Use bits [1:0] of \a M \n
///      1: Use the current MXCSR setting \n
///    Bits [1:0] contain the rounding control definition: \n
///      00: Nearest \n
````
- **L217 EN**: Comment explains nearby logic, constraints, or intent: `argument and returns the rounded values in a 128-bit vector of`.
  **L217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`argument and returns the rounded values in a 128-bit vector of`。
- **L218 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float].`.
  **L218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float].`。
- **L219 EN**: Separator comment used for visual grouping.
  **L219 CN**: 用于视觉分组的分隔注释。
- **L220 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L221 EN**: Separator comment used for visual grouping.
  **L221 CN**: 用于视觉分组的分隔注释。
- **L222 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L222 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L223 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_round_ps(__m128 X, const int M);`.
  **L223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_round_ps(__m128 X, const int M);`。
- **L224 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L225 EN**: Separator comment used for visual grouping.
  **L225 CN**: 用于视觉分组的分隔注释。
- **L226 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.`.
  **L226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDPS / ROUNDPS </c> instruction.`。
- **L227 EN**: Separator comment used for visual grouping.
  **L227 CN**: 用于视觉分组的分隔注释。
- **L228 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L229 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L230 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L231 EN**: Comment explains nearby logic, constraints, or intent: `An integer value that specifies the rounding operation. n`.
  **L231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer value that specifies the rounding operation. n`。
- **L232 EN**: Comment explains nearby logic, constraints, or intent: `Bits [7:4] are reserved. n`.
  **L232 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [7:4] are reserved. n`。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `Bit [3] is a precision exception value: n`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [3] is a precision exception value: n`。
- **L234 EN**: Comment explains nearby logic, constraints, or intent: `0: A normal PE exception is used n`.
  **L234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: A normal PE exception is used n`。
- **L235 EN**: Comment explains nearby logic, constraints, or intent: `1: The PE field is not updated n`.
  **L235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: The PE field is not updated n`。
- **L236 EN**: Comment explains nearby logic, constraints, or intent: `Bit [2] is the rounding control source: n`.
  **L236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [2] is the rounding control source: n`。
- **L237 EN**: Comment explains nearby logic, constraints, or intent: `0: Use bits [1:0] of a M n`.
  **L237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: Use bits [1:0] of a M n`。
- **L238 EN**: Comment explains nearby logic, constraints, or intent: `1: Use the current MXCSR setting n`.
  **L238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: Use the current MXCSR setting n`。
- **L239 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0] contain the rounding control definition: n`.
  **L239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0] contain the rounding control definition: n`。
- **L240 EN**: Comment explains nearby logic, constraints, or intent: `00: Nearest n`.
  **L240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Nearest n`。

### Lines 241-264

````c
///      01: Downward (toward negative infinity) \n
///      10: Upward (toward positive infinity) \n
///      11: Truncated
/// \returns A 128-bit vector of [4 x float] containing the rounded values.
#define _mm_round_ps(X, M)                                                     \
  ((__m128)__builtin_ia32_roundps((__v4sf)(__m128)(X), (M)))

/// Copies three upper elements of the first 128-bit vector operand to
///    the corresponding three upper elements of the 128-bit result vector of
///    [4 x float]. Rounds the lowest element of the second 128-bit vector
///    operand to an integer value according to the rounding control specified
///    by the third argument and copies it to the lowest element of the 128-bit
///    result vector of [4 x float].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128 _mm_round_ss(__m128 X, __m128 Y, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.
///
/// \param X
///    A 128-bit vector of [4 x float]. The values stored in bits [127:32] are
````
- **L241 EN**: Comment explains nearby logic, constraints, or intent: `01: Downward (toward negative infinity) n`.
  **L241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Downward (toward negative infinity) n`。
- **L242 EN**: Comment explains nearby logic, constraints, or intent: `10: Upward (toward positive infinity) n`.
  **L242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Upward (toward positive infinity) n`。
- **L243 EN**: Comment explains nearby logic, constraints, or intent: `11: Truncated`.
  **L243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Truncated`。
- **L244 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the rounded values.`.
  **L244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the rounded values.`。
- **L245 EN**: Defines macro `_mm_round_ps(X, M)` for conditional compilation, shorthand, or API generation.
  **L245 CN**: 定义宏 `_mm_round_ps(X, M)`，用于条件编译、简写或 API 生成。
- **L246 EN**: Continues logic associated with callable symbol `__builtin_ia32_roundps`.
  **L246 CN**: 继续与可调用符号 `__builtin_ia32_roundps` 相关的逻辑。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L248 EN**: Comment explains nearby logic, constraints, or intent: `Copies three upper elements of the first 128-bit vector operand to`.
  **L248 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Copies three upper elements of the first 128-bit vector operand to`。
- **L249 EN**: Comment explains nearby logic, constraints, or intent: `the corresponding three upper elements of the 128-bit result vector of`.
  **L249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the corresponding three upper elements of the 128-bit result vector of`。
- **L250 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float]. Rounds the lowest element of the second 128-bit vector`.
  **L250 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float]. Rounds the lowest element of the second 128-bit vector`。
- **L251 EN**: Comment explains nearby logic, constraints, or intent: `operand to an integer value according to the rounding control specified`.
  **L251 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operand to an integer value according to the rounding control specified`。
- **L252 EN**: Comment explains nearby logic, constraints, or intent: `by the third argument and copies it to the lowest element of the 128-bit`.
  **L252 CN**: 注释解释附近代码的逻辑、约束或设计意图：`by the third argument and copies it to the lowest element of the 128-bit`。
- **L253 EN**: Comment explains nearby logic, constraints, or intent: `result vector of [4 x float].`.
  **L253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result vector of [4 x float].`。
- **L254 EN**: Separator comment used for visual grouping.
  **L254 CN**: 用于视觉分组的分隔注释。
- **L255 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L256 EN**: Separator comment used for visual grouping.
  **L256 CN**: 用于视觉分组的分隔注释。
- **L257 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L258 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_round_ss(__m128 X, __m128 Y, const int M);`.
  **L258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_round_ss(__m128 X, __m128 Y, const int M);`。
- **L259 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L260 EN**: Separator comment used for visual grouping.
  **L260 CN**: 用于视觉分组的分隔注释。
- **L261 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.`.
  **L261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDSS / ROUNDSS </c> instruction.`。
- **L262 EN**: Separator comment used for visual grouping.
  **L262 CN**: 用于视觉分组的分隔注释。
- **L263 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L264 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float]. The values stored in bits [127:32] are`.
  **L264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float]. The values stored in bits [127:32] are`。

### Lines 265-288

````c
///    copied to the corresponding bits of the result.
/// \param Y
///    A 128-bit vector of [4 x float]. The value stored in bits [31:0] is
///    rounded to the nearest integer using the specified rounding control and
///    copied to the corresponding bits of the result.
/// \param M
///    An integer value that specifies the rounding operation. \n
///    Bits [7:4] are reserved. \n
///    Bit [3] is a precision exception value: \n
///      0: A normal PE exception is used \n
///      1: The PE field is not updated \n
///    Bit [2] is the rounding control source: \n
///      0: Use bits [1:0] of \a M \n
///      1: Use the current MXCSR setting \n
///    Bits [1:0] contain the rounding control definition: \n
///      00: Nearest \n
///      01: Downward (toward negative infinity) \n
///      10: Upward (toward positive infinity) \n
///      11: Truncated
/// \returns A 128-bit vector of [4 x float] containing the copied and rounded
///    values.
#define _mm_round_ss(X, Y, M)                                                  \
  ((__m128)__builtin_ia32_roundss((__v4sf)(__m128)(X), (__v4sf)(__m128)(Y),    \
                                  (M)))
````
- **L265 EN**: Comment explains nearby logic, constraints, or intent: `copied to the corresponding bits of the result.`.
  **L265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copied to the corresponding bits of the result.`。
- **L266 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L267 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float]. The value stored in bits [31:0] is`.
  **L267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float]. The value stored in bits [31:0] is`。
- **L268 EN**: Comment explains nearby logic, constraints, or intent: `rounded to the nearest integer using the specified rounding control and`.
  **L268 CN**: 注释解释附近代码的逻辑、约束或设计意图：`rounded to the nearest integer using the specified rounding control and`。
- **L269 EN**: Comment explains nearby logic, constraints, or intent: `copied to the corresponding bits of the result.`.
  **L269 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copied to the corresponding bits of the result.`。
- **L270 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L270 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L271 EN**: Comment explains nearby logic, constraints, or intent: `An integer value that specifies the rounding operation. n`.
  **L271 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer value that specifies the rounding operation. n`。
- **L272 EN**: Comment explains nearby logic, constraints, or intent: `Bits [7:4] are reserved. n`.
  **L272 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [7:4] are reserved. n`。
- **L273 EN**: Comment explains nearby logic, constraints, or intent: `Bit [3] is a precision exception value: n`.
  **L273 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [3] is a precision exception value: n`。
- **L274 EN**: Comment explains nearby logic, constraints, or intent: `0: A normal PE exception is used n`.
  **L274 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: A normal PE exception is used n`。
- **L275 EN**: Comment explains nearby logic, constraints, or intent: `1: The PE field is not updated n`.
  **L275 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: The PE field is not updated n`。
- **L276 EN**: Comment explains nearby logic, constraints, or intent: `Bit [2] is the rounding control source: n`.
  **L276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [2] is the rounding control source: n`。
- **L277 EN**: Comment explains nearby logic, constraints, or intent: `0: Use bits [1:0] of a M n`.
  **L277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: Use bits [1:0] of a M n`。
- **L278 EN**: Comment explains nearby logic, constraints, or intent: `1: Use the current MXCSR setting n`.
  **L278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: Use the current MXCSR setting n`。
- **L279 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0] contain the rounding control definition: n`.
  **L279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0] contain the rounding control definition: n`。
- **L280 EN**: Comment explains nearby logic, constraints, or intent: `00: Nearest n`.
  **L280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Nearest n`。
- **L281 EN**: Comment explains nearby logic, constraints, or intent: `01: Downward (toward negative infinity) n`.
  **L281 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Downward (toward negative infinity) n`。
- **L282 EN**: Comment explains nearby logic, constraints, or intent: `10: Upward (toward positive infinity) n`.
  **L282 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Upward (toward positive infinity) n`。
- **L283 EN**: Comment explains nearby logic, constraints, or intent: `11: Truncated`.
  **L283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Truncated`。
- **L284 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the copied and rounded`.
  **L284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the copied and rounded`。
- **L285 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L286 EN**: Defines macro `_mm_round_ss(X, Y, M)` for conditional compilation, shorthand, or API generation.
  **L286 CN**: 定义宏 `_mm_round_ss(X, Y, M)`，用于条件编译、简写或 API 生成。
- **L287 EN**: Continues logic associated with callable symbol `__builtin_ia32_roundss`.
  **L287 CN**: 继续与可调用符号 `__builtin_ia32_roundss` 相关的逻辑。
- **L288 EN**: Continues the surrounding expression or declaration: `(M)))`.
  **L288 CN**: 继续构造周围的表达式或声明：`(M)))`。

### Lines 289-312

````c

/// Rounds each element of the 128-bit vector of [2 x double] to an
///    integer value according to the rounding control specified by the second
///    argument and returns the rounded values in a 128-bit vector of
///    [2 x double].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128d _mm_round_pd(__m128d X, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.
///
/// \param X
///    A 128-bit vector of [2 x double].
/// \param M
///    An integer value that specifies the rounding operation. \n
///    Bits [7:4] are reserved. \n
///    Bit [3] is a precision exception value: \n
///      0: A normal PE exception is used \n
///      1: The PE field is not updated \n
///    Bit [2] is the rounding control source: \n
///      0: Use bits [1:0] of \a M \n
````
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L290 EN**: Comment explains nearby logic, constraints, or intent: `Rounds each element of the 128-bit vector of [2 x double] to an`.
  **L290 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds each element of the 128-bit vector of [2 x double] to an`。
- **L291 EN**: Comment explains nearby logic, constraints, or intent: `integer value according to the rounding control specified by the second`.
  **L291 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer value according to the rounding control specified by the second`。
- **L292 EN**: Comment explains nearby logic, constraints, or intent: `argument and returns the rounded values in a 128-bit vector of`.
  **L292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`argument and returns the rounded values in a 128-bit vector of`。
- **L293 EN**: Comment explains nearby logic, constraints, or intent: `[2 x double].`.
  **L293 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x double].`。
- **L294 EN**: Separator comment used for visual grouping.
  **L294 CN**: 用于视觉分组的分隔注释。
- **L295 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L295 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L296 EN**: Separator comment used for visual grouping.
  **L296 CN**: 用于视觉分组的分隔注释。
- **L297 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L298 EN**: Comment explains nearby logic, constraints, or intent: `__m128d _mm_round_pd(__m128d X, const int M);`.
  **L298 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128d _mm_round_pd(__m128d X, const int M);`。
- **L299 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L299 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L300 EN**: Separator comment used for visual grouping.
  **L300 CN**: 用于视觉分组的分隔注释。
- **L301 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.`.
  **L301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDPD / ROUNDPD </c> instruction.`。
- **L302 EN**: Separator comment used for visual grouping.
  **L302 CN**: 用于视觉分组的分隔注释。
- **L303 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L304 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double].`.
  **L304 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double].`。
- **L305 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L306 EN**: Comment explains nearby logic, constraints, or intent: `An integer value that specifies the rounding operation. n`.
  **L306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer value that specifies the rounding operation. n`。
- **L307 EN**: Comment explains nearby logic, constraints, or intent: `Bits [7:4] are reserved. n`.
  **L307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [7:4] are reserved. n`。
- **L308 EN**: Comment explains nearby logic, constraints, or intent: `Bit [3] is a precision exception value: n`.
  **L308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [3] is a precision exception value: n`。
- **L309 EN**: Comment explains nearby logic, constraints, or intent: `0: A normal PE exception is used n`.
  **L309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: A normal PE exception is used n`。
- **L310 EN**: Comment explains nearby logic, constraints, or intent: `1: The PE field is not updated n`.
  **L310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: The PE field is not updated n`。
- **L311 EN**: Comment explains nearby logic, constraints, or intent: `Bit [2] is the rounding control source: n`.
  **L311 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [2] is the rounding control source: n`。
- **L312 EN**: Comment explains nearby logic, constraints, or intent: `0: Use bits [1:0] of a M n`.
  **L312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: Use bits [1:0] of a M n`。

### Lines 313-336

````c
///      1: Use the current MXCSR setting \n
///    Bits [1:0] contain the rounding control definition: \n
///      00: Nearest \n
///      01: Downward (toward negative infinity) \n
///      10: Upward (toward positive infinity) \n
///      11: Truncated
/// \returns A 128-bit vector of [2 x double] containing the rounded values.
#define _mm_round_pd(X, M)                                                     \
  ((__m128d)__builtin_ia32_roundpd((__v2df)(__m128d)(X), (M)))

/// Copies the upper element of the first 128-bit vector operand to the
///    corresponding upper element of the 128-bit result vector of [2 x double].
///    Rounds the lower element of the second 128-bit vector operand to an
///    integer value according to the rounding control specified by the third
///    argument and copies it to the lower element of the 128-bit result vector
///    of [2 x double].
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128d _mm_round_sd(__m128d X, __m128d Y, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.
````
- **L313 EN**: Comment explains nearby logic, constraints, or intent: `1: Use the current MXCSR setting n`.
  **L313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: Use the current MXCSR setting n`。
- **L314 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0] contain the rounding control definition: n`.
  **L314 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0] contain the rounding control definition: n`。
- **L315 EN**: Comment explains nearby logic, constraints, or intent: `00: Nearest n`.
  **L315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Nearest n`。
- **L316 EN**: Comment explains nearby logic, constraints, or intent: `01: Downward (toward negative infinity) n`.
  **L316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Downward (toward negative infinity) n`。
- **L317 EN**: Comment explains nearby logic, constraints, or intent: `10: Upward (toward positive infinity) n`.
  **L317 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Upward (toward positive infinity) n`。
- **L318 EN**: Comment explains nearby logic, constraints, or intent: `11: Truncated`.
  **L318 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Truncated`。
- **L319 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the rounded values.`.
  **L319 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the rounded values.`。
- **L320 EN**: Defines macro `_mm_round_pd(X, M)` for conditional compilation, shorthand, or API generation.
  **L320 CN**: 定义宏 `_mm_round_pd(X, M)`，用于条件编译、简写或 API 生成。
- **L321 EN**: Continues logic associated with callable symbol `__builtin_ia32_roundpd`.
  **L321 CN**: 继续与可调用符号 `__builtin_ia32_roundpd` 相关的逻辑。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Comment explains nearby logic, constraints, or intent: `Copies the upper element of the first 128-bit vector operand to the`.
  **L323 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Copies the upper element of the first 128-bit vector operand to the`。
- **L324 EN**: Comment explains nearby logic, constraints, or intent: `corresponding upper element of the 128-bit result vector of [2 x double].`.
  **L324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding upper element of the 128-bit result vector of [2 x double].`。
- **L325 EN**: Comment explains nearby logic, constraints, or intent: `Rounds the lower element of the second 128-bit vector operand to an`.
  **L325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounds the lower element of the second 128-bit vector operand to an`。
- **L326 EN**: Comment explains nearby logic, constraints, or intent: `integer value according to the rounding control specified by the third`.
  **L326 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer value according to the rounding control specified by the third`。
- **L327 EN**: Comment explains nearby logic, constraints, or intent: `argument and copies it to the lower element of the 128-bit result vector`.
  **L327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`argument and copies it to the lower element of the 128-bit result vector`。
- **L328 EN**: Comment explains nearby logic, constraints, or intent: `of [2 x double].`.
  **L328 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [2 x double].`。
- **L329 EN**: Separator comment used for visual grouping.
  **L329 CN**: 用于视觉分组的分隔注释。
- **L330 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L330 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L331 EN**: Separator comment used for visual grouping.
  **L331 CN**: 用于视觉分组的分隔注释。
- **L332 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L333 EN**: Comment explains nearby logic, constraints, or intent: `__m128d _mm_round_sd(__m128d X, __m128d Y, const int M);`.
  **L333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128d _mm_round_sd(__m128d X, __m128d Y, const int M);`。
- **L334 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L335 EN**: Separator comment used for visual grouping.
  **L335 CN**: 用于视觉分组的分隔注释。
- **L336 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.`.
  **L336 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VROUNDSD / ROUNDSD </c> instruction.`。

### Lines 337-360

````c
///
/// \param X
///    A 128-bit vector of [2 x double]. The value stored in bits [127:64] is
///    copied to the corresponding bits of the result.
/// \param Y
///    A 128-bit vector of [2 x double]. The value stored in bits [63:0] is
///    rounded to the nearest integer using the specified rounding control and
///    copied to the corresponding bits of the result.
/// \param M
///    An integer value that specifies the rounding operation. \n
///    Bits [7:4] are reserved. \n
///    Bit [3] is a precision exception value: \n
///      0: A normal PE exception is used \n
///      1: The PE field is not updated \n
///    Bit [2] is the rounding control source: \n
///      0: Use bits [1:0] of \a M \n
///      1: Use the current MXCSR setting \n
///    Bits [1:0] contain the rounding control definition: \n
///      00: Nearest \n
///      01: Downward (toward negative infinity) \n
///      10: Upward (toward positive infinity) \n
///      11: Truncated
/// \returns A 128-bit vector of [2 x double] containing the copied and rounded
///    values.
````
- **L337 EN**: Separator comment used for visual grouping.
  **L337 CN**: 用于视觉分组的分隔注释。
- **L338 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L339 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double]. The value stored in bits [127:64] is`.
  **L339 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double]. The value stored in bits [127:64] is`。
- **L340 EN**: Comment explains nearby logic, constraints, or intent: `copied to the corresponding bits of the result.`.
  **L340 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copied to the corresponding bits of the result.`。
- **L341 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L342 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double]. The value stored in bits [63:0] is`.
  **L342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double]. The value stored in bits [63:0] is`。
- **L343 EN**: Comment explains nearby logic, constraints, or intent: `rounded to the nearest integer using the specified rounding control and`.
  **L343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`rounded to the nearest integer using the specified rounding control and`。
- **L344 EN**: Comment explains nearby logic, constraints, or intent: `copied to the corresponding bits of the result.`.
  **L344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copied to the corresponding bits of the result.`。
- **L345 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L346 EN**: Comment explains nearby logic, constraints, or intent: `An integer value that specifies the rounding operation. n`.
  **L346 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer value that specifies the rounding operation. n`。
- **L347 EN**: Comment explains nearby logic, constraints, or intent: `Bits [7:4] are reserved. n`.
  **L347 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [7:4] are reserved. n`。
- **L348 EN**: Comment explains nearby logic, constraints, or intent: `Bit [3] is a precision exception value: n`.
  **L348 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [3] is a precision exception value: n`。
- **L349 EN**: Comment explains nearby logic, constraints, or intent: `0: A normal PE exception is used n`.
  **L349 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: A normal PE exception is used n`。
- **L350 EN**: Comment explains nearby logic, constraints, or intent: `1: The PE field is not updated n`.
  **L350 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: The PE field is not updated n`。
- **L351 EN**: Comment explains nearby logic, constraints, or intent: `Bit [2] is the rounding control source: n`.
  **L351 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [2] is the rounding control source: n`。
- **L352 EN**: Comment explains nearby logic, constraints, or intent: `0: Use bits [1:0] of a M n`.
  **L352 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: Use bits [1:0] of a M n`。
- **L353 EN**: Comment explains nearby logic, constraints, or intent: `1: Use the current MXCSR setting n`.
  **L353 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: Use the current MXCSR setting n`。
- **L354 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0] contain the rounding control definition: n`.
  **L354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0] contain the rounding control definition: n`。
- **L355 EN**: Comment explains nearby logic, constraints, or intent: `00: Nearest n`.
  **L355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Nearest n`。
- **L356 EN**: Comment explains nearby logic, constraints, or intent: `01: Downward (toward negative infinity) n`.
  **L356 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Downward (toward negative infinity) n`。
- **L357 EN**: Comment explains nearby logic, constraints, or intent: `10: Upward (toward positive infinity) n`.
  **L357 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Upward (toward positive infinity) n`。
- **L358 EN**: Comment explains nearby logic, constraints, or intent: `11: Truncated`.
  **L358 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Truncated`。
- **L359 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the copied and rounded`.
  **L359 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the copied and rounded`。
- **L360 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L360 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。

### Lines 361-384

````c
#define _mm_round_sd(X, Y, M)                                                  \
  ((__m128d)__builtin_ia32_roundsd((__v2df)(__m128d)(X), (__v2df)(__m128d)(Y), \
                                   (M)))

/* SSE4 Packed Blending Intrinsics.  */
/// Returns a 128-bit vector of [2 x double] where the values are
///    selected from either the first or second operand as specified by the
///    third operand, the control mask.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128d _mm_blend_pd(__m128d V1, __m128d V2, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VBLENDPD / BLENDPD </c> instruction.
///
/// \param V1
///    A 128-bit vector of [2 x double].
/// \param V2
///    A 128-bit vector of [2 x double].
/// \param M
///    An immediate integer operand, with mask bits [1:0] specifying how the
///    values are to be copied. The position of the mask bit corresponds to the
````
- **L361 EN**: Defines macro `_mm_round_sd(X, Y, M)` for conditional compilation, shorthand, or API generation.
  **L361 CN**: 定义宏 `_mm_round_sd(X, Y, M)`，用于条件编译、简写或 API 生成。
- **L362 EN**: Continues logic associated with callable symbol `__builtin_ia32_roundsd`.
  **L362 CN**: 继续与可调用符号 `__builtin_ia32_roundsd` 相关的逻辑。
- **L363 EN**: Continues the surrounding expression or declaration: `(M)))`.
  **L363 CN**: 继续构造周围的表达式或声明：`(M)))`。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L365 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Packed Blending Intrinsics.`.
  **L365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Packed Blending Intrinsics.`。
- **L366 EN**: Comment explains nearby logic, constraints, or intent: `Returns a 128-bit vector of [2 x double] where the values are`.
  **L366 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a 128-bit vector of [2 x double] where the values are`。
- **L367 EN**: Comment explains nearby logic, constraints, or intent: `selected from either the first or second operand as specified by the`.
  **L367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`selected from either the first or second operand as specified by the`。
- **L368 EN**: Comment explains nearby logic, constraints, or intent: `third operand, the control mask.`.
  **L368 CN**: 注释解释附近代码的逻辑、约束或设计意图：`third operand, the control mask.`。
- **L369 EN**: Separator comment used for visual grouping.
  **L369 CN**: 用于视觉分组的分隔注释。
- **L370 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L370 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L371 EN**: Separator comment used for visual grouping.
  **L371 CN**: 用于视觉分组的分隔注释。
- **L372 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L372 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L373 EN**: Comment explains nearby logic, constraints, or intent: `__m128d _mm_blend_pd(__m128d V1, __m128d V2, const int M);`.
  **L373 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128d _mm_blend_pd(__m128d V1, __m128d V2, const int M);`。
- **L374 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L375 EN**: Separator comment used for visual grouping.
  **L375 CN**: 用于视觉分组的分隔注释。
- **L376 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VBLENDPD / BLENDPD </c> instruction.`.
  **L376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VBLENDPD / BLENDPD </c> instruction.`。
- **L377 EN**: Separator comment used for visual grouping.
  **L377 CN**: 用于视觉分组的分隔注释。
- **L378 EN**: Comment explains nearby logic, constraints, or intent: `param V1`.
  **L378 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V1`。
- **L379 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double].`.
  **L379 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double].`。
- **L380 EN**: Comment explains nearby logic, constraints, or intent: `param V2`.
  **L380 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V2`。
- **L381 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double].`.
  **L381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double].`。
- **L382 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L382 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L383 EN**: Comment explains nearby logic, constraints, or intent: `An immediate integer operand, with mask bits [1:0] specifying how the`.
  **L383 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate integer operand, with mask bits [1:0] specifying how the`。
- **L384 EN**: Comment explains nearby logic, constraints, or intent: `values are to be copied. The position of the mask bit corresponds to the`.
  **L384 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values are to be copied. The position of the mask bit corresponds to the`。

### Lines 385-408

````c
///    index of a copied value. When a mask bit is 0, the corresponding 64-bit
///    element in operand \a V1 is copied to the same position in the result.
///    When a mask bit is 1, the corresponding 64-bit element in operand \a V2
///    is copied to the same position in the result.
/// \returns A 128-bit vector of [2 x double] containing the copied values.
#define _mm_blend_pd(V1, V2, M)                                                \
  ((__m128d)__builtin_ia32_blendpd((__v2df)(__m128d)(V1),                      \
                                   (__v2df)(__m128d)(V2), (int)(M)))

/// Returns a 128-bit vector of [4 x float] where the values are selected
///    from either the first or second operand as specified by the third
///    operand, the control mask.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128 _mm_blend_ps(__m128 V1, __m128 V2, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VBLENDPS / BLENDPS </c> instruction.
///
/// \param V1
///    A 128-bit vector of [4 x float].
/// \param V2
````
- **L385 EN**: Comment explains nearby logic, constraints, or intent: `index of a copied value. When a mask bit is 0, the corresponding 64-bit`.
  **L385 CN**: 注释解释附近代码的逻辑、约束或设计意图：`index of a copied value. When a mask bit is 0, the corresponding 64-bit`。
- **L386 EN**: Comment explains nearby logic, constraints, or intent: `element in operand a V1 is copied to the same position in the result.`.
  **L386 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element in operand a V1 is copied to the same position in the result.`。
- **L387 EN**: Comment explains nearby logic, constraints, or intent: `When a mask bit is 1, the corresponding 64-bit element in operand a V2`.
  **L387 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When a mask bit is 1, the corresponding 64-bit element in operand a V2`。
- **L388 EN**: Comment explains nearby logic, constraints, or intent: `is copied to the same position in the result.`.
  **L388 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is copied to the same position in the result.`。
- **L389 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the copied values.`.
  **L389 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the copied values.`。
- **L390 EN**: Defines macro `_mm_blend_pd(V1, V2, M)` for conditional compilation, shorthand, or API generation.
  **L390 CN**: 定义宏 `_mm_blend_pd(V1, V2, M)`，用于条件编译、简写或 API 生成。
- **L391 EN**: Continues logic associated with callable symbol `__builtin_ia32_blendpd`.
  **L391 CN**: 继续与可调用符号 `__builtin_ia32_blendpd` 相关的逻辑。
- **L392 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(V2), (int)(M)))`.
  **L392 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(V2), (int)(M)))`。
- **L393 EN**: Blank line separating nearby declarations or logic blocks.
  **L393 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L394 EN**: Comment explains nearby logic, constraints, or intent: `Returns a 128-bit vector of [4 x float] where the values are selected`.
  **L394 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a 128-bit vector of [4 x float] where the values are selected`。
- **L395 EN**: Comment explains nearby logic, constraints, or intent: `from either the first or second operand as specified by the third`.
  **L395 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from either the first or second operand as specified by the third`。
- **L396 EN**: Comment explains nearby logic, constraints, or intent: `operand, the control mask.`.
  **L396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operand, the control mask.`。
- **L397 EN**: Separator comment used for visual grouping.
  **L397 CN**: 用于视觉分组的分隔注释。
- **L398 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L399 EN**: Separator comment used for visual grouping.
  **L399 CN**: 用于视觉分组的分隔注释。
- **L400 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L400 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L401 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_blend_ps(__m128 V1, __m128 V2, const int M);`.
  **L401 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_blend_ps(__m128 V1, __m128 V2, const int M);`。
- **L402 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L402 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L403 EN**: Separator comment used for visual grouping.
  **L403 CN**: 用于视觉分组的分隔注释。
- **L404 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VBLENDPS / BLENDPS </c> instruction.`.
  **L404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VBLENDPS / BLENDPS </c> instruction.`。
- **L405 EN**: Separator comment used for visual grouping.
  **L405 CN**: 用于视觉分组的分隔注释。
- **L406 EN**: Comment explains nearby logic, constraints, or intent: `param V1`.
  **L406 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V1`。
- **L407 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L407 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L408 EN**: Comment explains nearby logic, constraints, or intent: `param V2`.
  **L408 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V2`。

### Lines 409-432

````c
///    A 128-bit vector of [4 x float].
/// \param M
///    An immediate integer operand, with mask bits [3:0] specifying how the
///    values are to be copied. The position of the mask bit corresponds to the
///    index of a copied value. When a mask bit is 0, the corresponding 32-bit
///    element in operand \a V1 is copied to the same position in the result.
///    When a mask bit is 1, the corresponding 32-bit element in operand \a V2
///    is copied to the same position in the result.
/// \returns A 128-bit vector of [4 x float] containing the copied values.
#define _mm_blend_ps(V1, V2, M)                                                \
  ((__m128)__builtin_ia32_blendps((__v4sf)(__m128)(V1), (__v4sf)(__m128)(V2),  \
                                  (int)(M)))

/// Returns a 128-bit vector of [2 x double] where the values are
///    selected from either the first or second operand as specified by the
///    third operand, the control mask.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VBLENDVPD / BLENDVPD </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [2 x double].
/// \param __V2
````
- **L409 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L409 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L410 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L410 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L411 EN**: Comment explains nearby logic, constraints, or intent: `An immediate integer operand, with mask bits [3:0] specifying how the`.
  **L411 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate integer operand, with mask bits [3:0] specifying how the`。
- **L412 EN**: Comment explains nearby logic, constraints, or intent: `values are to be copied. The position of the mask bit corresponds to the`.
  **L412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values are to be copied. The position of the mask bit corresponds to the`。
- **L413 EN**: Comment explains nearby logic, constraints, or intent: `index of a copied value. When a mask bit is 0, the corresponding 32-bit`.
  **L413 CN**: 注释解释附近代码的逻辑、约束或设计意图：`index of a copied value. When a mask bit is 0, the corresponding 32-bit`。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `element in operand a V1 is copied to the same position in the result.`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element in operand a V1 is copied to the same position in the result.`。
- **L415 EN**: Comment explains nearby logic, constraints, or intent: `When a mask bit is 1, the corresponding 32-bit element in operand a V2`.
  **L415 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When a mask bit is 1, the corresponding 32-bit element in operand a V2`。
- **L416 EN**: Comment explains nearby logic, constraints, or intent: `is copied to the same position in the result.`.
  **L416 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is copied to the same position in the result.`。
- **L417 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the copied values.`.
  **L417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the copied values.`。
- **L418 EN**: Defines macro `_mm_blend_ps(V1, V2, M)` for conditional compilation, shorthand, or API generation.
  **L418 CN**: 定义宏 `_mm_blend_ps(V1, V2, M)`，用于条件编译、简写或 API 生成。
- **L419 EN**: Continues logic associated with callable symbol `__builtin_ia32_blendps`.
  **L419 CN**: 继续与可调用符号 `__builtin_ia32_blendps` 相关的逻辑。
- **L420 EN**: Continues the surrounding expression or declaration: `(int)(M)))`.
  **L420 CN**: 继续构造周围的表达式或声明：`(int)(M)))`。
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L422 EN**: Comment explains nearby logic, constraints, or intent: `Returns a 128-bit vector of [2 x double] where the values are`.
  **L422 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a 128-bit vector of [2 x double] where the values are`。
- **L423 EN**: Comment explains nearby logic, constraints, or intent: `selected from either the first or second operand as specified by the`.
  **L423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`selected from either the first or second operand as specified by the`。
- **L424 EN**: Comment explains nearby logic, constraints, or intent: `third operand, the control mask.`.
  **L424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`third operand, the control mask.`。
- **L425 EN**: Separator comment used for visual grouping.
  **L425 CN**: 用于视觉分组的分隔注释。
- **L426 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L427 EN**: Separator comment used for visual grouping.
  **L427 CN**: 用于视觉分组的分隔注释。
- **L428 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VBLENDVPD / BLENDVPD </c> instruction.`.
  **L428 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VBLENDVPD / BLENDVPD </c> instruction.`。
- **L429 EN**: Separator comment used for visual grouping.
  **L429 CN**: 用于视觉分组的分隔注释。
- **L430 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L430 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L431 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double].`.
  **L431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double].`。
- **L432 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L432 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。

### Lines 433-456

````c
///    A 128-bit vector of [2 x double].
/// \param __M
///    A 128-bit vector operand, with mask bits 127 and 63 specifying how the
///    values are to be copied. The position of the mask bit corresponds to the
///    most significant bit of a copied value. When a mask bit is 0, the
///    corresponding 64-bit element in operand \a __V1 is copied to the same
///    position in the result. When a mask bit is 1, the corresponding 64-bit
///    element in operand \a __V2 is copied to the same position in the result.
/// \returns A 128-bit vector of [2 x double] containing the copied values.
static __inline__ __m128d __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_blendv_pd(__m128d __V1, __m128d __V2, __m128d __M) {
  return (__m128d)__builtin_ia32_blendvpd((__v2df)__V1, (__v2df)__V2,
                                          (__v2df)__M);
}

/// Returns a 128-bit vector of [4 x float] where the values are
///    selected from either the first or second operand as specified by the
///    third operand, the control mask.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VBLENDVPS / BLENDVPS </c> instruction.
///
/// \param __V1
````
- **L433 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double].`.
  **L433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double].`。
- **L434 EN**: Comment explains nearby logic, constraints, or intent: `param __M`.
  **L434 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __M`。
- **L435 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector operand, with mask bits 127 and 63 specifying how the`.
  **L435 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector operand, with mask bits 127 and 63 specifying how the`。
- **L436 EN**: Comment explains nearby logic, constraints, or intent: `values are to be copied. The position of the mask bit corresponds to the`.
  **L436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values are to be copied. The position of the mask bit corresponds to the`。
- **L437 EN**: Comment explains nearby logic, constraints, or intent: `most significant bit of a copied value. When a mask bit is 0, the`.
  **L437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`most significant bit of a copied value. When a mask bit is 0, the`。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `corresponding 64-bit element in operand a __V1 is copied to the same`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding 64-bit element in operand a __V1 is copied to the same`。
- **L439 EN**: Comment explains nearby logic, constraints, or intent: `position in the result. When a mask bit is 1, the corresponding 64-bit`.
  **L439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`position in the result. When a mask bit is 1, the corresponding 64-bit`。
- **L440 EN**: Comment explains nearby logic, constraints, or intent: `element in operand a __V2 is copied to the same position in the result.`.
  **L440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element in operand a __V2 is copied to the same position in the result.`。
- **L441 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the copied values.`.
  **L441 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the copied values.`。
- **L442 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L442 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L443 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blendv_pd(__m128d __V1, __m128d __V2, __m128d __M) {`.
  **L443 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blendv_pd(__m128d __V1, __m128d __V2, __m128d __M) {`。
- **L444 EN**: Returns from the current function with `(__m128d)__builtin_ia32_blendvpd((__v2df)__V1, (__v2df)__V2,`.
  **L444 CN**: 以 `(__m128d)__builtin_ia32_blendvpd((__v2df)__V1, (__v2df)__V2,` 从当前函数返回。
- **L445 EN**: Executes a call or declaration centered on `statement`.
  **L445 CN**: 执行以 `statement` 为核心的调用或声明。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Comment explains nearby logic, constraints, or intent: `Returns a 128-bit vector of [4 x float] where the values are`.
  **L448 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a 128-bit vector of [4 x float] where the values are`。
- **L449 EN**: Comment explains nearby logic, constraints, or intent: `selected from either the first or second operand as specified by the`.
  **L449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`selected from either the first or second operand as specified by the`。
- **L450 EN**: Comment explains nearby logic, constraints, or intent: `third operand, the control mask.`.
  **L450 CN**: 注释解释附近代码的逻辑、约束或设计意图：`third operand, the control mask.`。
- **L451 EN**: Separator comment used for visual grouping.
  **L451 CN**: 用于视觉分组的分隔注释。
- **L452 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L452 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L453 EN**: Separator comment used for visual grouping.
  **L453 CN**: 用于视觉分组的分隔注释。
- **L454 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VBLENDVPS / BLENDVPS </c> instruction.`.
  **L454 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VBLENDVPS / BLENDVPS </c> instruction.`。
- **L455 EN**: Separator comment used for visual grouping.
  **L455 CN**: 用于视觉分组的分隔注释。
- **L456 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L456 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。

### Lines 457-480

````c
///    A 128-bit vector of [4 x float].
/// \param __V2
///    A 128-bit vector of [4 x float].
/// \param __M
///    A 128-bit vector operand, with mask bits 127, 95, 63, and 31 specifying
///    how the values are to be copied. The position of the mask bit corresponds
///    to the most significant bit of a copied value. When a mask bit is 0, the
///    corresponding 32-bit element in operand \a __V1 is copied to the same
///    position in the result. When a mask bit is 1, the corresponding 32-bit
///    element in operand \a __V2 is copied to the same position in the result.
/// \returns A 128-bit vector of [4 x float] containing the copied values.
static __inline__ __m128 __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_blendv_ps(__m128 __V1, __m128 __V2, __m128 __M) {
  return (__m128)__builtin_ia32_blendvps((__v4sf)__V1, (__v4sf)__V2,
                                         (__v4sf)__M);
}

/// Returns a 128-bit vector of [16 x i8] where the values are selected
///    from either of the first or second operand as specified by the third
///    operand, the control mask.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPBLENDVB / PBLENDVB </c> instruction.
````
- **L457 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L458 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L458 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L459 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L459 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L460 EN**: Comment explains nearby logic, constraints, or intent: `param __M`.
  **L460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __M`。
- **L461 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector operand, with mask bits 127, 95, 63, and 31 specifying`.
  **L461 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector operand, with mask bits 127, 95, 63, and 31 specifying`。
- **L462 EN**: Comment explains nearby logic, constraints, or intent: `how the values are to be copied. The position of the mask bit corresponds`.
  **L462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`how the values are to be copied. The position of the mask bit corresponds`。
- **L463 EN**: Comment explains nearby logic, constraints, or intent: `to the most significant bit of a copied value. When a mask bit is 0, the`.
  **L463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the most significant bit of a copied value. When a mask bit is 0, the`。
- **L464 EN**: Comment explains nearby logic, constraints, or intent: `corresponding 32-bit element in operand a __V1 is copied to the same`.
  **L464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding 32-bit element in operand a __V1 is copied to the same`。
- **L465 EN**: Comment explains nearby logic, constraints, or intent: `position in the result. When a mask bit is 1, the corresponding 32-bit`.
  **L465 CN**: 注释解释附近代码的逻辑、约束或设计意图：`position in the result. When a mask bit is 1, the corresponding 32-bit`。
- **L466 EN**: Comment explains nearby logic, constraints, or intent: `element in operand a __V2 is copied to the same position in the result.`.
  **L466 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element in operand a __V2 is copied to the same position in the result.`。
- **L467 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the copied values.`.
  **L467 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the copied values.`。
- **L468 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L468 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L469 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blendv_ps(__m128 __V1, __m128 __V2, __m128 __M) {`.
  **L469 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blendv_ps(__m128 __V1, __m128 __V2, __m128 __M) {`。
- **L470 EN**: Returns from the current function with `(__m128)__builtin_ia32_blendvps((__v4sf)__V1, (__v4sf)__V2,`.
  **L470 CN**: 以 `(__m128)__builtin_ia32_blendvps((__v4sf)__V1, (__v4sf)__V2,` 从当前函数返回。
- **L471 EN**: Executes a call or declaration centered on `statement`.
  **L471 CN**: 执行以 `statement` 为核心的调用或声明。
- **L472 EN**: Closes the current lexical scope or compound statement.
  **L472 CN**: 结束当前词法作用域或复合语句块。
- **L473 EN**: Blank line separating nearby declarations or logic blocks.
  **L473 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L474 EN**: Comment explains nearby logic, constraints, or intent: `Returns a 128-bit vector of [16 x i8] where the values are selected`.
  **L474 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a 128-bit vector of [16 x i8] where the values are selected`。
- **L475 EN**: Comment explains nearby logic, constraints, or intent: `from either of the first or second operand as specified by the third`.
  **L475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from either of the first or second operand as specified by the third`。
- **L476 EN**: Comment explains nearby logic, constraints, or intent: `operand, the control mask.`.
  **L476 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operand, the control mask.`。
- **L477 EN**: Separator comment used for visual grouping.
  **L477 CN**: 用于视觉分组的分隔注释。
- **L478 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L479 EN**: Separator comment used for visual grouping.
  **L479 CN**: 用于视觉分组的分隔注释。
- **L480 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPBLENDVB / PBLENDVB </c> instruction.`.
  **L480 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPBLENDVB / PBLENDVB </c> instruction.`。

### Lines 481-504

````c
///
/// \param __V1
///    A 128-bit vector of [16 x i8].
/// \param __V2
///    A 128-bit vector of [16 x i8].
/// \param __M
///    A 128-bit vector operand, with mask bits 127, 119, 111...7 specifying
///    how the values are to be copied. The position of the mask bit corresponds
///    to the most significant bit of a copied value. When a mask bit is 0, the
///    corresponding 8-bit element in operand \a __V1 is copied to the same
///    position in the result. When a mask bit is 1, the corresponding 8-bit
///    element in operand \a __V2 is copied to the same position in the result.
/// \returns A 128-bit vector of [16 x i8] containing the copied values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_blendv_epi8(__m128i __V1, __m128i __V2, __m128i __M) {
  return (__m128i)__builtin_ia32_pblendvb128((__v16qi)__V1, (__v16qi)__V2,
                                             (__v16qi)__M);
}

/// Returns a 128-bit vector of [8 x i16] where the values are selected
///    from either of the first or second operand as specified by the third
///    operand, the control mask.
///
/// \headerfile <x86intrin.h>
````
- **L481 EN**: Separator comment used for visual grouping.
  **L481 CN**: 用于视觉分组的分隔注释。
- **L482 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L482 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L483 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8].`.
  **L483 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8].`。
- **L484 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L484 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L485 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8].`.
  **L485 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8].`。
- **L486 EN**: Comment explains nearby logic, constraints, or intent: `param __M`.
  **L486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __M`。
- **L487 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector operand, with mask bits 127, 119, 111...7 specifying`.
  **L487 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector operand, with mask bits 127, 119, 111...7 specifying`。
- **L488 EN**: Comment explains nearby logic, constraints, or intent: `how the values are to be copied. The position of the mask bit corresponds`.
  **L488 CN**: 注释解释附近代码的逻辑、约束或设计意图：`how the values are to be copied. The position of the mask bit corresponds`。
- **L489 EN**: Comment explains nearby logic, constraints, or intent: `to the most significant bit of a copied value. When a mask bit is 0, the`.
  **L489 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the most significant bit of a copied value. When a mask bit is 0, the`。
- **L490 EN**: Comment explains nearby logic, constraints, or intent: `corresponding 8-bit element in operand a __V1 is copied to the same`.
  **L490 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding 8-bit element in operand a __V1 is copied to the same`。
- **L491 EN**: Comment explains nearby logic, constraints, or intent: `position in the result. When a mask bit is 1, the corresponding 8-bit`.
  **L491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`position in the result. When a mask bit is 1, the corresponding 8-bit`。
- **L492 EN**: Comment explains nearby logic, constraints, or intent: `element in operand a __V2 is copied to the same position in the result.`.
  **L492 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element in operand a __V2 is copied to the same position in the result.`。
- **L493 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [16 x i8] containing the copied values.`.
  **L493 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [16 x i8] containing the copied values.`。
- **L494 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L494 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L495 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blendv_epi8(__m128i __V1, __m128i __V2, __m128i __M) {`.
  **L495 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blendv_epi8(__m128i __V1, __m128i __V2, __m128i __M) {`。
- **L496 EN**: Returns from the current function with `(__m128i)__builtin_ia32_pblendvb128((__v16qi)__V1, (__v16qi)__V2,`.
  **L496 CN**: 以 `(__m128i)__builtin_ia32_pblendvb128((__v16qi)__V1, (__v16qi)__V2,` 从当前函数返回。
- **L497 EN**: Executes a call or declaration centered on `statement`.
  **L497 CN**: 执行以 `statement` 为核心的调用或声明。
- **L498 EN**: Closes the current lexical scope or compound statement.
  **L498 CN**: 结束当前词法作用域或复合语句块。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L500 EN**: Comment explains nearby logic, constraints, or intent: `Returns a 128-bit vector of [8 x i16] where the values are selected`.
  **L500 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns a 128-bit vector of [8 x i16] where the values are selected`。
- **L501 EN**: Comment explains nearby logic, constraints, or intent: `from either of the first or second operand as specified by the third`.
  **L501 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from either of the first or second operand as specified by the third`。
- **L502 EN**: Comment explains nearby logic, constraints, or intent: `operand, the control mask.`.
  **L502 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operand, the control mask.`。
- **L503 EN**: Separator comment used for visual grouping.
  **L503 CN**: 用于视觉分组的分隔注释。
- **L504 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。

### Lines 505-528

````c
///
/// \code
/// __m128i _mm_blend_epi16(__m128i V1, __m128i V2, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPBLENDW / PBLENDW </c> instruction.
///
/// \param V1
///    A 128-bit vector of [8 x i16].
/// \param V2
///    A 128-bit vector of [8 x i16].
/// \param M
///    An immediate integer operand, with mask bits [7:0] specifying how the
///    values are to be copied. The position of the mask bit corresponds to the
///    index of a copied value. When a mask bit is 0, the corresponding 16-bit
///    element in operand \a V1 is copied to the same position in the result.
///    When a mask bit is 1, the corresponding 16-bit element in operand \a V2
///    is copied to the same position in the result.
/// \returns A 128-bit vector of [8 x i16] containing the copied values.
#define _mm_blend_epi16(V1, V2, M)                                             \
  ((__m128i)__builtin_ia32_pblendw128((__v8hi)(__m128i)(V1),                   \
                                      (__v8hi)(__m128i)(V2), (int)(M)))

/* SSE4 Dword Multiply Instructions.  */
````
- **L505 EN**: Separator comment used for visual grouping.
  **L505 CN**: 用于视觉分组的分隔注释。
- **L506 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L506 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L507 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_blend_epi16(__m128i V1, __m128i V2, const int M);`.
  **L507 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_blend_epi16(__m128i V1, __m128i V2, const int M);`。
- **L508 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L508 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L509 EN**: Separator comment used for visual grouping.
  **L509 CN**: 用于视觉分组的分隔注释。
- **L510 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPBLENDW / PBLENDW </c> instruction.`.
  **L510 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPBLENDW / PBLENDW </c> instruction.`。
- **L511 EN**: Separator comment used for visual grouping.
  **L511 CN**: 用于视觉分组的分隔注释。
- **L512 EN**: Comment explains nearby logic, constraints, or intent: `param V1`.
  **L512 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V1`。
- **L513 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x i16].`.
  **L513 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x i16].`。
- **L514 EN**: Comment explains nearby logic, constraints, or intent: `param V2`.
  **L514 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V2`。
- **L515 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x i16].`.
  **L515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x i16].`。
- **L516 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L516 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L517 EN**: Comment explains nearby logic, constraints, or intent: `An immediate integer operand, with mask bits [7:0] specifying how the`.
  **L517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate integer operand, with mask bits [7:0] specifying how the`。
- **L518 EN**: Comment explains nearby logic, constraints, or intent: `values are to be copied. The position of the mask bit corresponds to the`.
  **L518 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values are to be copied. The position of the mask bit corresponds to the`。
- **L519 EN**: Comment explains nearby logic, constraints, or intent: `index of a copied value. When a mask bit is 0, the corresponding 16-bit`.
  **L519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`index of a copied value. When a mask bit is 0, the corresponding 16-bit`。
- **L520 EN**: Comment explains nearby logic, constraints, or intent: `element in operand a V1 is copied to the same position in the result.`.
  **L520 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element in operand a V1 is copied to the same position in the result.`。
- **L521 EN**: Comment explains nearby logic, constraints, or intent: `When a mask bit is 1, the corresponding 16-bit element in operand a V2`.
  **L521 CN**: 注释解释附近代码的逻辑、约束或设计意图：`When a mask bit is 1, the corresponding 16-bit element in operand a V2`。
- **L522 EN**: Comment explains nearby logic, constraints, or intent: `is copied to the same position in the result.`.
  **L522 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is copied to the same position in the result.`。
- **L523 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [8 x i16] containing the copied values.`.
  **L523 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [8 x i16] containing the copied values.`。
- **L524 EN**: Defines macro `_mm_blend_epi16(V1, V2, M)` for conditional compilation, shorthand, or API generation.
  **L524 CN**: 定义宏 `_mm_blend_epi16(V1, V2, M)`，用于条件编译、简写或 API 生成。
- **L525 EN**: Continues logic associated with callable symbol `__builtin_ia32_pblendw128`.
  **L525 CN**: 继续与可调用符号 `__builtin_ia32_pblendw128` 相关的逻辑。
- **L526 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(V2), (int)(M)))`.
  **L526 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(V2), (int)(M)))`。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L528 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Dword Multiply Instructions.`.
  **L528 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Dword Multiply Instructions.`。

### Lines 529-552

````c
/// Multiples corresponding elements of two 128-bit vectors of [4 x i32]
///    and returns the lower 32 bits of the each product in a 128-bit vector of
///    [4 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMULLD / PMULLD </c> instruction.
///
/// \param __V1
///    A 128-bit integer vector.
/// \param __V2
///    A 128-bit integer vector.
/// \returns A 128-bit integer vector containing the products of both operands.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_mullo_epi32(__m128i __V1, __m128i __V2) {
  return (__m128i)((__v4su)__V1 * (__v4su)__V2);
}

/// Multiplies corresponding even-indexed elements of two 128-bit
///    vectors of [4 x i32] and returns a 128-bit vector of [2 x i64]
///    containing the products.
///
/// \headerfile <x86intrin.h>
///
````
- **L529 EN**: Comment explains nearby logic, constraints, or intent: `Multiples corresponding elements of two 128-bit vectors of [4 x i32]`.
  **L529 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiples corresponding elements of two 128-bit vectors of [4 x i32]`。
- **L530 EN**: Comment explains nearby logic, constraints, or intent: `and returns the lower 32 bits of the each product in a 128-bit vector of`.
  **L530 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and returns the lower 32 bits of the each product in a 128-bit vector of`。
- **L531 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i32].`.
  **L531 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i32].`。
- **L532 EN**: Separator comment used for visual grouping.
  **L532 CN**: 用于视觉分组的分隔注释。
- **L533 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L533 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L534 EN**: Separator comment used for visual grouping.
  **L534 CN**: 用于视觉分组的分隔注释。
- **L535 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMULLD / PMULLD </c> instruction.`.
  **L535 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMULLD / PMULLD </c> instruction.`。
- **L536 EN**: Separator comment used for visual grouping.
  **L536 CN**: 用于视觉分组的分隔注释。
- **L537 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L537 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L538 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L538 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。
- **L539 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L539 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L540 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L540 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。
- **L541 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit integer vector containing the products of both operands.`.
  **L541 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit integer vector containing the products of both operands.`。
- **L542 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L542 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L543 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mullo_epi32(__m128i __V1, __m128i __V2) {`.
  **L543 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mullo_epi32(__m128i __V1, __m128i __V2) {`。
- **L544 EN**: Returns from the current function with `(__m128i)((__v4su)__V1 * (__v4su)__V2)`.
  **L544 CN**: 以 `(__m128i)((__v4su)__V1 * (__v4su)__V2)` 从当前函数返回。
- **L545 EN**: Closes the current lexical scope or compound statement.
  **L545 CN**: 结束当前词法作用域或复合语句块。
- **L546 EN**: Blank line separating nearby declarations or logic blocks.
  **L546 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L547 EN**: Comment explains nearby logic, constraints, or intent: `Multiplies corresponding even-indexed elements of two 128-bit`.
  **L547 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplies corresponding even-indexed elements of two 128-bit`。
- **L548 EN**: Comment explains nearby logic, constraints, or intent: `vectors of [4 x i32] and returns a 128-bit vector of [2 x i64]`.
  **L548 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vectors of [4 x i32] and returns a 128-bit vector of [2 x i64]`。
- **L549 EN**: Comment explains nearby logic, constraints, or intent: `containing the products.`.
  **L549 CN**: 注释解释附近代码的逻辑、约束或设计意图：`containing the products.`。
- **L550 EN**: Separator comment used for visual grouping.
  **L550 CN**: 用于视觉分组的分隔注释。
- **L551 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L551 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L552 EN**: Separator comment used for visual grouping.
  **L552 CN**: 用于视觉分组的分隔注释。

### Lines 553-576

````c
/// This intrinsic corresponds to the <c> VPMULDQ / PMULDQ </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [4 x i32].
/// \param __V2
///    A 128-bit vector of [4 x i32].
/// \returns A 128-bit vector of [2 x i64] containing the products of both
///    operands.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_mul_epi32(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_ia32_pmuldq128((__v4si)__V1, (__v4si)__V2);
}

/* SSE4 Floating Point Dot Product Instructions.  */
/// Computes the dot product of the two 128-bit vectors of [4 x float]
///    and returns it in the elements of the 128-bit result vector of
///    [4 x float].
///
///    The immediate integer operand controls which input elements
///    will contribute to the dot product, and where the final results are
///    returned.
///
/// \headerfile <x86intrin.h>
///
````
- **L553 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMULDQ / PMULDQ </c> instruction.`.
  **L553 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMULDQ / PMULDQ </c> instruction.`。
- **L554 EN**: Separator comment used for visual grouping.
  **L554 CN**: 用于视觉分组的分隔注释。
- **L555 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L555 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L556 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32].`.
  **L556 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32].`。
- **L557 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L557 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L558 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32].`.
  **L558 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32].`。
- **L559 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x i64] containing the products of both`.
  **L559 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x i64] containing the products of both`。
- **L560 EN**: Comment explains nearby logic, constraints, or intent: `operands.`.
  **L560 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operands.`。
- **L561 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L561 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L562 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mul_epi32(__m128i __V1, __m128i __V2) {`.
  **L562 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mul_epi32(__m128i __V1, __m128i __V2) {`。
- **L563 EN**: Returns from the current function with `(__m128i)__builtin_ia32_pmuldq128((__v4si)__V1, (__v4si)__V2)`.
  **L563 CN**: 以 `(__m128i)__builtin_ia32_pmuldq128((__v4si)__V1, (__v4si)__V2)` 从当前函数返回。
- **L564 EN**: Closes the current lexical scope or compound statement.
  **L564 CN**: 结束当前词法作用域或复合语句块。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L566 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Floating Point Dot Product Instructions.`.
  **L566 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Floating Point Dot Product Instructions.`。
- **L567 EN**: Comment explains nearby logic, constraints, or intent: `Computes the dot product of the two 128-bit vectors of [4 x float]`.
  **L567 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes the dot product of the two 128-bit vectors of [4 x float]`。
- **L568 EN**: Comment explains nearby logic, constraints, or intent: `and returns it in the elements of the 128-bit result vector of`.
  **L568 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and returns it in the elements of the 128-bit result vector of`。
- **L569 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float].`.
  **L569 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float].`。
- **L570 EN**: Separator comment used for visual grouping.
  **L570 CN**: 用于视觉分组的分隔注释。
- **L571 EN**: Comment explains nearby logic, constraints, or intent: `The immediate integer operand controls which input elements`.
  **L571 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The immediate integer operand controls which input elements`。
- **L572 EN**: Comment explains nearby logic, constraints, or intent: `will contribute to the dot product, and where the final results are`.
  **L572 CN**: 注释解释附近代码的逻辑、约束或设计意图：`will contribute to the dot product, and where the final results are`。
- **L573 EN**: Comment explains nearby logic, constraints, or intent: `returned.`.
  **L573 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returned.`。
- **L574 EN**: Separator comment used for visual grouping.
  **L574 CN**: 用于视觉分组的分隔注释。
- **L575 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L575 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L576 EN**: Separator comment used for visual grouping.
  **L576 CN**: 用于视觉分组的分隔注释。

### Lines 577-600

````c
/// \code
/// __m128 _mm_dp_ps(__m128 X, __m128 Y, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VDPPS / DPPS </c> instruction.
///
/// \param X
///    A 128-bit vector of [4 x float].
/// \param Y
///    A 128-bit vector of [4 x float].
/// \param M
///    An immediate integer operand. Mask bits [7:4] determine which elements
///    of the input vectors are used, with bit [4] corresponding to the lowest
///    element and bit [7] corresponding to the highest element of each [4 x
///    float] vector. If a bit is set, the corresponding elements from the two
///    input vectors are used as an input for dot product; otherwise that input
///    is treated as zero. Bits [3:0] determine which elements of the result
///    will receive a copy of the final dot product, with bit [0] corresponding
///    to the lowest element and bit [3] corresponding to the highest element of
///    each [4 x float] subvector. If a bit is set, the dot product is returned
///    in the corresponding element; otherwise that element is set to zero.
/// \returns A 128-bit vector of [4 x float] containing the dot product.
#define _mm_dp_ps(X, Y, M)                                                     \
  ((__m128)__builtin_ia32_dpps((__v4sf)(__m128)(X), (__v4sf)(__m128)(Y), (M)))
````
- **L577 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L577 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L578 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_dp_ps(__m128 X, __m128 Y, const int M);`.
  **L578 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_dp_ps(__m128 X, __m128 Y, const int M);`。
- **L579 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L579 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L580 EN**: Separator comment used for visual grouping.
  **L580 CN**: 用于视觉分组的分隔注释。
- **L581 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VDPPS / DPPS </c> instruction.`.
  **L581 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VDPPS / DPPS </c> instruction.`。
- **L582 EN**: Separator comment used for visual grouping.
  **L582 CN**: 用于视觉分组的分隔注释。
- **L583 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L583 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L584 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L584 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L585 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L585 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L586 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L586 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L587 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L587 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L588 EN**: Comment explains nearby logic, constraints, or intent: `An immediate integer operand. Mask bits [7:4] determine which elements`.
  **L588 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate integer operand. Mask bits [7:4] determine which elements`。
- **L589 EN**: Comment explains nearby logic, constraints, or intent: `of the input vectors are used, with bit [4] corresponding to the lowest`.
  **L589 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the input vectors are used, with bit [4] corresponding to the lowest`。
- **L590 EN**: Comment explains nearby logic, constraints, or intent: `element and bit [7] corresponding to the highest element of each [4 x`.
  **L590 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element and bit [7] corresponding to the highest element of each [4 x`。
- **L591 EN**: Comment explains nearby logic, constraints, or intent: `float] vector. If a bit is set, the corresponding elements from the two`.
  **L591 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float] vector. If a bit is set, the corresponding elements from the two`。
- **L592 EN**: Comment explains nearby logic, constraints, or intent: `input vectors are used as an input for dot product; otherwise that input`.
  **L592 CN**: 注释解释附近代码的逻辑、约束或设计意图：`input vectors are used as an input for dot product; otherwise that input`。
- **L593 EN**: Comment explains nearby logic, constraints, or intent: `is treated as zero. Bits [3:0] determine which elements of the result`.
  **L593 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is treated as zero. Bits [3:0] determine which elements of the result`。
- **L594 EN**: Comment explains nearby logic, constraints, or intent: `will receive a copy of the final dot product, with bit [0] corresponding`.
  **L594 CN**: 注释解释附近代码的逻辑、约束或设计意图：`will receive a copy of the final dot product, with bit [0] corresponding`。
- **L595 EN**: Comment explains nearby logic, constraints, or intent: `to the lowest element and bit [3] corresponding to the highest element of`.
  **L595 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the lowest element and bit [3] corresponding to the highest element of`。
- **L596 EN**: Comment explains nearby logic, constraints, or intent: `each [4 x float] subvector. If a bit is set, the dot product is returned`.
  **L596 CN**: 注释解释附近代码的逻辑、约束或设计意图：`each [4 x float] subvector. If a bit is set, the dot product is returned`。
- **L597 EN**: Comment explains nearby logic, constraints, or intent: `in the corresponding element; otherwise that element is set to zero.`.
  **L597 CN**: 注释解释附近代码的逻辑、约束或设计意图：`in the corresponding element; otherwise that element is set to zero.`。
- **L598 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the dot product.`.
  **L598 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the dot product.`。
- **L599 EN**: Defines macro `_mm_dp_ps(X, Y, M)` for conditional compilation, shorthand, or API generation.
  **L599 CN**: 定义宏 `_mm_dp_ps(X, Y, M)`，用于条件编译、简写或 API 生成。
- **L600 EN**: Continues logic associated with callable symbol `__builtin_ia32_dpps`.
  **L600 CN**: 继续与可调用符号 `__builtin_ia32_dpps` 相关的逻辑。

### Lines 601-624

````c

/// Computes the dot product of the two 128-bit vectors of [2 x double]
///    and returns it in the elements of the 128-bit result vector of
///    [2 x double].
///
///    The immediate integer operand controls which input
///    elements will contribute to the dot product, and where the final results
///    are returned.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128d _mm_dp_pd(__m128d X, __m128d Y, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VDPPD / DPPD </c> instruction.
///
/// \param X
///    A 128-bit vector of [2 x double].
/// \param Y
///    A 128-bit vector of [2 x double].
/// \param M
///    An immediate integer operand. Mask bits [5:4] determine which elements
///    of the input vectors are used, with bit [4] corresponding to the lowest
````
- **L601 EN**: Blank line separating nearby declarations or logic blocks.
  **L601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L602 EN**: Comment explains nearby logic, constraints, or intent: `Computes the dot product of the two 128-bit vectors of [2 x double]`.
  **L602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes the dot product of the two 128-bit vectors of [2 x double]`。
- **L603 EN**: Comment explains nearby logic, constraints, or intent: `and returns it in the elements of the 128-bit result vector of`.
  **L603 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and returns it in the elements of the 128-bit result vector of`。
- **L604 EN**: Comment explains nearby logic, constraints, or intent: `[2 x double].`.
  **L604 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x double].`。
- **L605 EN**: Separator comment used for visual grouping.
  **L605 CN**: 用于视觉分组的分隔注释。
- **L606 EN**: Comment explains nearby logic, constraints, or intent: `The immediate integer operand controls which input`.
  **L606 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The immediate integer operand controls which input`。
- **L607 EN**: Comment explains nearby logic, constraints, or intent: `elements will contribute to the dot product, and where the final results`.
  **L607 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements will contribute to the dot product, and where the final results`。
- **L608 EN**: Comment explains nearby logic, constraints, or intent: `are returned.`.
  **L608 CN**: 注释解释附近代码的逻辑、约束或设计意图：`are returned.`。
- **L609 EN**: Separator comment used for visual grouping.
  **L609 CN**: 用于视觉分组的分隔注释。
- **L610 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L610 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L611 EN**: Separator comment used for visual grouping.
  **L611 CN**: 用于视觉分组的分隔注释。
- **L612 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L613 EN**: Comment explains nearby logic, constraints, or intent: `__m128d _mm_dp_pd(__m128d X, __m128d Y, const int M);`.
  **L613 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128d _mm_dp_pd(__m128d X, __m128d Y, const int M);`。
- **L614 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L614 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L615 EN**: Separator comment used for visual grouping.
  **L615 CN**: 用于视觉分组的分隔注释。
- **L616 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VDPPD / DPPD </c> instruction.`.
  **L616 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VDPPD / DPPD </c> instruction.`。
- **L617 EN**: Separator comment used for visual grouping.
  **L617 CN**: 用于视觉分组的分隔注释。
- **L618 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L618 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L619 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double].`.
  **L619 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double].`。
- **L620 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L620 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L621 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double].`.
  **L621 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double].`。
- **L622 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L623 EN**: Comment explains nearby logic, constraints, or intent: `An immediate integer operand. Mask bits [5:4] determine which elements`.
  **L623 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate integer operand. Mask bits [5:4] determine which elements`。
- **L624 EN**: Comment explains nearby logic, constraints, or intent: `of the input vectors are used, with bit [4] corresponding to the lowest`.
  **L624 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the input vectors are used, with bit [4] corresponding to the lowest`。

### Lines 625-648

````c
///    element and bit [5] corresponding to the highest element of each of [2 x
///    double] vector. If a bit is set, the corresponding elements from the two
///    input vectors are used as an input for dot product; otherwise that input
///    is treated as zero. Bits [1:0] determine which elements of the result
///    will receive a copy of the final dot product, with bit [0] corresponding
///    to the lowest element and bit [1] corresponding to the highest element of
///    each [2 x double] vector. If a bit is set, the dot product is returned in
///    the corresponding element; otherwise that element is set to zero.
#define _mm_dp_pd(X, Y, M)                                                     \
  ((__m128d)__builtin_ia32_dppd((__v2df)(__m128d)(X), (__v2df)(__m128d)(Y),    \
                                (M)))

/* SSE4 Streaming Load Hint Instruction.  */
/// Loads integer values from a 128-bit aligned memory location to a
///    128-bit integer vector.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VMOVNTDQA / MOVNTDQA </c> instruction.
///
/// \param __V
///    A pointer to a 128-bit aligned memory location that contains the integer
///    values.
/// \returns A 128-bit integer vector containing the data stored at the
````
- **L625 EN**: Comment explains nearby logic, constraints, or intent: `element and bit [5] corresponding to the highest element of each of [2 x`.
  **L625 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element and bit [5] corresponding to the highest element of each of [2 x`。
- **L626 EN**: Comment explains nearby logic, constraints, or intent: `double] vector. If a bit is set, the corresponding elements from the two`.
  **L626 CN**: 注释解释附近代码的逻辑、约束或设计意图：`double] vector. If a bit is set, the corresponding elements from the two`。
- **L627 EN**: Comment explains nearby logic, constraints, or intent: `input vectors are used as an input for dot product; otherwise that input`.
  **L627 CN**: 注释解释附近代码的逻辑、约束或设计意图：`input vectors are used as an input for dot product; otherwise that input`。
- **L628 EN**: Comment explains nearby logic, constraints, or intent: `is treated as zero. Bits [1:0] determine which elements of the result`.
  **L628 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is treated as zero. Bits [1:0] determine which elements of the result`。
- **L629 EN**: Comment explains nearby logic, constraints, or intent: `will receive a copy of the final dot product, with bit [0] corresponding`.
  **L629 CN**: 注释解释附近代码的逻辑、约束或设计意图：`will receive a copy of the final dot product, with bit [0] corresponding`。
- **L630 EN**: Comment explains nearby logic, constraints, or intent: `to the lowest element and bit [1] corresponding to the highest element of`.
  **L630 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the lowest element and bit [1] corresponding to the highest element of`。
- **L631 EN**: Comment explains nearby logic, constraints, or intent: `each [2 x double] vector. If a bit is set, the dot product is returned in`.
  **L631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`each [2 x double] vector. If a bit is set, the dot product is returned in`。
- **L632 EN**: Comment explains nearby logic, constraints, or intent: `the corresponding element; otherwise that element is set to zero.`.
  **L632 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the corresponding element; otherwise that element is set to zero.`。
- **L633 EN**: Defines macro `_mm_dp_pd(X, Y, M)` for conditional compilation, shorthand, or API generation.
  **L633 CN**: 定义宏 `_mm_dp_pd(X, Y, M)`，用于条件编译、简写或 API 生成。
- **L634 EN**: Continues logic associated with callable symbol `__builtin_ia32_dppd`.
  **L634 CN**: 继续与可调用符号 `__builtin_ia32_dppd` 相关的逻辑。
- **L635 EN**: Continues the surrounding expression or declaration: `(M)))`.
  **L635 CN**: 继续构造周围的表达式或声明：`(M)))`。
- **L636 EN**: Blank line separating nearby declarations or logic blocks.
  **L636 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L637 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Streaming Load Hint Instruction.`.
  **L637 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Streaming Load Hint Instruction.`。
- **L638 EN**: Comment explains nearby logic, constraints, or intent: `Loads integer values from a 128-bit aligned memory location to a`.
  **L638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Loads integer values from a 128-bit aligned memory location to a`。
- **L639 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector.`.
  **L639 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector.`。
- **L640 EN**: Separator comment used for visual grouping.
  **L640 CN**: 用于视觉分组的分隔注释。
- **L641 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L641 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L642 EN**: Separator comment used for visual grouping.
  **L642 CN**: 用于视觉分组的分隔注释。
- **L643 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VMOVNTDQA / MOVNTDQA </c> instruction.`.
  **L643 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VMOVNTDQA / MOVNTDQA </c> instruction.`。
- **L644 EN**: Separator comment used for visual grouping.
  **L644 CN**: 用于视觉分组的分隔注释。
- **L645 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L645 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L646 EN**: Comment explains nearby logic, constraints, or intent: `A pointer to a 128-bit aligned memory location that contains the integer`.
  **L646 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A pointer to a 128-bit aligned memory location that contains the integer`。
- **L647 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L647 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L648 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit integer vector containing the data stored at the`.
  **L648 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit integer vector containing the data stored at the`。

### Lines 649-672

````c
///    specified memory location.
static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_stream_load_si128(const void *__V) {
  return (__m128i)__builtin_nontemporal_load((const __v2di *)__V);
}

/* SSE4 Packed Integer Min/Max Instructions.  */
/// Compares the corresponding elements of two 128-bit vectors of
///    [16 x i8] and returns a 128-bit vector of [16 x i8] containing the lesser
///    of the two values.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMINSB / PMINSB </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [16 x i8].
/// \param __V2
///    A 128-bit vector of [16 x i8]
/// \returns A 128-bit vector of [16 x i8] containing the lesser values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_min_epi8(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_elementwise_min((__v16qs)__V1, (__v16qs)__V2);
}
````
- **L649 EN**: Comment explains nearby logic, constraints, or intent: `specified memory location.`.
  **L649 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified memory location.`。
- **L650 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L650 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L651 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_stream_load_si128(const void *__V) {`.
  **L651 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_stream_load_si128(const void *__V) {`。
- **L652 EN**: Returns from the current function with `(__m128i)__builtin_nontemporal_load((const __v2di *)__V)`.
  **L652 CN**: 以 `(__m128i)__builtin_nontemporal_load((const __v2di *)__V)` 从当前函数返回。
- **L653 EN**: Closes the current lexical scope or compound statement.
  **L653 CN**: 结束当前词法作用域或复合语句块。
- **L654 EN**: Blank line separating nearby declarations or logic blocks.
  **L654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L655 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Packed Integer Min/Max Instructions.`.
  **L655 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Packed Integer Min/Max Instructions.`。
- **L656 EN**: Comment explains nearby logic, constraints, or intent: `Compares the corresponding elements of two 128-bit vectors of`.
  **L656 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the corresponding elements of two 128-bit vectors of`。
- **L657 EN**: Comment explains nearby logic, constraints, or intent: `[16 x i8] and returns a 128-bit vector of [16 x i8] containing the lesser`.
  **L657 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[16 x i8] and returns a 128-bit vector of [16 x i8] containing the lesser`。
- **L658 EN**: Comment explains nearby logic, constraints, or intent: `of the two values.`.
  **L658 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the two values.`。
- **L659 EN**: Separator comment used for visual grouping.
  **L659 CN**: 用于视觉分组的分隔注释。
- **L660 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L661 EN**: Separator comment used for visual grouping.
  **L661 CN**: 用于视觉分组的分隔注释。
- **L662 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMINSB / PMINSB </c> instruction.`.
  **L662 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMINSB / PMINSB </c> instruction.`。
- **L663 EN**: Separator comment used for visual grouping.
  **L663 CN**: 用于视觉分组的分隔注释。
- **L664 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L665 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8].`.
  **L665 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8].`。
- **L666 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L666 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L667 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8]`.
  **L667 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8]`。
- **L668 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [16 x i8] containing the lesser values.`.
  **L668 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [16 x i8] containing the lesser values.`。
- **L669 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L669 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L670 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epi8(__m128i __V1, __m128i __V2) {`.
  **L670 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epi8(__m128i __V1, __m128i __V2) {`。
- **L671 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_min((__v16qs)__V1, (__v16qs)__V2)`.
  **L671 CN**: 以 `(__m128i)__builtin_elementwise_min((__v16qs)__V1, (__v16qs)__V2)` 从当前函数返回。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````c

/// Compares the corresponding elements of two 128-bit vectors of
///    [16 x i8] and returns a 128-bit vector of [16 x i8] containing the
///    greater value of the two.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMAXSB / PMAXSB </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [16 x i8].
/// \param __V2
///    A 128-bit vector of [16 x i8].
/// \returns A 128-bit vector of [16 x i8] containing the greater values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_max_epi8(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_elementwise_max((__v16qs)__V1, (__v16qs)__V2);
}

/// Compares the corresponding elements of two 128-bit vectors of
///    [8 x u16] and returns a 128-bit vector of [8 x u16] containing the lesser
///    value of the two.
///
/// \headerfile <x86intrin.h>
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Comment explains nearby logic, constraints, or intent: `Compares the corresponding elements of two 128-bit vectors of`.
  **L674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the corresponding elements of two 128-bit vectors of`。
- **L675 EN**: Comment explains nearby logic, constraints, or intent: `[16 x i8] and returns a 128-bit vector of [16 x i8] containing the`.
  **L675 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[16 x i8] and returns a 128-bit vector of [16 x i8] containing the`。
- **L676 EN**: Comment explains nearby logic, constraints, or intent: `greater value of the two.`.
  **L676 CN**: 注释解释附近代码的逻辑、约束或设计意图：`greater value of the two.`。
- **L677 EN**: Separator comment used for visual grouping.
  **L677 CN**: 用于视觉分组的分隔注释。
- **L678 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L679 EN**: Separator comment used for visual grouping.
  **L679 CN**: 用于视觉分组的分隔注释。
- **L680 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMAXSB / PMAXSB </c> instruction.`.
  **L680 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMAXSB / PMAXSB </c> instruction.`。
- **L681 EN**: Separator comment used for visual grouping.
  **L681 CN**: 用于视觉分组的分隔注释。
- **L682 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L682 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L683 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8].`.
  **L683 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8].`。
- **L684 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L685 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8].`.
  **L685 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8].`。
- **L686 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [16 x i8] containing the greater values.`.
  **L686 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [16 x i8] containing the greater values.`。
- **L687 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L687 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L688 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epi8(__m128i __V1, __m128i __V2) {`.
  **L688 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epi8(__m128i __V1, __m128i __V2) {`。
- **L689 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_max((__v16qs)__V1, (__v16qs)__V2)`.
  **L689 CN**: 以 `(__m128i)__builtin_elementwise_max((__v16qs)__V1, (__v16qs)__V2)` 从当前函数返回。
- **L690 EN**: Closes the current lexical scope or compound statement.
  **L690 CN**: 结束当前词法作用域或复合语句块。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L692 EN**: Comment explains nearby logic, constraints, or intent: `Compares the corresponding elements of two 128-bit vectors of`.
  **L692 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the corresponding elements of two 128-bit vectors of`。
- **L693 EN**: Comment explains nearby logic, constraints, or intent: `[8 x u16] and returns a 128-bit vector of [8 x u16] containing the lesser`.
  **L693 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[8 x u16] and returns a 128-bit vector of [8 x u16] containing the lesser`。
- **L694 EN**: Comment explains nearby logic, constraints, or intent: `value of the two.`.
  **L694 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value of the two.`。
- **L695 EN**: Separator comment used for visual grouping.
  **L695 CN**: 用于视觉分组的分隔注释。
- **L696 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L696 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。

### Lines 697-720

````c
///
/// This intrinsic corresponds to the <c> VPMINUW / PMINUW </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [8 x u16].
/// \param __V2
///    A 128-bit vector of [8 x u16].
/// \returns A 128-bit vector of [8 x u16] containing the lesser values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_min_epu16(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_elementwise_min((__v8hu)__V1, (__v8hu)__V2);
}

/// Compares the corresponding elements of two 128-bit vectors of
///    [8 x u16] and returns a 128-bit vector of [8 x u16] containing the
///    greater value of the two.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMAXUW / PMAXUW </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [8 x u16].
/// \param __V2
````
- **L697 EN**: Separator comment used for visual grouping.
  **L697 CN**: 用于视觉分组的分隔注释。
- **L698 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMINUW / PMINUW </c> instruction.`.
  **L698 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMINUW / PMINUW </c> instruction.`。
- **L699 EN**: Separator comment used for visual grouping.
  **L699 CN**: 用于视觉分组的分隔注释。
- **L700 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L700 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L701 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x u16].`.
  **L701 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x u16].`。
- **L702 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L702 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L703 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x u16].`.
  **L703 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x u16].`。
- **L704 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [8 x u16] containing the lesser values.`.
  **L704 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [8 x u16] containing the lesser values.`。
- **L705 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L705 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L706 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epu16(__m128i __V1, __m128i __V2) {`.
  **L706 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epu16(__m128i __V1, __m128i __V2) {`。
- **L707 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_min((__v8hu)__V1, (__v8hu)__V2)`.
  **L707 CN**: 以 `(__m128i)__builtin_elementwise_min((__v8hu)__V1, (__v8hu)__V2)` 从当前函数返回。
- **L708 EN**: Closes the current lexical scope or compound statement.
  **L708 CN**: 结束当前词法作用域或复合语句块。
- **L709 EN**: Blank line separating nearby declarations or logic blocks.
  **L709 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L710 EN**: Comment explains nearby logic, constraints, or intent: `Compares the corresponding elements of two 128-bit vectors of`.
  **L710 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the corresponding elements of two 128-bit vectors of`。
- **L711 EN**: Comment explains nearby logic, constraints, or intent: `[8 x u16] and returns a 128-bit vector of [8 x u16] containing the`.
  **L711 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[8 x u16] and returns a 128-bit vector of [8 x u16] containing the`。
- **L712 EN**: Comment explains nearby logic, constraints, or intent: `greater value of the two.`.
  **L712 CN**: 注释解释附近代码的逻辑、约束或设计意图：`greater value of the two.`。
- **L713 EN**: Separator comment used for visual grouping.
  **L713 CN**: 用于视觉分组的分隔注释。
- **L714 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L715 EN**: Separator comment used for visual grouping.
  **L715 CN**: 用于视觉分组的分隔注释。
- **L716 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMAXUW / PMAXUW </c> instruction.`.
  **L716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMAXUW / PMAXUW </c> instruction.`。
- **L717 EN**: Separator comment used for visual grouping.
  **L717 CN**: 用于视觉分组的分隔注释。
- **L718 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L718 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L719 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x u16].`.
  **L719 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x u16].`。
- **L720 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L720 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。

### Lines 721-744

````c
///    A 128-bit vector of [8 x u16].
/// \returns A 128-bit vector of [8 x u16] containing the greater values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_max_epu16(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_elementwise_max((__v8hu)__V1, (__v8hu)__V2);
}

/// Compares the corresponding elements of two 128-bit vectors of
///    [4 x i32] and returns a 128-bit vector of [4 x i32] containing the lesser
///    value of the two.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMINSD / PMINSD </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [4 x i32].
/// \param __V2
///    A 128-bit vector of [4 x i32].
/// \returns A 128-bit vector of [4 x i32] containing the lesser values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_min_epi32(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_elementwise_min((__v4si)__V1, (__v4si)__V2);
}
````
- **L721 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x u16].`.
  **L721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x u16].`。
- **L722 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [8 x u16] containing the greater values.`.
  **L722 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [8 x u16] containing the greater values.`。
- **L723 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L723 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L724 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epu16(__m128i __V1, __m128i __V2) {`.
  **L724 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epu16(__m128i __V1, __m128i __V2) {`。
- **L725 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_max((__v8hu)__V1, (__v8hu)__V2)`.
  **L725 CN**: 以 `(__m128i)__builtin_elementwise_max((__v8hu)__V1, (__v8hu)__V2)` 从当前函数返回。
- **L726 EN**: Closes the current lexical scope or compound statement.
  **L726 CN**: 结束当前词法作用域或复合语句块。
- **L727 EN**: Blank line separating nearby declarations or logic blocks.
  **L727 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L728 EN**: Comment explains nearby logic, constraints, or intent: `Compares the corresponding elements of two 128-bit vectors of`.
  **L728 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the corresponding elements of two 128-bit vectors of`。
- **L729 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i32] and returns a 128-bit vector of [4 x i32] containing the lesser`.
  **L729 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i32] and returns a 128-bit vector of [4 x i32] containing the lesser`。
- **L730 EN**: Comment explains nearby logic, constraints, or intent: `value of the two.`.
  **L730 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value of the two.`。
- **L731 EN**: Separator comment used for visual grouping.
  **L731 CN**: 用于视觉分组的分隔注释。
- **L732 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L732 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L733 EN**: Separator comment used for visual grouping.
  **L733 CN**: 用于视觉分组的分隔注释。
- **L734 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMINSD / PMINSD </c> instruction.`.
  **L734 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMINSD / PMINSD </c> instruction.`。
- **L735 EN**: Separator comment used for visual grouping.
  **L735 CN**: 用于视觉分组的分隔注释。
- **L736 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L737 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32].`.
  **L737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32].`。
- **L738 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L738 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L739 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32].`.
  **L739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32].`。
- **L740 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x i32] containing the lesser values.`.
  **L740 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x i32] containing the lesser values.`。
- **L741 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L741 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L742 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epi32(__m128i __V1, __m128i __V2) {`.
  **L742 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epi32(__m128i __V1, __m128i __V2) {`。
- **L743 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_min((__v4si)__V1, (__v4si)__V2)`.
  **L743 CN**: 以 `(__m128i)__builtin_elementwise_min((__v4si)__V1, (__v4si)__V2)` 从当前函数返回。
- **L744 EN**: Closes the current lexical scope or compound statement.
  **L744 CN**: 结束当前词法作用域或复合语句块。

### Lines 745-768

````c

/// Compares the corresponding elements of two 128-bit vectors of
///    [4 x i32] and returns a 128-bit vector of [4 x i32] containing the
///    greater value of the two.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMAXSD / PMAXSD </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [4 x i32].
/// \param __V2
///    A 128-bit vector of [4 x i32].
/// \returns A 128-bit vector of [4 x i32] containing the greater values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_max_epi32(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_elementwise_max((__v4si)__V1, (__v4si)__V2);
}

/// Compares the corresponding elements of two 128-bit vectors of
///    [4 x u32] and returns a 128-bit vector of [4 x u32] containing the lesser
///    value of the two.
///
/// \headerfile <x86intrin.h>
````
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L746 EN**: Comment explains nearby logic, constraints, or intent: `Compares the corresponding elements of two 128-bit vectors of`.
  **L746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the corresponding elements of two 128-bit vectors of`。
- **L747 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i32] and returns a 128-bit vector of [4 x i32] containing the`.
  **L747 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i32] and returns a 128-bit vector of [4 x i32] containing the`。
- **L748 EN**: Comment explains nearby logic, constraints, or intent: `greater value of the two.`.
  **L748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`greater value of the two.`。
- **L749 EN**: Separator comment used for visual grouping.
  **L749 CN**: 用于视觉分组的分隔注释。
- **L750 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L750 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L751 EN**: Separator comment used for visual grouping.
  **L751 CN**: 用于视觉分组的分隔注释。
- **L752 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMAXSD / PMAXSD </c> instruction.`.
  **L752 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMAXSD / PMAXSD </c> instruction.`。
- **L753 EN**: Separator comment used for visual grouping.
  **L753 CN**: 用于视觉分组的分隔注释。
- **L754 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L754 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L755 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32].`.
  **L755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32].`。
- **L756 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L756 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L757 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32].`.
  **L757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32].`。
- **L758 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x i32] containing the greater values.`.
  **L758 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x i32] containing the greater values.`。
- **L759 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L759 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L760 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epi32(__m128i __V1, __m128i __V2) {`.
  **L760 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epi32(__m128i __V1, __m128i __V2) {`。
- **L761 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_max((__v4si)__V1, (__v4si)__V2)`.
  **L761 CN**: 以 `(__m128i)__builtin_elementwise_max((__v4si)__V1, (__v4si)__V2)` 从当前函数返回。
- **L762 EN**: Closes the current lexical scope or compound statement.
  **L762 CN**: 结束当前词法作用域或复合语句块。
- **L763 EN**: Blank line separating nearby declarations or logic blocks.
  **L763 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L764 EN**: Comment explains nearby logic, constraints, or intent: `Compares the corresponding elements of two 128-bit vectors of`.
  **L764 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the corresponding elements of two 128-bit vectors of`。
- **L765 EN**: Comment explains nearby logic, constraints, or intent: `[4 x u32] and returns a 128-bit vector of [4 x u32] containing the lesser`.
  **L765 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x u32] and returns a 128-bit vector of [4 x u32] containing the lesser`。
- **L766 EN**: Comment explains nearby logic, constraints, or intent: `value of the two.`.
  **L766 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value of the two.`。
- **L767 EN**: Separator comment used for visual grouping.
  **L767 CN**: 用于视觉分组的分隔注释。
- **L768 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L768 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。

### Lines 769-792

````c
///
/// This intrinsic corresponds to the <c> VPMINUD / PMINUD </c>  instruction.
///
/// \param __V1
///    A 128-bit vector of [4 x u32].
/// \param __V2
///    A 128-bit vector of [4 x u32].
/// \returns A 128-bit vector of [4 x u32] containing the lesser values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_min_epu32(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_elementwise_min((__v4su)__V1, (__v4su)__V2);
}

/// Compares the corresponding elements of two 128-bit vectors of
///    [4 x u32] and returns a 128-bit vector of [4 x u32] containing the
///    greater value of the two.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMAXUD / PMAXUD </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [4 x u32].
/// \param __V2
````
- **L769 EN**: Separator comment used for visual grouping.
  **L769 CN**: 用于视觉分组的分隔注释。
- **L770 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMINUD / PMINUD </c> instruction.`.
  **L770 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMINUD / PMINUD </c> instruction.`。
- **L771 EN**: Separator comment used for visual grouping.
  **L771 CN**: 用于视觉分组的分隔注释。
- **L772 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L773 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x u32].`.
  **L773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x u32].`。
- **L774 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L774 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L775 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x u32].`.
  **L775 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x u32].`。
- **L776 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x u32] containing the lesser values.`.
  **L776 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x u32] containing the lesser values.`。
- **L777 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L777 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L778 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epu32(__m128i __V1, __m128i __V2) {`.
  **L778 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epu32(__m128i __V1, __m128i __V2) {`。
- **L779 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_min((__v4su)__V1, (__v4su)__V2)`.
  **L779 CN**: 以 `(__m128i)__builtin_elementwise_min((__v4su)__V1, (__v4su)__V2)` 从当前函数返回。
- **L780 EN**: Closes the current lexical scope or compound statement.
  **L780 CN**: 结束当前词法作用域或复合语句块。
- **L781 EN**: Blank line separating nearby declarations or logic blocks.
  **L781 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L782 EN**: Comment explains nearby logic, constraints, or intent: `Compares the corresponding elements of two 128-bit vectors of`.
  **L782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the corresponding elements of two 128-bit vectors of`。
- **L783 EN**: Comment explains nearby logic, constraints, or intent: `[4 x u32] and returns a 128-bit vector of [4 x u32] containing the`.
  **L783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x u32] and returns a 128-bit vector of [4 x u32] containing the`。
- **L784 EN**: Comment explains nearby logic, constraints, or intent: `greater value of the two.`.
  **L784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`greater value of the two.`。
- **L785 EN**: Separator comment used for visual grouping.
  **L785 CN**: 用于视觉分组的分隔注释。
- **L786 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L787 EN**: Separator comment used for visual grouping.
  **L787 CN**: 用于视觉分组的分隔注释。
- **L788 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMAXUD / PMAXUD </c> instruction.`.
  **L788 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMAXUD / PMAXUD </c> instruction.`。
- **L789 EN**: Separator comment used for visual grouping.
  **L789 CN**: 用于视觉分组的分隔注释。
- **L790 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L790 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L791 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x u32].`.
  **L791 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x u32].`。
- **L792 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L792 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。

### Lines 793-816

````c
///    A 128-bit vector of [4 x u32].
/// \returns A 128-bit vector of [4 x u32] containing the greater values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_max_epu32(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_elementwise_max((__v4su)__V1, (__v4su)__V2);
}

/* SSE4 Insertion and Extraction from XMM Register Instructions.  */
/// Takes the first argument \a X and inserts an element from the second
///    argument \a Y as selected by the third argument \a N. That result then
///    has elements zeroed out also as selected by the third argument \a N. The
///    resulting 128-bit vector of [4 x float] is then returned.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128 _mm_insert_ps(__m128 X, __m128 Y, const int N);
/// \endcode
///
/// This intrinsic corresponds to the <c> VINSERTPS </c> instruction.
///
/// \param X
///    A 128-bit vector source operand of [4 x float]. With the exception of
///    those bits in the result copied from parameter \a Y and zeroed by bits
````
- **L793 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x u32].`.
  **L793 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x u32].`。
- **L794 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x u32] containing the greater values.`.
  **L794 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x u32] containing the greater values.`。
- **L795 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L795 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L796 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epu32(__m128i __V1, __m128i __V2) {`.
  **L796 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epu32(__m128i __V1, __m128i __V2) {`。
- **L797 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_max((__v4su)__V1, (__v4su)__V2)`.
  **L797 CN**: 以 `(__m128i)__builtin_elementwise_max((__v4su)__V1, (__v4su)__V2)` 从当前函数返回。
- **L798 EN**: Closes the current lexical scope or compound statement.
  **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L800 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Insertion and Extraction from XMM Register Instructions.`.
  **L800 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Insertion and Extraction from XMM Register Instructions.`。
- **L801 EN**: Comment explains nearby logic, constraints, or intent: `Takes the first argument a X and inserts an element from the second`.
  **L801 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Takes the first argument a X and inserts an element from the second`。
- **L802 EN**: Comment explains nearby logic, constraints, or intent: `argument a Y as selected by the third argument a N. That result then`.
  **L802 CN**: 注释解释附近代码的逻辑、约束或设计意图：`argument a Y as selected by the third argument a N. That result then`。
- **L803 EN**: Comment explains nearby logic, constraints, or intent: `has elements zeroed out also as selected by the third argument a N. The`.
  **L803 CN**: 注释解释附近代码的逻辑、约束或设计意图：`has elements zeroed out also as selected by the third argument a N. The`。
- **L804 EN**: Comment explains nearby logic, constraints, or intent: `resulting 128-bit vector of [4 x float] is then returned.`.
  **L804 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting 128-bit vector of [4 x float] is then returned.`。
- **L805 EN**: Separator comment used for visual grouping.
  **L805 CN**: 用于视觉分组的分隔注释。
- **L806 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L806 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L807 EN**: Separator comment used for visual grouping.
  **L807 CN**: 用于视觉分组的分隔注释。
- **L808 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L808 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L809 EN**: Comment explains nearby logic, constraints, or intent: `__m128 _mm_insert_ps(__m128 X, __m128 Y, const int N);`.
  **L809 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128 _mm_insert_ps(__m128 X, __m128 Y, const int N);`。
- **L810 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L810 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L811 EN**: Separator comment used for visual grouping.
  **L811 CN**: 用于视觉分组的分隔注释。
- **L812 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VINSERTPS </c> instruction.`.
  **L812 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VINSERTPS </c> instruction.`。
- **L813 EN**: Separator comment used for visual grouping.
  **L813 CN**: 用于视觉分组的分隔注释。
- **L814 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L814 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L815 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector source operand of [4 x float]. With the exception of`.
  **L815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector source operand of [4 x float]. With the exception of`。
- **L816 EN**: Comment explains nearby logic, constraints, or intent: `those bits in the result copied from parameter a Y and zeroed by bits`.
  **L816 CN**: 注释解释附近代码的逻辑、约束或设计意图：`those bits in the result copied from parameter a Y and zeroed by bits`。

### Lines 817-840

````c
///    [3:0] of \a N, all bits from this parameter are copied to the result.
/// \param Y
///    A 128-bit vector source operand of [4 x float]. One single-precision
///    floating-point element from this source, as determined by the immediate
///    parameter, is copied to the result.
/// \param N
///    Specifies which bits from operand \a Y will be copied, which bits in the
///    result they will be copied to, and which bits in the result will be
///    cleared. The following assignments are made: \n
///    Bits [7:6] specify the bits to copy from operand \a Y: \n
///      00: Selects bits [31:0] from operand \a Y. \n
///      01: Selects bits [63:32] from operand \a Y. \n
///      10: Selects bits [95:64] from operand \a Y. \n
///      11: Selects bits [127:96] from operand \a Y. \n
///    Bits [5:4] specify the bits in the result to which the selected bits
///    from operand \a Y are copied: \n
///      00: Copies the selected bits from \a Y to result bits [31:0]. \n
///      01: Copies the selected bits from \a Y to result bits [63:32]. \n
///      10: Copies the selected bits from \a Y to result bits [95:64]. \n
///      11: Copies the selected bits from \a Y to result bits [127:96]. \n
///    Bits[3:0]: If any of these bits are set, the corresponding result
///    element is cleared.
/// \returns A 128-bit vector of [4 x float] containing the copied
///    single-precision floating point elements from the operands.
````
- **L817 EN**: Comment explains nearby logic, constraints, or intent: `[3:0] of a N, all bits from this parameter are copied to the result.`.
  **L817 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[3:0] of a N, all bits from this parameter are copied to the result.`。
- **L818 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L818 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L819 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector source operand of [4 x float]. One single-precision`.
  **L819 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector source operand of [4 x float]. One single-precision`。
- **L820 EN**: Comment explains nearby logic, constraints, or intent: `floating-point element from this source, as determined by the immediate`.
  **L820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point element from this source, as determined by the immediate`。
- **L821 EN**: Comment explains nearby logic, constraints, or intent: `parameter, is copied to the result.`.
  **L821 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter, is copied to the result.`。
- **L822 EN**: Comment explains nearby logic, constraints, or intent: `param N`.
  **L822 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param N`。
- **L823 EN**: Comment explains nearby logic, constraints, or intent: `Specifies which bits from operand a Y will be copied, which bits in the`.
  **L823 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Specifies which bits from operand a Y will be copied, which bits in the`。
- **L824 EN**: Comment explains nearby logic, constraints, or intent: `result they will be copied to, and which bits in the result will be`.
  **L824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result they will be copied to, and which bits in the result will be`。
- **L825 EN**: Comment explains nearby logic, constraints, or intent: `cleared. The following assignments are made: n`.
  **L825 CN**: 注释解释附近代码的逻辑、约束或设计意图：`cleared. The following assignments are made: n`。
- **L826 EN**: Comment explains nearby logic, constraints, or intent: `Bits [7:6] specify the bits to copy from operand a Y: n`.
  **L826 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [7:6] specify the bits to copy from operand a Y: n`。
- **L827 EN**: Comment explains nearby logic, constraints, or intent: `00: Selects bits [31:0] from operand a Y. n`.
  **L827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Selects bits [31:0] from operand a Y. n`。
- **L828 EN**: Comment explains nearby logic, constraints, or intent: `01: Selects bits [63:32] from operand a Y. n`.
  **L828 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Selects bits [63:32] from operand a Y. n`。
- **L829 EN**: Comment explains nearby logic, constraints, or intent: `10: Selects bits [95:64] from operand a Y. n`.
  **L829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Selects bits [95:64] from operand a Y. n`。
- **L830 EN**: Comment explains nearby logic, constraints, or intent: `11: Selects bits [127:96] from operand a Y. n`.
  **L830 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Selects bits [127:96] from operand a Y. n`。
- **L831 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4] specify the bits in the result to which the selected bits`.
  **L831 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4] specify the bits in the result to which the selected bits`。
- **L832 EN**: Comment explains nearby logic, constraints, or intent: `from operand a Y are copied: n`.
  **L832 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from operand a Y are copied: n`。
- **L833 EN**: Comment explains nearby logic, constraints, or intent: `00: Copies the selected bits from a Y to result bits [31:0]. n`.
  **L833 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Copies the selected bits from a Y to result bits [31:0]. n`。
- **L834 EN**: Comment explains nearby logic, constraints, or intent: `01: Copies the selected bits from a Y to result bits [63:32]. n`.
  **L834 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Copies the selected bits from a Y to result bits [63:32]. n`。
- **L835 EN**: Comment explains nearby logic, constraints, or intent: `10: Copies the selected bits from a Y to result bits [95:64]. n`.
  **L835 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Copies the selected bits from a Y to result bits [95:64]. n`。
- **L836 EN**: Comment explains nearby logic, constraints, or intent: `11: Copies the selected bits from a Y to result bits [127:96]. n`.
  **L836 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Copies the selected bits from a Y to result bits [127:96]. n`。
- **L837 EN**: Comment explains nearby logic, constraints, or intent: `Bits[3:0]: If any of these bits are set, the corresponding result`.
  **L837 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits[3:0]: If any of these bits are set, the corresponding result`。
- **L838 EN**: Comment explains nearby logic, constraints, or intent: `element is cleared.`.
  **L838 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element is cleared.`。
- **L839 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the copied`.
  **L839 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the copied`。
- **L840 EN**: Comment explains nearby logic, constraints, or intent: `single-precision floating point elements from the operands.`.
  **L840 CN**: 注释解释附近代码的逻辑、约束或设计意图：`single-precision floating point elements from the operands.`。

### Lines 841-864

````c
#define _mm_insert_ps(X, Y, N) __builtin_ia32_insertps128((X), (Y), (N))

/// Extracts a 32-bit integer from a 128-bit vector of [4 x float] and
///    returns it, using the immediate value parameter \a N as a selector.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_extract_ps(__m128 X, const int N);
/// \endcode
///
/// This intrinsic corresponds to the <c> VEXTRACTPS / EXTRACTPS </c>
/// instruction.
///
/// \param X
///    A 128-bit vector of [4 x float].
/// \param N
///    An immediate value. Bits [1:0] determines which bits from the argument
///    \a X are extracted and returned: \n
///    00: Bits [31:0] of parameter \a X are returned. \n
///    01: Bits [63:32] of parameter \a X are returned. \n
///    10: Bits [95:64] of parameter \a X are returned. \n
///    11: Bits [127:96] of parameter \a X are returned.
/// \returns A 32-bit integer containing the extracted 32 bits of float data.
````
- **L841 EN**: Defines macro `_mm_insert_ps(X, Y, N)` for conditional compilation, shorthand, or API generation.
  **L841 CN**: 定义宏 `_mm_insert_ps(X, Y, N)`，用于条件编译、简写或 API 生成。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L843 EN**: Comment explains nearby logic, constraints, or intent: `Extracts a 32-bit integer from a 128-bit vector of [4 x float] and`.
  **L843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extracts a 32-bit integer from a 128-bit vector of [4 x float] and`。
- **L844 EN**: Comment explains nearby logic, constraints, or intent: `returns it, using the immediate value parameter a N as a selector.`.
  **L844 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns it, using the immediate value parameter a N as a selector.`。
- **L845 EN**: Separator comment used for visual grouping.
  **L845 CN**: 用于视觉分组的分隔注释。
- **L846 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L846 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L847 EN**: Separator comment used for visual grouping.
  **L847 CN**: 用于视觉分组的分隔注释。
- **L848 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L848 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L849 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_extract_ps(__m128 X, const int N);`.
  **L849 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_extract_ps(__m128 X, const int N);`。
- **L850 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L850 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L851 EN**: Separator comment used for visual grouping.
  **L851 CN**: 用于视觉分组的分隔注释。
- **L852 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VEXTRACTPS / EXTRACTPS </c>`.
  **L852 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VEXTRACTPS / EXTRACTPS </c>`。
- **L853 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L853 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L854 EN**: Separator comment used for visual grouping.
  **L854 CN**: 用于视觉分组的分隔注释。
- **L855 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L855 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L856 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L857 EN**: Comment explains nearby logic, constraints, or intent: `param N`.
  **L857 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param N`。
- **L858 EN**: Comment explains nearby logic, constraints, or intent: `An immediate value. Bits [1:0] determines which bits from the argument`.
  **L858 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate value. Bits [1:0] determines which bits from the argument`。
- **L859 EN**: Comment explains nearby logic, constraints, or intent: `a X are extracted and returned: n`.
  **L859 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a X are extracted and returned: n`。
- **L860 EN**: Comment explains nearby logic, constraints, or intent: `00: Bits [31:0] of parameter a X are returned. n`.
  **L860 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Bits [31:0] of parameter a X are returned. n`。
- **L861 EN**: Comment explains nearby logic, constraints, or intent: `01: Bits [63:32] of parameter a X are returned. n`.
  **L861 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Bits [63:32] of parameter a X are returned. n`。
- **L862 EN**: Comment explains nearby logic, constraints, or intent: `10: Bits [95:64] of parameter a X are returned. n`.
  **L862 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Bits [95:64] of parameter a X are returned. n`。
- **L863 EN**: Comment explains nearby logic, constraints, or intent: `11: Bits [127:96] of parameter a X are returned.`.
  **L863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Bits [127:96] of parameter a X are returned.`。
- **L864 EN**: Comment explains nearby logic, constraints, or intent: `returns A 32-bit integer containing the extracted 32 bits of float data.`.
  **L864 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 32-bit integer containing the extracted 32 bits of float data.`。

### Lines 865-888

````c
#define _mm_extract_ps(X, N)                                                   \
  __builtin_bit_cast(                                                          \
      int, __builtin_ia32_vec_ext_v4sf((__v4sf)(__m128)(X), (int)(N)))

/* Miscellaneous insert and extract macros.  */
/* Extract a single-precision float from X at index N into D.  */
#define _MM_EXTRACT_FLOAT(D, X, N)                                             \
  do {                                                                         \
    (D) = __builtin_ia32_vec_ext_v4sf((__v4sf)(__m128)(X), (int)(N));          \
  } while (0)

/* Or together 2 sets of indexes (X and Y) with the zeroing bits (Z) to create
   an index suitable for _mm_insert_ps.  */
#define _MM_MK_INSERTPS_NDX(X, Y, Z) (((X) << 6) | ((Y) << 4) | (Z))

/* Extract a float from X at index N into the first index of the return.  */
#define _MM_PICK_OUT_PS(X, N)                                                  \
  _mm_insert_ps(_mm_setzero_ps(), (X), _MM_MK_INSERTPS_NDX((N), 0, 0x0e))

/* Insert int into packed integer array at index.  */
/// Constructs a 128-bit vector of [16 x i8] by first making a copy of
///    the 128-bit integer vector parameter, and then inserting the lower 8 bits
///    of an integer parameter \a I into an offset specified by the immediate
///    value parameter \a N.
````
- **L865 EN**: Defines macro `_mm_extract_ps(X, N)` for conditional compilation, shorthand, or API generation.
  **L865 CN**: 定义宏 `_mm_extract_ps(X, N)`，用于条件编译、简写或 API 生成。
- **L866 EN**: Continues logic associated with callable symbol `__builtin_bit_cast`.
  **L866 CN**: 继续与可调用符号 `__builtin_bit_cast` 相关的逻辑。
- **L867 EN**: Continues logic associated with callable symbol `__builtin_ia32_vec_ext_v4sf`.
  **L867 CN**: 继续与可调用符号 `__builtin_ia32_vec_ext_v4sf` 相关的逻辑。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L869 EN**: Comment explains nearby logic, constraints, or intent: `Miscellaneous insert and extract macros.`.
  **L869 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Miscellaneous insert and extract macros.`。
- **L870 EN**: Comment explains nearby logic, constraints, or intent: `Extract a single-precision float from X at index N into D.`.
  **L870 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract a single-precision float from X at index N into D.`。
- **L871 EN**: Defines macro `_MM_EXTRACT_FLOAT(D, X, N)` for conditional compilation, shorthand, or API generation.
  **L871 CN**: 定义宏 `_MM_EXTRACT_FLOAT(D, X, N)`，用于条件编译、简写或 API 生成。
- **L872 EN**: Continues the surrounding expression or declaration: `do {                                                                         \`.
  **L872 CN**: 继续构造周围的表达式或声明：`do {                                                                         \`。
- **L873 EN**: Continues logic associated with callable symbol `__builtin_ia32_vec_ext_v4sf`.
  **L873 CN**: 继续与可调用符号 `__builtin_ia32_vec_ext_v4sf` 相关的逻辑。
- **L874 EN**: Continues the surrounding expression or declaration: `} while (0)`.
  **L874 CN**: 继续构造周围的表达式或声明：`} while (0)`。
- **L875 EN**: Blank line separating nearby declarations or logic blocks.
  **L875 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L876 EN**: Comment explains nearby logic, constraints, or intent: `Or together 2 sets of indexes (X and Y) with the zeroing bits (Z) to create`.
  **L876 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Or together 2 sets of indexes (X and Y) with the zeroing bits (Z) to create`。
- **L877 EN**: Continues the surrounding expression or declaration: `an index suitable for _mm_insert_ps.  */`.
  **L877 CN**: 继续构造周围的表达式或声明：`an index suitable for _mm_insert_ps.  */`。
- **L878 EN**: Defines macro `_MM_MK_INSERTPS_NDX(X, Y, Z)` for conditional compilation, shorthand, or API generation.
  **L878 CN**: 定义宏 `_MM_MK_INSERTPS_NDX(X, Y, Z)`，用于条件编译、简写或 API 生成。
- **L879 EN**: Blank line separating nearby declarations or logic blocks.
  **L879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L880 EN**: Comment explains nearby logic, constraints, or intent: `Extract a float from X at index N into the first index of the return.`.
  **L880 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract a float from X at index N into the first index of the return.`。
- **L881 EN**: Defines macro `_MM_PICK_OUT_PS(X, N)` for conditional compilation, shorthand, or API generation.
  **L881 CN**: 定义宏 `_MM_PICK_OUT_PS(X, N)`，用于条件编译、简写或 API 生成。
- **L882 EN**: Continues logic associated with callable symbol `_mm_insert_ps`.
  **L882 CN**: 继续与可调用符号 `_mm_insert_ps` 相关的逻辑。
- **L883 EN**: Blank line separating nearby declarations or logic blocks.
  **L883 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L884 EN**: Comment explains nearby logic, constraints, or intent: `Insert int into packed integer array at index.`.
  **L884 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert int into packed integer array at index.`。
- **L885 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 128-bit vector of [16 x i8] by first making a copy of`.
  **L885 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 128-bit vector of [16 x i8] by first making a copy of`。
- **L886 EN**: Comment explains nearby logic, constraints, or intent: `the 128-bit integer vector parameter, and then inserting the lower 8 bits`.
  **L886 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the 128-bit integer vector parameter, and then inserting the lower 8 bits`。
- **L887 EN**: Comment explains nearby logic, constraints, or intent: `of an integer parameter a I into an offset specified by the immediate`.
  **L887 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of an integer parameter a I into an offset specified by the immediate`。
- **L888 EN**: Comment explains nearby logic, constraints, or intent: `value parameter a N.`.
  **L888 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value parameter a N.`。

### Lines 889-912

````c
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128i _mm_insert_epi8(__m128i X, int I, const int N);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPINSRB / PINSRB </c> instruction.
///
/// \param X
///    A 128-bit integer vector of [16 x i8]. This vector is copied to the
///    result and then one of the sixteen elements in the result vector is
///    replaced by the lower 8 bits of \a I.
/// \param I
///    An integer. The lower 8 bits of this operand are written to the result
///    beginning at the offset specified by \a N.
/// \param N
///    An immediate value. Bits [3:0] specify the bit offset in the result at
///    which the lower 8 bits of \a I are written. \n
///    0000: Bits [7:0] of the result are used for insertion. \n
///    0001: Bits [15:8] of the result are used for insertion. \n
///    0010: Bits [23:16] of the result are used for insertion. \n
///    0011: Bits [31:24] of the result are used for insertion. \n
///    0100: Bits [39:32] of the result are used for insertion. \n
````
- **L889 EN**: Separator comment used for visual grouping.
  **L889 CN**: 用于视觉分组的分隔注释。
- **L890 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L890 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L891 EN**: Separator comment used for visual grouping.
  **L891 CN**: 用于视觉分组的分隔注释。
- **L892 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L892 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L893 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_insert_epi8(__m128i X, int I, const int N);`.
  **L893 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_insert_epi8(__m128i X, int I, const int N);`。
- **L894 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L894 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L895 EN**: Separator comment used for visual grouping.
  **L895 CN**: 用于视觉分组的分隔注释。
- **L896 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPINSRB / PINSRB </c> instruction.`.
  **L896 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPINSRB / PINSRB </c> instruction.`。
- **L897 EN**: Separator comment used for visual grouping.
  **L897 CN**: 用于视觉分组的分隔注释。
- **L898 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L898 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L899 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector of [16 x i8]. This vector is copied to the`.
  **L899 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector of [16 x i8]. This vector is copied to the`。
- **L900 EN**: Comment explains nearby logic, constraints, or intent: `result and then one of the sixteen elements in the result vector is`.
  **L900 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result and then one of the sixteen elements in the result vector is`。
- **L901 EN**: Comment explains nearby logic, constraints, or intent: `replaced by the lower 8 bits of a I.`.
  **L901 CN**: 注释解释附近代码的逻辑、约束或设计意图：`replaced by the lower 8 bits of a I.`。
- **L902 EN**: Comment explains nearby logic, constraints, or intent: `param I`.
  **L902 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param I`。
- **L903 EN**: Comment explains nearby logic, constraints, or intent: `An integer. The lower 8 bits of this operand are written to the result`.
  **L903 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer. The lower 8 bits of this operand are written to the result`。
- **L904 EN**: Comment explains nearby logic, constraints, or intent: `beginning at the offset specified by a N.`.
  **L904 CN**: 注释解释附近代码的逻辑、约束或设计意图：`beginning at the offset specified by a N.`。
- **L905 EN**: Comment explains nearby logic, constraints, or intent: `param N`.
  **L905 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param N`。
- **L906 EN**: Comment explains nearby logic, constraints, or intent: `An immediate value. Bits [3:0] specify the bit offset in the result at`.
  **L906 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate value. Bits [3:0] specify the bit offset in the result at`。
- **L907 EN**: Comment explains nearby logic, constraints, or intent: `which the lower 8 bits of a I are written. n`.
  **L907 CN**: 注释解释附近代码的逻辑、约束或设计意图：`which the lower 8 bits of a I are written. n`。
- **L908 EN**: Comment explains nearby logic, constraints, or intent: `0000: Bits [7:0] of the result are used for insertion. n`.
  **L908 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0000: Bits [7:0] of the result are used for insertion. n`。
- **L909 EN**: Comment explains nearby logic, constraints, or intent: `0001: Bits [15:8] of the result are used for insertion. n`.
  **L909 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0001: Bits [15:8] of the result are used for insertion. n`。
- **L910 EN**: Comment explains nearby logic, constraints, or intent: `0010: Bits [23:16] of the result are used for insertion. n`.
  **L910 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0010: Bits [23:16] of the result are used for insertion. n`。
- **L911 EN**: Comment explains nearby logic, constraints, or intent: `0011: Bits [31:24] of the result are used for insertion. n`.
  **L911 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0011: Bits [31:24] of the result are used for insertion. n`。
- **L912 EN**: Comment explains nearby logic, constraints, or intent: `0100: Bits [39:32] of the result are used for insertion. n`.
  **L912 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0100: Bits [39:32] of the result are used for insertion. n`。

### Lines 913-936

````c
///    0101: Bits [47:40] of the result are used for insertion. \n
///    0110: Bits [55:48] of the result are used for insertion. \n
///    0111: Bits [63:56] of the result are used for insertion. \n
///    1000: Bits [71:64] of the result are used for insertion. \n
///    1001: Bits [79:72] of the result are used for insertion. \n
///    1010: Bits [87:80] of the result are used for insertion. \n
///    1011: Bits [95:88] of the result are used for insertion. \n
///    1100: Bits [103:96] of the result are used for insertion. \n
///    1101: Bits [111:104] of the result are used for insertion. \n
///    1110: Bits [119:112] of the result are used for insertion. \n
///    1111: Bits [127:120] of the result are used for insertion.
/// \returns A 128-bit integer vector containing the constructed values.
#define _mm_insert_epi8(X, I, N)                                               \
  ((__m128i)__builtin_ia32_vec_set_v16qi((__v16qi)(__m128i)(X), (int)(I),      \
                                         (int)(N)))

/// Constructs a 128-bit vector of [4 x i32] by first making a copy of
///    the 128-bit integer vector parameter, and then inserting the 32-bit
///    integer parameter \a I at the offset specified by the immediate value
///    parameter \a N.
///
/// \headerfile <x86intrin.h>
///
/// \code
````
- **L913 EN**: Comment explains nearby logic, constraints, or intent: `0101: Bits [47:40] of the result are used for insertion. n`.
  **L913 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0101: Bits [47:40] of the result are used for insertion. n`。
- **L914 EN**: Comment explains nearby logic, constraints, or intent: `0110: Bits [55:48] of the result are used for insertion. n`.
  **L914 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0110: Bits [55:48] of the result are used for insertion. n`。
- **L915 EN**: Comment explains nearby logic, constraints, or intent: `0111: Bits [63:56] of the result are used for insertion. n`.
  **L915 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0111: Bits [63:56] of the result are used for insertion. n`。
- **L916 EN**: Comment explains nearby logic, constraints, or intent: `1000: Bits [71:64] of the result are used for insertion. n`.
  **L916 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1000: Bits [71:64] of the result are used for insertion. n`。
- **L917 EN**: Comment explains nearby logic, constraints, or intent: `1001: Bits [79:72] of the result are used for insertion. n`.
  **L917 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1001: Bits [79:72] of the result are used for insertion. n`。
- **L918 EN**: Comment explains nearby logic, constraints, or intent: `1010: Bits [87:80] of the result are used for insertion. n`.
  **L918 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1010: Bits [87:80] of the result are used for insertion. n`。
- **L919 EN**: Comment explains nearby logic, constraints, or intent: `1011: Bits [95:88] of the result are used for insertion. n`.
  **L919 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1011: Bits [95:88] of the result are used for insertion. n`。
- **L920 EN**: Comment explains nearby logic, constraints, or intent: `1100: Bits [103:96] of the result are used for insertion. n`.
  **L920 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1100: Bits [103:96] of the result are used for insertion. n`。
- **L921 EN**: Comment explains nearby logic, constraints, or intent: `1101: Bits [111:104] of the result are used for insertion. n`.
  **L921 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1101: Bits [111:104] of the result are used for insertion. n`。
- **L922 EN**: Comment explains nearby logic, constraints, or intent: `1110: Bits [119:112] of the result are used for insertion. n`.
  **L922 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1110: Bits [119:112] of the result are used for insertion. n`。
- **L923 EN**: Comment explains nearby logic, constraints, or intent: `1111: Bits [127:120] of the result are used for insertion.`.
  **L923 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1111: Bits [127:120] of the result are used for insertion.`。
- **L924 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit integer vector containing the constructed values.`.
  **L924 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit integer vector containing the constructed values.`。
- **L925 EN**: Defines macro `_mm_insert_epi8(X, I, N)` for conditional compilation, shorthand, or API generation.
  **L925 CN**: 定义宏 `_mm_insert_epi8(X, I, N)`，用于条件编译、简写或 API 生成。
- **L926 EN**: Continues logic associated with callable symbol `__builtin_ia32_vec_set_v16qi`.
  **L926 CN**: 继续与可调用符号 `__builtin_ia32_vec_set_v16qi` 相关的逻辑。
- **L927 EN**: Continues the surrounding expression or declaration: `(int)(N)))`.
  **L927 CN**: 继续构造周围的表达式或声明：`(int)(N)))`。
- **L928 EN**: Blank line separating nearby declarations or logic blocks.
  **L928 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L929 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 128-bit vector of [4 x i32] by first making a copy of`.
  **L929 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 128-bit vector of [4 x i32] by first making a copy of`。
- **L930 EN**: Comment explains nearby logic, constraints, or intent: `the 128-bit integer vector parameter, and then inserting the 32-bit`.
  **L930 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the 128-bit integer vector parameter, and then inserting the 32-bit`。
- **L931 EN**: Comment explains nearby logic, constraints, or intent: `integer parameter a I at the offset specified by the immediate value`.
  **L931 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer parameter a I at the offset specified by the immediate value`。
- **L932 EN**: Comment explains nearby logic, constraints, or intent: `parameter a N.`.
  **L932 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter a N.`。
- **L933 EN**: Separator comment used for visual grouping.
  **L933 CN**: 用于视觉分组的分隔注释。
- **L934 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L934 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L935 EN**: Separator comment used for visual grouping.
  **L935 CN**: 用于视觉分组的分隔注释。
- **L936 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L936 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。

### Lines 937-960

````c
/// __m128i _mm_insert_epi32(__m128i X, int I, const int N);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPINSRD / PINSRD </c> instruction.
///
/// \param X
///    A 128-bit integer vector of [4 x i32]. This vector is copied to the
///    result and then one of the four elements in the result vector is
///    replaced by \a I.
/// \param I
///    A 32-bit integer that is written to the result beginning at the offset
///    specified by \a N.
/// \param N
///    An immediate value. Bits [1:0] specify the bit offset in the result at
///    which the integer \a I is written. \n
///    00: Bits [31:0] of the result are used for insertion. \n
///    01: Bits [63:32] of the result are used for insertion. \n
///    10: Bits [95:64] of the result are used for insertion. \n
///    11: Bits [127:96] of the result are used for insertion.
/// \returns A 128-bit integer vector containing the constructed values.
#define _mm_insert_epi32(X, I, N)                                              \
  ((__m128i)__builtin_ia32_vec_set_v4si((__v4si)(__m128i)(X), (int)(I),        \
                                        (int)(N)))

````
- **L937 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_insert_epi32(__m128i X, int I, const int N);`.
  **L937 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_insert_epi32(__m128i X, int I, const int N);`。
- **L938 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L938 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L939 EN**: Separator comment used for visual grouping.
  **L939 CN**: 用于视觉分组的分隔注释。
- **L940 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPINSRD / PINSRD </c> instruction.`.
  **L940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPINSRD / PINSRD </c> instruction.`。
- **L941 EN**: Separator comment used for visual grouping.
  **L941 CN**: 用于视觉分组的分隔注释。
- **L942 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L942 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L943 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector of [4 x i32]. This vector is copied to the`.
  **L943 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector of [4 x i32]. This vector is copied to the`。
- **L944 EN**: Comment explains nearby logic, constraints, or intent: `result and then one of the four elements in the result vector is`.
  **L944 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result and then one of the four elements in the result vector is`。
- **L945 EN**: Comment explains nearby logic, constraints, or intent: `replaced by a I.`.
  **L945 CN**: 注释解释附近代码的逻辑、约束或设计意图：`replaced by a I.`。
- **L946 EN**: Comment explains nearby logic, constraints, or intent: `param I`.
  **L946 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param I`。
- **L947 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer that is written to the result beginning at the offset`.
  **L947 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer that is written to the result beginning at the offset`。
- **L948 EN**: Comment explains nearby logic, constraints, or intent: `specified by a N.`.
  **L948 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified by a N.`。
- **L949 EN**: Comment explains nearby logic, constraints, or intent: `param N`.
  **L949 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param N`。
- **L950 EN**: Comment explains nearby logic, constraints, or intent: `An immediate value. Bits [1:0] specify the bit offset in the result at`.
  **L950 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate value. Bits [1:0] specify the bit offset in the result at`。
- **L951 EN**: Comment explains nearby logic, constraints, or intent: `which the integer a I is written. n`.
  **L951 CN**: 注释解释附近代码的逻辑、约束或设计意图：`which the integer a I is written. n`。
- **L952 EN**: Comment explains nearby logic, constraints, or intent: `00: Bits [31:0] of the result are used for insertion. n`.
  **L952 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Bits [31:0] of the result are used for insertion. n`。
- **L953 EN**: Comment explains nearby logic, constraints, or intent: `01: Bits [63:32] of the result are used for insertion. n`.
  **L953 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Bits [63:32] of the result are used for insertion. n`。
- **L954 EN**: Comment explains nearby logic, constraints, or intent: `10: Bits [95:64] of the result are used for insertion. n`.
  **L954 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Bits [95:64] of the result are used for insertion. n`。
- **L955 EN**: Comment explains nearby logic, constraints, or intent: `11: Bits [127:96] of the result are used for insertion.`.
  **L955 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Bits [127:96] of the result are used for insertion.`。
- **L956 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit integer vector containing the constructed values.`.
  **L956 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit integer vector containing the constructed values.`。
- **L957 EN**: Defines macro `_mm_insert_epi32(X, I, N)` for conditional compilation, shorthand, or API generation.
  **L957 CN**: 定义宏 `_mm_insert_epi32(X, I, N)`，用于条件编译、简写或 API 生成。
- **L958 EN**: Continues logic associated with callable symbol `__builtin_ia32_vec_set_v4si`.
  **L958 CN**: 继续与可调用符号 `__builtin_ia32_vec_set_v4si` 相关的逻辑。
- **L959 EN**: Continues the surrounding expression or declaration: `(int)(N)))`.
  **L959 CN**: 继续构造周围的表达式或声明：`(int)(N)))`。
- **L960 EN**: Blank line separating nearby declarations or logic blocks.
  **L960 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 961-984

````c
#ifdef __x86_64__
/// Constructs a 128-bit vector of [2 x i64] by first making a copy of
///    the 128-bit integer vector parameter, and then inserting the 64-bit
///    integer parameter \a I, using the immediate value parameter \a N as an
///    insertion location selector.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128i _mm_insert_epi64(__m128i X, long long I, const int N);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPINSRQ / PINSRQ </c> instruction.
///
/// \param X
///    A 128-bit integer vector of [2 x i64]. This vector is copied to the
///    result and then one of the two elements in the result vector is replaced
///    by \a I.
/// \param I
///    A 64-bit integer that is written to the result beginning at the offset
///    specified by \a N.
/// \param N
///    An immediate value. Bit [0] specifies the bit offset in the result at
///    which the integer \a I is written. \n
````
- **L961 EN**: Starts a preprocessor conditional block: `#ifdef __x86_64__`.
  **L961 CN**: 开始一个预处理条件块：`#ifdef __x86_64__`。
- **L962 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 128-bit vector of [2 x i64] by first making a copy of`.
  **L962 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 128-bit vector of [2 x i64] by first making a copy of`。
- **L963 EN**: Comment explains nearby logic, constraints, or intent: `the 128-bit integer vector parameter, and then inserting the 64-bit`.
  **L963 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the 128-bit integer vector parameter, and then inserting the 64-bit`。
- **L964 EN**: Comment explains nearby logic, constraints, or intent: `integer parameter a I, using the immediate value parameter a N as an`.
  **L964 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer parameter a I, using the immediate value parameter a N as an`。
- **L965 EN**: Comment explains nearby logic, constraints, or intent: `insertion location selector.`.
  **L965 CN**: 注释解释附近代码的逻辑、约束或设计意图：`insertion location selector.`。
- **L966 EN**: Separator comment used for visual grouping.
  **L966 CN**: 用于视觉分组的分隔注释。
- **L967 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L967 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L968 EN**: Separator comment used for visual grouping.
  **L968 CN**: 用于视觉分组的分隔注释。
- **L969 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L969 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L970 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_insert_epi64(__m128i X, long long I, const int N);`.
  **L970 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_insert_epi64(__m128i X, long long I, const int N);`。
- **L971 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L971 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L972 EN**: Separator comment used for visual grouping.
  **L972 CN**: 用于视觉分组的分隔注释。
- **L973 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPINSRQ / PINSRQ </c> instruction.`.
  **L973 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPINSRQ / PINSRQ </c> instruction.`。
- **L974 EN**: Separator comment used for visual grouping.
  **L974 CN**: 用于视觉分组的分隔注释。
- **L975 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L975 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L976 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector of [2 x i64]. This vector is copied to the`.
  **L976 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector of [2 x i64]. This vector is copied to the`。
- **L977 EN**: Comment explains nearby logic, constraints, or intent: `result and then one of the two elements in the result vector is replaced`.
  **L977 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result and then one of the two elements in the result vector is replaced`。
- **L978 EN**: Comment explains nearby logic, constraints, or intent: `by a I.`.
  **L978 CN**: 注释解释附近代码的逻辑、约束或设计意图：`by a I.`。
- **L979 EN**: Comment explains nearby logic, constraints, or intent: `param I`.
  **L979 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param I`。
- **L980 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer that is written to the result beginning at the offset`.
  **L980 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer that is written to the result beginning at the offset`。
- **L981 EN**: Comment explains nearby logic, constraints, or intent: `specified by a N.`.
  **L981 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified by a N.`。
- **L982 EN**: Comment explains nearby logic, constraints, or intent: `param N`.
  **L982 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param N`。
- **L983 EN**: Comment explains nearby logic, constraints, or intent: `An immediate value. Bit [0] specifies the bit offset in the result at`.
  **L983 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate value. Bit [0] specifies the bit offset in the result at`。
- **L984 EN**: Comment explains nearby logic, constraints, or intent: `which the integer a I is written. n`.
  **L984 CN**: 注释解释附近代码的逻辑、约束或设计意图：`which the integer a I is written. n`。

### Lines 985-1008

````c
///    0: Bits [63:0] of the result are used for insertion. \n
///    1: Bits [127:64] of the result are used for insertion. \n
/// \returns A 128-bit integer vector containing the constructed values.
#define _mm_insert_epi64(X, I, N)                                              \
  ((__m128i)__builtin_ia32_vec_set_v2di((__v2di)(__m128i)(X), (long long)(I),  \
                                        (int)(N)))
#endif /* __x86_64__ */

/* Extract int from packed integer array at index.  This returns the element
 * as a zero extended value, so it is unsigned.
 */
/// Extracts an 8-bit element from the 128-bit integer vector of
///    [16 x i8], using the immediate value parameter \a N as a selector.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_extract_epi8(__m128i X, const int N);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPEXTRB / PEXTRB </c> instruction.
///
/// \param X
///    A 128-bit integer vector.
````
- **L985 EN**: Comment explains nearby logic, constraints, or intent: `0: Bits [63:0] of the result are used for insertion. n`.
  **L985 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: Bits [63:0] of the result are used for insertion. n`。
- **L986 EN**: Comment explains nearby logic, constraints, or intent: `1: Bits [127:64] of the result are used for insertion. n`.
  **L986 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: Bits [127:64] of the result are used for insertion. n`。
- **L987 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit integer vector containing the constructed values.`.
  **L987 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit integer vector containing the constructed values.`。
- **L988 EN**: Defines macro `_mm_insert_epi64(X, I, N)` for conditional compilation, shorthand, or API generation.
  **L988 CN**: 定义宏 `_mm_insert_epi64(X, I, N)`，用于条件编译、简写或 API 生成。
- **L989 EN**: Continues logic associated with callable symbol `__builtin_ia32_vec_set_v2di`.
  **L989 CN**: 继续与可调用符号 `__builtin_ia32_vec_set_v2di` 相关的逻辑。
- **L990 EN**: Continues the surrounding expression or declaration: `(int)(N)))`.
  **L990 CN**: 继续构造周围的表达式或声明：`(int)(N)))`。
- **L991 EN**: Closes the current preprocessor conditional block.
  **L991 CN**: 结束当前预处理条件块。
- **L992 EN**: Blank line separating nearby declarations or logic blocks.
  **L992 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L993 EN**: Comment explains nearby logic, constraints, or intent: `Extract int from packed integer array at index. This returns the element`.
  **L993 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract int from packed integer array at index. This returns the element`。
- **L994 EN**: Comment explains nearby logic, constraints, or intent: `as a zero extended value, so it is unsigned.`.
  **L994 CN**: 注释解释附近代码的逻辑、约束或设计意图：`as a zero extended value, so it is unsigned.`。
- **L995 EN**: Separator comment used for visual grouping.
  **L995 CN**: 用于视觉分组的分隔注释。
- **L996 EN**: Comment explains nearby logic, constraints, or intent: `Extracts an 8-bit element from the 128-bit integer vector of`.
  **L996 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extracts an 8-bit element from the 128-bit integer vector of`。
- **L997 EN**: Comment explains nearby logic, constraints, or intent: `[16 x i8], using the immediate value parameter a N as a selector.`.
  **L997 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[16 x i8], using the immediate value parameter a N as a selector.`。
- **L998 EN**: Separator comment used for visual grouping.
  **L998 CN**: 用于视觉分组的分隔注释。
- **L999 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L999 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1000 EN**: Separator comment used for visual grouping.
  **L1000 CN**: 用于视觉分组的分隔注释。
- **L1001 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1001 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1002 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_extract_epi8(__m128i X, const int N);`.
  **L1002 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_extract_epi8(__m128i X, const int N);`。
- **L1003 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1003 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1004 EN**: Separator comment used for visual grouping.
  **L1004 CN**: 用于视觉分组的分隔注释。
- **L1005 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPEXTRB / PEXTRB </c> instruction.`.
  **L1005 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPEXTRB / PEXTRB </c> instruction.`。
- **L1006 EN**: Separator comment used for visual grouping.
  **L1006 CN**: 用于视觉分组的分隔注释。
- **L1007 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L1007 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L1008 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L1008 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。

### Lines 1009-1032

````c
/// \param N
///    An immediate value. Bits [3:0] specify which 8-bit vector element from
///    the argument \a X to extract and copy to the result. \n
///    0000: Bits [7:0] of parameter \a X are extracted. \n
///    0001: Bits [15:8] of the parameter \a X are extracted. \n
///    0010: Bits [23:16] of the parameter \a X are extracted. \n
///    0011: Bits [31:24] of the parameter \a X are extracted. \n
///    0100: Bits [39:32] of the parameter \a X are extracted. \n
///    0101: Bits [47:40] of the parameter \a X are extracted. \n
///    0110: Bits [55:48] of the parameter \a X are extracted. \n
///    0111: Bits [63:56] of the parameter \a X are extracted. \n
///    1000: Bits [71:64] of the parameter \a X are extracted. \n
///    1001: Bits [79:72] of the parameter \a X are extracted. \n
///    1010: Bits [87:80] of the parameter \a X are extracted. \n
///    1011: Bits [95:88] of the parameter \a X are extracted. \n
///    1100: Bits [103:96] of the parameter \a X are extracted. \n
///    1101: Bits [111:104] of the parameter \a X are extracted. \n
///    1110: Bits [119:112] of the parameter \a X are extracted. \n
///    1111: Bits [127:120] of the parameter \a X are extracted.
/// \returns  An unsigned integer, whose lower 8 bits are selected from the
///    128-bit integer vector parameter and the remaining bits are assigned
///    zeros.
#define _mm_extract_epi8(X, N)                                                 \
  ((int)(unsigned char)__builtin_ia32_vec_ext_v16qi((__v16qi)(__m128i)(X),     \
````
- **L1009 EN**: Comment explains nearby logic, constraints, or intent: `param N`.
  **L1009 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param N`。
- **L1010 EN**: Comment explains nearby logic, constraints, or intent: `An immediate value. Bits [3:0] specify which 8-bit vector element from`.
  **L1010 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate value. Bits [3:0] specify which 8-bit vector element from`。
- **L1011 EN**: Comment explains nearby logic, constraints, or intent: `the argument a X to extract and copy to the result. n`.
  **L1011 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the argument a X to extract and copy to the result. n`。
- **L1012 EN**: Comment explains nearby logic, constraints, or intent: `0000: Bits [7:0] of parameter a X are extracted. n`.
  **L1012 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0000: Bits [7:0] of parameter a X are extracted. n`。
- **L1013 EN**: Comment explains nearby logic, constraints, or intent: `0001: Bits [15:8] of the parameter a X are extracted. n`.
  **L1013 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0001: Bits [15:8] of the parameter a X are extracted. n`。
- **L1014 EN**: Comment explains nearby logic, constraints, or intent: `0010: Bits [23:16] of the parameter a X are extracted. n`.
  **L1014 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0010: Bits [23:16] of the parameter a X are extracted. n`。
- **L1015 EN**: Comment explains nearby logic, constraints, or intent: `0011: Bits [31:24] of the parameter a X are extracted. n`.
  **L1015 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0011: Bits [31:24] of the parameter a X are extracted. n`。
- **L1016 EN**: Comment explains nearby logic, constraints, or intent: `0100: Bits [39:32] of the parameter a X are extracted. n`.
  **L1016 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0100: Bits [39:32] of the parameter a X are extracted. n`。
- **L1017 EN**: Comment explains nearby logic, constraints, or intent: `0101: Bits [47:40] of the parameter a X are extracted. n`.
  **L1017 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0101: Bits [47:40] of the parameter a X are extracted. n`。
- **L1018 EN**: Comment explains nearby logic, constraints, or intent: `0110: Bits [55:48] of the parameter a X are extracted. n`.
  **L1018 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0110: Bits [55:48] of the parameter a X are extracted. n`。
- **L1019 EN**: Comment explains nearby logic, constraints, or intent: `0111: Bits [63:56] of the parameter a X are extracted. n`.
  **L1019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0111: Bits [63:56] of the parameter a X are extracted. n`。
- **L1020 EN**: Comment explains nearby logic, constraints, or intent: `1000: Bits [71:64] of the parameter a X are extracted. n`.
  **L1020 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1000: Bits [71:64] of the parameter a X are extracted. n`。
- **L1021 EN**: Comment explains nearby logic, constraints, or intent: `1001: Bits [79:72] of the parameter a X are extracted. n`.
  **L1021 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1001: Bits [79:72] of the parameter a X are extracted. n`。
- **L1022 EN**: Comment explains nearby logic, constraints, or intent: `1010: Bits [87:80] of the parameter a X are extracted. n`.
  **L1022 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1010: Bits [87:80] of the parameter a X are extracted. n`。
- **L1023 EN**: Comment explains nearby logic, constraints, or intent: `1011: Bits [95:88] of the parameter a X are extracted. n`.
  **L1023 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1011: Bits [95:88] of the parameter a X are extracted. n`。
- **L1024 EN**: Comment explains nearby logic, constraints, or intent: `1100: Bits [103:96] of the parameter a X are extracted. n`.
  **L1024 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1100: Bits [103:96] of the parameter a X are extracted. n`。
- **L1025 EN**: Comment explains nearby logic, constraints, or intent: `1101: Bits [111:104] of the parameter a X are extracted. n`.
  **L1025 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1101: Bits [111:104] of the parameter a X are extracted. n`。
- **L1026 EN**: Comment explains nearby logic, constraints, or intent: `1110: Bits [119:112] of the parameter a X are extracted. n`.
  **L1026 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1110: Bits [119:112] of the parameter a X are extracted. n`。
- **L1027 EN**: Comment explains nearby logic, constraints, or intent: `1111: Bits [127:120] of the parameter a X are extracted.`.
  **L1027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1111: Bits [127:120] of the parameter a X are extracted.`。
- **L1028 EN**: Comment explains nearby logic, constraints, or intent: `returns An unsigned integer, whose lower 8 bits are selected from the`.
  **L1028 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An unsigned integer, whose lower 8 bits are selected from the`。
- **L1029 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector parameter and the remaining bits are assigned`.
  **L1029 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector parameter and the remaining bits are assigned`。
- **L1030 EN**: Comment explains nearby logic, constraints, or intent: `zeros.`.
  **L1030 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zeros.`。
- **L1031 EN**: Defines macro `_mm_extract_epi8(X, N)` for conditional compilation, shorthand, or API generation.
  **L1031 CN**: 定义宏 `_mm_extract_epi8(X, N)`，用于条件编译、简写或 API 生成。
- **L1032 EN**: Continues logic associated with callable symbol `__builtin_ia32_vec_ext_v16qi`.
  **L1032 CN**: 继续与可调用符号 `__builtin_ia32_vec_ext_v16qi` 相关的逻辑。

### Lines 1033-1056

````c
                                                    (int)(N)))

/// Extracts a 32-bit element from the 128-bit integer vector of
///    [4 x i32], using the immediate value parameter \a N as a selector.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_extract_epi32(__m128i X, const int N);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPEXTRD / PEXTRD </c> instruction.
///
/// \param X
///    A 128-bit integer vector.
/// \param N
///    An immediate value. Bits [1:0] specify which 32-bit vector element from
///    the argument \a X to extract and copy to the result. \n
///    00: Bits [31:0] of the parameter \a X are extracted. \n
///    01: Bits [63:32] of the parameter \a X are extracted. \n
///    10: Bits [95:64] of the parameter \a X are extracted. \n
///    11: Bits [127:96] of the parameter \a X are exracted.
/// \returns  An integer, whose lower 32 bits are selected from the 128-bit
///    integer vector parameter and the remaining bits are assigned zeros.
````
- **L1033 EN**: Continues the surrounding expression or declaration: `(int)(N)))`.
  **L1033 CN**: 继续构造周围的表达式或声明：`(int)(N)))`。
- **L1034 EN**: Blank line separating nearby declarations or logic blocks.
  **L1034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1035 EN**: Comment explains nearby logic, constraints, or intent: `Extracts a 32-bit element from the 128-bit integer vector of`.
  **L1035 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extracts a 32-bit element from the 128-bit integer vector of`。
- **L1036 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i32], using the immediate value parameter a N as a selector.`.
  **L1036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i32], using the immediate value parameter a N as a selector.`。
- **L1037 EN**: Separator comment used for visual grouping.
  **L1037 CN**: 用于视觉分组的分隔注释。
- **L1038 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1038 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1039 EN**: Separator comment used for visual grouping.
  **L1039 CN**: 用于视觉分组的分隔注释。
- **L1040 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1041 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_extract_epi32(__m128i X, const int N);`.
  **L1041 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_extract_epi32(__m128i X, const int N);`。
- **L1042 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1042 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1043 EN**: Separator comment used for visual grouping.
  **L1043 CN**: 用于视觉分组的分隔注释。
- **L1044 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPEXTRD / PEXTRD </c> instruction.`.
  **L1044 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPEXTRD / PEXTRD </c> instruction.`。
- **L1045 EN**: Separator comment used for visual grouping.
  **L1045 CN**: 用于视觉分组的分隔注释。
- **L1046 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L1046 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L1047 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L1047 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。
- **L1048 EN**: Comment explains nearby logic, constraints, or intent: `param N`.
  **L1048 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param N`。
- **L1049 EN**: Comment explains nearby logic, constraints, or intent: `An immediate value. Bits [1:0] specify which 32-bit vector element from`.
  **L1049 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate value. Bits [1:0] specify which 32-bit vector element from`。
- **L1050 EN**: Comment explains nearby logic, constraints, or intent: `the argument a X to extract and copy to the result. n`.
  **L1050 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the argument a X to extract and copy to the result. n`。
- **L1051 EN**: Comment explains nearby logic, constraints, or intent: `00: Bits [31:0] of the parameter a X are extracted. n`.
  **L1051 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Bits [31:0] of the parameter a X are extracted. n`。
- **L1052 EN**: Comment explains nearby logic, constraints, or intent: `01: Bits [63:32] of the parameter a X are extracted. n`.
  **L1052 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Bits [63:32] of the parameter a X are extracted. n`。
- **L1053 EN**: Comment explains nearby logic, constraints, or intent: `10: Bits [95:64] of the parameter a X are extracted. n`.
  **L1053 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Bits [95:64] of the parameter a X are extracted. n`。
- **L1054 EN**: Comment explains nearby logic, constraints, or intent: `11: Bits [127:96] of the parameter a X are exracted.`.
  **L1054 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Bits [127:96] of the parameter a X are exracted.`。
- **L1055 EN**: Comment explains nearby logic, constraints, or intent: `returns An integer, whose lower 32 bits are selected from the 128-bit`.
  **L1055 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An integer, whose lower 32 bits are selected from the 128-bit`。
- **L1056 EN**: Comment explains nearby logic, constraints, or intent: `integer vector parameter and the remaining bits are assigned zeros.`.
  **L1056 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector parameter and the remaining bits are assigned zeros.`。

### Lines 1057-1080

````c
#define _mm_extract_epi32(X, N)                                                \
  ((int)__builtin_ia32_vec_ext_v4si((__v4si)(__m128i)(X), (int)(N)))

/// Extracts a 64-bit element from the 128-bit integer vector of
///    [2 x i64], using the immediate value parameter \a N as a selector.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// long long _mm_extract_epi64(__m128i X, const int N);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPEXTRQ / PEXTRQ </c> instruction
/// in 64-bit mode.
///
/// \param X
///    A 128-bit integer vector.
/// \param N
///    An immediate value. Bit [0] specifies which 64-bit vector element from
///    the argument \a X to return. \n
///    0: Bits [63:0] are returned. \n
///    1: Bits [127:64] are returned. \n
/// \returns  A 64-bit integer.
#define _mm_extract_epi64(X, N)                                                \
````
- **L1057 EN**: Defines macro `_mm_extract_epi32(X, N)` for conditional compilation, shorthand, or API generation.
  **L1057 CN**: 定义宏 `_mm_extract_epi32(X, N)`，用于条件编译、简写或 API 生成。
- **L1058 EN**: Continues logic associated with callable symbol `__builtin_ia32_vec_ext_v4si`.
  **L1058 CN**: 继续与可调用符号 `__builtin_ia32_vec_ext_v4si` 相关的逻辑。
- **L1059 EN**: Blank line separating nearby declarations or logic blocks.
  **L1059 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1060 EN**: Comment explains nearby logic, constraints, or intent: `Extracts a 64-bit element from the 128-bit integer vector of`.
  **L1060 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extracts a 64-bit element from the 128-bit integer vector of`。
- **L1061 EN**: Comment explains nearby logic, constraints, or intent: `[2 x i64], using the immediate value parameter a N as a selector.`.
  **L1061 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x i64], using the immediate value parameter a N as a selector.`。
- **L1062 EN**: Separator comment used for visual grouping.
  **L1062 CN**: 用于视觉分组的分隔注释。
- **L1063 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1063 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1064 EN**: Separator comment used for visual grouping.
  **L1064 CN**: 用于视觉分组的分隔注释。
- **L1065 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1065 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1066 EN**: Comment explains nearby logic, constraints, or intent: `long long _mm_extract_epi64(__m128i X, const int N);`.
  **L1066 CN**: 注释解释附近代码的逻辑、约束或设计意图：`long long _mm_extract_epi64(__m128i X, const int N);`。
- **L1067 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1067 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1068 EN**: Separator comment used for visual grouping.
  **L1068 CN**: 用于视觉分组的分隔注释。
- **L1069 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPEXTRQ / PEXTRQ </c> instruction`.
  **L1069 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPEXTRQ / PEXTRQ </c> instruction`。
- **L1070 EN**: Comment explains nearby logic, constraints, or intent: `in 64-bit mode.`.
  **L1070 CN**: 注释解释附近代码的逻辑、约束或设计意图：`in 64-bit mode.`。
- **L1071 EN**: Separator comment used for visual grouping.
  **L1071 CN**: 用于视觉分组的分隔注释。
- **L1072 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L1072 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。
- **L1073 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L1073 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。
- **L1074 EN**: Comment explains nearby logic, constraints, or intent: `param N`.
  **L1074 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param N`。
- **L1075 EN**: Comment explains nearby logic, constraints, or intent: `An immediate value. Bit [0] specifies which 64-bit vector element from`.
  **L1075 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An immediate value. Bit [0] specifies which 64-bit vector element from`。
- **L1076 EN**: Comment explains nearby logic, constraints, or intent: `the argument a X to return. n`.
  **L1076 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the argument a X to return. n`。
- **L1077 EN**: Comment explains nearby logic, constraints, or intent: `0: Bits [63:0] are returned. n`.
  **L1077 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: Bits [63:0] are returned. n`。
- **L1078 EN**: Comment explains nearby logic, constraints, or intent: `1: Bits [127:64] are returned. n`.
  **L1078 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: Bits [127:64] are returned. n`。
- **L1079 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer.`.
  **L1079 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer.`。
- **L1080 EN**: Defines macro `_mm_extract_epi64(X, N)` for conditional compilation, shorthand, or API generation.
  **L1080 CN**: 定义宏 `_mm_extract_epi64(X, N)`，用于条件编译、简写或 API 生成。

### Lines 1081-1104

````c
  ((long long)__builtin_ia32_vec_ext_v2di((__v2di)(__m128i)(X), (int)(N)))

/* SSE4 128-bit Packed Integer Comparisons.  */
/// Tests whether the specified bits in a 128-bit integer vector are all
///    zeros.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.
///
/// \param __M
///    A 128-bit integer vector containing the bits to be tested.
/// \param __V
///    A 128-bit integer vector selecting which bits to test in operand \a __M.
/// \returns TRUE if the specified bits are all zeros; FALSE otherwise.
static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_testz_si128(__m128i __M, __m128i __V) {
  return __builtin_ia32_ptestz128((__v2di)__M, (__v2di)__V);
}

/// Tests whether the specified bits in a 128-bit integer vector are all
///    ones.
///
/// \headerfile <x86intrin.h>
````
- **L1081 EN**: Continues logic associated with callable symbol `__builtin_ia32_vec_ext_v2di`.
  **L1081 CN**: 继续与可调用符号 `__builtin_ia32_vec_ext_v2di` 相关的逻辑。
- **L1082 EN**: Blank line separating nearby declarations or logic blocks.
  **L1082 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1083 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 128-bit Packed Integer Comparisons.`.
  **L1083 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 128-bit Packed Integer Comparisons.`。
- **L1084 EN**: Comment explains nearby logic, constraints, or intent: `Tests whether the specified bits in a 128-bit integer vector are all`.
  **L1084 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Tests whether the specified bits in a 128-bit integer vector are all`。
- **L1085 EN**: Comment explains nearby logic, constraints, or intent: `zeros.`.
  **L1085 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zeros.`。
- **L1086 EN**: Separator comment used for visual grouping.
  **L1086 CN**: 用于视觉分组的分隔注释。
- **L1087 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1087 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1088 EN**: Separator comment used for visual grouping.
  **L1088 CN**: 用于视觉分组的分隔注释。
- **L1089 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`.
  **L1089 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`。
- **L1090 EN**: Separator comment used for visual grouping.
  **L1090 CN**: 用于视觉分组的分隔注释。
- **L1091 EN**: Comment explains nearby logic, constraints, or intent: `param __M`.
  **L1091 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __M`。
- **L1092 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing the bits to be tested.`.
  **L1092 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing the bits to be tested.`。
- **L1093 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1093 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1094 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector selecting which bits to test in operand a __M.`.
  **L1094 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector selecting which bits to test in operand a __M.`。
- **L1095 EN**: Comment explains nearby logic, constraints, or intent: `returns TRUE if the specified bits are all zeros; FALSE otherwise.`.
  **L1095 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns TRUE if the specified bits are all zeros; FALSE otherwise.`。
- **L1096 EN**: Continues the surrounding expression or declaration: `static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1096 CN**: 继续构造周围的表达式或声明：`static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1097 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_testz_si128(__m128i __M, __m128i __V) {`.
  **L1097 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_testz_si128(__m128i __M, __m128i __V) {`。
- **L1098 EN**: Returns from the current function with `__builtin_ia32_ptestz128((__v2di)__M, (__v2di)__V)`.
  **L1098 CN**: 以 `__builtin_ia32_ptestz128((__v2di)__M, (__v2di)__V)` 从当前函数返回。
- **L1099 EN**: Closes the current lexical scope or compound statement.
  **L1099 CN**: 结束当前词法作用域或复合语句块。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1101 EN**: Comment explains nearby logic, constraints, or intent: `Tests whether the specified bits in a 128-bit integer vector are all`.
  **L1101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Tests whether the specified bits in a 128-bit integer vector are all`。
- **L1102 EN**: Comment explains nearby logic, constraints, or intent: `ones.`.
  **L1102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ones.`。
- **L1103 EN**: Separator comment used for visual grouping.
  **L1103 CN**: 用于视觉分组的分隔注释。
- **L1104 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。

### Lines 1105-1128

````c
///
/// This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.
///
/// \param __M
///    A 128-bit integer vector containing the bits to be tested.
/// \param __V
///    A 128-bit integer vector selecting which bits to test in operand \a __M.
/// \returns TRUE if the specified bits are all ones; FALSE otherwise.
static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_testc_si128(__m128i __M, __m128i __V) {
  return __builtin_ia32_ptestc128((__v2di)__M, (__v2di)__V);
}

/// Tests whether the specified bits in a 128-bit integer vector are
///    neither all zeros nor all ones.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.
///
/// \param __M
///    A 128-bit integer vector containing the bits to be tested.
/// \param __V
///    A 128-bit integer vector selecting which bits to test in operand \a __M.
````
- **L1105 EN**: Separator comment used for visual grouping.
  **L1105 CN**: 用于视觉分组的分隔注释。
- **L1106 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`.
  **L1106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`。
- **L1107 EN**: Separator comment used for visual grouping.
  **L1107 CN**: 用于视觉分组的分隔注释。
- **L1108 EN**: Comment explains nearby logic, constraints, or intent: `param __M`.
  **L1108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __M`。
- **L1109 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing the bits to be tested.`.
  **L1109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing the bits to be tested.`。
- **L1110 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1111 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector selecting which bits to test in operand a __M.`.
  **L1111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector selecting which bits to test in operand a __M.`。
- **L1112 EN**: Comment explains nearby logic, constraints, or intent: `returns TRUE if the specified bits are all ones; FALSE otherwise.`.
  **L1112 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns TRUE if the specified bits are all ones; FALSE otherwise.`。
- **L1113 EN**: Continues the surrounding expression or declaration: `static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1113 CN**: 继续构造周围的表达式或声明：`static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1114 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_testc_si128(__m128i __M, __m128i __V) {`.
  **L1114 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_testc_si128(__m128i __M, __m128i __V) {`。
- **L1115 EN**: Returns from the current function with `__builtin_ia32_ptestc128((__v2di)__M, (__v2di)__V)`.
  **L1115 CN**: 以 `__builtin_ia32_ptestc128((__v2di)__M, (__v2di)__V)` 从当前函数返回。
- **L1116 EN**: Closes the current lexical scope or compound statement.
  **L1116 CN**: 结束当前词法作用域或复合语句块。
- **L1117 EN**: Blank line separating nearby declarations or logic blocks.
  **L1117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1118 EN**: Comment explains nearby logic, constraints, or intent: `Tests whether the specified bits in a 128-bit integer vector are`.
  **L1118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Tests whether the specified bits in a 128-bit integer vector are`。
- **L1119 EN**: Comment explains nearby logic, constraints, or intent: `neither all zeros nor all ones.`.
  **L1119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`neither all zeros nor all ones.`。
- **L1120 EN**: Separator comment used for visual grouping.
  **L1120 CN**: 用于视觉分组的分隔注释。
- **L1121 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1122 EN**: Separator comment used for visual grouping.
  **L1122 CN**: 用于视觉分组的分隔注释。
- **L1123 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`.
  **L1123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`。
- **L1124 EN**: Separator comment used for visual grouping.
  **L1124 CN**: 用于视觉分组的分隔注释。
- **L1125 EN**: Comment explains nearby logic, constraints, or intent: `param __M`.
  **L1125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __M`。
- **L1126 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing the bits to be tested.`.
  **L1126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing the bits to be tested.`。
- **L1127 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1128 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector selecting which bits to test in operand a __M.`.
  **L1128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector selecting which bits to test in operand a __M.`。

### Lines 1129-1152

````c
/// \returns TRUE if the specified bits are neither all zeros nor all ones;
///    FALSE otherwise.
static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_testnzc_si128(__m128i __M, __m128i __V) {
  return __builtin_ia32_ptestnzc128((__v2di)__M, (__v2di)__V);
}

/// Tests whether the specified bits in a 128-bit integer vector are all
///    ones.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_test_all_ones(__m128i V);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.
///
/// \param V
///    A 128-bit integer vector containing the bits to be tested.
/// \returns TRUE if the bits specified in the operand are all set to 1; FALSE
///    otherwise.
#define _mm_test_all_ones(V) _mm_testc_si128((V), _mm_set1_epi32(-1))

````
- **L1129 EN**: Comment explains nearby logic, constraints, or intent: `returns TRUE if the specified bits are neither all zeros nor all ones;`.
  **L1129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns TRUE if the specified bits are neither all zeros nor all ones;`。
- **L1130 EN**: Comment explains nearby logic, constraints, or intent: `FALSE otherwise.`.
  **L1130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FALSE otherwise.`。
- **L1131 EN**: Continues the surrounding expression or declaration: `static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1131 CN**: 继续构造周围的表达式或声明：`static __inline__ int __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1132 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_testnzc_si128(__m128i __M, __m128i __V) {`.
  **L1132 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_testnzc_si128(__m128i __M, __m128i __V) {`。
- **L1133 EN**: Returns from the current function with `__builtin_ia32_ptestnzc128((__v2di)__M, (__v2di)__V)`.
  **L1133 CN**: 以 `__builtin_ia32_ptestnzc128((__v2di)__M, (__v2di)__V)` 从当前函数返回。
- **L1134 EN**: Closes the current lexical scope or compound statement.
  **L1134 CN**: 结束当前词法作用域或复合语句块。
- **L1135 EN**: Blank line separating nearby declarations or logic blocks.
  **L1135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1136 EN**: Comment explains nearby logic, constraints, or intent: `Tests whether the specified bits in a 128-bit integer vector are all`.
  **L1136 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Tests whether the specified bits in a 128-bit integer vector are all`。
- **L1137 EN**: Comment explains nearby logic, constraints, or intent: `ones.`.
  **L1137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ones.`。
- **L1138 EN**: Separator comment used for visual grouping.
  **L1138 CN**: 用于视觉分组的分隔注释。
- **L1139 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1140 EN**: Separator comment used for visual grouping.
  **L1140 CN**: 用于视觉分组的分隔注释。
- **L1141 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1142 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_test_all_ones(__m128i V);`.
  **L1142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_test_all_ones(__m128i V);`。
- **L1143 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1144 EN**: Separator comment used for visual grouping.
  **L1144 CN**: 用于视觉分组的分隔注释。
- **L1145 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`.
  **L1145 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`。
- **L1146 EN**: Separator comment used for visual grouping.
  **L1146 CN**: 用于视觉分组的分隔注释。
- **L1147 EN**: Comment explains nearby logic, constraints, or intent: `param V`.
  **L1147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V`。
- **L1148 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing the bits to be tested.`.
  **L1148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing the bits to be tested.`。
- **L1149 EN**: Comment explains nearby logic, constraints, or intent: `returns TRUE if the bits specified in the operand are all set to 1; FALSE`.
  **L1149 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns TRUE if the bits specified in the operand are all set to 1; FALSE`。
- **L1150 EN**: Comment explains nearby logic, constraints, or intent: `otherwise.`.
  **L1150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`otherwise.`。
- **L1151 EN**: Defines macro `_mm_test_all_ones(V)` for conditional compilation, shorthand, or API generation.
  **L1151 CN**: 定义宏 `_mm_test_all_ones(V)`，用于条件编译、简写或 API 生成。
- **L1152 EN**: Blank line separating nearby declarations or logic blocks.
  **L1152 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1153-1176

````c
/// Tests whether the specified bits in a 128-bit integer vector are
///    neither all zeros nor all ones.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_test_mix_ones_zeros(__m128i M, __m128i V);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.
///
/// \param M
///    A 128-bit integer vector containing the bits to be tested.
/// \param V
///    A 128-bit integer vector selecting which bits to test in operand \a M.
/// \returns TRUE if the specified bits are neither all zeros nor all ones;
///    FALSE otherwise.
#define _mm_test_mix_ones_zeros(M, V) _mm_testnzc_si128((M), (V))

/// Tests whether the specified bits in a 128-bit integer vector are all
///    zeros.
///
/// \headerfile <x86intrin.h>
///
````
- **L1153 EN**: Comment explains nearby logic, constraints, or intent: `Tests whether the specified bits in a 128-bit integer vector are`.
  **L1153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Tests whether the specified bits in a 128-bit integer vector are`。
- **L1154 EN**: Comment explains nearby logic, constraints, or intent: `neither all zeros nor all ones.`.
  **L1154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`neither all zeros nor all ones.`。
- **L1155 EN**: Separator comment used for visual grouping.
  **L1155 CN**: 用于视觉分组的分隔注释。
- **L1156 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1156 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1157 EN**: Separator comment used for visual grouping.
  **L1157 CN**: 用于视觉分组的分隔注释。
- **L1158 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1159 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_test_mix_ones_zeros(__m128i M, __m128i V);`.
  **L1159 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_test_mix_ones_zeros(__m128i M, __m128i V);`。
- **L1160 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1161 EN**: Separator comment used for visual grouping.
  **L1161 CN**: 用于视觉分组的分隔注释。
- **L1162 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`.
  **L1162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`。
- **L1163 EN**: Separator comment used for visual grouping.
  **L1163 CN**: 用于视觉分组的分隔注释。
- **L1164 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1164 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1165 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing the bits to be tested.`.
  **L1165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing the bits to be tested.`。
- **L1166 EN**: Comment explains nearby logic, constraints, or intent: `param V`.
  **L1166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V`。
- **L1167 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector selecting which bits to test in operand a M.`.
  **L1167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector selecting which bits to test in operand a M.`。
- **L1168 EN**: Comment explains nearby logic, constraints, or intent: `returns TRUE if the specified bits are neither all zeros nor all ones;`.
  **L1168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns TRUE if the specified bits are neither all zeros nor all ones;`。
- **L1169 EN**: Comment explains nearby logic, constraints, or intent: `FALSE otherwise.`.
  **L1169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FALSE otherwise.`。
- **L1170 EN**: Defines macro `_mm_test_mix_ones_zeros(M, V)` for conditional compilation, shorthand, or API generation.
  **L1170 CN**: 定义宏 `_mm_test_mix_ones_zeros(M, V)`，用于条件编译、简写或 API 生成。
- **L1171 EN**: Blank line separating nearby declarations or logic blocks.
  **L1171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1172 EN**: Comment explains nearby logic, constraints, or intent: `Tests whether the specified bits in a 128-bit integer vector are all`.
  **L1172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Tests whether the specified bits in a 128-bit integer vector are all`。
- **L1173 EN**: Comment explains nearby logic, constraints, or intent: `zeros.`.
  **L1173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zeros.`。
- **L1174 EN**: Separator comment used for visual grouping.
  **L1174 CN**: 用于视觉分组的分隔注释。
- **L1175 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1176 EN**: Separator comment used for visual grouping.
  **L1176 CN**: 用于视觉分组的分隔注释。

### Lines 1177-1200

````c
/// \code
/// int _mm_test_all_zeros(__m128i M, __m128i V);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.
///
/// \param M
///    A 128-bit integer vector containing the bits to be tested.
/// \param V
///    A 128-bit integer vector selecting which bits to test in operand \a M.
/// \returns TRUE if the specified bits are all zeros; FALSE otherwise.
#define _mm_test_all_zeros(M, V) _mm_testz_si128((M), (V))

/* SSE4 64-bit Packed Integer Comparisons.  */
/// Compares each of the corresponding 64-bit values of the 128-bit
///    integer vectors for equality.
///
///    Each comparison returns 0x0 for false, 0xFFFFFFFFFFFFFFFF for true.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPCMPEQQ / PCMPEQQ </c> instruction.
///
/// \param __V1
````
- **L1177 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1178 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_test_all_zeros(__m128i M, __m128i V);`.
  **L1178 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_test_all_zeros(__m128i M, __m128i V);`。
- **L1179 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1180 EN**: Separator comment used for visual grouping.
  **L1180 CN**: 用于视觉分组的分隔注释。
- **L1181 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`.
  **L1181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPTEST / PTEST </c> instruction.`。
- **L1182 EN**: Separator comment used for visual grouping.
  **L1182 CN**: 用于视觉分组的分隔注释。
- **L1183 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1184 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing the bits to be tested.`.
  **L1184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing the bits to be tested.`。
- **L1185 EN**: Comment explains nearby logic, constraints, or intent: `param V`.
  **L1185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param V`。
- **L1186 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector selecting which bits to test in operand a M.`.
  **L1186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector selecting which bits to test in operand a M.`。
- **L1187 EN**: Comment explains nearby logic, constraints, or intent: `returns TRUE if the specified bits are all zeros; FALSE otherwise.`.
  **L1187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns TRUE if the specified bits are all zeros; FALSE otherwise.`。
- **L1188 EN**: Defines macro `_mm_test_all_zeros(M, V)` for conditional compilation, shorthand, or API generation.
  **L1188 CN**: 定义宏 `_mm_test_all_zeros(M, V)`，用于条件编译、简写或 API 生成。
- **L1189 EN**: Blank line separating nearby declarations or logic blocks.
  **L1189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1190 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 64-bit Packed Integer Comparisons.`.
  **L1190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 64-bit Packed Integer Comparisons.`。
- **L1191 EN**: Comment explains nearby logic, constraints, or intent: `Compares each of the corresponding 64-bit values of the 128-bit`.
  **L1191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares each of the corresponding 64-bit values of the 128-bit`。
- **L1192 EN**: Comment explains nearby logic, constraints, or intent: `integer vectors for equality.`.
  **L1192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vectors for equality.`。
- **L1193 EN**: Separator comment used for visual grouping.
  **L1193 CN**: 用于视觉分组的分隔注释。
- **L1194 EN**: Comment explains nearby logic, constraints, or intent: `Each comparison returns 0x0 for false, 0xFFFFFFFFFFFFFFFF for true.`.
  **L1194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each comparison returns 0x0 for false, 0xFFFFFFFFFFFFFFFF for true.`。
- **L1195 EN**: Separator comment used for visual grouping.
  **L1195 CN**: 用于视觉分组的分隔注释。
- **L1196 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1197 EN**: Separator comment used for visual grouping.
  **L1197 CN**: 用于视觉分组的分隔注释。
- **L1198 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPEQQ / PCMPEQQ </c> instruction.`.
  **L1198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPEQQ / PCMPEQQ </c> instruction.`。
- **L1199 EN**: Separator comment used for visual grouping.
  **L1199 CN**: 用于视觉分组的分隔注释。
- **L1200 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L1200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。

### Lines 1201-1224

````c
///    A 128-bit integer vector.
/// \param __V2
///    A 128-bit integer vector.
/// \returns A 128-bit integer vector containing the comparison results.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cmpeq_epi64(__m128i __V1, __m128i __V2) {
  return (__m128i)((__v2di)__V1 == (__v2di)__V2);
}

/* SSE4 Packed Integer Sign-Extension.  */
/// Sign-extends each of the lower eight 8-bit integer elements of a
///    128-bit vector of [16 x i8] to 16-bit values and returns them in a
///    128-bit vector of [8 x i16]. The upper eight elements of the input vector
///    are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVSXBW / PMOVSXBW </c> instruction.
///
/// \param __V
///    A 128-bit vector of [16 x i8]. The lower eight 8-bit elements are
///    sign-extended to 16-bit values.
/// \returns A 128-bit vector of [8 x i16] containing the sign-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
````
- **L1201 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L1201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。
- **L1202 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L1202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L1203 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L1203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。
- **L1204 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit integer vector containing the comparison results.`.
  **L1204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit integer vector containing the comparison results.`。
- **L1205 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1205 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1206 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_epi64(__m128i __V1, __m128i __V2) {`.
  **L1206 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_epi64(__m128i __V1, __m128i __V2) {`。
- **L1207 EN**: Returns from the current function with `(__m128i)((__v2di)__V1 == (__v2di)__V2)`.
  **L1207 CN**: 以 `(__m128i)((__v2di)__V1 == (__v2di)__V2)` 从当前函数返回。
- **L1208 EN**: Closes the current lexical scope or compound statement.
  **L1208 CN**: 结束当前词法作用域或复合语句块。
- **L1209 EN**: Blank line separating nearby declarations or logic blocks.
  **L1209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1210 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Packed Integer Sign-Extension.`.
  **L1210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Packed Integer Sign-Extension.`。
- **L1211 EN**: Comment explains nearby logic, constraints, or intent: `Sign-extends each of the lower eight 8-bit integer elements of a`.
  **L1211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sign-extends each of the lower eight 8-bit integer elements of a`。
- **L1212 EN**: Comment explains nearby logic, constraints, or intent: `128-bit vector of [16 x i8] to 16-bit values and returns them in a`.
  **L1212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit vector of [16 x i8] to 16-bit values and returns them in a`。
- **L1213 EN**: Comment explains nearby logic, constraints, or intent: `128-bit vector of [8 x i16]. The upper eight elements of the input vector`.
  **L1213 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit vector of [8 x i16]. The upper eight elements of the input vector`。
- **L1214 EN**: Comment explains nearby logic, constraints, or intent: `are unused.`.
  **L1214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`are unused.`。
- **L1215 EN**: Separator comment used for visual grouping.
  **L1215 CN**: 用于视觉分组的分隔注释。
- **L1216 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1217 EN**: Separator comment used for visual grouping.
  **L1217 CN**: 用于视觉分组的分隔注释。
- **L1218 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVSXBW / PMOVSXBW </c> instruction.`.
  **L1218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVSXBW / PMOVSXBW </c> instruction.`。
- **L1219 EN**: Separator comment used for visual grouping.
  **L1219 CN**: 用于视觉分组的分隔注释。
- **L1220 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1221 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8]. The lower eight 8-bit elements are`.
  **L1221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8]. The lower eight 8-bit elements are`。
- **L1222 EN**: Comment explains nearby logic, constraints, or intent: `sign-extended to 16-bit values.`.
  **L1222 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sign-extended to 16-bit values.`。
- **L1223 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [8 x i16] containing the sign-extended values.`.
  **L1223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [8 x i16] containing the sign-extended values.`。
- **L1224 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1224 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。

### Lines 1225-1248

````c
_mm_cvtepi8_epi16(__m128i __V) {
  /* This function always performs a signed extension, but __v16qi is a char
     which may be signed or unsigned, so use __v16qs. */
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v16qs)__V, (__v16qs)__V, 0, 1, 2, 3, 4, 5, 6,
                              7),
      __v8hi);
}

/// Sign-extends each of the lower four 8-bit integer elements of a
///    128-bit vector of [16 x i8] to 32-bit values and returns them in a
///    128-bit vector of [4 x i32]. The upper twelve elements of the input
///    vector are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVSXBD / PMOVSXBD </c> instruction.
///
/// \param __V
///    A 128-bit vector of [16 x i8]. The lower four 8-bit elements are
///    sign-extended to 32-bit values.
/// \returns A 128-bit vector of [4 x i32] containing the sign-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepi8_epi32(__m128i __V) {
````
- **L1225 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi8_epi16(__m128i __V) {`.
  **L1225 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi8_epi16(__m128i __V) {`。
- **L1226 EN**: Comment explains nearby logic, constraints, or intent: `This function always performs a signed extension, but __v16qi is a char`.
  **L1226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This function always performs a signed extension, but __v16qi is a char`。
- **L1227 EN**: Continues the surrounding expression or declaration: `which may be signed or unsigned, so use __v16qs. */`.
  **L1227 CN**: 继续构造周围的表达式或声明：`which may be signed or unsigned, so use __v16qs. */`。
- **L1228 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1228 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_shufflevector((__v16qs)__V, (__v16qs)__V, 0, 1, 2, 3, 4, 5, 6,`.
  **L1229 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_shufflevector((__v16qs)__V, (__v16qs)__V, 0, 1, 2, 3, 4, 5, 6,`。
- **L1230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `7),`.
  **L1230 CN**: 继续一个多行参数列表、初始化器或聚合项：`7),`。
- **L1231 EN**: Adds a standalone statement or declaration: `__v8hi);`.
  **L1231 CN**: 添加一条独立语句或声明：`__v8hi);`。
- **L1232 EN**: Closes the current lexical scope or compound statement.
  **L1232 CN**: 结束当前词法作用域或复合语句块。
- **L1233 EN**: Blank line separating nearby declarations or logic blocks.
  **L1233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1234 EN**: Comment explains nearby logic, constraints, or intent: `Sign-extends each of the lower four 8-bit integer elements of a`.
  **L1234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sign-extends each of the lower four 8-bit integer elements of a`。
- **L1235 EN**: Comment explains nearby logic, constraints, or intent: `128-bit vector of [16 x i8] to 32-bit values and returns them in a`.
  **L1235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit vector of [16 x i8] to 32-bit values and returns them in a`。
- **L1236 EN**: Comment explains nearby logic, constraints, or intent: `128-bit vector of [4 x i32]. The upper twelve elements of the input`.
  **L1236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit vector of [4 x i32]. The upper twelve elements of the input`。
- **L1237 EN**: Comment explains nearby logic, constraints, or intent: `vector are unused.`.
  **L1237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector are unused.`。
- **L1238 EN**: Separator comment used for visual grouping.
  **L1238 CN**: 用于视觉分组的分隔注释。
- **L1239 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1240 EN**: Separator comment used for visual grouping.
  **L1240 CN**: 用于视觉分组的分隔注释。
- **L1241 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVSXBD / PMOVSXBD </c> instruction.`.
  **L1241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVSXBD / PMOVSXBD </c> instruction.`。
- **L1242 EN**: Separator comment used for visual grouping.
  **L1242 CN**: 用于视觉分组的分隔注释。
- **L1243 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1244 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8]. The lower four 8-bit elements are`.
  **L1244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8]. The lower four 8-bit elements are`。
- **L1245 EN**: Comment explains nearby logic, constraints, or intent: `sign-extended to 32-bit values.`.
  **L1245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sign-extended to 32-bit values.`。
- **L1246 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x i32] containing the sign-extended values.`.
  **L1246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x i32] containing the sign-extended values.`。
- **L1247 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1247 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1248 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi8_epi32(__m128i __V) {`.
  **L1248 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi8_epi32(__m128i __V) {`。

### Lines 1249-1272

````c
  /* This function always performs a signed extension, but __v16qi is a char
     which may be signed or unsigned, so use __v16qs. */
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v16qs)__V, (__v16qs)__V, 0, 1, 2, 3), __v4si);
}

/// Sign-extends each of the lower two 8-bit integer elements of a
///    128-bit integer vector of [16 x i8] to 64-bit values and returns them in
///    a 128-bit vector of [2 x i64]. The upper fourteen elements of the input
///    vector are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVSXBQ / PMOVSXBQ </c> instruction.
///
/// \param __V
///    A 128-bit vector of [16 x i8]. The lower two 8-bit elements are
///    sign-extended to 64-bit values.
/// \returns A 128-bit vector of [2 x i64] containing the sign-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepi8_epi64(__m128i __V) {
  /* This function always performs a signed extension, but __v16qi is a char
     which may be signed or unsigned, so use __v16qs. */
  return (__m128i) __builtin_convertvector(
````
- **L1249 EN**: Comment explains nearby logic, constraints, or intent: `This function always performs a signed extension, but __v16qi is a char`.
  **L1249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This function always performs a signed extension, but __v16qi is a char`。
- **L1250 EN**: Continues the surrounding expression or declaration: `which may be signed or unsigned, so use __v16qs. */`.
  **L1250 CN**: 继续构造周围的表达式或声明：`which may be signed or unsigned, so use __v16qs. */`。
- **L1251 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1251 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1252 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1252 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1253 EN**: Closes the current lexical scope or compound statement.
  **L1253 CN**: 结束当前词法作用域或复合语句块。
- **L1254 EN**: Blank line separating nearby declarations or logic blocks.
  **L1254 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1255 EN**: Comment explains nearby logic, constraints, or intent: `Sign-extends each of the lower two 8-bit integer elements of a`.
  **L1255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sign-extends each of the lower two 8-bit integer elements of a`。
- **L1256 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector of [16 x i8] to 64-bit values and returns them in`.
  **L1256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector of [16 x i8] to 64-bit values and returns them in`。
- **L1257 EN**: Comment explains nearby logic, constraints, or intent: `a 128-bit vector of [2 x i64]. The upper fourteen elements of the input`.
  **L1257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a 128-bit vector of [2 x i64]. The upper fourteen elements of the input`。
- **L1258 EN**: Comment explains nearby logic, constraints, or intent: `vector are unused.`.
  **L1258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector are unused.`。
- **L1259 EN**: Separator comment used for visual grouping.
  **L1259 CN**: 用于视觉分组的分隔注释。
- **L1260 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1261 EN**: Separator comment used for visual grouping.
  **L1261 CN**: 用于视觉分组的分隔注释。
- **L1262 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVSXBQ / PMOVSXBQ </c> instruction.`.
  **L1262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVSXBQ / PMOVSXBQ </c> instruction.`。
- **L1263 EN**: Separator comment used for visual grouping.
  **L1263 CN**: 用于视觉分组的分隔注释。
- **L1264 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1265 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8]. The lower two 8-bit elements are`.
  **L1265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8]. The lower two 8-bit elements are`。
- **L1266 EN**: Comment explains nearby logic, constraints, or intent: `sign-extended to 64-bit values.`.
  **L1266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sign-extended to 64-bit values.`。
- **L1267 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x i64] containing the sign-extended values.`.
  **L1267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x i64] containing the sign-extended values.`。
- **L1268 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1268 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1269 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi8_epi64(__m128i __V) {`.
  **L1269 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi8_epi64(__m128i __V) {`。
- **L1270 EN**: Comment explains nearby logic, constraints, or intent: `This function always performs a signed extension, but __v16qi is a char`.
  **L1270 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This function always performs a signed extension, but __v16qi is a char`。
- **L1271 EN**: Continues the surrounding expression or declaration: `which may be signed or unsigned, so use __v16qs. */`.
  **L1271 CN**: 继续构造周围的表达式或声明：`which may be signed or unsigned, so use __v16qs. */`。
- **L1272 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1272 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。

### Lines 1273-1296

````c
      __builtin_shufflevector((__v16qs)__V, (__v16qs)__V, 0, 1), __v2di);
}

/// Sign-extends each of the lower four 16-bit integer elements of a
///    128-bit integer vector of [8 x i16] to 32-bit values and returns them in
///    a 128-bit vector of [4 x i32]. The upper four elements of the input
///    vector are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVSXWD / PMOVSXWD </c> instruction.
///
/// \param __V
///    A 128-bit vector of [8 x i16]. The lower four 16-bit elements are
///    sign-extended to 32-bit values.
/// \returns A 128-bit vector of [4 x i32] containing the sign-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepi16_epi32(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v8hi)__V, (__v8hi)__V, 0, 1, 2, 3), __v4si);
}

/// Sign-extends each of the lower two 16-bit integer elements of a
///    128-bit integer vector of [8 x i16] to 64-bit values and returns them in
````
- **L1273 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1273 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1274 EN**: Closes the current lexical scope or compound statement.
  **L1274 CN**: 结束当前词法作用域或复合语句块。
- **L1275 EN**: Blank line separating nearby declarations or logic blocks.
  **L1275 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1276 EN**: Comment explains nearby logic, constraints, or intent: `Sign-extends each of the lower four 16-bit integer elements of a`.
  **L1276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sign-extends each of the lower four 16-bit integer elements of a`。
- **L1277 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector of [8 x i16] to 32-bit values and returns them in`.
  **L1277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector of [8 x i16] to 32-bit values and returns them in`。
- **L1278 EN**: Comment explains nearby logic, constraints, or intent: `a 128-bit vector of [4 x i32]. The upper four elements of the input`.
  **L1278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a 128-bit vector of [4 x i32]. The upper four elements of the input`。
- **L1279 EN**: Comment explains nearby logic, constraints, or intent: `vector are unused.`.
  **L1279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector are unused.`。
- **L1280 EN**: Separator comment used for visual grouping.
  **L1280 CN**: 用于视觉分组的分隔注释。
- **L1281 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1281 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1282 EN**: Separator comment used for visual grouping.
  **L1282 CN**: 用于视觉分组的分隔注释。
- **L1283 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVSXWD / PMOVSXWD </c> instruction.`.
  **L1283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVSXWD / PMOVSXWD </c> instruction.`。
- **L1284 EN**: Separator comment used for visual grouping.
  **L1284 CN**: 用于视觉分组的分隔注释。
- **L1285 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1286 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x i16]. The lower four 16-bit elements are`.
  **L1286 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x i16]. The lower four 16-bit elements are`。
- **L1287 EN**: Comment explains nearby logic, constraints, or intent: `sign-extended to 32-bit values.`.
  **L1287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sign-extended to 32-bit values.`。
- **L1288 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x i32] containing the sign-extended values.`.
  **L1288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x i32] containing the sign-extended values.`。
- **L1289 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1289 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1290 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi16_epi32(__m128i __V) {`.
  **L1290 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi16_epi32(__m128i __V) {`。
- **L1291 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1291 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1292 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1292 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1293 EN**: Closes the current lexical scope or compound statement.
  **L1293 CN**: 结束当前词法作用域或复合语句块。
- **L1294 EN**: Blank line separating nearby declarations or logic blocks.
  **L1294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1295 EN**: Comment explains nearby logic, constraints, or intent: `Sign-extends each of the lower two 16-bit integer elements of a`.
  **L1295 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sign-extends each of the lower two 16-bit integer elements of a`。
- **L1296 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector of [8 x i16] to 64-bit values and returns them in`.
  **L1296 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector of [8 x i16] to 64-bit values and returns them in`。

### Lines 1297-1320

````c
///    a 128-bit vector of [2 x i64]. The upper six elements of the input
///    vector are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVSXWQ / PMOVSXWQ </c> instruction.
///
/// \param __V
///    A 128-bit vector of [8 x i16]. The lower two 16-bit elements are
///     sign-extended to 64-bit values.
/// \returns A 128-bit vector of [2 x i64] containing the sign-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepi16_epi64(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v8hi)__V, (__v8hi)__V, 0, 1), __v2di);
}

/// Sign-extends each of the lower two 32-bit integer elements of a
///    128-bit integer vector of [4 x i32] to 64-bit values and returns them in
///    a 128-bit vector of [2 x i64]. The upper two elements of the input vector
///    are unused.
///
/// \headerfile <x86intrin.h>
///
````
- **L1297 EN**: Comment explains nearby logic, constraints, or intent: `a 128-bit vector of [2 x i64]. The upper six elements of the input`.
  **L1297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a 128-bit vector of [2 x i64]. The upper six elements of the input`。
- **L1298 EN**: Comment explains nearby logic, constraints, or intent: `vector are unused.`.
  **L1298 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector are unused.`。
- **L1299 EN**: Separator comment used for visual grouping.
  **L1299 CN**: 用于视觉分组的分隔注释。
- **L1300 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1300 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1301 EN**: Separator comment used for visual grouping.
  **L1301 CN**: 用于视觉分组的分隔注释。
- **L1302 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVSXWQ / PMOVSXWQ </c> instruction.`.
  **L1302 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVSXWQ / PMOVSXWQ </c> instruction.`。
- **L1303 EN**: Separator comment used for visual grouping.
  **L1303 CN**: 用于视觉分组的分隔注释。
- **L1304 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1304 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1305 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x i16]. The lower two 16-bit elements are`.
  **L1305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x i16]. The lower two 16-bit elements are`。
- **L1306 EN**: Comment explains nearby logic, constraints, or intent: `sign-extended to 64-bit values.`.
  **L1306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sign-extended to 64-bit values.`。
- **L1307 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x i64] containing the sign-extended values.`.
  **L1307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x i64] containing the sign-extended values.`。
- **L1308 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1308 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1309 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi16_epi64(__m128i __V) {`.
  **L1309 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi16_epi64(__m128i __V) {`。
- **L1310 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1310 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1311 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1311 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1312 EN**: Closes the current lexical scope or compound statement.
  **L1312 CN**: 结束当前词法作用域或复合语句块。
- **L1313 EN**: Blank line separating nearby declarations or logic blocks.
  **L1313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1314 EN**: Comment explains nearby logic, constraints, or intent: `Sign-extends each of the lower two 32-bit integer elements of a`.
  **L1314 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sign-extends each of the lower two 32-bit integer elements of a`。
- **L1315 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector of [4 x i32] to 64-bit values and returns them in`.
  **L1315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector of [4 x i32] to 64-bit values and returns them in`。
- **L1316 EN**: Comment explains nearby logic, constraints, or intent: `a 128-bit vector of [2 x i64]. The upper two elements of the input vector`.
  **L1316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a 128-bit vector of [2 x i64]. The upper two elements of the input vector`。
- **L1317 EN**: Comment explains nearby logic, constraints, or intent: `are unused.`.
  **L1317 CN**: 注释解释附近代码的逻辑、约束或设计意图：`are unused.`。
- **L1318 EN**: Separator comment used for visual grouping.
  **L1318 CN**: 用于视觉分组的分隔注释。
- **L1319 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1319 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1320 EN**: Separator comment used for visual grouping.
  **L1320 CN**: 用于视觉分组的分隔注释。

### Lines 1321-1344

````c
/// This intrinsic corresponds to the <c> VPMOVSXDQ / PMOVSXDQ </c> instruction.
///
/// \param __V
///    A 128-bit vector of [4 x i32]. The lower two 32-bit elements are
///    sign-extended to 64-bit values.
/// \returns A 128-bit vector of [2 x i64] containing the sign-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepi32_epi64(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v4si)__V, (__v4si)__V, 0, 1), __v2di);
}

/* SSE4 Packed Integer Zero-Extension.  */
/// Zero-extends each of the lower eight 8-bit integer elements of a
///    128-bit vector of [16 x i8] to 16-bit values and returns them in a
///    128-bit vector of [8 x i16]. The upper eight elements of the input vector
///    are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVZXBW / PMOVZXBW </c> instruction.
///
/// \param __V
///    A 128-bit vector of [16 x i8]. The lower eight 8-bit elements are
````
- **L1321 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVSXDQ / PMOVSXDQ </c> instruction.`.
  **L1321 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVSXDQ / PMOVSXDQ </c> instruction.`。
- **L1322 EN**: Separator comment used for visual grouping.
  **L1322 CN**: 用于视觉分组的分隔注释。
- **L1323 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1323 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1324 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32]. The lower two 32-bit elements are`.
  **L1324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32]. The lower two 32-bit elements are`。
- **L1325 EN**: Comment explains nearby logic, constraints, or intent: `sign-extended to 64-bit values.`.
  **L1325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sign-extended to 64-bit values.`。
- **L1326 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x i64] containing the sign-extended values.`.
  **L1326 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x i64] containing the sign-extended values.`。
- **L1327 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1327 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1328 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi32_epi64(__m128i __V) {`.
  **L1328 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi32_epi64(__m128i __V) {`。
- **L1329 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1329 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1330 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1330 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1331 EN**: Closes the current lexical scope or compound statement.
  **L1331 CN**: 结束当前词法作用域或复合语句块。
- **L1332 EN**: Blank line separating nearby declarations or logic blocks.
  **L1332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1333 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Packed Integer Zero-Extension.`.
  **L1333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Packed Integer Zero-Extension.`。
- **L1334 EN**: Comment explains nearby logic, constraints, or intent: `Zero-extends each of the lower eight 8-bit integer elements of a`.
  **L1334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zero-extends each of the lower eight 8-bit integer elements of a`。
- **L1335 EN**: Comment explains nearby logic, constraints, or intent: `128-bit vector of [16 x i8] to 16-bit values and returns them in a`.
  **L1335 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit vector of [16 x i8] to 16-bit values and returns them in a`。
- **L1336 EN**: Comment explains nearby logic, constraints, or intent: `128-bit vector of [8 x i16]. The upper eight elements of the input vector`.
  **L1336 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit vector of [8 x i16]. The upper eight elements of the input vector`。
- **L1337 EN**: Comment explains nearby logic, constraints, or intent: `are unused.`.
  **L1337 CN**: 注释解释附近代码的逻辑、约束或设计意图：`are unused.`。
- **L1338 EN**: Separator comment used for visual grouping.
  **L1338 CN**: 用于视觉分组的分隔注释。
- **L1339 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1339 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1340 EN**: Separator comment used for visual grouping.
  **L1340 CN**: 用于视觉分组的分隔注释。
- **L1341 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVZXBW / PMOVZXBW </c> instruction.`.
  **L1341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVZXBW / PMOVZXBW </c> instruction.`。
- **L1342 EN**: Separator comment used for visual grouping.
  **L1342 CN**: 用于视觉分组的分隔注释。
- **L1343 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1344 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8]. The lower eight 8-bit elements are`.
  **L1344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8]. The lower eight 8-bit elements are`。

### Lines 1345-1368

````c
///    zero-extended to 16-bit values.
/// \returns A 128-bit vector of [8 x i16] containing the zero-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepu8_epi16(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v16qu)__V, (__v16qu)__V, 0, 1, 2, 3, 4, 5, 6,
                              7),
      __v8hi);
}

/// Zero-extends each of the lower four 8-bit integer elements of a
///    128-bit vector of [16 x i8] to 32-bit values and returns them in a
///    128-bit vector of [4 x i32]. The upper twelve elements of the input
///    vector are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVZXBD / PMOVZXBD </c> instruction.
///
/// \param __V
///    A 128-bit vector of [16 x i8]. The lower four 8-bit elements are
///    zero-extended to 32-bit values.
/// \returns A 128-bit vector of [4 x i32] containing the zero-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
````
- **L1345 EN**: Comment explains nearby logic, constraints, or intent: `zero-extended to 16-bit values.`.
  **L1345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero-extended to 16-bit values.`。
- **L1346 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [8 x i16] containing the zero-extended values.`.
  **L1346 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [8 x i16] containing the zero-extended values.`。
- **L1347 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1347 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1348 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu8_epi16(__m128i __V) {`.
  **L1348 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu8_epi16(__m128i __V) {`。
- **L1349 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1349 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_shufflevector((__v16qu)__V, (__v16qu)__V, 0, 1, 2, 3, 4, 5, 6,`.
  **L1350 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_shufflevector((__v16qu)__V, (__v16qu)__V, 0, 1, 2, 3, 4, 5, 6,`。
- **L1351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `7),`.
  **L1351 CN**: 继续一个多行参数列表、初始化器或聚合项：`7),`。
- **L1352 EN**: Adds a standalone statement or declaration: `__v8hi);`.
  **L1352 CN**: 添加一条独立语句或声明：`__v8hi);`。
- **L1353 EN**: Closes the current lexical scope or compound statement.
  **L1353 CN**: 结束当前词法作用域或复合语句块。
- **L1354 EN**: Blank line separating nearby declarations or logic blocks.
  **L1354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1355 EN**: Comment explains nearby logic, constraints, or intent: `Zero-extends each of the lower four 8-bit integer elements of a`.
  **L1355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zero-extends each of the lower four 8-bit integer elements of a`。
- **L1356 EN**: Comment explains nearby logic, constraints, or intent: `128-bit vector of [16 x i8] to 32-bit values and returns them in a`.
  **L1356 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit vector of [16 x i8] to 32-bit values and returns them in a`。
- **L1357 EN**: Comment explains nearby logic, constraints, or intent: `128-bit vector of [4 x i32]. The upper twelve elements of the input`.
  **L1357 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit vector of [4 x i32]. The upper twelve elements of the input`。
- **L1358 EN**: Comment explains nearby logic, constraints, or intent: `vector are unused.`.
  **L1358 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector are unused.`。
- **L1359 EN**: Separator comment used for visual grouping.
  **L1359 CN**: 用于视觉分组的分隔注释。
- **L1360 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1360 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1361 EN**: Separator comment used for visual grouping.
  **L1361 CN**: 用于视觉分组的分隔注释。
- **L1362 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVZXBD / PMOVZXBD </c> instruction.`.
  **L1362 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVZXBD / PMOVZXBD </c> instruction.`。
- **L1363 EN**: Separator comment used for visual grouping.
  **L1363 CN**: 用于视觉分组的分隔注释。
- **L1364 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1364 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1365 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8]. The lower four 8-bit elements are`.
  **L1365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8]. The lower four 8-bit elements are`。
- **L1366 EN**: Comment explains nearby logic, constraints, or intent: `zero-extended to 32-bit values.`.
  **L1366 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero-extended to 32-bit values.`。
- **L1367 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x i32] containing the zero-extended values.`.
  **L1367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x i32] containing the zero-extended values.`。
- **L1368 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1368 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。

### Lines 1369-1392

````c
_mm_cvtepu8_epi32(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v16qu)__V, (__v16qu)__V, 0, 1, 2, 3), __v4si);
}

/// Zero-extends each of the lower two 8-bit integer elements of a
///    128-bit integer vector of [16 x i8] to 64-bit values and returns them in
///    a 128-bit vector of [2 x i64]. The upper fourteen elements of the input
///    vector are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVZXBQ / PMOVZXBQ </c> instruction.
///
/// \param __V
///    A 128-bit vector of [16 x i8]. The lower two 8-bit elements are
///    zero-extended to 64-bit values.
/// \returns A 128-bit vector of [2 x i64] containing the zero-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepu8_epi64(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v16qu)__V, (__v16qu)__V, 0, 1), __v2di);
}

````
- **L1369 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu8_epi32(__m128i __V) {`.
  **L1369 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu8_epi32(__m128i __V) {`。
- **L1370 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1370 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1371 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1371 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1372 EN**: Closes the current lexical scope or compound statement.
  **L1372 CN**: 结束当前词法作用域或复合语句块。
- **L1373 EN**: Blank line separating nearby declarations or logic blocks.
  **L1373 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1374 EN**: Comment explains nearby logic, constraints, or intent: `Zero-extends each of the lower two 8-bit integer elements of a`.
  **L1374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zero-extends each of the lower two 8-bit integer elements of a`。
- **L1375 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector of [16 x i8] to 64-bit values and returns them in`.
  **L1375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector of [16 x i8] to 64-bit values and returns them in`。
- **L1376 EN**: Comment explains nearby logic, constraints, or intent: `a 128-bit vector of [2 x i64]. The upper fourteen elements of the input`.
  **L1376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a 128-bit vector of [2 x i64]. The upper fourteen elements of the input`。
- **L1377 EN**: Comment explains nearby logic, constraints, or intent: `vector are unused.`.
  **L1377 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector are unused.`。
- **L1378 EN**: Separator comment used for visual grouping.
  **L1378 CN**: 用于视觉分组的分隔注释。
- **L1379 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1379 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1380 EN**: Separator comment used for visual grouping.
  **L1380 CN**: 用于视觉分组的分隔注释。
- **L1381 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVZXBQ / PMOVZXBQ </c> instruction.`.
  **L1381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVZXBQ / PMOVZXBQ </c> instruction.`。
- **L1382 EN**: Separator comment used for visual grouping.
  **L1382 CN**: 用于视觉分组的分隔注释。
- **L1383 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1383 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1384 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8]. The lower two 8-bit elements are`.
  **L1384 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8]. The lower two 8-bit elements are`。
- **L1385 EN**: Comment explains nearby logic, constraints, or intent: `zero-extended to 64-bit values.`.
  **L1385 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero-extended to 64-bit values.`。
- **L1386 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x i64] containing the zero-extended values.`.
  **L1386 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x i64] containing the zero-extended values.`。
- **L1387 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1387 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1388 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu8_epi64(__m128i __V) {`.
  **L1388 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu8_epi64(__m128i __V) {`。
- **L1389 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1389 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1390 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1390 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1391 EN**: Closes the current lexical scope or compound statement.
  **L1391 CN**: 结束当前词法作用域或复合语句块。
- **L1392 EN**: Blank line separating nearby declarations or logic blocks.
  **L1392 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1393-1416

````c
/// Zero-extends each of the lower four 16-bit integer elements of a
///    128-bit integer vector of [8 x i16] to 32-bit values and returns them in
///    a 128-bit vector of [4 x i32]. The upper four elements of the input
///    vector are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVZXWD / PMOVZXWD </c> instruction.
///
/// \param __V
///    A 128-bit vector of [8 x i16]. The lower four 16-bit elements are
///    zero-extended to 32-bit values.
/// \returns A 128-bit vector of [4 x i32] containing the zero-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepu16_epi32(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v8hu)__V, (__v8hu)__V, 0, 1, 2, 3), __v4si);
}

/// Zero-extends each of the lower two 16-bit integer elements of a
///    128-bit integer vector of [8 x i16] to 64-bit values and returns them in
///    a 128-bit vector of [2 x i64]. The upper six elements of the input vector
///    are unused.
///
````
- **L1393 EN**: Comment explains nearby logic, constraints, or intent: `Zero-extends each of the lower four 16-bit integer elements of a`.
  **L1393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zero-extends each of the lower four 16-bit integer elements of a`。
- **L1394 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector of [8 x i16] to 32-bit values and returns them in`.
  **L1394 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector of [8 x i16] to 32-bit values and returns them in`。
- **L1395 EN**: Comment explains nearby logic, constraints, or intent: `a 128-bit vector of [4 x i32]. The upper four elements of the input`.
  **L1395 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a 128-bit vector of [4 x i32]. The upper four elements of the input`。
- **L1396 EN**: Comment explains nearby logic, constraints, or intent: `vector are unused.`.
  **L1396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector are unused.`。
- **L1397 EN**: Separator comment used for visual grouping.
  **L1397 CN**: 用于视觉分组的分隔注释。
- **L1398 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1399 EN**: Separator comment used for visual grouping.
  **L1399 CN**: 用于视觉分组的分隔注释。
- **L1400 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVZXWD / PMOVZXWD </c> instruction.`.
  **L1400 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVZXWD / PMOVZXWD </c> instruction.`。
- **L1401 EN**: Separator comment used for visual grouping.
  **L1401 CN**: 用于视觉分组的分隔注释。
- **L1402 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1402 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1403 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x i16]. The lower four 16-bit elements are`.
  **L1403 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x i16]. The lower four 16-bit elements are`。
- **L1404 EN**: Comment explains nearby logic, constraints, or intent: `zero-extended to 32-bit values.`.
  **L1404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero-extended to 32-bit values.`。
- **L1405 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x i32] containing the zero-extended values.`.
  **L1405 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x i32] containing the zero-extended values.`。
- **L1406 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1406 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1407 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu16_epi32(__m128i __V) {`.
  **L1407 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu16_epi32(__m128i __V) {`。
- **L1408 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1408 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1409 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1409 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1410 EN**: Closes the current lexical scope or compound statement.
  **L1410 CN**: 结束当前词法作用域或复合语句块。
- **L1411 EN**: Blank line separating nearby declarations or logic blocks.
  **L1411 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1412 EN**: Comment explains nearby logic, constraints, or intent: `Zero-extends each of the lower two 16-bit integer elements of a`.
  **L1412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zero-extends each of the lower two 16-bit integer elements of a`。
- **L1413 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector of [8 x i16] to 64-bit values and returns them in`.
  **L1413 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector of [8 x i16] to 64-bit values and returns them in`。
- **L1414 EN**: Comment explains nearby logic, constraints, or intent: `a 128-bit vector of [2 x i64]. The upper six elements of the input vector`.
  **L1414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a 128-bit vector of [2 x i64]. The upper six elements of the input vector`。
- **L1415 EN**: Comment explains nearby logic, constraints, or intent: `are unused.`.
  **L1415 CN**: 注释解释附近代码的逻辑、约束或设计意图：`are unused.`。
- **L1416 EN**: Separator comment used for visual grouping.
  **L1416 CN**: 用于视觉分组的分隔注释。

### Lines 1417-1440

````c
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVZXWQ / PMOVZXWQ </c> instruction.
///
/// \param __V
///    A 128-bit vector of [8 x i16]. The lower two 16-bit elements are
///    zero-extended to 64-bit values.
/// \returns A 128-bit vector of [2 x i64] containing the zero-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepu16_epi64(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v8hu)__V, (__v8hu)__V, 0, 1), __v2di);
}

/// Zero-extends each of the lower two 32-bit integer elements of a
///    128-bit integer vector of [4 x i32] to 64-bit values and returns them in
///    a 128-bit vector of [2 x i64]. The upper two elements of the input vector
///    are unused.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPMOVZXDQ / PMOVZXDQ </c> instruction.
///
/// \param __V
````
- **L1417 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1418 EN**: Separator comment used for visual grouping.
  **L1418 CN**: 用于视觉分组的分隔注释。
- **L1419 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVZXWQ / PMOVZXWQ </c> instruction.`.
  **L1419 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVZXWQ / PMOVZXWQ </c> instruction.`。
- **L1420 EN**: Separator comment used for visual grouping.
  **L1420 CN**: 用于视觉分组的分隔注释。
- **L1421 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1421 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1422 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x i16]. The lower two 16-bit elements are`.
  **L1422 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x i16]. The lower two 16-bit elements are`。
- **L1423 EN**: Comment explains nearby logic, constraints, or intent: `zero-extended to 64-bit values.`.
  **L1423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero-extended to 64-bit values.`。
- **L1424 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x i64] containing the zero-extended values.`.
  **L1424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x i64] containing the zero-extended values.`。
- **L1425 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1425 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1426 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu16_epi64(__m128i __V) {`.
  **L1426 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu16_epi64(__m128i __V) {`。
- **L1427 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1427 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1428 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1428 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1429 EN**: Closes the current lexical scope or compound statement.
  **L1429 CN**: 结束当前词法作用域或复合语句块。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1431 EN**: Comment explains nearby logic, constraints, or intent: `Zero-extends each of the lower two 32-bit integer elements of a`.
  **L1431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zero-extends each of the lower two 32-bit integer elements of a`。
- **L1432 EN**: Comment explains nearby logic, constraints, or intent: `128-bit integer vector of [4 x i32] to 64-bit values and returns them in`.
  **L1432 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit integer vector of [4 x i32] to 64-bit values and returns them in`。
- **L1433 EN**: Comment explains nearby logic, constraints, or intent: `a 128-bit vector of [2 x i64]. The upper two elements of the input vector`.
  **L1433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a 128-bit vector of [2 x i64]. The upper two elements of the input vector`。
- **L1434 EN**: Comment explains nearby logic, constraints, or intent: `are unused.`.
  **L1434 CN**: 注释解释附近代码的逻辑、约束或设计意图：`are unused.`。
- **L1435 EN**: Separator comment used for visual grouping.
  **L1435 CN**: 用于视觉分组的分隔注释。
- **L1436 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1437 EN**: Separator comment used for visual grouping.
  **L1437 CN**: 用于视觉分组的分隔注释。
- **L1438 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPMOVZXDQ / PMOVZXDQ </c> instruction.`.
  **L1438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPMOVZXDQ / PMOVZXDQ </c> instruction.`。
- **L1439 EN**: Separator comment used for visual grouping.
  **L1439 CN**: 用于视觉分组的分隔注释。
- **L1440 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。

### Lines 1441-1464

````c
///    A 128-bit vector of [4 x i32]. The lower two 32-bit elements are
///    zero-extended to 64-bit values.
/// \returns A 128-bit vector of [2 x i64] containing the zero-extended values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cvtepu32_epi64(__m128i __V) {
  return (__m128i) __builtin_convertvector(
      __builtin_shufflevector((__v4su)__V, (__v4su)__V, 0, 1), __v2di);
}

/* SSE4 Pack with Unsigned Saturation.  */
/// Converts, with saturation, 32-bit signed integers from both 128-bit integer
///    vector operands into 16-bit unsigned integers, and returns the packed
///    result.
///
///    Values greater than 0xFFFF are saturated to 0xFFFF. Values less than
///    0x0000 are saturated to 0x0000.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPACKUSDW / PACKUSDW </c> instruction.
///
/// \param __V1
///    A 128-bit vector of [4 x i32]. The converted [4 x i16] values are
///    written to the lower 64 bits of the result.
````
- **L1441 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32]. The lower two 32-bit elements are`.
  **L1441 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32]. The lower two 32-bit elements are`。
- **L1442 EN**: Comment explains nearby logic, constraints, or intent: `zero-extended to 64-bit values.`.
  **L1442 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero-extended to 64-bit values.`。
- **L1443 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x i64] containing the zero-extended values.`.
  **L1443 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x i64] containing the zero-extended values.`。
- **L1444 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1444 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1445 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu32_epi64(__m128i __V) {`.
  **L1445 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu32_epi64(__m128i __V) {`。
- **L1446 EN**: Returns from the current function with `(__m128i) __builtin_convertvector(`.
  **L1446 CN**: 以 `(__m128i) __builtin_convertvector(` 从当前函数返回。
- **L1447 EN**: Executes a call or declaration centered on `__builtin_shufflevector`.
  **L1447 CN**: 执行以 `__builtin_shufflevector` 为核心的调用或声明。
- **L1448 EN**: Closes the current lexical scope or compound statement.
  **L1448 CN**: 结束当前词法作用域或复合语句块。
- **L1449 EN**: Blank line separating nearby declarations or logic blocks.
  **L1449 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1450 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Pack with Unsigned Saturation.`.
  **L1450 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Pack with Unsigned Saturation.`。
- **L1451 EN**: Comment explains nearby logic, constraints, or intent: `Converts, with saturation, 32-bit signed integers from both 128-bit integer`.
  **L1451 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converts, with saturation, 32-bit signed integers from both 128-bit integer`。
- **L1452 EN**: Comment explains nearby logic, constraints, or intent: `vector operands into 16-bit unsigned integers, and returns the packed`.
  **L1452 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector operands into 16-bit unsigned integers, and returns the packed`。
- **L1453 EN**: Comment explains nearby logic, constraints, or intent: `result.`.
  **L1453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result.`。
- **L1454 EN**: Separator comment used for visual grouping.
  **L1454 CN**: 用于视觉分组的分隔注释。
- **L1455 EN**: Comment explains nearby logic, constraints, or intent: `Values greater than 0xFFFF are saturated to 0xFFFF. Values less than`.
  **L1455 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Values greater than 0xFFFF are saturated to 0xFFFF. Values less than`。
- **L1456 EN**: Comment explains nearby logic, constraints, or intent: `0x0000 are saturated to 0x0000.`.
  **L1456 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0x0000 are saturated to 0x0000.`。
- **L1457 EN**: Separator comment used for visual grouping.
  **L1457 CN**: 用于视觉分组的分隔注释。
- **L1458 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1458 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1459 EN**: Separator comment used for visual grouping.
  **L1459 CN**: 用于视觉分组的分隔注释。
- **L1460 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPACKUSDW / PACKUSDW </c> instruction.`.
  **L1460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPACKUSDW / PACKUSDW </c> instruction.`。
- **L1461 EN**: Separator comment used for visual grouping.
  **L1461 CN**: 用于视觉分组的分隔注释。
- **L1462 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L1462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L1463 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32]. The converted [4 x i16] values are`.
  **L1463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32]. The converted [4 x i16] values are`。
- **L1464 EN**: Comment explains nearby logic, constraints, or intent: `written to the lower 64 bits of the result.`.
  **L1464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`written to the lower 64 bits of the result.`。

### Lines 1465-1488

````c
/// \param __V2
///    A 128-bit vector of [4 x i32]. The converted [4 x i16] values are
///    written to the higher 64 bits of the result.
/// \returns A 128-bit vector of [8 x i16] containing the converted values.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_packus_epi32(__m128i __V1, __m128i __V2) {
  return (__m128i)__builtin_ia32_packusdw128((__v4si)__V1, (__v4si)__V2);
}

/* SSE4 Multiple Packed Sums of Absolute Difference.  */
/// Subtracts 8-bit unsigned integer values and computes the absolute
///    values of the differences to the corresponding bits in the destination.
///    Then sums of the absolute differences are returned according to the bit
///    fields in the immediate operand.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128i _mm_mpsadbw_epu8(__m128i X, __m128i Y, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VMPSADBW / MPSADBW </c> instruction.
///
/// \param X
````
- **L1465 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L1465 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L1466 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x i32]. The converted [4 x i16] values are`.
  **L1466 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x i32]. The converted [4 x i16] values are`。
- **L1467 EN**: Comment explains nearby logic, constraints, or intent: `written to the higher 64 bits of the result.`.
  **L1467 CN**: 注释解释附近代码的逻辑、约束或设计意图：`written to the higher 64 bits of the result.`。
- **L1468 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [8 x i16] containing the converted values.`.
  **L1468 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [8 x i16] containing the converted values.`。
- **L1469 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1469 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1470 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_packus_epi32(__m128i __V1, __m128i __V2) {`.
  **L1470 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_packus_epi32(__m128i __V1, __m128i __V2) {`。
- **L1471 EN**: Returns from the current function with `(__m128i)__builtin_ia32_packusdw128((__v4si)__V1, (__v4si)__V2)`.
  **L1471 CN**: 以 `(__m128i)__builtin_ia32_packusdw128((__v4si)__V1, (__v4si)__V2)` 从当前函数返回。
- **L1472 EN**: Closes the current lexical scope or compound statement.
  **L1472 CN**: 结束当前词法作用域或复合语句块。
- **L1473 EN**: Blank line separating nearby declarations or logic blocks.
  **L1473 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1474 EN**: Comment explains nearby logic, constraints, or intent: `SSE4 Multiple Packed Sums of Absolute Difference.`.
  **L1474 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4 Multiple Packed Sums of Absolute Difference.`。
- **L1475 EN**: Comment explains nearby logic, constraints, or intent: `Subtracts 8-bit unsigned integer values and computes the absolute`.
  **L1475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtracts 8-bit unsigned integer values and computes the absolute`。
- **L1476 EN**: Comment explains nearby logic, constraints, or intent: `values of the differences to the corresponding bits in the destination.`.
  **L1476 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values of the differences to the corresponding bits in the destination.`。
- **L1477 EN**: Comment explains nearby logic, constraints, or intent: `Then sums of the absolute differences are returned according to the bit`.
  **L1477 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then sums of the absolute differences are returned according to the bit`。
- **L1478 EN**: Comment explains nearby logic, constraints, or intent: `fields in the immediate operand.`.
  **L1478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fields in the immediate operand.`。
- **L1479 EN**: Separator comment used for visual grouping.
  **L1479 CN**: 用于视觉分组的分隔注释。
- **L1480 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1480 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1481 EN**: Separator comment used for visual grouping.
  **L1481 CN**: 用于视觉分组的分隔注释。
- **L1482 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1482 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1483 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_mpsadbw_epu8(__m128i X, __m128i Y, const int M);`.
  **L1483 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_mpsadbw_epu8(__m128i X, __m128i Y, const int M);`。
- **L1484 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1484 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1485 EN**: Separator comment used for visual grouping.
  **L1485 CN**: 用于视觉分组的分隔注释。
- **L1486 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VMPSADBW / MPSADBW </c> instruction.`.
  **L1486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VMPSADBW / MPSADBW </c> instruction.`。
- **L1487 EN**: Separator comment used for visual grouping.
  **L1487 CN**: 用于视觉分组的分隔注释。
- **L1488 EN**: Comment explains nearby logic, constraints, or intent: `param X`.
  **L1488 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param X`。

### Lines 1489-1512

````c
///    A 128-bit vector of [16 x i8].
/// \param Y
///    A 128-bit vector of [16 x i8].
/// \param M
///    An 8-bit immediate operand specifying how the absolute differences are to
///    be calculated, according to the following algorithm:
///    \code
///    // M2 represents bit 2 of the immediate operand
///    // M10 represents bits [1:0] of the immediate operand
///    i = M2 * 4;
///    j = M10 * 4;
///    for (k = 0; k < 8; k = k + 1) {
///      d0 = abs(X[i + k + 0] - Y[j + 0]);
///      d1 = abs(X[i + k + 1] - Y[j + 1]);
///      d2 = abs(X[i + k + 2] - Y[j + 2]);
///      d3 = abs(X[i + k + 3] - Y[j + 3]);
///      r[k] = d0 + d1 + d2 + d3;
///    }
///    \endcode
/// \returns A 128-bit integer vector containing the sums of the sets of
///    absolute differences between both operands.
#define _mm_mpsadbw_epu8(X, Y, M)                                              \
  ((__m128i)__builtin_ia32_mpsadbw128((__v16qi)(__m128i)(X),                   \
                                      (__v16qi)(__m128i)(Y), (M)))
````
- **L1489 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8].`.
  **L1489 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8].`。
- **L1490 EN**: Comment explains nearby logic, constraints, or intent: `param Y`.
  **L1490 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param Y`。
- **L1491 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x i8].`.
  **L1491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x i8].`。
- **L1492 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1492 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1493 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying how the absolute differences are to`.
  **L1493 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying how the absolute differences are to`。
- **L1494 EN**: Comment explains nearby logic, constraints, or intent: `be calculated, according to the following algorithm:`.
  **L1494 CN**: 注释解释附近代码的逻辑、约束或设计意图：`be calculated, according to the following algorithm:`。
- **L1495 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1495 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1496 EN**: Comment explains nearby logic, constraints, or intent: `M2 represents bit 2 of the immediate operand`.
  **L1496 CN**: 注释解释附近代码的逻辑、约束或设计意图：`M2 represents bit 2 of the immediate operand`。
- **L1497 EN**: Comment explains nearby logic, constraints, or intent: `M10 represents bits [1:0] of the immediate operand`.
  **L1497 CN**: 注释解释附近代码的逻辑、约束或设计意图：`M10 represents bits [1:0] of the immediate operand`。
- **L1498 EN**: Comment explains nearby logic, constraints, or intent: `i M2 * 4;`.
  **L1498 CN**: 注释解释附近代码的逻辑、约束或设计意图：`i M2 * 4;`。
- **L1499 EN**: Comment explains nearby logic, constraints, or intent: `j M10 * 4;`.
  **L1499 CN**: 注释解释附近代码的逻辑、约束或设计意图：`j M10 * 4;`。
- **L1500 EN**: Comment explains nearby logic, constraints, or intent: `for (k 0; k < 8; k k + 1) {`.
  **L1500 CN**: 注释解释附近代码的逻辑、约束或设计意图：`for (k 0; k < 8; k k + 1) {`。
- **L1501 EN**: Comment explains nearby logic, constraints, or intent: `d0 abs(X[i + k + 0] - Y[j + 0]);`.
  **L1501 CN**: 注释解释附近代码的逻辑、约束或设计意图：`d0 abs(X[i + k + 0] - Y[j + 0]);`。
- **L1502 EN**: Comment explains nearby logic, constraints, or intent: `d1 abs(X[i + k + 1] - Y[j + 1]);`.
  **L1502 CN**: 注释解释附近代码的逻辑、约束或设计意图：`d1 abs(X[i + k + 1] - Y[j + 1]);`。
- **L1503 EN**: Comment explains nearby logic, constraints, or intent: `d2 abs(X[i + k + 2] - Y[j + 2]);`.
  **L1503 CN**: 注释解释附近代码的逻辑、约束或设计意图：`d2 abs(X[i + k + 2] - Y[j + 2]);`。
- **L1504 EN**: Comment explains nearby logic, constraints, or intent: `d3 abs(X[i + k + 3] - Y[j + 3]);`.
  **L1504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`d3 abs(X[i + k + 3] - Y[j + 3]);`。
- **L1505 EN**: Comment explains nearby logic, constraints, or intent: `r[k] d0 + d1 + d2 + d3;`.
  **L1505 CN**: 注释解释附近代码的逻辑、约束或设计意图：`r[k] d0 + d1 + d2 + d3;`。
- **L1506 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L1506 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L1507 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1507 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1508 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit integer vector containing the sums of the sets of`.
  **L1508 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit integer vector containing the sums of the sets of`。
- **L1509 EN**: Comment explains nearby logic, constraints, or intent: `absolute differences between both operands.`.
  **L1509 CN**: 注释解释附近代码的逻辑、约束或设计意图：`absolute differences between both operands.`。
- **L1510 EN**: Defines macro `_mm_mpsadbw_epu8(X, Y, M)` for conditional compilation, shorthand, or API generation.
  **L1510 CN**: 定义宏 `_mm_mpsadbw_epu8(X, Y, M)`，用于条件编译、简写或 API 生成。
- **L1511 EN**: Continues logic associated with callable symbol `__builtin_ia32_mpsadbw128`.
  **L1511 CN**: 继续与可调用符号 `__builtin_ia32_mpsadbw128` 相关的逻辑。
- **L1512 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(Y), (M)))`.
  **L1512 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(Y), (M)))`。

### Lines 1513-1536

````c

/// Finds the minimum unsigned 16-bit element in the input 128-bit
///    vector of [8 x u16] and returns it and along with its index.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPHMINPOSUW / PHMINPOSUW </c>
/// instruction.
///
/// \param __V
///    A 128-bit vector of [8 x u16].
/// \returns A 128-bit value where bits [15:0] contain the minimum value found
///    in parameter \a __V, bits [18:16] contain the index of the minimum value
///    and the remaining bits are set to 0.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_minpos_epu16(__m128i __V) {
  return (__m128i)__builtin_ia32_phminposuw128((__v8hi)__V);
}

/* Handle the sse4.2 definitions here. */

/* These definitions are normally in nmmintrin.h, but gcc puts them in here
   so we'll do the same.  */

````
- **L1513 EN**: Blank line separating nearby declarations or logic blocks.
  **L1513 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1514 EN**: Comment explains nearby logic, constraints, or intent: `Finds the minimum unsigned 16-bit element in the input 128-bit`.
  **L1514 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Finds the minimum unsigned 16-bit element in the input 128-bit`。
- **L1515 EN**: Comment explains nearby logic, constraints, or intent: `vector of [8 x u16] and returns it and along with its index.`.
  **L1515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [8 x u16] and returns it and along with its index.`。
- **L1516 EN**: Separator comment used for visual grouping.
  **L1516 CN**: 用于视觉分组的分隔注释。
- **L1517 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1518 EN**: Separator comment used for visual grouping.
  **L1518 CN**: 用于视觉分组的分隔注释。
- **L1519 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPHMINPOSUW / PHMINPOSUW </c>`.
  **L1519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPHMINPOSUW / PHMINPOSUW </c>`。
- **L1520 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1520 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1521 EN**: Separator comment used for visual grouping.
  **L1521 CN**: 用于视觉分组的分隔注释。
- **L1522 EN**: Comment explains nearby logic, constraints, or intent: `param __V`.
  **L1522 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V`。
- **L1523 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x u16].`.
  **L1523 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x u16].`。
- **L1524 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit value where bits [15:0] contain the minimum value found`.
  **L1524 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit value where bits [15:0] contain the minimum value found`。
- **L1525 EN**: Comment explains nearby logic, constraints, or intent: `in parameter a __V, bits [18:16] contain the index of the minimum value`.
  **L1525 CN**: 注释解释附近代码的逻辑、约束或设计意图：`in parameter a __V, bits [18:16] contain the index of the minimum value`。
- **L1526 EN**: Comment explains nearby logic, constraints, or intent: `and the remaining bits are set to 0.`.
  **L1526 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and the remaining bits are set to 0.`。
- **L1527 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1527 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1528 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_minpos_epu16(__m128i __V) {`.
  **L1528 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_minpos_epu16(__m128i __V) {`。
- **L1529 EN**: Returns from the current function with `(__m128i)__builtin_ia32_phminposuw128((__v8hi)__V)`.
  **L1529 CN**: 以 `(__m128i)__builtin_ia32_phminposuw128((__v8hi)__V)` 从当前函数返回。
- **L1530 EN**: Closes the current lexical scope or compound statement.
  **L1530 CN**: 结束当前词法作用域或复合语句块。
- **L1531 EN**: Blank line separating nearby declarations or logic blocks.
  **L1531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1532 EN**: Comment explains nearby logic, constraints, or intent: `Handle the sse4.2 definitions here.`.
  **L1532 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Handle the sse4.2 definitions here.`。
- **L1533 EN**: Blank line separating nearby declarations or logic blocks.
  **L1533 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1534 EN**: Comment explains nearby logic, constraints, or intent: `These definitions are normally in nmmintrin.h, but gcc puts them in here`.
  **L1534 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These definitions are normally in nmmintrin.h, but gcc puts them in here`。
- **L1535 EN**: Continues the surrounding expression or declaration: `so we'll do the same.  */`.
  **L1535 CN**: 继续构造周围的表达式或声明：`so we'll do the same.  */`。
- **L1536 EN**: Blank line separating nearby declarations or logic blocks.
  **L1536 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1537-1560

````c
#undef __DEFAULT_FN_ATTRS
#undef __DEFAULT_FN_ATTRS_CONSTEXPR
#define __DEFAULT_FN_ATTRS                                                     \
  __attribute__((__always_inline__, __nodebug__, __target__("sse4.2")))

#if defined(__cplusplus) && (__cplusplus >= 201103L)
#define __DEFAULT_FN_ATTRS_CONSTEXPR __DEFAULT_FN_ATTRS constexpr
#else
#define __DEFAULT_FN_ATTRS_CONSTEXPR __DEFAULT_FN_ATTRS
#endif

/* These specify the type of data that we're comparing.  */
#define _SIDD_UBYTE_OPS 0x00
#define _SIDD_UWORD_OPS 0x01
#define _SIDD_SBYTE_OPS 0x02
#define _SIDD_SWORD_OPS 0x03

/* These specify the type of comparison operation.  */
#define _SIDD_CMP_EQUAL_ANY 0x00
#define _SIDD_CMP_RANGES 0x04
#define _SIDD_CMP_EQUAL_EACH 0x08
#define _SIDD_CMP_EQUAL_ORDERED 0x0c

/* These macros specify the polarity of the operation.  */
````
- **L1537 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS`.
  **L1537 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS`。
- **L1538 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L1538 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L1539 EN**: Defines macro `__DEFAULT_FN_ATTRS` for conditional compilation, shorthand, or API generation.
  **L1539 CN**: 定义宏 `__DEFAULT_FN_ATTRS`，用于条件编译、简写或 API 生成。
- **L1540 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("sse4.2")))`.
  **L1540 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("sse4.2")))`。
- **L1541 EN**: Blank line separating nearby declarations or logic blocks.
  **L1541 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1542 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L1542 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L1543 EN**: Defines macro `__DEFAULT_FN_ATTRS_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L1543 CN**: 定义宏 `__DEFAULT_FN_ATTRS_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L1544 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1544 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1545 EN**: Defines macro `__DEFAULT_FN_ATTRS_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L1545 CN**: 定义宏 `__DEFAULT_FN_ATTRS_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L1546 EN**: Closes the current preprocessor conditional block.
  **L1546 CN**: 结束当前预处理条件块。
- **L1547 EN**: Blank line separating nearby declarations or logic blocks.
  **L1547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1548 EN**: Comment explains nearby logic, constraints, or intent: `These specify the type of data that we're comparing.`.
  **L1548 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These specify the type of data that we're comparing.`。
- **L1549 EN**: Defines macro `_SIDD_UBYTE_OPS` for conditional compilation, shorthand, or API generation.
  **L1549 CN**: 定义宏 `_SIDD_UBYTE_OPS`，用于条件编译、简写或 API 生成。
- **L1550 EN**: Defines macro `_SIDD_UWORD_OPS` for conditional compilation, shorthand, or API generation.
  **L1550 CN**: 定义宏 `_SIDD_UWORD_OPS`，用于条件编译、简写或 API 生成。
- **L1551 EN**: Defines macro `_SIDD_SBYTE_OPS` for conditional compilation, shorthand, or API generation.
  **L1551 CN**: 定义宏 `_SIDD_SBYTE_OPS`，用于条件编译、简写或 API 生成。
- **L1552 EN**: Defines macro `_SIDD_SWORD_OPS` for conditional compilation, shorthand, or API generation.
  **L1552 CN**: 定义宏 `_SIDD_SWORD_OPS`，用于条件编译、简写或 API 生成。
- **L1553 EN**: Blank line separating nearby declarations or logic blocks.
  **L1553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1554 EN**: Comment explains nearby logic, constraints, or intent: `These specify the type of comparison operation.`.
  **L1554 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These specify the type of comparison operation.`。
- **L1555 EN**: Defines macro `_SIDD_CMP_EQUAL_ANY` for conditional compilation, shorthand, or API generation.
  **L1555 CN**: 定义宏 `_SIDD_CMP_EQUAL_ANY`，用于条件编译、简写或 API 生成。
- **L1556 EN**: Defines macro `_SIDD_CMP_RANGES` for conditional compilation, shorthand, or API generation.
  **L1556 CN**: 定义宏 `_SIDD_CMP_RANGES`，用于条件编译、简写或 API 生成。
- **L1557 EN**: Defines macro `_SIDD_CMP_EQUAL_EACH` for conditional compilation, shorthand, or API generation.
  **L1557 CN**: 定义宏 `_SIDD_CMP_EQUAL_EACH`，用于条件编译、简写或 API 生成。
- **L1558 EN**: Defines macro `_SIDD_CMP_EQUAL_ORDERED` for conditional compilation, shorthand, or API generation.
  **L1558 CN**: 定义宏 `_SIDD_CMP_EQUAL_ORDERED`，用于条件编译、简写或 API 生成。
- **L1559 EN**: Blank line separating nearby declarations or logic blocks.
  **L1559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1560 EN**: Comment explains nearby logic, constraints, or intent: `These macros specify the polarity of the operation.`.
  **L1560 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These macros specify the polarity of the operation.`。

### Lines 1561-1584

````c
#define _SIDD_POSITIVE_POLARITY 0x00
#define _SIDD_NEGATIVE_POLARITY 0x10
#define _SIDD_MASKED_POSITIVE_POLARITY 0x20
#define _SIDD_MASKED_NEGATIVE_POLARITY 0x30

/* These macros are used in _mm_cmpXstri() to specify the return.  */
#define _SIDD_LEAST_SIGNIFICANT 0x00
#define _SIDD_MOST_SIGNIFICANT 0x40

/* These macros are used in _mm_cmpXstri() to specify the return.  */
#define _SIDD_BIT_MASK 0x00
#define _SIDD_UNIT_MASK 0x40

/* SSE4.2 Packed Comparison Intrinsics.  */
/// Uses the immediate operand \a M to perform a comparison of string
///    data with implicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns a 128-bit integer vector representing the result
///    mask of the comparison.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128i _mm_cmpistrm(__m128i A, __m128i B, const int M);
/// \endcode
````
- **L1561 EN**: Defines macro `_SIDD_POSITIVE_POLARITY` for conditional compilation, shorthand, or API generation.
  **L1561 CN**: 定义宏 `_SIDD_POSITIVE_POLARITY`，用于条件编译、简写或 API 生成。
- **L1562 EN**: Defines macro `_SIDD_NEGATIVE_POLARITY` for conditional compilation, shorthand, or API generation.
  **L1562 CN**: 定义宏 `_SIDD_NEGATIVE_POLARITY`，用于条件编译、简写或 API 生成。
- **L1563 EN**: Defines macro `_SIDD_MASKED_POSITIVE_POLARITY` for conditional compilation, shorthand, or API generation.
  **L1563 CN**: 定义宏 `_SIDD_MASKED_POSITIVE_POLARITY`，用于条件编译、简写或 API 生成。
- **L1564 EN**: Defines macro `_SIDD_MASKED_NEGATIVE_POLARITY` for conditional compilation, shorthand, or API generation.
  **L1564 CN**: 定义宏 `_SIDD_MASKED_NEGATIVE_POLARITY`，用于条件编译、简写或 API 生成。
- **L1565 EN**: Blank line separating nearby declarations or logic blocks.
  **L1565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1566 EN**: Comment explains nearby logic, constraints, or intent: `These macros are used in _mm_cmpXstri() to specify the return.`.
  **L1566 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These macros are used in _mm_cmpXstri() to specify the return.`。
- **L1567 EN**: Defines macro `_SIDD_LEAST_SIGNIFICANT` for conditional compilation, shorthand, or API generation.
  **L1567 CN**: 定义宏 `_SIDD_LEAST_SIGNIFICANT`，用于条件编译、简写或 API 生成。
- **L1568 EN**: Defines macro `_SIDD_MOST_SIGNIFICANT` for conditional compilation, shorthand, or API generation.
  **L1568 CN**: 定义宏 `_SIDD_MOST_SIGNIFICANT`，用于条件编译、简写或 API 生成。
- **L1569 EN**: Blank line separating nearby declarations or logic blocks.
  **L1569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1570 EN**: Comment explains nearby logic, constraints, or intent: `These macros are used in _mm_cmpXstri() to specify the return.`.
  **L1570 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These macros are used in _mm_cmpXstri() to specify the return.`。
- **L1571 EN**: Defines macro `_SIDD_BIT_MASK` for conditional compilation, shorthand, or API generation.
  **L1571 CN**: 定义宏 `_SIDD_BIT_MASK`，用于条件编译、简写或 API 生成。
- **L1572 EN**: Defines macro `_SIDD_UNIT_MASK` for conditional compilation, shorthand, or API generation.
  **L1572 CN**: 定义宏 `_SIDD_UNIT_MASK`，用于条件编译、简写或 API 生成。
- **L1573 EN**: Blank line separating nearby declarations or logic blocks.
  **L1573 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1574 EN**: Comment explains nearby logic, constraints, or intent: `SSE4.2 Packed Comparison Intrinsics.`.
  **L1574 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4.2 Packed Comparison Intrinsics.`。
- **L1575 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L1575 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L1576 EN**: Comment explains nearby logic, constraints, or intent: `data with implicitly defined lengths that is contained in source operands`.
  **L1576 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with implicitly defined lengths that is contained in source operands`。
- **L1577 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns a 128-bit integer vector representing the result`.
  **L1577 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns a 128-bit integer vector representing the result`。
- **L1578 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison.`.
  **L1578 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison.`。
- **L1579 EN**: Separator comment used for visual grouping.
  **L1579 CN**: 用于视觉分组的分隔注释。
- **L1580 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1580 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1581 EN**: Separator comment used for visual grouping.
  **L1581 CN**: 用于视觉分组的分隔注释。
- **L1582 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1582 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1583 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_cmpistrm(__m128i A, __m128i B, const int M);`.
  **L1583 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_cmpistrm(__m128i A, __m128i B, const int M);`。
- **L1584 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1584 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。

### Lines 1585-1608

````c
///
/// This intrinsic corresponds to the <c> VPCMPISTRM / PCMPISTRM </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words, the type of comparison to perform, and the format of the return
///    value. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
````
- **L1585 EN**: Separator comment used for visual grouping.
  **L1585 CN**: 用于视觉分组的分隔注释。
- **L1586 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPISTRM / PCMPISTRM </c>`.
  **L1586 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPISTRM / PCMPISTRM </c>`。
- **L1587 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1587 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1588 EN**: Separator comment used for visual grouping.
  **L1588 CN**: 用于视觉分组的分隔注释。
- **L1589 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L1589 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L1590 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1590 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1591 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1591 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1592 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L1592 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L1593 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1593 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1594 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1594 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1595 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1595 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1596 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L1596 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L1597 EN**: Comment explains nearby logic, constraints, or intent: `words, the type of comparison to perform, and the format of the return`.
  **L1597 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words, the type of comparison to perform, and the format of the return`。
- **L1598 EN**: Comment explains nearby logic, constraints, or intent: `value. n`.
  **L1598 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value. n`。
- **L1599 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L1599 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L1600 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L1600 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L1601 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L1601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L1602 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L1602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L1603 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L1603 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L1604 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L1604 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L1605 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L1605 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L1606 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L1606 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L1607 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L1607 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L1608 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L1608 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。

### Lines 1609-1632

````c
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B. \n
///    Bit [6]: Determines whether the result is zero-extended or expanded to 16
///             bytes. \n
///      0: The result is zero-extended to 16 bytes. \n
///      1: The result is expanded to 16 bytes (this expansion is performed by
///         repeating each bit 8 or 16 times).
/// \returns Returns a 128-bit integer vector representing the result mask of
///    the comparison.
#define _mm_cmpistrm(A, B, M)                                                  \
  ((__m128i)__builtin_ia32_pcmpistrm128((__v16qi)(__m128i)(A),                 \
                                        (__v16qi)(__m128i)(B), (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with implicitly defined lengths that is contained in source operands
````
- **L1609 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L1609 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L1610 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L1610 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L1611 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L1611 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L1612 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L1612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L1613 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L1613 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L1614 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L1614 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L1615 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L1615 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L1616 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L1616 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L1617 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L1617 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L1618 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L1618 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L1619 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L1619 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L1620 EN**: Comment explains nearby logic, constraints, or intent: `Bit [6]: Determines whether the result is zero-extended or expanded to 16`.
  **L1620 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [6]: Determines whether the result is zero-extended or expanded to 16`。
- **L1621 EN**: Comment explains nearby logic, constraints, or intent: `bytes. n`.
  **L1621 CN**: 注释解释附近代码的逻辑、约束或设计意图：`bytes. n`。
- **L1622 EN**: Comment explains nearby logic, constraints, or intent: `0: The result is zero-extended to 16 bytes. n`.
  **L1622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: The result is zero-extended to 16 bytes. n`。
- **L1623 EN**: Comment explains nearby logic, constraints, or intent: `1: The result is expanded to 16 bytes (this expansion is performed by`.
  **L1623 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: The result is expanded to 16 bytes (this expansion is performed by`。
- **L1624 EN**: Comment explains nearby logic, constraints, or intent: `repeating each bit 8 or 16 times).`.
  **L1624 CN**: 注释解释附近代码的逻辑、约束或设计意图：`repeating each bit 8 or 16 times).`。
- **L1625 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns a 128-bit integer vector representing the result mask of`.
  **L1625 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns a 128-bit integer vector representing the result mask of`。
- **L1626 EN**: Comment explains nearby logic, constraints, or intent: `the comparison.`.
  **L1626 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the comparison.`。
- **L1627 EN**: Defines macro `_mm_cmpistrm(A, B, M)` for conditional compilation, shorthand, or API generation.
  **L1627 CN**: 定义宏 `_mm_cmpistrm(A, B, M)`，用于条件编译、简写或 API 生成。
- **L1628 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpistrm128`.
  **L1628 CN**: 继续与可调用符号 `__builtin_ia32_pcmpistrm128` 相关的逻辑。
- **L1629 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(M)))`.
  **L1629 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(M)))`。
- **L1630 EN**: Blank line separating nearby declarations or logic blocks.
  **L1630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1631 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L1631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L1632 EN**: Comment explains nearby logic, constraints, or intent: `data with implicitly defined lengths that is contained in source operands`.
  **L1632 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with implicitly defined lengths that is contained in source operands`。

### Lines 1633-1656

````c
///    \a A and \a B. Returns an integer representing the result index of the
///    comparison.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpistri(__m128i A, __m128i B, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words, the type of comparison to perform, and the format of the return
///    value. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
````
- **L1633 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns an integer representing the result index of the`.
  **L1633 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns an integer representing the result index of the`。
- **L1634 EN**: Comment explains nearby logic, constraints, or intent: `comparison.`.
  **L1634 CN**: 注释解释附近代码的逻辑、约束或设计意图：`comparison.`。
- **L1635 EN**: Separator comment used for visual grouping.
  **L1635 CN**: 用于视觉分组的分隔注释。
- **L1636 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1636 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1637 EN**: Separator comment used for visual grouping.
  **L1637 CN**: 用于视觉分组的分隔注释。
- **L1638 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1639 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpistri(__m128i A, __m128i B, const int M);`.
  **L1639 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpistri(__m128i A, __m128i B, const int M);`。
- **L1640 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1640 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1641 EN**: Separator comment used for visual grouping.
  **L1641 CN**: 用于视觉分组的分隔注释。
- **L1642 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`.
  **L1642 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`。
- **L1643 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1643 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1644 EN**: Separator comment used for visual grouping.
  **L1644 CN**: 用于视觉分组的分隔注释。
- **L1645 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L1645 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L1646 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1646 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1647 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1647 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1648 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L1648 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L1649 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1649 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1650 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1650 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1651 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1651 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1652 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L1652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L1653 EN**: Comment explains nearby logic, constraints, or intent: `words, the type of comparison to perform, and the format of the return`.
  **L1653 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words, the type of comparison to perform, and the format of the return`。
- **L1654 EN**: Comment explains nearby logic, constraints, or intent: `value. n`.
  **L1654 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value. n`。
- **L1655 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L1655 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L1656 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L1656 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。

### Lines 1657-1680

````c
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B. \n
///    Bit [6]: Determines whether the index of the lowest set bit or the
///             highest set bit is returned. \n
///      0: The index of the least significant set bit. \n
///      1: The index of the most significant set bit. \n
/// \returns Returns an integer representing the result index of the comparison.
````
- **L1657 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L1657 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L1658 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L1658 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L1659 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L1659 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L1660 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L1660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L1661 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L1661 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L1662 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L1662 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L1663 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L1663 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L1664 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L1664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L1665 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L1665 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L1666 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L1666 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L1667 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L1667 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L1668 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search B for substring matches of a A. n`.
  **L1668 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search B for substring matches of a A. n`。
- **L1669 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L1669 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L1670 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L1670 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L1671 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L1671 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L1672 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L1672 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L1673 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L1673 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L1674 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L1674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L1675 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L1675 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L1676 EN**: Comment explains nearby logic, constraints, or intent: `Bit [6]: Determines whether the index of the lowest set bit or the`.
  **L1676 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [6]: Determines whether the index of the lowest set bit or the`。
- **L1677 EN**: Comment explains nearby logic, constraints, or intent: `highest set bit is returned. n`.
  **L1677 CN**: 注释解释附近代码的逻辑、约束或设计意图：`highest set bit is returned. n`。
- **L1678 EN**: Comment explains nearby logic, constraints, or intent: `0: The index of the least significant set bit. n`.
  **L1678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: The index of the least significant set bit. n`。
- **L1679 EN**: Comment explains nearby logic, constraints, or intent: `1: The index of the most significant set bit. n`.
  **L1679 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: The index of the most significant set bit. n`。
- **L1680 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns an integer representing the result index of the comparison.`.
  **L1680 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns an integer representing the result index of the comparison.`。

### Lines 1681-1704

````c
#define _mm_cmpistri(A, B, M)                                                  \
  ((int)__builtin_ia32_pcmpistri128((__v16qi)(__m128i)(A),                     \
                                    (__v16qi)(__m128i)(B), (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with explicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns a 128-bit integer vector representing the result
///    mask of the comparison.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// __m128i _mm_cmpestrm(__m128i A, int LA, __m128i B, int LB, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPESTRM / PCMPESTRM </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LA
///    An integer that specifies the length of the string in \a A.
/// \param B
````
- **L1681 EN**: Defines macro `_mm_cmpistri(A, B, M)` for conditional compilation, shorthand, or API generation.
  **L1681 CN**: 定义宏 `_mm_cmpistri(A, B, M)`，用于条件编译、简写或 API 生成。
- **L1682 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpistri128`.
  **L1682 CN**: 继续与可调用符号 `__builtin_ia32_pcmpistri128` 相关的逻辑。
- **L1683 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(M)))`.
  **L1683 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(M)))`。
- **L1684 EN**: Blank line separating nearby declarations or logic blocks.
  **L1684 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1685 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L1685 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L1686 EN**: Comment explains nearby logic, constraints, or intent: `data with explicitly defined lengths that is contained in source operands`.
  **L1686 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with explicitly defined lengths that is contained in source operands`。
- **L1687 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns a 128-bit integer vector representing the result`.
  **L1687 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns a 128-bit integer vector representing the result`。
- **L1688 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison.`.
  **L1688 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison.`。
- **L1689 EN**: Separator comment used for visual grouping.
  **L1689 CN**: 用于视觉分组的分隔注释。
- **L1690 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1690 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1691 EN**: Separator comment used for visual grouping.
  **L1691 CN**: 用于视觉分组的分隔注释。
- **L1692 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1692 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1693 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_cmpestrm(__m128i A, int LA, __m128i B, int LB, const int M);`.
  **L1693 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_cmpestrm(__m128i A, int LA, __m128i B, int LB, const int M);`。
- **L1694 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1694 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1695 EN**: Separator comment used for visual grouping.
  **L1695 CN**: 用于视觉分组的分隔注释。
- **L1696 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPESTRM / PCMPESTRM </c>`.
  **L1696 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPESTRM / PCMPESTRM </c>`。
- **L1697 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1697 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1698 EN**: Separator comment used for visual grouping.
  **L1698 CN**: 用于视觉分组的分隔注释。
- **L1699 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L1699 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L1700 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1700 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1701 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1701 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1702 EN**: Comment explains nearby logic, constraints, or intent: `param LA`.
  **L1702 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LA`。
- **L1703 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a A.`.
  **L1703 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a A.`。
- **L1704 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L1704 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。

### Lines 1705-1728

````c
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LB
///    An integer that specifies the length of the string in \a B.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words, the type of comparison to perform, and the format of the return
///    value. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
````
- **L1705 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1706 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1706 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1707 EN**: Comment explains nearby logic, constraints, or intent: `param LB`.
  **L1707 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LB`。
- **L1708 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a B.`.
  **L1708 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a B.`。
- **L1709 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1709 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1710 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L1710 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L1711 EN**: Comment explains nearby logic, constraints, or intent: `words, the type of comparison to perform, and the format of the return`.
  **L1711 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words, the type of comparison to perform, and the format of the return`。
- **L1712 EN**: Comment explains nearby logic, constraints, or intent: `value. n`.
  **L1712 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value. n`。
- **L1713 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L1713 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L1714 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L1714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L1715 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L1715 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L1716 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L1716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L1717 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L1717 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L1718 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L1718 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L1719 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L1719 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L1720 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L1720 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L1721 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L1721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L1722 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L1722 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L1723 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L1723 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L1724 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L1724 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L1725 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L1725 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L1726 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L1726 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L1727 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L1727 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L1728 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L1728 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。

### Lines 1729-1752

````c
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B. \n
///    Bit [6]: Determines whether the result is zero-extended or expanded to 16
///             bytes. \n
///      0: The result is zero-extended to 16 bytes. \n
///      1: The result is expanded to 16 bytes (this expansion is performed by
///         repeating each bit 8 or 16 times). \n
/// \returns Returns a 128-bit integer vector representing the result mask of
///    the comparison.
#define _mm_cmpestrm(A, LA, B, LB, M)                                          \
  ((__m128i)__builtin_ia32_pcmpestrm128((__v16qi)(__m128i)(A), (int)(LA),      \
                                        (__v16qi)(__m128i)(B), (int)(LB),      \
                                        (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with explicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns an integer representing the result index of the
///    comparison.
///
/// \headerfile <x86intrin.h>
///
````
- **L1729 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L1729 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L1730 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L1730 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L1731 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L1731 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L1732 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L1732 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L1733 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L1733 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L1734 EN**: Comment explains nearby logic, constraints, or intent: `Bit [6]: Determines whether the result is zero-extended or expanded to 16`.
  **L1734 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [6]: Determines whether the result is zero-extended or expanded to 16`。
- **L1735 EN**: Comment explains nearby logic, constraints, or intent: `bytes. n`.
  **L1735 CN**: 注释解释附近代码的逻辑、约束或设计意图：`bytes. n`。
- **L1736 EN**: Comment explains nearby logic, constraints, or intent: `0: The result is zero-extended to 16 bytes. n`.
  **L1736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: The result is zero-extended to 16 bytes. n`。
- **L1737 EN**: Comment explains nearby logic, constraints, or intent: `1: The result is expanded to 16 bytes (this expansion is performed by`.
  **L1737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: The result is expanded to 16 bytes (this expansion is performed by`。
- **L1738 EN**: Comment explains nearby logic, constraints, or intent: `repeating each bit 8 or 16 times). n`.
  **L1738 CN**: 注释解释附近代码的逻辑、约束或设计意图：`repeating each bit 8 or 16 times). n`。
- **L1739 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns a 128-bit integer vector representing the result mask of`.
  **L1739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns a 128-bit integer vector representing the result mask of`。
- **L1740 EN**: Comment explains nearby logic, constraints, or intent: `the comparison.`.
  **L1740 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the comparison.`。
- **L1741 EN**: Defines macro `_mm_cmpestrm(A, LA, B, LB, M)` for conditional compilation, shorthand, or API generation.
  **L1741 CN**: 定义宏 `_mm_cmpestrm(A, LA, B, LB, M)`，用于条件编译、简写或 API 生成。
- **L1742 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpestrm128`.
  **L1742 CN**: 继续与可调用符号 `__builtin_ia32_pcmpestrm128` 相关的逻辑。
- **L1743 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(LB),      \`.
  **L1743 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(LB),      \`。
- **L1744 EN**: Continues the surrounding expression or declaration: `(int)(M)))`.
  **L1744 CN**: 继续构造周围的表达式或声明：`(int)(M)))`。
- **L1745 EN**: Blank line separating nearby declarations or logic blocks.
  **L1745 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1746 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L1746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L1747 EN**: Comment explains nearby logic, constraints, or intent: `data with explicitly defined lengths that is contained in source operands`.
  **L1747 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with explicitly defined lengths that is contained in source operands`。
- **L1748 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns an integer representing the result index of the`.
  **L1748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns an integer representing the result index of the`。
- **L1749 EN**: Comment explains nearby logic, constraints, or intent: `comparison.`.
  **L1749 CN**: 注释解释附近代码的逻辑、约束或设计意图：`comparison.`。
- **L1750 EN**: Separator comment used for visual grouping.
  **L1750 CN**: 用于视觉分组的分隔注释。
- **L1751 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1751 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1752 EN**: Separator comment used for visual grouping.
  **L1752 CN**: 用于视觉分组的分隔注释。

### Lines 1753-1776

````c
/// \code
/// int _mm_cmpestri(__m128i A, int LA, __m128i B, int LB, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LA
///    An integer that specifies the length of the string in \a A.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LB
///    An integer that specifies the length of the string in \a B.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words, the type of comparison to perform, and the format of the return
///    value. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
````
- **L1753 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1753 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1754 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpestri(__m128i A, int LA, __m128i B, int LB, const int M);`.
  **L1754 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpestri(__m128i A, int LA, __m128i B, int LB, const int M);`。
- **L1755 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1756 EN**: Separator comment used for visual grouping.
  **L1756 CN**: 用于视觉分组的分隔注释。
- **L1757 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`.
  **L1757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`。
- **L1758 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1758 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1759 EN**: Separator comment used for visual grouping.
  **L1759 CN**: 用于视觉分组的分隔注释。
- **L1760 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L1760 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L1761 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1761 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1762 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1762 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1763 EN**: Comment explains nearby logic, constraints, or intent: `param LA`.
  **L1763 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LA`。
- **L1764 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a A.`.
  **L1764 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a A.`。
- **L1765 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L1765 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L1766 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1766 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1767 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1767 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1768 EN**: Comment explains nearby logic, constraints, or intent: `param LB`.
  **L1768 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LB`。
- **L1769 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a B.`.
  **L1769 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a B.`。
- **L1770 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1770 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1771 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L1771 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L1772 EN**: Comment explains nearby logic, constraints, or intent: `words, the type of comparison to perform, and the format of the return`.
  **L1772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words, the type of comparison to perform, and the format of the return`。
- **L1773 EN**: Comment explains nearby logic, constraints, or intent: `value. n`.
  **L1773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value. n`。
- **L1774 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L1774 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L1775 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L1775 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L1776 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L1776 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。

### Lines 1777-1800

````c
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B. \n
///    Bit [6]: Determines whether the index of the lowest set bit or the
///             highest set bit is returned. \n
///      0: The index of the least significant set bit. \n
///      1: The index of the most significant set bit. \n
/// \returns Returns an integer representing the result index of the comparison.
#define _mm_cmpestri(A, LA, B, LB, M)                                          \
````
- **L1777 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L1777 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L1778 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L1778 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L1779 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L1779 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L1780 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L1780 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L1781 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L1781 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L1782 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L1782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L1783 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L1783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L1784 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L1784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L1785 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L1785 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L1786 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L1786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L1787 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search B for substring matches of a A. n`.
  **L1787 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search B for substring matches of a A. n`。
- **L1788 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L1788 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L1789 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L1789 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L1790 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L1790 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L1791 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L1791 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L1792 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L1792 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L1793 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L1793 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L1794 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L1794 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L1795 EN**: Comment explains nearby logic, constraints, or intent: `Bit [6]: Determines whether the index of the lowest set bit or the`.
  **L1795 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit [6]: Determines whether the index of the lowest set bit or the`。
- **L1796 EN**: Comment explains nearby logic, constraints, or intent: `highest set bit is returned. n`.
  **L1796 CN**: 注释解释附近代码的逻辑、约束或设计意图：`highest set bit is returned. n`。
- **L1797 EN**: Comment explains nearby logic, constraints, or intent: `0: The index of the least significant set bit. n`.
  **L1797 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0: The index of the least significant set bit. n`。
- **L1798 EN**: Comment explains nearby logic, constraints, or intent: `1: The index of the most significant set bit. n`.
  **L1798 CN**: 注释解释附近代码的逻辑、约束或设计意图：`1: The index of the most significant set bit. n`。
- **L1799 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns an integer representing the result index of the comparison.`.
  **L1799 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns an integer representing the result index of the comparison.`。
- **L1800 EN**: Defines macro `_mm_cmpestri(A, LA, B, LB, M)` for conditional compilation, shorthand, or API generation.
  **L1800 CN**: 定义宏 `_mm_cmpestri(A, LA, B, LB, M)`，用于条件编译、简写或 API 生成。

### Lines 1801-1824

````c
  ((int)__builtin_ia32_pcmpestri128((__v16qi)(__m128i)(A), (int)(LA),          \
                                    (__v16qi)(__m128i)(B), (int)(LB),          \
                                    (int)(M)))

/* SSE4.2 Packed Comparison Intrinsics and EFlag Reading.  */
/// Uses the immediate operand \a M to perform a comparison of string
///    data with implicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns 1 if the bit mask is zero and the length of the
///    string in \a B is the maximum, otherwise, returns 0.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpistra(__m128i A, __m128i B, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
````
- **L1801 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpestri128`.
  **L1801 CN**: 继续与可调用符号 `__builtin_ia32_pcmpestri128` 相关的逻辑。
- **L1802 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(LB),          \`.
  **L1802 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(LB),          \`。
- **L1803 EN**: Continues the surrounding expression or declaration: `(int)(M)))`.
  **L1803 CN**: 继续构造周围的表达式或声明：`(int)(M)))`。
- **L1804 EN**: Blank line separating nearby declarations or logic blocks.
  **L1804 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1805 EN**: Comment explains nearby logic, constraints, or intent: `SSE4.2 Packed Comparison Intrinsics and EFlag Reading.`.
  **L1805 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4.2 Packed Comparison Intrinsics and EFlag Reading.`。
- **L1806 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L1806 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L1807 EN**: Comment explains nearby logic, constraints, or intent: `data with implicitly defined lengths that is contained in source operands`.
  **L1807 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with implicitly defined lengths that is contained in source operands`。
- **L1808 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns 1 if the bit mask is zero and the length of the`.
  **L1808 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns 1 if the bit mask is zero and the length of the`。
- **L1809 EN**: Comment explains nearby logic, constraints, or intent: `string in a B is the maximum, otherwise, returns 0.`.
  **L1809 CN**: 注释解释附近代码的逻辑、约束或设计意图：`string in a B is the maximum, otherwise, returns 0.`。
- **L1810 EN**: Separator comment used for visual grouping.
  **L1810 CN**: 用于视觉分组的分隔注释。
- **L1811 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1811 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1812 EN**: Separator comment used for visual grouping.
  **L1812 CN**: 用于视觉分组的分隔注释。
- **L1813 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1813 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1814 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpistra(__m128i A, __m128i B, const int M);`.
  **L1814 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpistra(__m128i A, __m128i B, const int M);`。
- **L1815 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1816 EN**: Separator comment used for visual grouping.
  **L1816 CN**: 用于视觉分组的分隔注释。
- **L1817 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`.
  **L1817 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`。
- **L1818 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1818 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1819 EN**: Separator comment used for visual grouping.
  **L1819 CN**: 用于视觉分组的分隔注释。
- **L1820 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L1820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L1821 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1821 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1822 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1822 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1823 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L1823 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L1824 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。

### Lines 1825-1848

````c
///    compared.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
````
- **L1825 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1825 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1826 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1826 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1827 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L1827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L1828 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L1828 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L1829 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L1829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L1830 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L1830 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L1831 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L1831 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L1832 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L1832 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L1833 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L1833 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L1834 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L1834 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L1835 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L1835 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L1836 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L1836 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L1837 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L1837 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L1838 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L1838 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L1839 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L1839 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L1840 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L1840 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L1841 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L1841 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L1842 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L1842 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L1843 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L1843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L1844 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L1844 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L1845 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L1845 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L1846 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L1846 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L1847 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L1847 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L1848 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L1848 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。

### Lines 1849-1872

````c
///          to the size of \a A or \a B. \n
/// \returns Returns 1 if the bit mask is zero and the length of the string in
///    \a B is the maximum; otherwise, returns 0.
#define _mm_cmpistra(A, B, M)                                                  \
  ((int)__builtin_ia32_pcmpistria128((__v16qi)(__m128i)(A),                    \
                                     (__v16qi)(__m128i)(B), (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with implicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns 1 if the bit mask is non-zero, otherwise, returns
///    0.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpistrc(__m128i A, __m128i B, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
````
- **L1849 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L1849 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L1850 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns 1 if the bit mask is zero and the length of the string in`.
  **L1850 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns 1 if the bit mask is zero and the length of the string in`。
- **L1851 EN**: Comment explains nearby logic, constraints, or intent: `a B is the maximum; otherwise, returns 0.`.
  **L1851 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B is the maximum; otherwise, returns 0.`。
- **L1852 EN**: Defines macro `_mm_cmpistra(A, B, M)` for conditional compilation, shorthand, or API generation.
  **L1852 CN**: 定义宏 `_mm_cmpistra(A, B, M)`，用于条件编译、简写或 API 生成。
- **L1853 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpistria128`.
  **L1853 CN**: 继续与可调用符号 `__builtin_ia32_pcmpistria128` 相关的逻辑。
- **L1854 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(M)))`.
  **L1854 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(M)))`。
- **L1855 EN**: Blank line separating nearby declarations or logic blocks.
  **L1855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1856 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L1856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L1857 EN**: Comment explains nearby logic, constraints, or intent: `data with implicitly defined lengths that is contained in source operands`.
  **L1857 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with implicitly defined lengths that is contained in source operands`。
- **L1858 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns 1 if the bit mask is non-zero, otherwise, returns`.
  **L1858 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns 1 if the bit mask is non-zero, otherwise, returns`。
- **L1859 EN**: Comment explains nearby logic, constraints, or intent: `0.`.
  **L1859 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0.`。
- **L1860 EN**: Separator comment used for visual grouping.
  **L1860 CN**: 用于视觉分组的分隔注释。
- **L1861 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1861 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1862 EN**: Separator comment used for visual grouping.
  **L1862 CN**: 用于视觉分组的分隔注释。
- **L1863 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1864 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpistrc(__m128i A, __m128i B, const int M);`.
  **L1864 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpistrc(__m128i A, __m128i B, const int M);`。
- **L1865 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1865 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1866 EN**: Separator comment used for visual grouping.
  **L1866 CN**: 用于视觉分组的分隔注释。
- **L1867 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`.
  **L1867 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`。
- **L1868 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1868 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1869 EN**: Separator comment used for visual grouping.
  **L1869 CN**: 用于视觉分组的分隔注释。
- **L1870 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L1870 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L1871 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1871 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1872 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1872 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。

### Lines 1873-1896

````c
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
````
- **L1873 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L1873 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L1874 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1874 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1875 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1875 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1876 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1876 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1877 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L1877 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L1878 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L1878 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L1879 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L1879 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L1880 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L1880 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L1881 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L1881 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L1882 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L1882 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L1883 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L1883 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L1884 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L1884 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L1885 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L1885 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L1886 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L1886 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L1887 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L1887 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L1888 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L1888 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L1889 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L1889 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L1890 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L1890 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L1891 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L1891 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L1892 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search B for substring matches of a A. n`.
  **L1892 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search B for substring matches of a A. n`。
- **L1893 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L1893 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L1894 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L1894 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L1895 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L1895 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L1896 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L1896 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。

### Lines 1897-1920

````c
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B.
/// \returns Returns 1 if the bit mask is non-zero, otherwise, returns 0.
#define _mm_cmpistrc(A, B, M)                                                  \
  ((int)__builtin_ia32_pcmpistric128((__v16qi)(__m128i)(A),                    \
                                     (__v16qi)(__m128i)(B), (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with implicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns bit 0 of the resulting bit mask.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpistro(__m128i A, __m128i B, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
````
- **L1897 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L1897 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L1898 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L1898 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L1899 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B.`.
  **L1899 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B.`。
- **L1900 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns 1 if the bit mask is non-zero, otherwise, returns 0.`.
  **L1900 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns 1 if the bit mask is non-zero, otherwise, returns 0.`。
- **L1901 EN**: Defines macro `_mm_cmpistrc(A, B, M)` for conditional compilation, shorthand, or API generation.
  **L1901 CN**: 定义宏 `_mm_cmpistrc(A, B, M)`，用于条件编译、简写或 API 生成。
- **L1902 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpistric128`.
  **L1902 CN**: 继续与可调用符号 `__builtin_ia32_pcmpistric128` 相关的逻辑。
- **L1903 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(M)))`.
  **L1903 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(M)))`。
- **L1904 EN**: Blank line separating nearby declarations or logic blocks.
  **L1904 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1905 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L1905 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L1906 EN**: Comment explains nearby logic, constraints, or intent: `data with implicitly defined lengths that is contained in source operands`.
  **L1906 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with implicitly defined lengths that is contained in source operands`。
- **L1907 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns bit 0 of the resulting bit mask.`.
  **L1907 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns bit 0 of the resulting bit mask.`。
- **L1908 EN**: Separator comment used for visual grouping.
  **L1908 CN**: 用于视觉分组的分隔注释。
- **L1909 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1909 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1910 EN**: Separator comment used for visual grouping.
  **L1910 CN**: 用于视觉分组的分隔注释。
- **L1911 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1911 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1912 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpistro(__m128i A, __m128i B, const int M);`.
  **L1912 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpistro(__m128i A, __m128i B, const int M);`。
- **L1913 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1913 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1914 EN**: Separator comment used for visual grouping.
  **L1914 CN**: 用于视觉分组的分隔注释。
- **L1915 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`.
  **L1915 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`。
- **L1916 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1916 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1917 EN**: Separator comment used for visual grouping.
  **L1917 CN**: 用于视觉分组的分隔注释。
- **L1918 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L1918 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L1919 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1919 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1920 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1920 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。

### Lines 1921-1944

````c
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
````
- **L1921 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L1921 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L1922 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1922 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1923 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1923 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1924 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1924 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1925 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L1925 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L1926 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L1926 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L1927 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L1927 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L1928 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L1928 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L1929 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L1929 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L1930 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L1930 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L1931 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L1931 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L1932 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L1932 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L1933 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L1933 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L1934 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L1934 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L1935 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L1935 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L1936 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L1936 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L1937 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L1937 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L1938 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L1938 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L1939 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L1939 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L1940 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search B for substring matches of a A. n`.
  **L1940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search B for substring matches of a A. n`。
- **L1941 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L1941 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L1942 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L1942 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L1943 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L1943 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L1944 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L1944 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。

### Lines 1945-1968

````c
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B. \n
/// \returns Returns bit 0 of the resulting bit mask.
#define _mm_cmpistro(A, B, M)                                                  \
  ((int)__builtin_ia32_pcmpistrio128((__v16qi)(__m128i)(A),                    \
                                     (__v16qi)(__m128i)(B), (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with implicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns 1 if the length of the string in \a A is less than
///    the maximum, otherwise, returns 0.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpistrs(__m128i A, __m128i B, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
````
- **L1945 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L1945 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L1946 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L1946 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L1947 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L1947 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L1948 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns bit 0 of the resulting bit mask.`.
  **L1948 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns bit 0 of the resulting bit mask.`。
- **L1949 EN**: Defines macro `_mm_cmpistro(A, B, M)` for conditional compilation, shorthand, or API generation.
  **L1949 CN**: 定义宏 `_mm_cmpistro(A, B, M)`，用于条件编译、简写或 API 生成。
- **L1950 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpistrio128`.
  **L1950 CN**: 继续与可调用符号 `__builtin_ia32_pcmpistrio128` 相关的逻辑。
- **L1951 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(M)))`.
  **L1951 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(M)))`。
- **L1952 EN**: Blank line separating nearby declarations or logic blocks.
  **L1952 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1953 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L1953 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L1954 EN**: Comment explains nearby logic, constraints, or intent: `data with implicitly defined lengths that is contained in source operands`.
  **L1954 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with implicitly defined lengths that is contained in source operands`。
- **L1955 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns 1 if the length of the string in a A is less than`.
  **L1955 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns 1 if the length of the string in a A is less than`。
- **L1956 EN**: Comment explains nearby logic, constraints, or intent: `the maximum, otherwise, returns 0.`.
  **L1956 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the maximum, otherwise, returns 0.`。
- **L1957 EN**: Separator comment used for visual grouping.
  **L1957 CN**: 用于视觉分组的分隔注释。
- **L1958 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1958 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1959 EN**: Separator comment used for visual grouping.
  **L1959 CN**: 用于视觉分组的分隔注释。
- **L1960 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L1960 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L1961 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpistrs(__m128i A, __m128i B, const int M);`.
  **L1961 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpistrs(__m128i A, __m128i B, const int M);`。
- **L1962 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1962 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1963 EN**: Separator comment used for visual grouping.
  **L1963 CN**: 用于视觉分组的分隔注释。
- **L1964 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`.
  **L1964 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`。
- **L1965 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1965 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1966 EN**: Separator comment used for visual grouping.
  **L1966 CN**: 用于视觉分组的分隔注释。
- **L1967 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L1967 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L1968 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1968 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。

### Lines 1969-1992

````c
///    compared.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
````
- **L1969 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1969 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1970 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L1970 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L1971 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L1971 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L1972 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L1972 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L1973 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L1973 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L1974 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L1974 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L1975 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L1975 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L1976 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L1976 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L1977 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L1977 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L1978 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L1978 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L1979 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L1979 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L1980 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L1980 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L1981 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L1981 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L1982 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L1982 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L1983 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L1983 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L1984 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L1984 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L1985 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L1985 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L1986 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L1986 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L1987 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L1987 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L1988 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L1988 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L1989 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L1989 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L1990 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L1990 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L1991 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L1991 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L1992 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L1992 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。

### Lines 1993-2016

````c
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B. \n
/// \returns Returns 1 if the length of the string in \a A is less than the
///    maximum, otherwise, returns 0.
#define _mm_cmpistrs(A, B, M)                                                  \
  ((int)__builtin_ia32_pcmpistris128((__v16qi)(__m128i)(A),                    \
                                     (__v16qi)(__m128i)(B), (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with implicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns 1 if the length of the string in \a B is less than
///    the maximum, otherwise, returns 0.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpistrz(__m128i A, __m128i B, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>
/// instruction.
///
````
- **L1993 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L1993 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L1994 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L1994 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L1995 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L1995 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L1996 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L1996 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L1997 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns 1 if the length of the string in a A is less than the`.
  **L1997 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns 1 if the length of the string in a A is less than the`。
- **L1998 EN**: Comment explains nearby logic, constraints, or intent: `maximum, otherwise, returns 0.`.
  **L1998 CN**: 注释解释附近代码的逻辑、约束或设计意图：`maximum, otherwise, returns 0.`。
- **L1999 EN**: Defines macro `_mm_cmpistrs(A, B, M)` for conditional compilation, shorthand, or API generation.
  **L1999 CN**: 定义宏 `_mm_cmpistrs(A, B, M)`，用于条件编译、简写或 API 生成。
- **L2000 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpistris128`.
  **L2000 CN**: 继续与可调用符号 `__builtin_ia32_pcmpistris128` 相关的逻辑。
- **L2001 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(M)))`.
  **L2001 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(M)))`。
- **L2002 EN**: Blank line separating nearby declarations or logic blocks.
  **L2002 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2003 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L2003 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L2004 EN**: Comment explains nearby logic, constraints, or intent: `data with implicitly defined lengths that is contained in source operands`.
  **L2004 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with implicitly defined lengths that is contained in source operands`。
- **L2005 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns 1 if the length of the string in a B is less than`.
  **L2005 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns 1 if the length of the string in a B is less than`。
- **L2006 EN**: Comment explains nearby logic, constraints, or intent: `the maximum, otherwise, returns 0.`.
  **L2006 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the maximum, otherwise, returns 0.`。
- **L2007 EN**: Separator comment used for visual grouping.
  **L2007 CN**: 用于视觉分组的分隔注释。
- **L2008 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L2008 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L2009 EN**: Separator comment used for visual grouping.
  **L2009 CN**: 用于视觉分组的分隔注释。
- **L2010 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L2010 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L2011 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpistrz(__m128i A, __m128i B, const int M);`.
  **L2011 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpistrz(__m128i A, __m128i B, const int M);`。
- **L2012 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2012 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2013 EN**: Separator comment used for visual grouping.
  **L2013 CN**: 用于视觉分组的分隔注释。
- **L2014 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`.
  **L2014 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPISTRI / PCMPISTRI </c>`。
- **L2015 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L2015 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L2016 EN**: Separator comment used for visual grouping.
  **L2016 CN**: 用于视觉分组的分隔注释。

### Lines 2017-2040

````c
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
````
- **L2017 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L2017 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L2018 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2018 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2019 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2020 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L2020 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L2021 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2021 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2022 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2022 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2023 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L2023 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L2024 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L2024 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L2025 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L2025 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L2026 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L2026 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L2027 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L2027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L2028 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L2028 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L2029 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L2029 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L2030 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L2030 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L2031 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L2031 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L2032 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L2032 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L2033 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L2033 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L2034 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L2034 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L2035 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L2035 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L2036 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L2036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L2037 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L2037 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L2038 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L2038 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L2039 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L2039 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L2040 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L2040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。

### Lines 2041-2064

````c
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B.
/// \returns Returns 1 if the length of the string in \a B is less than the
///    maximum, otherwise, returns 0.
#define _mm_cmpistrz(A, B, M)                                                  \
  ((int)__builtin_ia32_pcmpistriz128((__v16qi)(__m128i)(A),                    \
                                     (__v16qi)(__m128i)(B), (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with explicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns 1 if the bit mask is zero and the length of the
///    string in \a B is the maximum, otherwise, returns 0.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpestra(__m128i A, int LA, __m128i B, int LB, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>
````
- **L2041 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L2041 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L2042 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L2042 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L2043 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L2043 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L2044 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L2044 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L2045 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L2045 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L2046 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B.`.
  **L2046 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B.`。
- **L2047 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns 1 if the length of the string in a B is less than the`.
  **L2047 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns 1 if the length of the string in a B is less than the`。
- **L2048 EN**: Comment explains nearby logic, constraints, or intent: `maximum, otherwise, returns 0.`.
  **L2048 CN**: 注释解释附近代码的逻辑、约束或设计意图：`maximum, otherwise, returns 0.`。
- **L2049 EN**: Defines macro `_mm_cmpistrz(A, B, M)` for conditional compilation, shorthand, or API generation.
  **L2049 CN**: 定义宏 `_mm_cmpistrz(A, B, M)`，用于条件编译、简写或 API 生成。
- **L2050 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpistriz128`.
  **L2050 CN**: 继续与可调用符号 `__builtin_ia32_pcmpistriz128` 相关的逻辑。
- **L2051 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(M)))`.
  **L2051 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(M)))`。
- **L2052 EN**: Blank line separating nearby declarations or logic blocks.
  **L2052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2053 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L2053 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L2054 EN**: Comment explains nearby logic, constraints, or intent: `data with explicitly defined lengths that is contained in source operands`.
  **L2054 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with explicitly defined lengths that is contained in source operands`。
- **L2055 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns 1 if the bit mask is zero and the length of the`.
  **L2055 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns 1 if the bit mask is zero and the length of the`。
- **L2056 EN**: Comment explains nearby logic, constraints, or intent: `string in a B is the maximum, otherwise, returns 0.`.
  **L2056 CN**: 注释解释附近代码的逻辑、约束或设计意图：`string in a B is the maximum, otherwise, returns 0.`。
- **L2057 EN**: Separator comment used for visual grouping.
  **L2057 CN**: 用于视觉分组的分隔注释。
- **L2058 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L2058 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L2059 EN**: Separator comment used for visual grouping.
  **L2059 CN**: 用于视觉分组的分隔注释。
- **L2060 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L2060 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L2061 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpestra(__m128i A, int LA, __m128i B, int LB, const int M);`.
  **L2061 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpestra(__m128i A, int LA, __m128i B, int LB, const int M);`。
- **L2062 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2062 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2063 EN**: Separator comment used for visual grouping.
  **L2063 CN**: 用于视觉分组的分隔注释。
- **L2064 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`.
  **L2064 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`。

### Lines 2065-2088

````c
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LA
///    An integer that specifies the length of the string in \a A.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LB
///    An integer that specifies the length of the string in \a B.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
````
- **L2065 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L2065 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L2066 EN**: Separator comment used for visual grouping.
  **L2066 CN**: 用于视觉分组的分隔注释。
- **L2067 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L2067 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L2068 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2068 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2069 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2069 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2070 EN**: Comment explains nearby logic, constraints, or intent: `param LA`.
  **L2070 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LA`。
- **L2071 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a A.`.
  **L2071 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a A.`。
- **L2072 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L2072 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L2073 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2073 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2074 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2074 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2075 EN**: Comment explains nearby logic, constraints, or intent: `param LB`.
  **L2075 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LB`。
- **L2076 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a B.`.
  **L2076 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a B.`。
- **L2077 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L2077 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L2078 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L2078 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L2079 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L2079 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L2080 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L2080 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L2081 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L2081 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L2082 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L2082 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L2083 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L2083 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L2084 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L2084 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L2085 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L2085 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L2086 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L2086 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L2087 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L2087 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L2088 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L2088 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。

### Lines 2089-2112

````c
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B.
/// \returns Returns 1 if the bit mask is zero and the length of the string in
///    \a B is the maximum, otherwise, returns 0.
#define _mm_cmpestra(A, LA, B, LB, M)                                          \
  ((int)__builtin_ia32_pcmpestria128((__v16qi)(__m128i)(A), (int)(LA),         \
                                     (__v16qi)(__m128i)(B), (int)(LB),         \
                                     (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with explicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns 1 if the resulting mask is non-zero, otherwise,
///    returns 0.
///
````
- **L2089 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L2089 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L2090 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L2090 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L2091 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L2091 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L2092 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L2092 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L2093 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L2093 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L2094 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L2094 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L2095 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L2095 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L2096 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L2096 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L2097 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L2097 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L2098 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L2098 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L2099 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L2099 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L2100 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B.`.
  **L2100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B.`。
- **L2101 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns 1 if the bit mask is zero and the length of the string in`.
  **L2101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns 1 if the bit mask is zero and the length of the string in`。
- **L2102 EN**: Comment explains nearby logic, constraints, or intent: `a B is the maximum, otherwise, returns 0.`.
  **L2102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B is the maximum, otherwise, returns 0.`。
- **L2103 EN**: Defines macro `_mm_cmpestra(A, LA, B, LB, M)` for conditional compilation, shorthand, or API generation.
  **L2103 CN**: 定义宏 `_mm_cmpestra(A, LA, B, LB, M)`，用于条件编译、简写或 API 生成。
- **L2104 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpestria128`.
  **L2104 CN**: 继续与可调用符号 `__builtin_ia32_pcmpestria128` 相关的逻辑。
- **L2105 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(LB),         \`.
  **L2105 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(LB),         \`。
- **L2106 EN**: Continues the surrounding expression or declaration: `(int)(M)))`.
  **L2106 CN**: 继续构造周围的表达式或声明：`(int)(M)))`。
- **L2107 EN**: Blank line separating nearby declarations or logic blocks.
  **L2107 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2108 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L2108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L2109 EN**: Comment explains nearby logic, constraints, or intent: `data with explicitly defined lengths that is contained in source operands`.
  **L2109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with explicitly defined lengths that is contained in source operands`。
- **L2110 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns 1 if the resulting mask is non-zero, otherwise,`.
  **L2110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns 1 if the resulting mask is non-zero, otherwise,`。
- **L2111 EN**: Comment explains nearby logic, constraints, or intent: `returns 0.`.
  **L2111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns 0.`。
- **L2112 EN**: Separator comment used for visual grouping.
  **L2112 CN**: 用于视觉分组的分隔注释。

### Lines 2113-2136

````c
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpestrc(__m128i A, int LA, __m128i B, int LB, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LA
///    An integer that specifies the length of the string in \a A.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LB
///    An integer that specifies the length of the string in \a B.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
````
- **L2113 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L2113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L2114 EN**: Separator comment used for visual grouping.
  **L2114 CN**: 用于视觉分组的分隔注释。
- **L2115 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L2115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L2116 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpestrc(__m128i A, int LA, __m128i B, int LB, const int M);`.
  **L2116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpestrc(__m128i A, int LA, __m128i B, int LB, const int M);`。
- **L2117 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2118 EN**: Separator comment used for visual grouping.
  **L2118 CN**: 用于视觉分组的分隔注释。
- **L2119 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`.
  **L2119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`。
- **L2120 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L2120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L2121 EN**: Separator comment used for visual grouping.
  **L2121 CN**: 用于视觉分组的分隔注释。
- **L2122 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L2122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L2123 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2124 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2125 EN**: Comment explains nearby logic, constraints, or intent: `param LA`.
  **L2125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LA`。
- **L2126 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a A.`.
  **L2126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a A.`。
- **L2127 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L2127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L2128 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2129 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2130 EN**: Comment explains nearby logic, constraints, or intent: `param LB`.
  **L2130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LB`。
- **L2131 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a B.`.
  **L2131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a B.`。
- **L2132 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L2132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L2133 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L2133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L2134 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L2134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L2135 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L2135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L2136 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L2136 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。

### Lines 2137-2160

````c
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B. \n
/// \returns Returns 1 if the resulting mask is non-zero, otherwise, returns 0.
#define _mm_cmpestrc(A, LA, B, LB, M)                                          \
  ((int)__builtin_ia32_pcmpestric128((__v16qi)(__m128i)(A), (int)(LA),         \
                                     (__v16qi)(__m128i)(B), (int)(LB),         \
                                     (int)(M)))
````
- **L2137 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L2137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L2138 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L2138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L2139 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L2139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L2140 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L2140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L2141 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L2141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L2142 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L2142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L2143 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L2143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L2144 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L2144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L2145 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L2145 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L2146 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L2146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L2147 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L2147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L2148 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L2148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L2149 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L2149 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L2150 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L2150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L2151 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L2151 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L2152 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L2152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L2153 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L2153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L2154 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L2154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L2155 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L2155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L2156 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns 1 if the resulting mask is non-zero, otherwise, returns 0.`.
  **L2156 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns 1 if the resulting mask is non-zero, otherwise, returns 0.`。
- **L2157 EN**: Defines macro `_mm_cmpestrc(A, LA, B, LB, M)` for conditional compilation, shorthand, or API generation.
  **L2157 CN**: 定义宏 `_mm_cmpestrc(A, LA, B, LB, M)`，用于条件编译、简写或 API 生成。
- **L2158 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpestric128`.
  **L2158 CN**: 继续与可调用符号 `__builtin_ia32_pcmpestric128` 相关的逻辑。
- **L2159 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(LB),         \`.
  **L2159 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(LB),         \`。
- **L2160 EN**: Continues the surrounding expression or declaration: `(int)(M)))`.
  **L2160 CN**: 继续构造周围的表达式或声明：`(int)(M)))`。

### Lines 2161-2184

````c

/// Uses the immediate operand \a M to perform a comparison of string
///    data with explicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns bit 0 of the resulting bit mask.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpestro(__m128i A, int LA, __m128i B, int LB, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LA
///    An integer that specifies the length of the string in \a A.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LB
///    An integer that specifies the length of the string in \a B.
````
- **L2161 EN**: Blank line separating nearby declarations or logic blocks.
  **L2161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2162 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L2162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L2163 EN**: Comment explains nearby logic, constraints, or intent: `data with explicitly defined lengths that is contained in source operands`.
  **L2163 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with explicitly defined lengths that is contained in source operands`。
- **L2164 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns bit 0 of the resulting bit mask.`.
  **L2164 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns bit 0 of the resulting bit mask.`。
- **L2165 EN**: Separator comment used for visual grouping.
  **L2165 CN**: 用于视觉分组的分隔注释。
- **L2166 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L2166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L2167 EN**: Separator comment used for visual grouping.
  **L2167 CN**: 用于视觉分组的分隔注释。
- **L2168 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L2168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L2169 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpestro(__m128i A, int LA, __m128i B, int LB, const int M);`.
  **L2169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpestro(__m128i A, int LA, __m128i B, int LB, const int M);`。
- **L2170 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2170 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2171 EN**: Separator comment used for visual grouping.
  **L2171 CN**: 用于视觉分组的分隔注释。
- **L2172 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`.
  **L2172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`。
- **L2173 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L2173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L2174 EN**: Separator comment used for visual grouping.
  **L2174 CN**: 用于视觉分组的分隔注释。
- **L2175 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L2175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L2176 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2176 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2177 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2178 EN**: Comment explains nearby logic, constraints, or intent: `param LA`.
  **L2178 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LA`。
- **L2179 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a A.`.
  **L2179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a A.`。
- **L2180 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L2180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L2181 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2182 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2183 EN**: Comment explains nearby logic, constraints, or intent: `param LB`.
  **L2183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LB`。
- **L2184 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a B.`.
  **L2184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a B.`。

### Lines 2185-2208

````c
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B.
````
- **L2185 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L2185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L2186 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L2186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L2187 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L2187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L2188 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L2188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L2189 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L2189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L2190 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L2190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L2191 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L2191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L2192 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L2192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L2193 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L2193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L2194 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L2194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L2195 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L2195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L2196 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L2196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L2197 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L2197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L2198 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L2198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L2199 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L2199 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L2200 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L2200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L2201 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L2201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L2202 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L2202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L2203 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L2203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L2204 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L2204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L2205 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L2205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L2206 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L2206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L2207 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L2207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L2208 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B.`.
  **L2208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B.`。

### Lines 2209-2232

````c
/// \returns Returns bit 0 of the resulting bit mask.
#define _mm_cmpestro(A, LA, B, LB, M)                                          \
  ((int)__builtin_ia32_pcmpestrio128((__v16qi)(__m128i)(A), (int)(LA),         \
                                     (__v16qi)(__m128i)(B), (int)(LB),         \
                                     (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with explicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns 1 if the length of the string in \a A is less than
///    the maximum, otherwise, returns 0.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpestrs(__m128i A, int LA, __m128i B, int LB, const int M);
/// \endcode
///
/// This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>
/// instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LA
````
- **L2209 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns bit 0 of the resulting bit mask.`.
  **L2209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns bit 0 of the resulting bit mask.`。
- **L2210 EN**: Defines macro `_mm_cmpestro(A, LA, B, LB, M)` for conditional compilation, shorthand, or API generation.
  **L2210 CN**: 定义宏 `_mm_cmpestro(A, LA, B, LB, M)`，用于条件编译、简写或 API 生成。
- **L2211 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpestrio128`.
  **L2211 CN**: 继续与可调用符号 `__builtin_ia32_pcmpestrio128` 相关的逻辑。
- **L2212 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(LB),         \`.
  **L2212 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(LB),         \`。
- **L2213 EN**: Continues the surrounding expression or declaration: `(int)(M)))`.
  **L2213 CN**: 继续构造周围的表达式或声明：`(int)(M)))`。
- **L2214 EN**: Blank line separating nearby declarations or logic blocks.
  **L2214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2215 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L2215 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L2216 EN**: Comment explains nearby logic, constraints, or intent: `data with explicitly defined lengths that is contained in source operands`.
  **L2216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with explicitly defined lengths that is contained in source operands`。
- **L2217 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns 1 if the length of the string in a A is less than`.
  **L2217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns 1 if the length of the string in a A is less than`。
- **L2218 EN**: Comment explains nearby logic, constraints, or intent: `the maximum, otherwise, returns 0.`.
  **L2218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the maximum, otherwise, returns 0.`。
- **L2219 EN**: Separator comment used for visual grouping.
  **L2219 CN**: 用于视觉分组的分隔注释。
- **L2220 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L2220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L2221 EN**: Separator comment used for visual grouping.
  **L2221 CN**: 用于视觉分组的分隔注释。
- **L2222 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L2222 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L2223 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpestrs(__m128i A, int LA, __m128i B, int LB, const int M);`.
  **L2223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpestrs(__m128i A, int LA, __m128i B, int LB, const int M);`。
- **L2224 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2225 EN**: Separator comment used for visual grouping.
  **L2225 CN**: 用于视觉分组的分隔注释。
- **L2226 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`.
  **L2226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPESTRI / PCMPESTRI </c>`。
- **L2227 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L2227 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L2228 EN**: Separator comment used for visual grouping.
  **L2228 CN**: 用于视觉分组的分隔注释。
- **L2229 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L2229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L2230 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2231 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2232 EN**: Comment explains nearby logic, constraints, or intent: `param LA`.
  **L2232 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LA`。

### Lines 2233-2256

````c
///    An integer that specifies the length of the string in \a A.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LB
///    An integer that specifies the length of the string in \a B.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement in the bit
````
- **L2233 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a A.`.
  **L2233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a A.`。
- **L2234 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L2234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L2235 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2236 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2237 EN**: Comment explains nearby logic, constraints, or intent: `param LB`.
  **L2237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LB`。
- **L2238 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a B.`.
  **L2238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a B.`。
- **L2239 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L2239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L2240 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L2240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L2241 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L2241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L2242 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L2242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L2243 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L2243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L2244 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L2244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L2245 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L2245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L2246 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L2246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L2247 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L2247 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L2248 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L2248 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L2249 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L2249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L2250 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L2250 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。
- **L2251 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L2251 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L2252 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L2252 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L2253 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L2253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L2254 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L2254 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L2255 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L2255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L2256 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement in the bit`.
  **L2256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement in the bit`。

### Lines 2257-2280

````c
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B. \n
/// \returns Returns 1 if the length of the string in \a A is less than the
///    maximum, otherwise, returns 0.
#define _mm_cmpestrs(A, LA, B, LB, M)                                          \
  ((int)__builtin_ia32_pcmpestris128((__v16qi)(__m128i)(A), (int)(LA),         \
                                     (__v16qi)(__m128i)(B), (int)(LB),         \
                                     (int)(M)))

/// Uses the immediate operand \a M to perform a comparison of string
///    data with explicitly defined lengths that is contained in source operands
///    \a A and \a B. Returns 1 if the length of the string in \a B is less than
///    the maximum, otherwise, returns 0.
///
/// \headerfile <x86intrin.h>
///
/// \code
/// int _mm_cmpestrz(__m128i A, int LA, __m128i B, int LB, const int M);
/// \endcode
///
````
- **L2257 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L2257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L2258 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L2258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L2259 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L2259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L2260 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L2260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L2261 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L2261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L2262 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B. n`.
  **L2262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B. n`。
- **L2263 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns 1 if the length of the string in a A is less than the`.
  **L2263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns 1 if the length of the string in a A is less than the`。
- **L2264 EN**: Comment explains nearby logic, constraints, or intent: `maximum, otherwise, returns 0.`.
  **L2264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`maximum, otherwise, returns 0.`。
- **L2265 EN**: Defines macro `_mm_cmpestrs(A, LA, B, LB, M)` for conditional compilation, shorthand, or API generation.
  **L2265 CN**: 定义宏 `_mm_cmpestrs(A, LA, B, LB, M)`，用于条件编译、简写或 API 生成。
- **L2266 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpestris128`.
  **L2266 CN**: 继续与可调用符号 `__builtin_ia32_pcmpestris128` 相关的逻辑。
- **L2267 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(LB),         \`.
  **L2267 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(LB),         \`。
- **L2268 EN**: Continues the surrounding expression or declaration: `(int)(M)))`.
  **L2268 CN**: 继续构造周围的表达式或声明：`(int)(M)))`。
- **L2269 EN**: Blank line separating nearby declarations or logic blocks.
  **L2269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2270 EN**: Comment explains nearby logic, constraints, or intent: `Uses the immediate operand a M to perform a comparison of string`.
  **L2270 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Uses the immediate operand a M to perform a comparison of string`。
- **L2271 EN**: Comment explains nearby logic, constraints, or intent: `data with explicitly defined lengths that is contained in source operands`.
  **L2271 CN**: 注释解释附近代码的逻辑、约束或设计意图：`data with explicitly defined lengths that is contained in source operands`。
- **L2272 EN**: Comment explains nearby logic, constraints, or intent: `a A and a B. Returns 1 if the length of the string in a B is less than`.
  **L2272 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a A and a B. Returns 1 if the length of the string in a B is less than`。
- **L2273 EN**: Comment explains nearby logic, constraints, or intent: `the maximum, otherwise, returns 0.`.
  **L2273 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the maximum, otherwise, returns 0.`。
- **L2274 EN**: Separator comment used for visual grouping.
  **L2274 CN**: 用于视觉分组的分隔注释。
- **L2275 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L2275 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L2276 EN**: Separator comment used for visual grouping.
  **L2276 CN**: 用于视觉分组的分隔注释。
- **L2277 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L2277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L2278 EN**: Comment explains nearby logic, constraints, or intent: `int _mm_cmpestrz(__m128i A, int LA, __m128i B, int LB, const int M);`.
  **L2278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`int _mm_cmpestrz(__m128i A, int LA, __m128i B, int LB, const int M);`。
- **L2279 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2280 EN**: Separator comment used for visual grouping.
  **L2280 CN**: 用于视觉分组的分隔注释。

### Lines 2281-2304

````c
/// This intrinsic corresponds to the <c> VPCMPESTRI </c> instruction.
///
/// \param A
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LA
///    An integer that specifies the length of the string in \a A.
/// \param B
///    A 128-bit integer vector containing one of the source operands to be
///    compared.
/// \param LB
///    An integer that specifies the length of the string in \a B.
/// \param M
///    An 8-bit immediate operand specifying whether the characters are bytes or
///    words and the type of comparison to perform. \n
///    Bits [1:0]: Determine source data format. \n
///      00: 16 unsigned bytes  \n
///      01: 8 unsigned words \n
///      10: 16 signed bytes \n
///      11: 8 signed words \n
///    Bits [3:2]: Determine comparison type and aggregation method. \n
///      00: Subset: Each character in \a B is compared for equality with all
///          the characters in \a A. \n
///      01: Ranges: Each character in \a B is compared to \a A. The comparison
````
- **L2281 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPESTRI </c> instruction.`.
  **L2281 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPESTRI </c> instruction.`。
- **L2282 EN**: Separator comment used for visual grouping.
  **L2282 CN**: 用于视觉分组的分隔注释。
- **L2283 EN**: Comment explains nearby logic, constraints, or intent: `param A`.
  **L2283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param A`。
- **L2284 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2285 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2286 EN**: Comment explains nearby logic, constraints, or intent: `param LA`.
  **L2286 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LA`。
- **L2287 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a A.`.
  **L2287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a A.`。
- **L2288 EN**: Comment explains nearby logic, constraints, or intent: `param B`.
  **L2288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param B`。
- **L2289 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector containing one of the source operands to be`.
  **L2289 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector containing one of the source operands to be`。
- **L2290 EN**: Comment explains nearby logic, constraints, or intent: `compared.`.
  **L2290 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compared.`。
- **L2291 EN**: Comment explains nearby logic, constraints, or intent: `param LB`.
  **L2291 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LB`。
- **L2292 EN**: Comment explains nearby logic, constraints, or intent: `An integer that specifies the length of the string in a B.`.
  **L2292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An integer that specifies the length of the string in a B.`。
- **L2293 EN**: Comment explains nearby logic, constraints, or intent: `param M`.
  **L2293 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param M`。
- **L2294 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit immediate operand specifying whether the characters are bytes or`.
  **L2294 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit immediate operand specifying whether the characters are bytes or`。
- **L2295 EN**: Comment explains nearby logic, constraints, or intent: `words and the type of comparison to perform. n`.
  **L2295 CN**: 注释解释附近代码的逻辑、约束或设计意图：`words and the type of comparison to perform. n`。
- **L2296 EN**: Comment explains nearby logic, constraints, or intent: `Bits [1:0]: Determine source data format. n`.
  **L2296 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [1:0]: Determine source data format. n`。
- **L2297 EN**: Comment explains nearby logic, constraints, or intent: `00: 16 unsigned bytes n`.
  **L2297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: 16 unsigned bytes n`。
- **L2298 EN**: Comment explains nearby logic, constraints, or intent: `01: 8 unsigned words n`.
  **L2298 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: 8 unsigned words n`。
- **L2299 EN**: Comment explains nearby logic, constraints, or intent: `10: 16 signed bytes n`.
  **L2299 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: 16 signed bytes n`。
- **L2300 EN**: Comment explains nearby logic, constraints, or intent: `11: 8 signed words n`.
  **L2300 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: 8 signed words n`。
- **L2301 EN**: Comment explains nearby logic, constraints, or intent: `Bits [3:2]: Determine comparison type and aggregation method. n`.
  **L2301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [3:2]: Determine comparison type and aggregation method. n`。
- **L2302 EN**: Comment explains nearby logic, constraints, or intent: `00: Subset: Each character in a B is compared for equality with all`.
  **L2302 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: Subset: Each character in a B is compared for equality with all`。
- **L2303 EN**: Comment explains nearby logic, constraints, or intent: `the characters in a A. n`.
  **L2303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the characters in a A. n`。
- **L2304 EN**: Comment explains nearby logic, constraints, or intent: `01: Ranges: Each character in a B is compared to a A. The comparison`.
  **L2304 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Ranges: Each character in a B is compared to a A. The comparison`。

### Lines 2305-2328

````c
///          basis is greater than or equal for even-indexed elements in \a A,
///          and less than or equal for odd-indexed elements in \a A. \n
///      10: Match: Compare each pair of corresponding characters in \a A and
///          \a B for equality. \n
///      11: Substring: Search \a B for substring matches of \a A. \n
///    Bits [5:4]: Determine whether to perform a one's complement on the bit
///                mask of the comparison results. \n
///      00: No effect. \n
///      01: Negate the bit mask. \n
///      10: No effect. \n
///      11: Negate the bit mask only for bits with an index less than or equal
///          to the size of \a A or \a B.
/// \returns Returns 1 if the length of the string in \a B is less than the
///    maximum, otherwise, returns 0.
#define _mm_cmpestrz(A, LA, B, LB, M)                                          \
  ((int)__builtin_ia32_pcmpestriz128((__v16qi)(__m128i)(A), (int)(LA),         \
                                     (__v16qi)(__m128i)(B), (int)(LB),         \
                                     (int)(M)))

/* SSE4.2 Compare Packed Data -- Greater Than.  */
/// Compares each of the corresponding 64-bit values of the 128-bit
///    integer vectors to determine if the values in the first operand are
///    greater than those in the second operand.
///
````
- **L2305 EN**: Comment explains nearby logic, constraints, or intent: `basis is greater than or equal for even-indexed elements in a A,`.
  **L2305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`basis is greater than or equal for even-indexed elements in a A,`。
- **L2306 EN**: Comment explains nearby logic, constraints, or intent: `and less than or equal for odd-indexed elements in a A. n`.
  **L2306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and less than or equal for odd-indexed elements in a A. n`。
- **L2307 EN**: Comment explains nearby logic, constraints, or intent: `10: Match: Compare each pair of corresponding characters in a A and`.
  **L2307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: Match: Compare each pair of corresponding characters in a A and`。
- **L2308 EN**: Comment explains nearby logic, constraints, or intent: `a B for equality. n`.
  **L2308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a B for equality. n`。
- **L2309 EN**: Comment explains nearby logic, constraints, or intent: `11: Substring: Search a B for substring matches of a A. n`.
  **L2309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Substring: Search a B for substring matches of a A. n`。
- **L2310 EN**: Comment explains nearby logic, constraints, or intent: `Bits [5:4]: Determine whether to perform a one's complement on the bit`.
  **L2310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [5:4]: Determine whether to perform a one's complement on the bit`。
- **L2311 EN**: Comment explains nearby logic, constraints, or intent: `mask of the comparison results. n`.
  **L2311 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mask of the comparison results. n`。
- **L2312 EN**: Comment explains nearby logic, constraints, or intent: `00: No effect. n`.
  **L2312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`00: No effect. n`。
- **L2313 EN**: Comment explains nearby logic, constraints, or intent: `01: Negate the bit mask. n`.
  **L2313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`01: Negate the bit mask. n`。
- **L2314 EN**: Comment explains nearby logic, constraints, or intent: `10: No effect. n`.
  **L2314 CN**: 注释解释附近代码的逻辑、约束或设计意图：`10: No effect. n`。
- **L2315 EN**: Comment explains nearby logic, constraints, or intent: `11: Negate the bit mask only for bits with an index less than or equal`.
  **L2315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11: Negate the bit mask only for bits with an index less than or equal`。
- **L2316 EN**: Comment explains nearby logic, constraints, or intent: `to the size of a A or a B.`.
  **L2316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the size of a A or a B.`。
- **L2317 EN**: Comment explains nearby logic, constraints, or intent: `returns Returns 1 if the length of the string in a B is less than the`.
  **L2317 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns Returns 1 if the length of the string in a B is less than the`。
- **L2318 EN**: Comment explains nearby logic, constraints, or intent: `maximum, otherwise, returns 0.`.
  **L2318 CN**: 注释解释附近代码的逻辑、约束或设计意图：`maximum, otherwise, returns 0.`。
- **L2319 EN**: Defines macro `_mm_cmpestrz(A, LA, B, LB, M)` for conditional compilation, shorthand, or API generation.
  **L2319 CN**: 定义宏 `_mm_cmpestrz(A, LA, B, LB, M)`，用于条件编译、简写或 API 生成。
- **L2320 EN**: Continues logic associated with callable symbol `__builtin_ia32_pcmpestriz128`.
  **L2320 CN**: 继续与可调用符号 `__builtin_ia32_pcmpestriz128` 相关的逻辑。
- **L2321 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(LB),         \`.
  **L2321 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(LB),         \`。
- **L2322 EN**: Continues the surrounding expression or declaration: `(int)(M)))`.
  **L2322 CN**: 继续构造周围的表达式或声明：`(int)(M)))`。
- **L2323 EN**: Blank line separating nearby declarations or logic blocks.
  **L2323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2324 EN**: Comment explains nearby logic, constraints, or intent: `SSE4.2 Compare Packed Data Greater Than.`.
  **L2324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE4.2 Compare Packed Data Greater Than.`。
- **L2325 EN**: Comment explains nearby logic, constraints, or intent: `Compares each of the corresponding 64-bit values of the 128-bit`.
  **L2325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares each of the corresponding 64-bit values of the 128-bit`。
- **L2326 EN**: Comment explains nearby logic, constraints, or intent: `integer vectors to determine if the values in the first operand are`.
  **L2326 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vectors to determine if the values in the first operand are`。
- **L2327 EN**: Comment explains nearby logic, constraints, or intent: `greater than those in the second operand.`.
  **L2327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`greater than those in the second operand.`。
- **L2328 EN**: Separator comment used for visual grouping.
  **L2328 CN**: 用于视觉分组的分隔注释。

### Lines 2329-2352

````c
///    Each comparison returns 0x0 for false, 0xFFFFFFFFFFFFFFFF for true.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> VPCMPGTQ / PCMPGTQ </c> instruction.
///
/// \param __V1
///    A 128-bit integer vector.
/// \param __V2
///    A 128-bit integer vector.
/// \returns A 128-bit integer vector containing the comparison results.
static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cmpgt_epi64(__m128i __V1, __m128i __V2) {
  return (__m128i)((__v2di)__V1 > (__v2di)__V2);
}

#undef __DEFAULT_FN_ATTRS
#undef __DEFAULT_FN_ATTRS_CONSTEXPR

#include <popcntintrin.h>

#include <crc32intrin.h>

#endif /* __SMMINTRIN_H */
````
- **L2329 EN**: Comment explains nearby logic, constraints, or intent: `Each comparison returns 0x0 for false, 0xFFFFFFFFFFFFFFFF for true.`.
  **L2329 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each comparison returns 0x0 for false, 0xFFFFFFFFFFFFFFFF for true.`。
- **L2330 EN**: Separator comment used for visual grouping.
  **L2330 CN**: 用于视觉分组的分隔注释。
- **L2331 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L2331 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L2332 EN**: Separator comment used for visual grouping.
  **L2332 CN**: 用于视觉分组的分隔注释。
- **L2333 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> VPCMPGTQ / PCMPGTQ </c> instruction.`.
  **L2333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> VPCMPGTQ / PCMPGTQ </c> instruction.`。
- **L2334 EN**: Separator comment used for visual grouping.
  **L2334 CN**: 用于视觉分组的分隔注释。
- **L2335 EN**: Comment explains nearby logic, constraints, or intent: `param __V1`.
  **L2335 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V1`。
- **L2336 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L2336 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。
- **L2337 EN**: Comment explains nearby logic, constraints, or intent: `param __V2`.
  **L2337 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __V2`。
- **L2338 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit integer vector.`.
  **L2338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit integer vector.`。
- **L2339 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit integer vector containing the comparison results.`.
  **L2339 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit integer vector containing the comparison results.`。
- **L2340 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L2340 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L2341 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_epi64(__m128i __V1, __m128i __V2) {`.
  **L2341 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_epi64(__m128i __V1, __m128i __V2) {`。
- **L2342 EN**: Returns from the current function with `(__m128i)((__v2di)__V1 > (__v2di)__V2)`.
  **L2342 CN**: 以 `(__m128i)((__v2di)__V1 > (__v2di)__V2)` 从当前函数返回。
- **L2343 EN**: Closes the current lexical scope or compound statement.
  **L2343 CN**: 结束当前词法作用域或复合语句块。
- **L2344 EN**: Blank line separating nearby declarations or logic blocks.
  **L2344 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2345 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS`.
  **L2345 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS`。
- **L2346 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L2346 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L2347 EN**: Blank line separating nearby declarations or logic blocks.
  **L2347 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2348 EN**: Includes <popcntintrin.h> to access related header declarations.
  **L2348 CN**: 引入 <popcntintrin.h> 以使用相关头文件声明。
- **L2349 EN**: Blank line separating nearby declarations or logic blocks.
  **L2349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2350 EN**: Includes <crc32intrin.h> to access related header declarations.
  **L2350 CN**: 引入 <crc32intrin.h> 以使用相关头文件声明。
- **L2351 EN**: Blank line separating nearby declarations or logic blocks.
  **L2351 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2352 EN**: Closes the current preprocessor conditional block.
  **L2352 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **Header composition through includes / 通过 include 组合头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Inline wrapper functions / 内联包装函数**
- **Compiler-specific attributes / 编译器特定属性**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `tmmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
  - `popcntintrin.h`: Provides related header declarations. / 提供相关头文件声明。
  - `crc32intrin.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `__SMMINTRIN_H`, `__i386__`, `__x86_64__`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_ia32_roundps`, `__builtin_ia32_roundss`, `__builtin_ia32_roundpd`, `__builtin_ia32_roundsd`, `__builtin_ia32_blendpd`, `__builtin_ia32_blendps`, `__builtin_ia32_blendvpd`, `__builtin_ia32_blendvps`, `__builtin_ia32_pblendvb128`, `__builtin_ia32_pblendw128`, `__builtin_ia32_pmuldq128`, `__builtin_ia32_dpps`
