# fmaintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/fmaintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: FMA intrinsics.
- **Purpose (CN)**: 提供 FMA intrinsic 接口。
- **Line Count / 行数**: 826

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- fmaintrin.h - FMA intrinsics -------------------------------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __IMMINTRIN_H
#error "Never use <fmaintrin.h> directly; include <immintrin.h> instead."
#endif

#ifndef __FMAINTRIN_H
#define __FMAINTRIN_H

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS128                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("fma"),            \
                 __min_vector_width__(128)))
#define __DEFAULT_FN_ATTRS256                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("fma"),            \
                 __min_vector_width__(256)))

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
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <fmaintrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error "Never use <fmaintrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifndef __FMAINTRIN_H`.
  **L14 CN**: 开始一个预处理条件块：`#ifndef __FMAINTRIN_H`。
- **L15 EN**: Defines macro `__FMAINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `__FMAINTRIN_H`，用于条件编译、简写或 API 生成。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L18 EN**: Defines macro `__DEFAULT_FN_ATTRS128` for conditional compilation, shorthand, or API generation.
  **L18 CN**: 定义宏 `__DEFAULT_FN_ATTRS128`，用于条件编译、简写或 API 生成。
- **L19 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("fma"),            \`.
  **L19 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("fma"),            \`。
- **L20 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L20 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L21 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L21 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L22 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("fma"),            \`.
  **L22 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("fma"),            \`。
- **L23 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L23 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
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

/// Computes a multiply-add of 128-bit vectors of [4 x float].
///    For each element, computes <c> (__A * __B) + __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADD213PS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier.
/// \param __C
///    A 128-bit vector of [4 x float] containing the addend.
/// \returns A 128-bit vector of [4 x float] containing the result.
static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fmadd_ps(__m128 __A, __m128 __B, __m128 __C)
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
- **L33 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply-add of 128-bit vectors of [4 x float].`.
  **L33 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply-add of 128-bit vectors of [4 x float].`。
- **L34 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> (__A * __B) + __C </c>.`.
  **L34 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> (__A * __B) + __C </c>.`。
- **L35 EN**: Separator comment used for visual grouping.
  **L35 CN**: 用于视觉分组的分隔注释。
- **L36 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L36 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L37 EN**: Separator comment used for visual grouping.
  **L37 CN**: 用于视觉分组的分隔注释。
- **L38 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADD213PS instruction.`.
  **L38 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADD213PS instruction.`。
- **L39 EN**: Separator comment used for visual grouping.
  **L39 CN**: 用于视觉分组的分隔注释。
- **L40 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L40 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L41 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand.`.
  **L41 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand.`。
- **L42 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L42 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L43 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier.`.
  **L43 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier.`。
- **L44 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L44 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L45 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the addend.`.
  **L45 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the addend.`。
- **L46 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result.`.
  **L46 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result.`。
- **L47 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L47 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L48 EN**: Continues logic associated with callable symbol `_mm_fmadd_ps`.
  **L48 CN**: 继续与可调用符号 `_mm_fmadd_ps` 相关的逻辑。

### Lines 49-72

````c
{
  return (__m128)__builtin_elementwise_fma((__v4sf)__A, (__v4sf)__B,
                                           (__v4sf)__C);
}

/// Computes a multiply-add of 128-bit vectors of [2 x double].
///    For each element, computes <c> (__A * __B) + __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADD213PD instruction.
///
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier.
/// \param __C
///    A 128-bit vector of [2 x double] containing the addend.
/// \returns A 128-bit [2 x double] vector containing the result.
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fmadd_pd(__m128d __A, __m128d __B, __m128d __C)
{
  return (__m128d)__builtin_elementwise_fma((__v2df)__A, (__v2df)__B,
                                            (__v2df)__C);
````
- **L49 EN**: Opens a new lexical scope or compound statement.
  **L49 CN**: 打开一个新的词法作用域或复合语句块。
- **L50 EN**: Returns from the current function with `(__m128)__builtin_elementwise_fma((__v4sf)__A, (__v4sf)__B,`.
  **L50 CN**: 以 `(__m128)__builtin_elementwise_fma((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L51 EN**: Executes a call or declaration centered on `statement`.
  **L51 CN**: 执行以 `statement` 为核心的调用或声明。
- **L52 EN**: Closes the current lexical scope or compound statement.
  **L52 CN**: 结束当前词法作用域或复合语句块。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L54 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply-add of 128-bit vectors of [2 x double].`.
  **L54 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply-add of 128-bit vectors of [2 x double].`。
- **L55 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> (__A * __B) + __C </c>.`.
  **L55 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> (__A * __B) + __C </c>.`。
- **L56 EN**: Separator comment used for visual grouping.
  **L56 CN**: 用于视觉分组的分隔注释。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L58 EN**: Separator comment used for visual grouping.
  **L58 CN**: 用于视觉分组的分隔注释。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADD213PD instruction.`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADD213PD instruction.`。
- **L60 EN**: Separator comment used for visual grouping.
  **L60 CN**: 用于视觉分组的分隔注释。
- **L61 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L61 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L62 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand.`.
  **L62 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand.`。
- **L63 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L63 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier.`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier.`。
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L66 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the addend.`.
  **L66 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the addend.`。
- **L67 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit [2 x double] vector containing the result.`.
  **L67 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit [2 x double] vector containing the result.`。
- **L68 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L68 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L69 EN**: Continues logic associated with callable symbol `_mm_fmadd_pd`.
  **L69 CN**: 继续与可调用符号 `_mm_fmadd_pd` 相关的逻辑。
- **L70 EN**: Opens a new lexical scope or compound statement.
  **L70 CN**: 打开一个新的词法作用域或复合语句块。
- **L71 EN**: Returns from the current function with `(__m128d)__builtin_elementwise_fma((__v2df)__A, (__v2df)__B,`.
  **L71 CN**: 以 `(__m128d)__builtin_elementwise_fma((__v2df)__A, (__v2df)__B,` 从当前函数返回。
- **L72 EN**: Executes a call or declaration centered on `statement`.
  **L72 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 73-96

````c
}

/// Computes a scalar multiply-add of the single-precision values in the
///    low 32 bits of 128-bit vectors of [4 x float].
///
/// \code{.operation}
/// result[31:0] = (__A[31:0] * __B[31:0]) + __C[31:0]
/// result[127:32] = __A[127:32]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADD213SS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand in the low
///    32 bits.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier in the low
///    32 bits.
/// \param __C
///    A 128-bit vector of [4 x float] containing the addend in the low
///    32 bits.
/// \returns A 128-bit vector of [4 x float] containing the result in the low
````
- **L73 EN**: Closes the current lexical scope or compound statement.
  **L73 CN**: 结束当前词法作用域或复合语句块。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `Computes a scalar multiply-add of the single-precision values in the`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a scalar multiply-add of the single-precision values in the`。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `low 32 bits of 128-bit vectors of [4 x float].`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`low 32 bits of 128-bit vectors of [4 x float].`。
- **L77 EN**: Separator comment used for visual grouping.
  **L77 CN**: 用于视觉分组的分隔注释。
- **L78 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L78 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L79 EN**: Comment explains nearby logic, constraints, or intent: `result[31:0] (__A[31:0] * __B[31:0]) + __C[31:0]`.
  **L79 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[31:0] (__A[31:0] * __B[31:0]) + __C[31:0]`。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `result[127:32] __A[127:32]`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:32] __A[127:32]`。
- **L81 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L81 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L82 EN**: Separator comment used for visual grouping.
  **L82 CN**: 用于视觉分组的分隔注释。
- **L83 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L83 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L84 EN**: Separator comment used for visual grouping.
  **L84 CN**: 用于视觉分组的分隔注释。
- **L85 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADD213SS instruction.`.
  **L85 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADD213SS instruction.`。
- **L86 EN**: Separator comment used for visual grouping.
  **L86 CN**: 用于视觉分组的分隔注释。
- **L87 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L87 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L88 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand in the low`.
  **L88 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand in the low`。
- **L89 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L89 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L90 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L90 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier in the low`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier in the low`。
- **L92 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L92 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L93 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L93 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L94 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the addend in the low`.
  **L94 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the addend in the low`。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result in the low`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result in the low`。

### Lines 97-120

````c
///    32 bits and a copy of \a __A[127:32] in the upper 96 bits.
static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fmadd_ss(__m128 __A, __m128 __B, __m128 __C) {
  __A[0] = __builtin_elementwise_fma(__A[0], __B[0], __C[0]);
  return __A;
}

/// Computes a scalar multiply-add of the double-precision values in the
///    low 64 bits of 128-bit vectors of [2 x double].
///
/// \code{.operation}
/// result[63:0] = (__A[63:0] * __B[63:0]) + __C[63:0]
/// result[127:64] = __A[127:64]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADD213SD instruction.
///
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand in the low
///    64 bits.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier in the low
````
- **L97 EN**: Comment explains nearby logic, constraints, or intent: `32 bits and a copy of a __A[127:32] in the upper 96 bits.`.
  **L97 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits and a copy of a __A[127:32] in the upper 96 bits.`。
- **L98 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L98 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L99 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_fmadd_ss(__m128 __A, __m128 __B, __m128 __C) {`.
  **L99 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_fmadd_ss(__m128 __A, __m128 __B, __m128 __C) {`。
- **L100 EN**: Executes a call or declaration centered on `__builtin_elementwise_fma`.
  **L100 CN**: 执行以 `__builtin_elementwise_fma` 为核心的调用或声明。
- **L101 EN**: Returns from the current function with `__A`.
  **L101 CN**: 以 `__A` 从当前函数返回。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L104 EN**: Comment explains nearby logic, constraints, or intent: `Computes a scalar multiply-add of the double-precision values in the`.
  **L104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a scalar multiply-add of the double-precision values in the`。
- **L105 EN**: Comment explains nearby logic, constraints, or intent: `low 64 bits of 128-bit vectors of [2 x double].`.
  **L105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`low 64 bits of 128-bit vectors of [2 x double].`。
- **L106 EN**: Separator comment used for visual grouping.
  **L106 CN**: 用于视觉分组的分隔注释。
- **L107 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L108 EN**: Comment explains nearby logic, constraints, or intent: `result[63:0] (__A[63:0] * __B[63:0]) + __C[63:0]`.
  **L108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:0] (__A[63:0] * __B[63:0]) + __C[63:0]`。
- **L109 EN**: Comment explains nearby logic, constraints, or intent: `result[127:64] __A[127:64]`.
  **L109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:64] __A[127:64]`。
- **L110 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L111 EN**: Separator comment used for visual grouping.
  **L111 CN**: 用于视觉分组的分隔注释。
- **L112 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L112 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L113 EN**: Separator comment used for visual grouping.
  **L113 CN**: 用于视觉分组的分隔注释。
- **L114 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADD213SD instruction.`.
  **L114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADD213SD instruction.`。
- **L115 EN**: Separator comment used for visual grouping.
  **L115 CN**: 用于视觉分组的分隔注释。
- **L116 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L117 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand in the low`.
  **L117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand in the low`。
- **L118 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L119 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L120 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier in the low`.
  **L120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier in the low`。

### Lines 121-144

````c
///    64 bits.
/// \param __C
///    A 128-bit vector of [2 x double] containing the addend in the low
///    64 bits.
/// \returns A 128-bit vector of [2 x double] containing the result in the low
///    64 bits and a copy of \a __A[127:64] in the upper 64 bits.
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fmadd_sd(__m128d __A, __m128d __B, __m128d __C) {
  __A[0] = __builtin_elementwise_fma(__A[0], __B[0], __C[0]);
  return __A;
}

/// Computes a multiply-subtract of 128-bit vectors of [4 x float].
///    For each element, computes <c> (__A * __B) - __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUB213PS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier.
/// \param __C
````
- **L121 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L122 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L123 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the addend in the low`.
  **L123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the addend in the low`。
- **L124 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L125 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result in the low`.
  **L125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result in the low`。
- **L126 EN**: Comment explains nearby logic, constraints, or intent: `64 bits and a copy of a __A[127:64] in the upper 64 bits.`.
  **L126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits and a copy of a __A[127:64] in the upper 64 bits.`。
- **L127 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L127 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L128 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_fmadd_sd(__m128d __A, __m128d __B, __m128d __C) {`.
  **L128 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_fmadd_sd(__m128d __A, __m128d __B, __m128d __C) {`。
- **L129 EN**: Executes a call or declaration centered on `__builtin_elementwise_fma`.
  **L129 CN**: 执行以 `__builtin_elementwise_fma` 为核心的调用或声明。
- **L130 EN**: Returns from the current function with `__A`.
  **L130 CN**: 以 `__A` 从当前函数返回。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L133 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply-subtract of 128-bit vectors of [4 x float].`.
  **L133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply-subtract of 128-bit vectors of [4 x float].`。
- **L134 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> (__A * __B) - __C </c>.`.
  **L134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> (__A * __B) - __C </c>.`。
- **L135 EN**: Separator comment used for visual grouping.
  **L135 CN**: 用于视觉分组的分隔注释。
- **L136 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L136 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L137 EN**: Separator comment used for visual grouping.
  **L137 CN**: 用于视觉分组的分隔注释。
- **L138 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUB213PS instruction.`.
  **L138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUB213PS instruction.`。
- **L139 EN**: Separator comment used for visual grouping.
  **L139 CN**: 用于视觉分组的分隔注释。
- **L140 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L141 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand.`.
  **L141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand.`。
- **L142 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier.`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier.`。
- **L144 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。

### Lines 145-168

````c
///    A 128-bit vector of [4 x float] containing the subtrahend.
/// \returns A 128-bit vector of [4 x float] containing the result.
static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fmsub_ps(__m128 __A, __m128 __B, __m128 __C)
{
  return (__m128)__builtin_elementwise_fma((__v4sf)__A, (__v4sf)__B,
                                           -(__v4sf)__C);
}

/// Computes a multiply-subtract of 128-bit vectors of [2 x double].
///    For each element, computes <c> (__A * __B) - __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUB213PD instruction.
///
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier.
/// \param __C
///    A 128-bit vector of [2 x double] containing the addend.
/// \returns A 128-bit vector of [2 x double] containing the result.
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
````
- **L145 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the subtrahend.`.
  **L145 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the subtrahend.`。
- **L146 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result.`.
  **L146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result.`。
- **L147 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L147 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L148 EN**: Continues logic associated with callable symbol `_mm_fmsub_ps`.
  **L148 CN**: 继续与可调用符号 `_mm_fmsub_ps` 相关的逻辑。
- **L149 EN**: Opens a new lexical scope or compound statement.
  **L149 CN**: 打开一个新的词法作用域或复合语句块。
- **L150 EN**: Returns from the current function with `(__m128)__builtin_elementwise_fma((__v4sf)__A, (__v4sf)__B,`.
  **L150 CN**: 以 `(__m128)__builtin_elementwise_fma((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L151 EN**: Executes a call or declaration centered on `-`.
  **L151 CN**: 执行以 `-` 为核心的调用或声明。
- **L152 EN**: Closes the current lexical scope or compound statement.
  **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L154 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply-subtract of 128-bit vectors of [2 x double].`.
  **L154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply-subtract of 128-bit vectors of [2 x double].`。
- **L155 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> (__A * __B) - __C </c>.`.
  **L155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> (__A * __B) - __C </c>.`。
- **L156 EN**: Separator comment used for visual grouping.
  **L156 CN**: 用于视觉分组的分隔注释。
- **L157 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L157 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L158 EN**: Separator comment used for visual grouping.
  **L158 CN**: 用于视觉分组的分隔注释。
- **L159 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUB213PD instruction.`.
  **L159 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUB213PD instruction.`。
- **L160 EN**: Separator comment used for visual grouping.
  **L160 CN**: 用于视觉分组的分隔注释。
- **L161 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L162 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand.`.
  **L162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand.`。
- **L163 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L163 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L164 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier.`.
  **L164 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier.`。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the addend.`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the addend.`。
- **L167 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result.`.
  **L167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result.`。
- **L168 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L168 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。

### Lines 169-192

````c
_mm_fmsub_pd(__m128d __A, __m128d __B, __m128d __C)
{
  return (__m128d)__builtin_elementwise_fma((__v2df)__A, (__v2df)__B,
                                            -(__v2df)__C);
}

/// Computes a scalar multiply-subtract of the single-precision values in
///    the low 32 bits of 128-bit vectors of [4 x float].
///
/// \code{.operation}
/// result[31:0] = (__A[31:0] * __B[31:0]) - __C[31:0]
/// result[127:32] = __A[127:32]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUB213SS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand in the low
///    32 bits.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier in the low
///    32 bits.
````
- **L169 EN**: Continues logic associated with callable symbol `_mm_fmsub_pd`.
  **L169 CN**: 继续与可调用符号 `_mm_fmsub_pd` 相关的逻辑。
- **L170 EN**: Opens a new lexical scope or compound statement.
  **L170 CN**: 打开一个新的词法作用域或复合语句块。
- **L171 EN**: Returns from the current function with `(__m128d)__builtin_elementwise_fma((__v2df)__A, (__v2df)__B,`.
  **L171 CN**: 以 `(__m128d)__builtin_elementwise_fma((__v2df)__A, (__v2df)__B,` 从当前函数返回。
- **L172 EN**: Executes a call or declaration centered on `-`.
  **L172 CN**: 执行以 `-` 为核心的调用或声明。
- **L173 EN**: Closes the current lexical scope or compound statement.
  **L173 CN**: 结束当前词法作用域或复合语句块。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L175 EN**: Comment explains nearby logic, constraints, or intent: `Computes a scalar multiply-subtract of the single-precision values in`.
  **L175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a scalar multiply-subtract of the single-precision values in`。
- **L176 EN**: Comment explains nearby logic, constraints, or intent: `the low 32 bits of 128-bit vectors of [4 x float].`.
  **L176 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the low 32 bits of 128-bit vectors of [4 x float].`。
- **L177 EN**: Separator comment used for visual grouping.
  **L177 CN**: 用于视觉分组的分隔注释。
- **L178 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L178 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L179 EN**: Comment explains nearby logic, constraints, or intent: `result[31:0] (__A[31:0] * __B[31:0]) - __C[31:0]`.
  **L179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[31:0] (__A[31:0] * __B[31:0]) - __C[31:0]`。
- **L180 EN**: Comment explains nearby logic, constraints, or intent: `result[127:32] __A[127:32]`.
  **L180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:32] __A[127:32]`。
- **L181 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L182 EN**: Separator comment used for visual grouping.
  **L182 CN**: 用于视觉分组的分隔注释。
- **L183 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L184 EN**: Separator comment used for visual grouping.
  **L184 CN**: 用于视觉分组的分隔注释。
- **L185 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUB213SS instruction.`.
  **L185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUB213SS instruction.`。
- **L186 EN**: Separator comment used for visual grouping.
  **L186 CN**: 用于视觉分组的分隔注释。
- **L187 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L188 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand in the low`.
  **L188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand in the low`。
- **L189 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L190 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L191 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier in the low`.
  **L191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier in the low`。
- **L192 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。

### Lines 193-216

````c
/// \param __C
///    A 128-bit vector of [4 x float] containing the subtrahend in the low
///   32 bits.
/// \returns A 128-bit vector of [4 x float] containing the result in the low
///    32 bits, and a copy of \a __A[127:32] in the upper 96 bits.
static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fmsub_ss(__m128 __A, __m128 __B, __m128 __C) {
  __A[0] = __builtin_elementwise_fma(__A[0], __B[0], -__C[0]);
  return __A;
}

/// Computes a scalar multiply-subtract of the double-precision values in
///    the low 64 bits of 128-bit vectors of [2 x double].
///
/// \code{.operation}
/// result[63:0] = (__A[63:0] * __B[63:0]) - __C[63:0]
/// result[127:64] = __A[127:64]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUB213SD instruction.
///
/// \param __A
````
- **L193 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the subtrahend in the low`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the subtrahend in the low`。
- **L195 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L196 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result in the low`.
  **L196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result in the low`。
- **L197 EN**: Comment explains nearby logic, constraints, or intent: `32 bits, and a copy of a __A[127:32] in the upper 96 bits.`.
  **L197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits, and a copy of a __A[127:32] in the upper 96 bits.`。
- **L198 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L198 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L199 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_fmsub_ss(__m128 __A, __m128 __B, __m128 __C) {`.
  **L199 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_fmsub_ss(__m128 __A, __m128 __B, __m128 __C) {`。
- **L200 EN**: Executes a call or declaration centered on `__builtin_elementwise_fma`.
  **L200 CN**: 执行以 `__builtin_elementwise_fma` 为核心的调用或声明。
- **L201 EN**: Returns from the current function with `__A`.
  **L201 CN**: 以 `__A` 从当前函数返回。
- **L202 EN**: Closes the current lexical scope or compound statement.
  **L202 CN**: 结束当前词法作用域或复合语句块。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L204 EN**: Comment explains nearby logic, constraints, or intent: `Computes a scalar multiply-subtract of the double-precision values in`.
  **L204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a scalar multiply-subtract of the double-precision values in`。
- **L205 EN**: Comment explains nearby logic, constraints, or intent: `the low 64 bits of 128-bit vectors of [2 x double].`.
  **L205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the low 64 bits of 128-bit vectors of [2 x double].`。
- **L206 EN**: Separator comment used for visual grouping.
  **L206 CN**: 用于视觉分组的分隔注释。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L208 EN**: Comment explains nearby logic, constraints, or intent: `result[63:0] (__A[63:0] * __B[63:0]) - __C[63:0]`.
  **L208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:0] (__A[63:0] * __B[63:0]) - __C[63:0]`。
- **L209 EN**: Comment explains nearby logic, constraints, or intent: `result[127:64] __A[127:64]`.
  **L209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:64] __A[127:64]`。
- **L210 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L211 EN**: Separator comment used for visual grouping.
  **L211 CN**: 用于视觉分组的分隔注释。
- **L212 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L213 EN**: Separator comment used for visual grouping.
  **L213 CN**: 用于视觉分组的分隔注释。
- **L214 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUB213SD instruction.`.
  **L214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUB213SD instruction.`。
- **L215 EN**: Separator comment used for visual grouping.
  **L215 CN**: 用于视觉分组的分隔注释。
- **L216 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。

### Lines 217-240

````c
///    A 128-bit vector of [2 x double] containing the multiplicand in the low
///    64 bits.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier in the low
///    64 bits.
/// \param __C
///    A 128-bit vector of [2 x double] containing the subtrahend in the low
///    64 bits.
/// \returns A 128-bit vector of [2 x double] containing the result in the low
///    64 bits, and a copy of \a __A[127:64] in the upper 64 bits.
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fmsub_sd(__m128d __A, __m128d __B, __m128d __C) {
  __A[0] = __builtin_elementwise_fma(__A[0], __B[0], -__C[0]);
  return __A;
}

/// Computes a negated multiply-add of 128-bit vectors of [4 x float].
///    For each element, computes <c> -(__A * __B) + __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFNMADD213DPS instruction.
///
/// \param __A
````
- **L217 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand in the low`.
  **L217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand in the low`。
- **L218 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L219 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L220 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier in the low`.
  **L220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier in the low`。
- **L221 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L222 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L222 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L223 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the subtrahend in the low`.
  **L223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the subtrahend in the low`。
- **L224 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L225 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result in the low`.
  **L225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result in the low`。
- **L226 EN**: Comment explains nearby logic, constraints, or intent: `64 bits, and a copy of a __A[127:64] in the upper 64 bits.`.
  **L226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits, and a copy of a __A[127:64] in the upper 64 bits.`。
- **L227 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L227 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L228 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_fmsub_sd(__m128d __A, __m128d __B, __m128d __C) {`.
  **L228 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_fmsub_sd(__m128d __A, __m128d __B, __m128d __C) {`。
- **L229 EN**: Executes a call or declaration centered on `__builtin_elementwise_fma`.
  **L229 CN**: 执行以 `__builtin_elementwise_fma` 为核心的调用或声明。
- **L230 EN**: Returns from the current function with `__A`.
  **L230 CN**: 以 `__A` 从当前函数返回。
- **L231 EN**: Closes the current lexical scope or compound statement.
  **L231 CN**: 结束当前词法作用域或复合语句块。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `Computes a negated multiply-add of 128-bit vectors of [4 x float].`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a negated multiply-add of 128-bit vectors of [4 x float].`。
- **L234 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> -(__A * __B) + __C </c>.`.
  **L234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> -(__A * __B) + __C </c>.`。
- **L235 EN**: Separator comment used for visual grouping.
  **L235 CN**: 用于视觉分组的分隔注释。
- **L236 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L237 EN**: Separator comment used for visual grouping.
  **L237 CN**: 用于视觉分组的分隔注释。
- **L238 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMADD213DPS instruction.`.
  **L238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMADD213DPS instruction.`。
- **L239 EN**: Separator comment used for visual grouping.
  **L239 CN**: 用于视觉分组的分隔注释。
- **L240 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。

### Lines 241-264

````c
///    A 128-bit vector of [4 x float] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier.
/// \param __C
///    A 128-bit vector of [4 x float] containing the addend.
/// \returns A 128-bit [4 x float] vector containing the result.
static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fnmadd_ps(__m128 __A, __m128 __B, __m128 __C)
{
  return (__m128)__builtin_elementwise_fma(-(__v4sf)__A, (__v4sf)__B,
                                           (__v4sf)__C);
}

/// Computes a negated multiply-add of 128-bit vectors of [2 x double].
///    For each element, computes <c> -(__A * __B) + __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFNMADD213PD instruction.
///
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier.
````
- **L241 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand.`.
  **L241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand.`。
- **L242 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L243 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier.`.
  **L243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier.`。
- **L244 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L245 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the addend.`.
  **L245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the addend.`。
- **L246 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit [4 x float] vector containing the result.`.
  **L246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit [4 x float] vector containing the result.`。
- **L247 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L247 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L248 EN**: Continues logic associated with callable symbol `_mm_fnmadd_ps`.
  **L248 CN**: 继续与可调用符号 `_mm_fnmadd_ps` 相关的逻辑。
- **L249 EN**: Opens a new lexical scope or compound statement.
  **L249 CN**: 打开一个新的词法作用域或复合语句块。
- **L250 EN**: Returns from the current function with `(__m128)__builtin_elementwise_fma(-(__v4sf)__A, (__v4sf)__B,`.
  **L250 CN**: 以 `(__m128)__builtin_elementwise_fma(-(__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L251 EN**: Executes a call or declaration centered on `statement`.
  **L251 CN**: 执行以 `statement` 为核心的调用或声明。
- **L252 EN**: Closes the current lexical scope or compound statement.
  **L252 CN**: 结束当前词法作用域或复合语句块。
- **L253 EN**: Blank line separating nearby declarations or logic blocks.
  **L253 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L254 EN**: Comment explains nearby logic, constraints, or intent: `Computes a negated multiply-add of 128-bit vectors of [2 x double].`.
  **L254 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a negated multiply-add of 128-bit vectors of [2 x double].`。
- **L255 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> -(__A * __B) + __C </c>.`.
  **L255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> -(__A * __B) + __C </c>.`。
- **L256 EN**: Separator comment used for visual grouping.
  **L256 CN**: 用于视觉分组的分隔注释。
- **L257 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L258 EN**: Separator comment used for visual grouping.
  **L258 CN**: 用于视觉分组的分隔注释。
- **L259 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMADD213PD instruction.`.
  **L259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMADD213PD instruction.`。
- **L260 EN**: Separator comment used for visual grouping.
  **L260 CN**: 用于视觉分组的分隔注释。
- **L261 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L262 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand.`.
  **L262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand.`。
- **L263 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L264 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier.`.
  **L264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier.`。

### Lines 265-288

````c
/// \param __C
///    A 128-bit vector of [2 x double] containing the addend.
/// \returns A 128-bit vector of [2 x double] containing the result.
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fnmadd_pd(__m128d __A, __m128d __B, __m128d __C)
{
  return (__m128d)__builtin_elementwise_fma(-(__v2df)__A, (__v2df)__B,
                                            (__v2df)__C);
}

/// Computes a scalar negated multiply-add of the single-precision values in
///    the low 32 bits of 128-bit vectors of [4 x float].
///
/// \code{.operation}
/// result[31:0] = -(__A[31:0] * __B[31:0]) + __C[31:0]
/// result[127:32] = __A[127:32]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFNMADD213SS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand in the low
````
- **L265 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L266 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the addend.`.
  **L266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the addend.`。
- **L267 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result.`.
  **L267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result.`。
- **L268 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L268 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L269 EN**: Continues logic associated with callable symbol `_mm_fnmadd_pd`.
  **L269 CN**: 继续与可调用符号 `_mm_fnmadd_pd` 相关的逻辑。
- **L270 EN**: Opens a new lexical scope or compound statement.
  **L270 CN**: 打开一个新的词法作用域或复合语句块。
- **L271 EN**: Returns from the current function with `(__m128d)__builtin_elementwise_fma(-(__v2df)__A, (__v2df)__B,`.
  **L271 CN**: 以 `(__m128d)__builtin_elementwise_fma(-(__v2df)__A, (__v2df)__B,` 从当前函数返回。
- **L272 EN**: Executes a call or declaration centered on `statement`.
  **L272 CN**: 执行以 `statement` 为核心的调用或声明。
- **L273 EN**: Closes the current lexical scope or compound statement.
  **L273 CN**: 结束当前词法作用域或复合语句块。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L275 EN**: Comment explains nearby logic, constraints, or intent: `Computes a scalar negated multiply-add of the single-precision values in`.
  **L275 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a scalar negated multiply-add of the single-precision values in`。
- **L276 EN**: Comment explains nearby logic, constraints, or intent: `the low 32 bits of 128-bit vectors of [4 x float].`.
  **L276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the low 32 bits of 128-bit vectors of [4 x float].`。
- **L277 EN**: Separator comment used for visual grouping.
  **L277 CN**: 用于视觉分组的分隔注释。
- **L278 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L279 EN**: Comment explains nearby logic, constraints, or intent: `result[31:0] -(__A[31:0] * __B[31:0]) + __C[31:0]`.
  **L279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[31:0] -(__A[31:0] * __B[31:0]) + __C[31:0]`。
- **L280 EN**: Comment explains nearby logic, constraints, or intent: `result[127:32] __A[127:32]`.
  **L280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:32] __A[127:32]`。
- **L281 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L281 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L282 EN**: Separator comment used for visual grouping.
  **L282 CN**: 用于视觉分组的分隔注释。
- **L283 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L284 EN**: Separator comment used for visual grouping.
  **L284 CN**: 用于视觉分组的分隔注释。
- **L285 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMADD213SS instruction.`.
  **L285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMADD213SS instruction.`。
- **L286 EN**: Separator comment used for visual grouping.
  **L286 CN**: 用于视觉分组的分隔注释。
- **L287 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L288 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand in the low`.
  **L288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand in the low`。

### Lines 289-312

````c
///    32 bits.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier in the low
///    32 bits.
/// \param __C
///    A 128-bit vector of [4 x float] containing the addend in the low
///    32 bits.
/// \returns A 128-bit vector of [4 x float] containing the result in the low
///    32 bits, and a copy of \a __A[127:32] in the upper 96 bits.
static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fnmadd_ss(__m128 __A, __m128 __B, __m128 __C) {
  __A[0] = __builtin_elementwise_fma(__A[0], -__B[0], __C[0]);
  return __A;
}

/// Computes a scalar negated multiply-add of the double-precision values
///    in the low 64 bits of 128-bit vectors of [2 x double].
///
/// \code{.operation}
/// result[63:0] = -(__A[63:0] * __B[63:0]) + __C[63:0]
/// result[127:64] = __A[127:64]
/// \endcode
///
/// \headerfile <immintrin.h>
````
- **L289 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L289 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L290 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L290 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L291 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier in the low`.
  **L291 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier in the low`。
- **L292 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L293 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L293 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L294 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the addend in the low`.
  **L294 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the addend in the low`。
- **L295 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L295 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L296 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result in the low`.
  **L296 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result in the low`。
- **L297 EN**: Comment explains nearby logic, constraints, or intent: `32 bits, and a copy of a __A[127:32] in the upper 96 bits.`.
  **L297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits, and a copy of a __A[127:32] in the upper 96 bits.`。
- **L298 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L298 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L299 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_fnmadd_ss(__m128 __A, __m128 __B, __m128 __C) {`.
  **L299 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_fnmadd_ss(__m128 __A, __m128 __B, __m128 __C) {`。
- **L300 EN**: Executes a call or declaration centered on `__builtin_elementwise_fma`.
  **L300 CN**: 执行以 `__builtin_elementwise_fma` 为核心的调用或声明。
- **L301 EN**: Returns from the current function with `__A`.
  **L301 CN**: 以 `__A` 从当前函数返回。
- **L302 EN**: Closes the current lexical scope or compound statement.
  **L302 CN**: 结束当前词法作用域或复合语句块。
- **L303 EN**: Blank line separating nearby declarations or logic blocks.
  **L303 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L304 EN**: Comment explains nearby logic, constraints, or intent: `Computes a scalar negated multiply-add of the double-precision values`.
  **L304 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a scalar negated multiply-add of the double-precision values`。
- **L305 EN**: Comment explains nearby logic, constraints, or intent: `in the low 64 bits of 128-bit vectors of [2 x double].`.
  **L305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`in the low 64 bits of 128-bit vectors of [2 x double].`。
- **L306 EN**: Separator comment used for visual grouping.
  **L306 CN**: 用于视觉分组的分隔注释。
- **L307 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L308 EN**: Comment explains nearby logic, constraints, or intent: `result[63:0] -(__A[63:0] * __B[63:0]) + __C[63:0]`.
  **L308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:0] -(__A[63:0] * __B[63:0]) + __C[63:0]`。
- **L309 EN**: Comment explains nearby logic, constraints, or intent: `result[127:64] __A[127:64]`.
  **L309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:64] __A[127:64]`。
- **L310 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L311 EN**: Separator comment used for visual grouping.
  **L311 CN**: 用于视觉分组的分隔注释。
- **L312 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。

### Lines 313-336

````c
///
/// This intrinsic corresponds to the \c VFNMADD213SD instruction.
///
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand in the low
///    64 bits.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier in the low
///    64 bits.
/// \param __C
///    A 128-bit vector of [2 x double] containing the addend in the low
///    64 bits.
/// \returns A 128-bit vector of [2 x double] containing the result in the low
///    64 bits, and a copy of \a __A[127:64] in the upper 64 bits.
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fnmadd_sd(__m128d __A, __m128d __B, __m128d __C) {
  __A[0] = __builtin_elementwise_fma(__A[0], -__B[0], __C[0]);
  return __A;
}

/// Computes a negated multiply-subtract of 128-bit vectors of [4 x float].
///    For each element, computes <c> -(__A * __B) - __C </c>.
///
/// \headerfile <immintrin.h>
````
- **L313 EN**: Separator comment used for visual grouping.
  **L313 CN**: 用于视觉分组的分隔注释。
- **L314 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMADD213SD instruction.`.
  **L314 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMADD213SD instruction.`。
- **L315 EN**: Separator comment used for visual grouping.
  **L315 CN**: 用于视觉分组的分隔注释。
- **L316 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L317 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand in the low`.
  **L317 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand in the low`。
- **L318 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L318 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L319 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L319 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L320 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier in the low`.
  **L320 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier in the low`。
- **L321 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L321 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L322 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L322 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L323 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the addend in the low`.
  **L323 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the addend in the low`。
- **L324 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L325 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result in the low`.
  **L325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result in the low`。
- **L326 EN**: Comment explains nearby logic, constraints, or intent: `64 bits, and a copy of a __A[127:64] in the upper 64 bits.`.
  **L326 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits, and a copy of a __A[127:64] in the upper 64 bits.`。
- **L327 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L327 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L328 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_fnmadd_sd(__m128d __A, __m128d __B, __m128d __C) {`.
  **L328 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_fnmadd_sd(__m128d __A, __m128d __B, __m128d __C) {`。
- **L329 EN**: Executes a call or declaration centered on `__builtin_elementwise_fma`.
  **L329 CN**: 执行以 `__builtin_elementwise_fma` 为核心的调用或声明。
- **L330 EN**: Returns from the current function with `__A`.
  **L330 CN**: 以 `__A` 从当前函数返回。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Comment explains nearby logic, constraints, or intent: `Computes a negated multiply-subtract of 128-bit vectors of [4 x float].`.
  **L333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a negated multiply-subtract of 128-bit vectors of [4 x float].`。
- **L334 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> -(__A * __B) - __C </c>.`.
  **L334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> -(__A * __B) - __C </c>.`。
- **L335 EN**: Separator comment used for visual grouping.
  **L335 CN**: 用于视觉分组的分隔注释。
- **L336 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L336 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。

### Lines 337-360

````c
///
/// This intrinsic corresponds to the \c VFNMSUB213PS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier.
/// \param __C
///    A 128-bit vector of [4 x float] containing the subtrahend.
/// \returns A 128-bit vector of [4 x float] containing the result.
static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fnmsub_ps(__m128 __A, __m128 __B, __m128 __C)
{
  return (__m128)__builtin_elementwise_fma(-(__v4sf)__A, (__v4sf)__B,
                                           -(__v4sf)__C);
}

/// Computes a negated multiply-subtract of 128-bit vectors of [2 x double].
///    For each element, computes <c> -(__A * __B) - __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFNMSUB213PD instruction.
///
````
- **L337 EN**: Separator comment used for visual grouping.
  **L337 CN**: 用于视觉分组的分隔注释。
- **L338 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMSUB213PS instruction.`.
  **L338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMSUB213PS instruction.`。
- **L339 EN**: Separator comment used for visual grouping.
  **L339 CN**: 用于视觉分组的分隔注释。
- **L340 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L340 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L341 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand.`.
  **L341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand.`。
- **L342 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L343 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier.`.
  **L343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier.`。
- **L344 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L345 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the subtrahend.`.
  **L345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the subtrahend.`。
- **L346 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result.`.
  **L346 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result.`。
- **L347 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L347 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L348 EN**: Continues logic associated with callable symbol `_mm_fnmsub_ps`.
  **L348 CN**: 继续与可调用符号 `_mm_fnmsub_ps` 相关的逻辑。
- **L349 EN**: Opens a new lexical scope or compound statement.
  **L349 CN**: 打开一个新的词法作用域或复合语句块。
- **L350 EN**: Returns from the current function with `(__m128)__builtin_elementwise_fma(-(__v4sf)__A, (__v4sf)__B,`.
  **L350 CN**: 以 `(__m128)__builtin_elementwise_fma(-(__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L351 EN**: Executes a call or declaration centered on `-`.
  **L351 CN**: 执行以 `-` 为核心的调用或声明。
- **L352 EN**: Closes the current lexical scope or compound statement.
  **L352 CN**: 结束当前词法作用域或复合语句块。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L354 EN**: Comment explains nearby logic, constraints, or intent: `Computes a negated multiply-subtract of 128-bit vectors of [2 x double].`.
  **L354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a negated multiply-subtract of 128-bit vectors of [2 x double].`。
- **L355 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> -(__A * __B) - __C </c>.`.
  **L355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> -(__A * __B) - __C </c>.`。
- **L356 EN**: Separator comment used for visual grouping.
  **L356 CN**: 用于视觉分组的分隔注释。
- **L357 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L357 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L358 EN**: Separator comment used for visual grouping.
  **L358 CN**: 用于视觉分组的分隔注释。
- **L359 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMSUB213PD instruction.`.
  **L359 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMSUB213PD instruction.`。
- **L360 EN**: Separator comment used for visual grouping.
  **L360 CN**: 用于视觉分组的分隔注释。

### Lines 361-384

````c
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier.
/// \param __C
///    A 128-bit vector of [2 x double] containing the subtrahend.
/// \returns A 128-bit vector of [2 x double] containing the result.
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fnmsub_pd(__m128d __A, __m128d __B, __m128d __C)
{
  return (__m128d)__builtin_elementwise_fma(-(__v2df)__A, (__v2df)__B,
                                            -(__v2df)__C);
}

/// Computes a scalar negated multiply-subtract of the single-precision
///    values in the low 32 bits of 128-bit vectors of [4 x float].
///
/// \code{.operation}
/// result[31:0] = -(__A[31:0] * __B[31:0]) - __C[31:0]
/// result[127:32] = __A[127:32]
/// \endcode
///
/// \headerfile <immintrin.h>
///
````
- **L361 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L361 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L362 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand.`.
  **L362 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand.`。
- **L363 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L363 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L364 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier.`.
  **L364 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier.`。
- **L365 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L366 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the subtrahend.`.
  **L366 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the subtrahend.`。
- **L367 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result.`.
  **L367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result.`。
- **L368 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L368 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L369 EN**: Continues logic associated with callable symbol `_mm_fnmsub_pd`.
  **L369 CN**: 继续与可调用符号 `_mm_fnmsub_pd` 相关的逻辑。
- **L370 EN**: Opens a new lexical scope or compound statement.
  **L370 CN**: 打开一个新的词法作用域或复合语句块。
- **L371 EN**: Returns from the current function with `(__m128d)__builtin_elementwise_fma(-(__v2df)__A, (__v2df)__B,`.
  **L371 CN**: 以 `(__m128d)__builtin_elementwise_fma(-(__v2df)__A, (__v2df)__B,` 从当前函数返回。
- **L372 EN**: Executes a call or declaration centered on `-`.
  **L372 CN**: 执行以 `-` 为核心的调用或声明。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L375 EN**: Comment explains nearby logic, constraints, or intent: `Computes a scalar negated multiply-subtract of the single-precision`.
  **L375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a scalar negated multiply-subtract of the single-precision`。
- **L376 EN**: Comment explains nearby logic, constraints, or intent: `values in the low 32 bits of 128-bit vectors of [4 x float].`.
  **L376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values in the low 32 bits of 128-bit vectors of [4 x float].`。
- **L377 EN**: Separator comment used for visual grouping.
  **L377 CN**: 用于视觉分组的分隔注释。
- **L378 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L378 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L379 EN**: Comment explains nearby logic, constraints, or intent: `result[31:0] -(__A[31:0] * __B[31:0]) - __C[31:0]`.
  **L379 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[31:0] -(__A[31:0] * __B[31:0]) - __C[31:0]`。
- **L380 EN**: Comment explains nearby logic, constraints, or intent: `result[127:32] __A[127:32]`.
  **L380 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:32] __A[127:32]`。
- **L381 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L382 EN**: Separator comment used for visual grouping.
  **L382 CN**: 用于视觉分组的分隔注释。
- **L383 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L383 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L384 EN**: Separator comment used for visual grouping.
  **L384 CN**: 用于视觉分组的分隔注释。

### Lines 385-408

````c
/// This intrinsic corresponds to the \c VFNMSUB213SS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand in the low
///    32 bits.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier in the low
///    32 bits.
/// \param __C
///    A 128-bit vector of [4 x float] containing the subtrahend in the low
///    32 bits.
/// \returns A 128-bit vector of [4 x float] containing the result in the low
///    32 bits, and a copy of \a __A[127:32] in the upper 96 bits.
static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fnmsub_ss(__m128 __A, __m128 __B, __m128 __C) {
  __A[0] = __builtin_elementwise_fma(__A[0], -__B[0], -__C[0]);
  return __A;
}

/// Computes a scalar negated multiply-subtract of the double-precision
///    values in the low 64 bits of 128-bit vectors of [2 x double].
///
/// \code{.operation}
/// result[63:0] = -(__A[63:0] * __B[63:0]) - __C[63:0]
````
- **L385 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMSUB213SS instruction.`.
  **L385 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMSUB213SS instruction.`。
- **L386 EN**: Separator comment used for visual grouping.
  **L386 CN**: 用于视觉分组的分隔注释。
- **L387 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L387 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L388 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand in the low`.
  **L388 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand in the low`。
- **L389 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L389 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L390 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L390 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L391 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier in the low`.
  **L391 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier in the low`。
- **L392 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L392 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L393 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L394 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the subtrahend in the low`.
  **L394 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the subtrahend in the low`。
- **L395 EN**: Comment explains nearby logic, constraints, or intent: `32 bits.`.
  **L395 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits.`。
- **L396 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result in the low`.
  **L396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result in the low`。
- **L397 EN**: Comment explains nearby logic, constraints, or intent: `32 bits, and a copy of a __A[127:32] in the upper 96 bits.`.
  **L397 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 bits, and a copy of a __A[127:32] in the upper 96 bits.`。
- **L398 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L398 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L399 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_fnmsub_ss(__m128 __A, __m128 __B, __m128 __C) {`.
  **L399 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_fnmsub_ss(__m128 __A, __m128 __B, __m128 __C) {`。
- **L400 EN**: Executes a call or declaration centered on `__builtin_elementwise_fma`.
  **L400 CN**: 执行以 `__builtin_elementwise_fma` 为核心的调用或声明。
- **L401 EN**: Returns from the current function with `__A`.
  **L401 CN**: 以 `__A` 从当前函数返回。
- **L402 EN**: Closes the current lexical scope or compound statement.
  **L402 CN**: 结束当前词法作用域或复合语句块。
- **L403 EN**: Blank line separating nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L404 EN**: Comment explains nearby logic, constraints, or intent: `Computes a scalar negated multiply-subtract of the double-precision`.
  **L404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a scalar negated multiply-subtract of the double-precision`。
- **L405 EN**: Comment explains nearby logic, constraints, or intent: `values in the low 64 bits of 128-bit vectors of [2 x double].`.
  **L405 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values in the low 64 bits of 128-bit vectors of [2 x double].`。
- **L406 EN**: Separator comment used for visual grouping.
  **L406 CN**: 用于视觉分组的分隔注释。
- **L407 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L407 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L408 EN**: Comment explains nearby logic, constraints, or intent: `result[63:0] -(__A[63:0] * __B[63:0]) - __C[63:0]`.
  **L408 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:0] -(__A[63:0] * __B[63:0]) - __C[63:0]`。

### Lines 409-432

````c
/// result[127:64] = __A[127:64]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFNMSUB213SD instruction.
///
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand in the low
///    64 bits.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier in the low
///    64 bits.
/// \param __C
///    A 128-bit vector of [2 x double] containing the subtrahend in the low
///    64 bits.
/// \returns A 128-bit vector of [2 x double] containing the result in the low
///    64 bits, and a copy of \a __A[127:64] in the upper 64 bits.
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_fnmsub_sd(__m128d __A, __m128d __B, __m128d __C) {
  __A[0] = __builtin_elementwise_fma(__A[0], -__B[0], -__C[0]);
  return __A;
}

````
- **L409 EN**: Comment explains nearby logic, constraints, or intent: `result[127:64] __A[127:64]`.
  **L409 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:64] __A[127:64]`。
- **L410 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L410 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L411 EN**: Separator comment used for visual grouping.
  **L411 CN**: 用于视觉分组的分隔注释。
- **L412 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L413 EN**: Separator comment used for visual grouping.
  **L413 CN**: 用于视觉分组的分隔注释。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMSUB213SD instruction.`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMSUB213SD instruction.`。
- **L415 EN**: Separator comment used for visual grouping.
  **L415 CN**: 用于视觉分组的分隔注释。
- **L416 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L416 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L417 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand in the low`.
  **L417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand in the low`。
- **L418 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L418 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L419 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L419 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L420 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier in the low`.
  **L420 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier in the low`。
- **L421 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L421 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L422 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L422 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L423 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the subtrahend in the low`.
  **L423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the subtrahend in the low`。
- **L424 EN**: Comment explains nearby logic, constraints, or intent: `64 bits.`.
  **L424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits.`。
- **L425 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result in the low`.
  **L425 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result in the low`。
- **L426 EN**: Comment explains nearby logic, constraints, or intent: `64 bits, and a copy of a __A[127:64] in the upper 64 bits.`.
  **L426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 bits, and a copy of a __A[127:64] in the upper 64 bits.`。
- **L427 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L427 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L428 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_fnmsub_sd(__m128d __A, __m128d __B, __m128d __C) {`.
  **L428 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_fnmsub_sd(__m128d __A, __m128d __B, __m128d __C) {`。
- **L429 EN**: Executes a call or declaration centered on `__builtin_elementwise_fma`.
  **L429 CN**: 执行以 `__builtin_elementwise_fma` 为核心的调用或声明。
- **L430 EN**: Returns from the current function with `__A`.
  **L430 CN**: 以 `__A` 从当前函数返回。
- **L431 EN**: Closes the current lexical scope or compound statement.
  **L431 CN**: 结束当前词法作用域或复合语句块。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 433-456

````c
/// Computes a multiply with alternating add/subtract of 128-bit vectors of
///    [4 x float].
///
/// \code{.operation}
/// result[31:0]  = (__A[31:0] * __B[31:0]) - __C[31:0]
/// result[63:32] = (__A[63:32] * __B[63:32]) + __C[63:32]
/// result[95:64] = (__A[95:64] * __B[95:64]) - __C[95:64]
/// result[127:96] = (__A[127:96] * __B[127:96]) + __C[127:96]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADDSUB213PS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier.
/// \param __C
///    A 128-bit vector of [4 x float] containing the addend/subtrahend.
/// \returns A 128-bit vector of [4 x float] containing the result.
static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_fmaddsub_ps(__m128 __A, __m128 __B, __m128 __C)
{
````
- **L433 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply with alternating add/subtract of 128-bit vectors of`.
  **L433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply with alternating add/subtract of 128-bit vectors of`。
- **L434 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float].`.
  **L434 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float].`。
- **L435 EN**: Separator comment used for visual grouping.
  **L435 CN**: 用于视觉分组的分隔注释。
- **L436 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L437 EN**: Comment explains nearby logic, constraints, or intent: `result[31:0] (__A[31:0] * __B[31:0]) - __C[31:0]`.
  **L437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[31:0] (__A[31:0] * __B[31:0]) - __C[31:0]`。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `result[63:32] (__A[63:32] * __B[63:32]) + __C[63:32]`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:32] (__A[63:32] * __B[63:32]) + __C[63:32]`。
- **L439 EN**: Comment explains nearby logic, constraints, or intent: `result[95:64] (__A[95:64] * __B[95:64]) - __C[95:64]`.
  **L439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[95:64] (__A[95:64] * __B[95:64]) - __C[95:64]`。
- **L440 EN**: Comment explains nearby logic, constraints, or intent: `result[127:96] (__A[127:96] * __B[127:96]) + __C[127:96]`.
  **L440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:96] (__A[127:96] * __B[127:96]) + __C[127:96]`。
- **L441 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L441 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L442 EN**: Separator comment used for visual grouping.
  **L442 CN**: 用于视觉分组的分隔注释。
- **L443 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L443 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L444 EN**: Separator comment used for visual grouping.
  **L444 CN**: 用于视觉分组的分隔注释。
- **L445 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADDSUB213PS instruction.`.
  **L445 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADDSUB213PS instruction.`。
- **L446 EN**: Separator comment used for visual grouping.
  **L446 CN**: 用于视觉分组的分隔注释。
- **L447 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L447 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L448 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand.`.
  **L448 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand.`。
- **L449 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L450 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier.`.
  **L450 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier.`。
- **L451 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L451 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L452 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the addend/subtrahend.`.
  **L452 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the addend/subtrahend.`。
- **L453 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result.`.
  **L453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result.`。
- **L454 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L454 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L455 EN**: Continues logic associated with callable symbol `_mm_fmaddsub_ps`.
  **L455 CN**: 继续与可调用符号 `_mm_fmaddsub_ps` 相关的逻辑。
- **L456 EN**: Opens a new lexical scope or compound statement.
  **L456 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 457-480

````c
  return (__m128)__builtin_ia32_vfmaddsubps((__v4sf)__A, (__v4sf)__B, (__v4sf)__C);
}

/// Computes a multiply with alternating add/subtract of 128-bit vectors of
///    [2 x double].
///
/// \code{.operation}
/// result[63:0]  = (__A[63:0] * __B[63:0]) - __C[63:0]
/// result[127:64] = (__A[127:64] * __B[127:64]) + __C[127:64]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADDSUB213PD instruction.
///
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier.
/// \param __C
///    A 128-bit vector of [2 x double] containing the addend/subtrahend.
/// \returns A 128-bit vector of [2 x double] containing the result.
static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_fmaddsub_pd(__m128d __A, __m128d __B, __m128d __C)
````
- **L457 EN**: Returns from the current function with `(__m128)__builtin_ia32_vfmaddsubps((__v4sf)__A, (__v4sf)__B, (__v4sf)__C)`.
  **L457 CN**: 以 `(__m128)__builtin_ia32_vfmaddsubps((__v4sf)__A, (__v4sf)__B, (__v4sf)__C)` 从当前函数返回。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L460 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply with alternating add/subtract of 128-bit vectors of`.
  **L460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply with alternating add/subtract of 128-bit vectors of`。
- **L461 EN**: Comment explains nearby logic, constraints, or intent: `[2 x double].`.
  **L461 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x double].`。
- **L462 EN**: Separator comment used for visual grouping.
  **L462 CN**: 用于视觉分组的分隔注释。
- **L463 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L464 EN**: Comment explains nearby logic, constraints, or intent: `result[63:0] (__A[63:0] * __B[63:0]) - __C[63:0]`.
  **L464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:0] (__A[63:0] * __B[63:0]) - __C[63:0]`。
- **L465 EN**: Comment explains nearby logic, constraints, or intent: `result[127:64] (__A[127:64] * __B[127:64]) + __C[127:64]`.
  **L465 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:64] (__A[127:64] * __B[127:64]) + __C[127:64]`。
- **L466 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L466 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L467 EN**: Separator comment used for visual grouping.
  **L467 CN**: 用于视觉分组的分隔注释。
- **L468 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L468 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L469 EN**: Separator comment used for visual grouping.
  **L469 CN**: 用于视觉分组的分隔注释。
- **L470 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADDSUB213PD instruction.`.
  **L470 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADDSUB213PD instruction.`。
- **L471 EN**: Separator comment used for visual grouping.
  **L471 CN**: 用于视觉分组的分隔注释。
- **L472 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L472 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L473 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand.`.
  **L473 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand.`。
- **L474 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L474 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L475 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier.`.
  **L475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier.`。
- **L476 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L476 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L477 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the addend/subtrahend.`.
  **L477 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the addend/subtrahend.`。
- **L478 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result.`.
  **L478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result.`。
- **L479 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L479 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L480 EN**: Continues logic associated with callable symbol `_mm_fmaddsub_pd`.
  **L480 CN**: 继续与可调用符号 `_mm_fmaddsub_pd` 相关的逻辑。

### Lines 481-504

````c
{
  return (__m128d)__builtin_ia32_vfmaddsubpd((__v2df)__A, (__v2df)__B, (__v2df)__C);
}

/// Computes a multiply with alternating add/subtract of 128-bit vectors of
///    [4 x float].
///
/// \code{.operation}
/// result[31:0]  = (__A[31:0] * __B[31:0]) + __C[31:0]
/// result[63:32] = (__A[63:32] * __B[63:32]) - __C[63:32]
/// result[95:64] = (__A[95:64] * __B[95:64]) + __C[95:64]
/// result[127:96 = (__A[127:96] * __B[127:96]) - __C[127:96]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUBADD213PS instruction.
///
/// \param __A
///    A 128-bit vector of [4 x float] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [4 x float] containing the multiplier.
/// \param __C
///    A 128-bit vector of [4 x float] containing the addend/subtrahend.
````
- **L481 EN**: Opens a new lexical scope or compound statement.
  **L481 CN**: 打开一个新的词法作用域或复合语句块。
- **L482 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vfmaddsubpd((__v2df)__A, (__v2df)__B, (__v2df)__C)`.
  **L482 CN**: 以 `(__m128d)__builtin_ia32_vfmaddsubpd((__v2df)__A, (__v2df)__B, (__v2df)__C)` 从当前函数返回。
- **L483 EN**: Closes the current lexical scope or compound statement.
  **L483 CN**: 结束当前词法作用域或复合语句块。
- **L484 EN**: Blank line separating nearby declarations or logic blocks.
  **L484 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L485 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply with alternating add/subtract of 128-bit vectors of`.
  **L485 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply with alternating add/subtract of 128-bit vectors of`。
- **L486 EN**: Comment explains nearby logic, constraints, or intent: `[4 x float].`.
  **L486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x float].`。
- **L487 EN**: Separator comment used for visual grouping.
  **L487 CN**: 用于视觉分组的分隔注释。
- **L488 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L488 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L489 EN**: Comment explains nearby logic, constraints, or intent: `result[31:0] (__A[31:0] * __B[31:0]) + __C[31:0]`.
  **L489 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[31:0] (__A[31:0] * __B[31:0]) + __C[31:0]`。
- **L490 EN**: Comment explains nearby logic, constraints, or intent: `result[63:32] (__A[63:32] * __B[63:32]) - __C[63:32]`.
  **L490 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:32] (__A[63:32] * __B[63:32]) - __C[63:32]`。
- **L491 EN**: Comment explains nearby logic, constraints, or intent: `result[95:64] (__A[95:64] * __B[95:64]) + __C[95:64]`.
  **L491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[95:64] (__A[95:64] * __B[95:64]) + __C[95:64]`。
- **L492 EN**: Comment explains nearby logic, constraints, or intent: `result[127:96 (__A[127:96] * __B[127:96]) - __C[127:96]`.
  **L492 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:96 (__A[127:96] * __B[127:96]) - __C[127:96]`。
- **L493 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L493 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L494 EN**: Separator comment used for visual grouping.
  **L494 CN**: 用于视觉分组的分隔注释。
- **L495 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L495 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L496 EN**: Separator comment used for visual grouping.
  **L496 CN**: 用于视觉分组的分隔注释。
- **L497 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUBADD213PS instruction.`.
  **L497 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUBADD213PS instruction.`。
- **L498 EN**: Separator comment used for visual grouping.
  **L498 CN**: 用于视觉分组的分隔注释。
- **L499 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L499 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L500 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplicand.`.
  **L500 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplicand.`。
- **L501 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L501 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L502 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the multiplier.`.
  **L502 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the multiplier.`。
- **L503 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L503 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L504 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float] containing the addend/subtrahend.`.
  **L504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float] containing the addend/subtrahend.`。

### Lines 505-528

````c
/// \returns A 128-bit vector of [4 x float] containing the result.
static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_fmsubadd_ps(__m128 __A, __m128 __B, __m128 __C)
{
  return (__m128)__builtin_ia32_vfmaddsubps((__v4sf)__A, (__v4sf)__B, -(__v4sf)__C);
}

/// Computes a multiply with alternating add/subtract of 128-bit vectors of
///    [2 x double].
///
/// \code{.operation}
/// result[63:0]  = (__A[63:0] * __B[63:0]) + __C[63:0]
/// result[127:64] = (__A[127:64] * __B[127:64]) - __C[127:64]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADDSUB213PD instruction.
///
/// \param __A
///    A 128-bit vector of [2 x double] containing the multiplicand.
/// \param __B
///    A 128-bit vector of [2 x double] containing the multiplier.
/// \param __C
````
- **L505 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [4 x float] containing the result.`.
  **L505 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [4 x float] containing the result.`。
- **L506 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L506 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L507 EN**: Continues logic associated with callable symbol `_mm_fmsubadd_ps`.
  **L507 CN**: 继续与可调用符号 `_mm_fmsubadd_ps` 相关的逻辑。
- **L508 EN**: Opens a new lexical scope or compound statement.
  **L508 CN**: 打开一个新的词法作用域或复合语句块。
- **L509 EN**: Returns from the current function with `(__m128)__builtin_ia32_vfmaddsubps((__v4sf)__A, (__v4sf)__B, -(__v4sf)__C)`.
  **L509 CN**: 以 `(__m128)__builtin_ia32_vfmaddsubps((__v4sf)__A, (__v4sf)__B, -(__v4sf)__C)` 从当前函数返回。
- **L510 EN**: Closes the current lexical scope or compound statement.
  **L510 CN**: 结束当前词法作用域或复合语句块。
- **L511 EN**: Blank line separating nearby declarations or logic blocks.
  **L511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L512 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply with alternating add/subtract of 128-bit vectors of`.
  **L512 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply with alternating add/subtract of 128-bit vectors of`。
- **L513 EN**: Comment explains nearby logic, constraints, or intent: `[2 x double].`.
  **L513 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x double].`。
- **L514 EN**: Separator comment used for visual grouping.
  **L514 CN**: 用于视觉分组的分隔注释。
- **L515 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L516 EN**: Comment explains nearby logic, constraints, or intent: `result[63:0] (__A[63:0] * __B[63:0]) + __C[63:0]`.
  **L516 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:0] (__A[63:0] * __B[63:0]) + __C[63:0]`。
- **L517 EN**: Comment explains nearby logic, constraints, or intent: `result[127:64] (__A[127:64] * __B[127:64]) - __C[127:64]`.
  **L517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:64] (__A[127:64] * __B[127:64]) - __C[127:64]`。
- **L518 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L518 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L519 EN**: Separator comment used for visual grouping.
  **L519 CN**: 用于视觉分组的分隔注释。
- **L520 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L520 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L521 EN**: Separator comment used for visual grouping.
  **L521 CN**: 用于视觉分组的分隔注释。
- **L522 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADDSUB213PD instruction.`.
  **L522 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADDSUB213PD instruction.`。
- **L523 EN**: Separator comment used for visual grouping.
  **L523 CN**: 用于视觉分组的分隔注释。
- **L524 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L524 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L525 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplicand.`.
  **L525 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplicand.`。
- **L526 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L526 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L527 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the multiplier.`.
  **L527 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the multiplier.`。
- **L528 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L528 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。

### Lines 529-552

````c
///    A 128-bit vector of [2 x double] containing the addend/subtrahend.
/// \returns A 128-bit vector of [2 x double] containing the result.
static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_fmsubadd_pd(__m128d __A, __m128d __B, __m128d __C)
{
  return (__m128d)__builtin_ia32_vfmaddsubpd((__v2df)__A, (__v2df)__B, -(__v2df)__C);
}

/// Computes a multiply-add of 256-bit vectors of [8 x float].
///    For each element, computes <c> (__A * __B) + __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADD213PS instruction.
///
/// \param __A
///    A 256-bit vector of [8 x float] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [8 x float] containing the multiplier.
/// \param __C
///    A 256-bit vector of [8 x float] containing the addend.
/// \returns A 256-bit vector of [8 x float] containing the result.
static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_fmadd_ps(__m256 __A, __m256 __B, __m256 __C)
````
- **L529 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [2 x double] containing the addend/subtrahend.`.
  **L529 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [2 x double] containing the addend/subtrahend.`。
- **L530 EN**: Comment explains nearby logic, constraints, or intent: `returns A 128-bit vector of [2 x double] containing the result.`.
  **L530 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 128-bit vector of [2 x double] containing the result.`。
- **L531 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L531 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L532 EN**: Continues logic associated with callable symbol `_mm_fmsubadd_pd`.
  **L532 CN**: 继续与可调用符号 `_mm_fmsubadd_pd` 相关的逻辑。
- **L533 EN**: Opens a new lexical scope or compound statement.
  **L533 CN**: 打开一个新的词法作用域或复合语句块。
- **L534 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vfmaddsubpd((__v2df)__A, (__v2df)__B, -(__v2df)__C)`.
  **L534 CN**: 以 `(__m128d)__builtin_ia32_vfmaddsubpd((__v2df)__A, (__v2df)__B, -(__v2df)__C)` 从当前函数返回。
- **L535 EN**: Closes the current lexical scope or compound statement.
  **L535 CN**: 结束当前词法作用域或复合语句块。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply-add of 256-bit vectors of [8 x float].`.
  **L537 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply-add of 256-bit vectors of [8 x float].`。
- **L538 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> (__A * __B) + __C </c>.`.
  **L538 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> (__A * __B) + __C </c>.`。
- **L539 EN**: Separator comment used for visual grouping.
  **L539 CN**: 用于视觉分组的分隔注释。
- **L540 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L540 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L541 EN**: Separator comment used for visual grouping.
  **L541 CN**: 用于视觉分组的分隔注释。
- **L542 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADD213PS instruction.`.
  **L542 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADD213PS instruction.`。
- **L543 EN**: Separator comment used for visual grouping.
  **L543 CN**: 用于视觉分组的分隔注释。
- **L544 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L544 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L545 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplicand.`.
  **L545 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplicand.`。
- **L546 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L546 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L547 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplier.`.
  **L547 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplier.`。
- **L548 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L548 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L549 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the addend.`.
  **L549 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the addend.`。
- **L550 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [8 x float] containing the result.`.
  **L550 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [8 x float] containing the result.`。
- **L551 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L551 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L552 EN**: Continues logic associated with callable symbol `_mm256_fmadd_ps`.
  **L552 CN**: 继续与可调用符号 `_mm256_fmadd_ps` 相关的逻辑。

### Lines 553-576

````c
{
  return (__m256)__builtin_elementwise_fma((__v8sf)__A, (__v8sf)__B,
                                           (__v8sf)__C);
}

/// Computes a multiply-add of 256-bit vectors of [4 x double].
///    For each element, computes <c> (__A * __B) + __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADD213PD instruction.
///
/// \param __A
///    A 256-bit vector of [4 x double] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [4 x double] containing the multiplier.
/// \param __C
///    A 256-bit vector of [4 x double] containing the addend.
/// \returns A 256-bit vector of [4 x double] containing the result.
static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_fmadd_pd(__m256d __A, __m256d __B, __m256d __C)
{
  return (__m256d)__builtin_elementwise_fma((__v4df)__A, (__v4df)__B,
                                            (__v4df)__C);
````
- **L553 EN**: Opens a new lexical scope or compound statement.
  **L553 CN**: 打开一个新的词法作用域或复合语句块。
- **L554 EN**: Returns from the current function with `(__m256)__builtin_elementwise_fma((__v8sf)__A, (__v8sf)__B,`.
  **L554 CN**: 以 `(__m256)__builtin_elementwise_fma((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L555 EN**: Executes a call or declaration centered on `statement`.
  **L555 CN**: 执行以 `statement` 为核心的调用或声明。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L558 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply-add of 256-bit vectors of [4 x double].`.
  **L558 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply-add of 256-bit vectors of [4 x double].`。
- **L559 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> (__A * __B) + __C </c>.`.
  **L559 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> (__A * __B) + __C </c>.`。
- **L560 EN**: Separator comment used for visual grouping.
  **L560 CN**: 用于视觉分组的分隔注释。
- **L561 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L561 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L562 EN**: Separator comment used for visual grouping.
  **L562 CN**: 用于视觉分组的分隔注释。
- **L563 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADD213PD instruction.`.
  **L563 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADD213PD instruction.`。
- **L564 EN**: Separator comment used for visual grouping.
  **L564 CN**: 用于视觉分组的分隔注释。
- **L565 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L565 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L566 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplicand.`.
  **L566 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplicand.`。
- **L567 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L567 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L568 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplier.`.
  **L568 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplier.`。
- **L569 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L569 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L570 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the addend.`.
  **L570 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the addend.`。
- **L571 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [4 x double] containing the result.`.
  **L571 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [4 x double] containing the result.`。
- **L572 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L572 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L573 EN**: Continues logic associated with callable symbol `_mm256_fmadd_pd`.
  **L573 CN**: 继续与可调用符号 `_mm256_fmadd_pd` 相关的逻辑。
- **L574 EN**: Opens a new lexical scope or compound statement.
  **L574 CN**: 打开一个新的词法作用域或复合语句块。
- **L575 EN**: Returns from the current function with `(__m256d)__builtin_elementwise_fma((__v4df)__A, (__v4df)__B,`.
  **L575 CN**: 以 `(__m256d)__builtin_elementwise_fma((__v4df)__A, (__v4df)__B,` 从当前函数返回。
- **L576 EN**: Executes a call or declaration centered on `statement`.
  **L576 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 577-600

````c
}

/// Computes a multiply-subtract of 256-bit vectors of [8 x float].
///    For each element, computes <c> (__A * __B) - __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUB213PS instruction.
///
/// \param __A
///    A 256-bit vector of [8 x float] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [8 x float] containing the multiplier.
/// \param __C
///    A 256-bit vector of [8 x float] containing the subtrahend.
/// \returns A 256-bit vector of [8 x float] containing the result.
static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_fmsub_ps(__m256 __A, __m256 __B, __m256 __C)
{
  return (__m256)__builtin_elementwise_fma((__v8sf)__A, (__v8sf)__B,
                                           -(__v8sf)__C);
}

/// Computes a multiply-subtract of 256-bit vectors of [4 x double].
````
- **L577 EN**: Closes the current lexical scope or compound statement.
  **L577 CN**: 结束当前词法作用域或复合语句块。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L579 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply-subtract of 256-bit vectors of [8 x float].`.
  **L579 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply-subtract of 256-bit vectors of [8 x float].`。
- **L580 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> (__A * __B) - __C </c>.`.
  **L580 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> (__A * __B) - __C </c>.`。
- **L581 EN**: Separator comment used for visual grouping.
  **L581 CN**: 用于视觉分组的分隔注释。
- **L582 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L582 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L583 EN**: Separator comment used for visual grouping.
  **L583 CN**: 用于视觉分组的分隔注释。
- **L584 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUB213PS instruction.`.
  **L584 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUB213PS instruction.`。
- **L585 EN**: Separator comment used for visual grouping.
  **L585 CN**: 用于视觉分组的分隔注释。
- **L586 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L586 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L587 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplicand.`.
  **L587 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplicand.`。
- **L588 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L588 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L589 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplier.`.
  **L589 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplier.`。
- **L590 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L590 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L591 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the subtrahend.`.
  **L591 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the subtrahend.`。
- **L592 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [8 x float] containing the result.`.
  **L592 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [8 x float] containing the result.`。
- **L593 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L593 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L594 EN**: Continues logic associated with callable symbol `_mm256_fmsub_ps`.
  **L594 CN**: 继续与可调用符号 `_mm256_fmsub_ps` 相关的逻辑。
- **L595 EN**: Opens a new lexical scope or compound statement.
  **L595 CN**: 打开一个新的词法作用域或复合语句块。
- **L596 EN**: Returns from the current function with `(__m256)__builtin_elementwise_fma((__v8sf)__A, (__v8sf)__B,`.
  **L596 CN**: 以 `(__m256)__builtin_elementwise_fma((__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L597 EN**: Executes a call or declaration centered on `-`.
  **L597 CN**: 执行以 `-` 为核心的调用或声明。
- **L598 EN**: Closes the current lexical scope or compound statement.
  **L598 CN**: 结束当前词法作用域或复合语句块。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L600 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply-subtract of 256-bit vectors of [4 x double].`.
  **L600 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply-subtract of 256-bit vectors of [4 x double].`。

### Lines 601-624

````c
///    For each element, computes <c> (__A * __B) - __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUB213PD instruction.
///
/// \param __A
///    A 256-bit vector of [4 x double] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [4 x double] containing the multiplier.
/// \param __C
///    A 256-bit vector of [4 x double] containing the subtrahend.
/// \returns A 256-bit vector of [4 x double] containing the result.
static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_fmsub_pd(__m256d __A, __m256d __B, __m256d __C)
{
  return (__m256d)__builtin_elementwise_fma((__v4df)__A, (__v4df)__B,
                                            -(__v4df)__C);
}

/// Computes a negated multiply-add of 256-bit vectors of [8 x float].
///    For each element, computes <c> -(__A * __B) + __C </c>.
///
/// \headerfile <immintrin.h>
````
- **L601 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> (__A * __B) - __C </c>.`.
  **L601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> (__A * __B) - __C </c>.`。
- **L602 EN**: Separator comment used for visual grouping.
  **L602 CN**: 用于视觉分组的分隔注释。
- **L603 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L603 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L604 EN**: Separator comment used for visual grouping.
  **L604 CN**: 用于视觉分组的分隔注释。
- **L605 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUB213PD instruction.`.
  **L605 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUB213PD instruction.`。
- **L606 EN**: Separator comment used for visual grouping.
  **L606 CN**: 用于视觉分组的分隔注释。
- **L607 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L607 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L608 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplicand.`.
  **L608 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplicand.`。
- **L609 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L609 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L610 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplier.`.
  **L610 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplier.`。
- **L611 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L611 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L612 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the subtrahend.`.
  **L612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the subtrahend.`。
- **L613 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [4 x double] containing the result.`.
  **L613 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [4 x double] containing the result.`。
- **L614 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L614 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L615 EN**: Continues logic associated with callable symbol `_mm256_fmsub_pd`.
  **L615 CN**: 继续与可调用符号 `_mm256_fmsub_pd` 相关的逻辑。
- **L616 EN**: Opens a new lexical scope or compound statement.
  **L616 CN**: 打开一个新的词法作用域或复合语句块。
- **L617 EN**: Returns from the current function with `(__m256d)__builtin_elementwise_fma((__v4df)__A, (__v4df)__B,`.
  **L617 CN**: 以 `(__m256d)__builtin_elementwise_fma((__v4df)__A, (__v4df)__B,` 从当前函数返回。
- **L618 EN**: Executes a call or declaration centered on `-`.
  **L618 CN**: 执行以 `-` 为核心的调用或声明。
- **L619 EN**: Closes the current lexical scope or compound statement.
  **L619 CN**: 结束当前词法作用域或复合语句块。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Comment explains nearby logic, constraints, or intent: `Computes a negated multiply-add of 256-bit vectors of [8 x float].`.
  **L621 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a negated multiply-add of 256-bit vectors of [8 x float].`。
- **L622 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> -(__A * __B) + __C </c>.`.
  **L622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> -(__A * __B) + __C </c>.`。
- **L623 EN**: Separator comment used for visual grouping.
  **L623 CN**: 用于视觉分组的分隔注释。
- **L624 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L624 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。

### Lines 625-648

````c
///
/// This intrinsic corresponds to the \c VFNMADD213PS instruction.
///
/// \param __A
///    A 256-bit vector of [8 x float] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [8 x float] containing the multiplier.
/// \param __C
///    A 256-bit vector of [8 x float] containing the addend.
/// \returns A 256-bit vector of [8 x float] containing the result.
static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_fnmadd_ps(__m256 __A, __m256 __B, __m256 __C)
{
  return (__m256)__builtin_elementwise_fma(-(__v8sf)__A, (__v8sf)__B,
                                           (__v8sf)__C);
}

/// Computes a negated multiply-add of 256-bit vectors of [4 x double].
///    For each element, computes <c> -(__A * __B) + __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFNMADD213PD instruction.
///
````
- **L625 EN**: Separator comment used for visual grouping.
  **L625 CN**: 用于视觉分组的分隔注释。
- **L626 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMADD213PS instruction.`.
  **L626 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMADD213PS instruction.`。
- **L627 EN**: Separator comment used for visual grouping.
  **L627 CN**: 用于视觉分组的分隔注释。
- **L628 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L628 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L629 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplicand.`.
  **L629 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplicand.`。
- **L630 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L630 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L631 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplier.`.
  **L631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplier.`。
- **L632 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L632 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L633 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the addend.`.
  **L633 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the addend.`。
- **L634 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [8 x float] containing the result.`.
  **L634 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [8 x float] containing the result.`。
- **L635 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L635 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L636 EN**: Continues logic associated with callable symbol `_mm256_fnmadd_ps`.
  **L636 CN**: 继续与可调用符号 `_mm256_fnmadd_ps` 相关的逻辑。
- **L637 EN**: Opens a new lexical scope or compound statement.
  **L637 CN**: 打开一个新的词法作用域或复合语句块。
- **L638 EN**: Returns from the current function with `(__m256)__builtin_elementwise_fma(-(__v8sf)__A, (__v8sf)__B,`.
  **L638 CN**: 以 `(__m256)__builtin_elementwise_fma(-(__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L639 EN**: Executes a call or declaration centered on `statement`.
  **L639 CN**: 执行以 `statement` 为核心的调用或声明。
- **L640 EN**: Closes the current lexical scope or compound statement.
  **L640 CN**: 结束当前词法作用域或复合语句块。
- **L641 EN**: Blank line separating nearby declarations or logic blocks.
  **L641 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L642 EN**: Comment explains nearby logic, constraints, or intent: `Computes a negated multiply-add of 256-bit vectors of [4 x double].`.
  **L642 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a negated multiply-add of 256-bit vectors of [4 x double].`。
- **L643 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> -(__A * __B) + __C </c>.`.
  **L643 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> -(__A * __B) + __C </c>.`。
- **L644 EN**: Separator comment used for visual grouping.
  **L644 CN**: 用于视觉分组的分隔注释。
- **L645 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L645 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L646 EN**: Separator comment used for visual grouping.
  **L646 CN**: 用于视觉分组的分隔注释。
- **L647 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMADD213PD instruction.`.
  **L647 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMADD213PD instruction.`。
- **L648 EN**: Separator comment used for visual grouping.
  **L648 CN**: 用于视觉分组的分隔注释。

### Lines 649-672

````c
/// \param __A
///    A 256-bit vector of [4 x double] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [4 x double] containing the multiplier.
/// \param __C
///    A 256-bit vector of [4 x double] containing the addend.
/// \returns A 256-bit vector of [4 x double] containing the result.
static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_fnmadd_pd(__m256d __A, __m256d __B, __m256d __C)
{
  return (__m256d)__builtin_elementwise_fma(-(__v4df)__A, (__v4df)__B,
                                            (__v4df)__C);
}

/// Computes a negated multiply-subtract of 256-bit vectors of [8 x float].
///    For each element, computes <c> -(__A * __B) - __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFNMSUB213PS instruction.
///
/// \param __A
///    A 256-bit vector of [8 x float] containing the multiplicand.
/// \param __B
````
- **L649 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L649 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L650 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplicand.`.
  **L650 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplicand.`。
- **L651 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L651 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L652 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplier.`.
  **L652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplier.`。
- **L653 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L653 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L654 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the addend.`.
  **L654 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the addend.`。
- **L655 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [4 x double] containing the result.`.
  **L655 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [4 x double] containing the result.`。
- **L656 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L656 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L657 EN**: Continues logic associated with callable symbol `_mm256_fnmadd_pd`.
  **L657 CN**: 继续与可调用符号 `_mm256_fnmadd_pd` 相关的逻辑。
- **L658 EN**: Opens a new lexical scope or compound statement.
  **L658 CN**: 打开一个新的词法作用域或复合语句块。
- **L659 EN**: Returns from the current function with `(__m256d)__builtin_elementwise_fma(-(__v4df)__A, (__v4df)__B,`.
  **L659 CN**: 以 `(__m256d)__builtin_elementwise_fma(-(__v4df)__A, (__v4df)__B,` 从当前函数返回。
- **L660 EN**: Executes a call or declaration centered on `statement`.
  **L660 CN**: 执行以 `statement` 为核心的调用或声明。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Blank line separating nearby declarations or logic blocks.
  **L662 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L663 EN**: Comment explains nearby logic, constraints, or intent: `Computes a negated multiply-subtract of 256-bit vectors of [8 x float].`.
  **L663 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a negated multiply-subtract of 256-bit vectors of [8 x float].`。
- **L664 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> -(__A * __B) - __C </c>.`.
  **L664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> -(__A * __B) - __C </c>.`。
- **L665 EN**: Separator comment used for visual grouping.
  **L665 CN**: 用于视觉分组的分隔注释。
- **L666 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L666 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L667 EN**: Separator comment used for visual grouping.
  **L667 CN**: 用于视觉分组的分隔注释。
- **L668 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMSUB213PS instruction.`.
  **L668 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMSUB213PS instruction.`。
- **L669 EN**: Separator comment used for visual grouping.
  **L669 CN**: 用于视觉分组的分隔注释。
- **L670 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L670 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L671 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplicand.`.
  **L671 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplicand.`。
- **L672 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L672 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。

### Lines 673-696

````c
///    A 256-bit vector of [8 x float] containing the multiplier.
/// \param __C
///    A 256-bit vector of [8 x float] containing the subtrahend.
/// \returns A 256-bit vector of [8 x float] containing the result.
static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_fnmsub_ps(__m256 __A, __m256 __B, __m256 __C)
{
  return (__m256)__builtin_elementwise_fma(-(__v8sf)__A, (__v8sf)__B,
                                           -(__v8sf)__C);
}

/// Computes a negated multiply-subtract of 256-bit vectors of [4 x double].
///    For each element, computes <c> -(__A * __B) - __C </c>.
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFNMSUB213PD instruction.
///
/// \param __A
///    A 256-bit vector of [4 x double] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [4 x double] containing the multiplier.
/// \param __C
///    A 256-bit vector of [4 x double] containing the subtrahend.
````
- **L673 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplier.`.
  **L673 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplier.`。
- **L674 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L675 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the subtrahend.`.
  **L675 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the subtrahend.`。
- **L676 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [8 x float] containing the result.`.
  **L676 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [8 x float] containing the result.`。
- **L677 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L677 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L678 EN**: Continues logic associated with callable symbol `_mm256_fnmsub_ps`.
  **L678 CN**: 继续与可调用符号 `_mm256_fnmsub_ps` 相关的逻辑。
- **L679 EN**: Opens a new lexical scope or compound statement.
  **L679 CN**: 打开一个新的词法作用域或复合语句块。
- **L680 EN**: Returns from the current function with `(__m256)__builtin_elementwise_fma(-(__v8sf)__A, (__v8sf)__B,`.
  **L680 CN**: 以 `(__m256)__builtin_elementwise_fma(-(__v8sf)__A, (__v8sf)__B,` 从当前函数返回。
- **L681 EN**: Executes a call or declaration centered on `-`.
  **L681 CN**: 执行以 `-` 为核心的调用或声明。
- **L682 EN**: Closes the current lexical scope or compound statement.
  **L682 CN**: 结束当前词法作用域或复合语句块。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L684 EN**: Comment explains nearby logic, constraints, or intent: `Computes a negated multiply-subtract of 256-bit vectors of [4 x double].`.
  **L684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a negated multiply-subtract of 256-bit vectors of [4 x double].`。
- **L685 EN**: Comment explains nearby logic, constraints, or intent: `For each element, computes <c> -(__A * __B) - __C </c>.`.
  **L685 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For each element, computes <c> -(__A * __B) - __C </c>.`。
- **L686 EN**: Separator comment used for visual grouping.
  **L686 CN**: 用于视觉分组的分隔注释。
- **L687 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L687 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L688 EN**: Separator comment used for visual grouping.
  **L688 CN**: 用于视觉分组的分隔注释。
- **L689 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFNMSUB213PD instruction.`.
  **L689 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFNMSUB213PD instruction.`。
- **L690 EN**: Separator comment used for visual grouping.
  **L690 CN**: 用于视觉分组的分隔注释。
- **L691 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L691 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L692 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplicand.`.
  **L692 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplicand.`。
- **L693 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L693 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L694 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplier.`.
  **L694 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplier.`。
- **L695 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L695 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L696 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the subtrahend.`.
  **L696 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the subtrahend.`。

### Lines 697-720

````c
/// \returns A 256-bit vector of [4 x double] containing the result.
static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_fnmsub_pd(__m256d __A, __m256d __B, __m256d __C)
{
  return (__m256d)__builtin_elementwise_fma(-(__v4df)__A, (__v4df)__B,
                                            -(__v4df)__C);
}

/// Computes a multiply with alternating add/subtract of 256-bit vectors of
///    [8 x float].
///
/// \code{.operation}
/// result[31:0] = (__A[31:0] * __B[31:0]) - __C[31:0]
/// result[63:32] = (__A[63:32] * __B[63:32]) + __C[63:32]
/// result[95:64] = (__A[95:64] * __B[95:64]) - __C[95:64]
/// result[127:96] = (__A[127:96] * __B[127:96]) + __C[127:96]
/// result[159:128] = (__A[159:128] * __B[159:128]) - __C[159:128]
/// result[191:160] = (__A[191:160] * __B[191:160]) + __C[191:160]
/// result[223:192] = (__A[223:192] * __B[223:192]) - __C[223:192]
/// result[255:224] = (__A[255:224] * __B[255:224]) + __C[255:224]
/// \endcode
///
/// \headerfile <immintrin.h>
///
````
- **L697 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [4 x double] containing the result.`.
  **L697 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [4 x double] containing the result.`。
- **L698 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L698 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L699 EN**: Continues logic associated with callable symbol `_mm256_fnmsub_pd`.
  **L699 CN**: 继续与可调用符号 `_mm256_fnmsub_pd` 相关的逻辑。
- **L700 EN**: Opens a new lexical scope or compound statement.
  **L700 CN**: 打开一个新的词法作用域或复合语句块。
- **L701 EN**: Returns from the current function with `(__m256d)__builtin_elementwise_fma(-(__v4df)__A, (__v4df)__B,`.
  **L701 CN**: 以 `(__m256d)__builtin_elementwise_fma(-(__v4df)__A, (__v4df)__B,` 从当前函数返回。
- **L702 EN**: Executes a call or declaration centered on `-`.
  **L702 CN**: 执行以 `-` 为核心的调用或声明。
- **L703 EN**: Closes the current lexical scope or compound statement.
  **L703 CN**: 结束当前词法作用域或复合语句块。
- **L704 EN**: Blank line separating nearby declarations or logic blocks.
  **L704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L705 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply with alternating add/subtract of 256-bit vectors of`.
  **L705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply with alternating add/subtract of 256-bit vectors of`。
- **L706 EN**: Comment explains nearby logic, constraints, or intent: `[8 x float].`.
  **L706 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[8 x float].`。
- **L707 EN**: Separator comment used for visual grouping.
  **L707 CN**: 用于视觉分组的分隔注释。
- **L708 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L708 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L709 EN**: Comment explains nearby logic, constraints, or intent: `result[31:0] (__A[31:0] * __B[31:0]) - __C[31:0]`.
  **L709 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[31:0] (__A[31:0] * __B[31:0]) - __C[31:0]`。
- **L710 EN**: Comment explains nearby logic, constraints, or intent: `result[63:32] (__A[63:32] * __B[63:32]) + __C[63:32]`.
  **L710 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:32] (__A[63:32] * __B[63:32]) + __C[63:32]`。
- **L711 EN**: Comment explains nearby logic, constraints, or intent: `result[95:64] (__A[95:64] * __B[95:64]) - __C[95:64]`.
  **L711 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[95:64] (__A[95:64] * __B[95:64]) - __C[95:64]`。
- **L712 EN**: Comment explains nearby logic, constraints, or intent: `result[127:96] (__A[127:96] * __B[127:96]) + __C[127:96]`.
  **L712 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:96] (__A[127:96] * __B[127:96]) + __C[127:96]`。
- **L713 EN**: Comment explains nearby logic, constraints, or intent: `result[159:128] (__A[159:128] * __B[159:128]) - __C[159:128]`.
  **L713 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[159:128] (__A[159:128] * __B[159:128]) - __C[159:128]`。
- **L714 EN**: Comment explains nearby logic, constraints, or intent: `result[191:160] (__A[191:160] * __B[191:160]) + __C[191:160]`.
  **L714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[191:160] (__A[191:160] * __B[191:160]) + __C[191:160]`。
- **L715 EN**: Comment explains nearby logic, constraints, or intent: `result[223:192] (__A[223:192] * __B[223:192]) - __C[223:192]`.
  **L715 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[223:192] (__A[223:192] * __B[223:192]) - __C[223:192]`。
- **L716 EN**: Comment explains nearby logic, constraints, or intent: `result[255:224] (__A[255:224] * __B[255:224]) + __C[255:224]`.
  **L716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[255:224] (__A[255:224] * __B[255:224]) + __C[255:224]`。
- **L717 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L717 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L718 EN**: Separator comment used for visual grouping.
  **L718 CN**: 用于视觉分组的分隔注释。
- **L719 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L719 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L720 EN**: Separator comment used for visual grouping.
  **L720 CN**: 用于视觉分组的分隔注释。

### Lines 721-744

````c
/// This intrinsic corresponds to the \c VFMADDSUB213PS instruction.
///
/// \param __A
///    A 256-bit vector of [8 x float] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [8 x float] containing the multiplier.
/// \param __C
///    A 256-bit vector of [8 x float] containing the addend/subtrahend.
/// \returns A 256-bit vector of [8 x float] containing the result.
static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_fmaddsub_ps(__m256 __A, __m256 __B, __m256 __C)
{
  return (__m256)__builtin_ia32_vfmaddsubps256((__v8sf)__A, (__v8sf)__B, (__v8sf)__C);
}

/// Computes a multiply with alternating add/subtract of 256-bit vectors of
///    [4 x double].
///
/// \code{.operation}
/// result[63:0] = (__A[63:0] * __B[63:0]) - __C[63:0]
/// result[127:64] = (__A[127:64] * __B[127:64]) + __C[127:64]
/// result[191:128] = (__A[191:128] * __B[191:128]) - __C[191:128]
/// result[255:192] = (__A[255:192] * __B[255:192]) + __C[255:192]
/// \endcode
````
- **L721 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADDSUB213PS instruction.`.
  **L721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADDSUB213PS instruction.`。
- **L722 EN**: Separator comment used for visual grouping.
  **L722 CN**: 用于视觉分组的分隔注释。
- **L723 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L723 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L724 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplicand.`.
  **L724 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplicand.`。
- **L725 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L725 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L726 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplier.`.
  **L726 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplier.`。
- **L727 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L727 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L728 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the addend/subtrahend.`.
  **L728 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the addend/subtrahend.`。
- **L729 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [8 x float] containing the result.`.
  **L729 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [8 x float] containing the result.`。
- **L730 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L730 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L731 EN**: Continues logic associated with callable symbol `_mm256_fmaddsub_ps`.
  **L731 CN**: 继续与可调用符号 `_mm256_fmaddsub_ps` 相关的逻辑。
- **L732 EN**: Opens a new lexical scope or compound statement.
  **L732 CN**: 打开一个新的词法作用域或复合语句块。
- **L733 EN**: Returns from the current function with `(__m256)__builtin_ia32_vfmaddsubps256((__v8sf)__A, (__v8sf)__B, (__v8sf)__C)`.
  **L733 CN**: 以 `(__m256)__builtin_ia32_vfmaddsubps256((__v8sf)__A, (__v8sf)__B, (__v8sf)__C)` 从当前函数返回。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L736 EN**: Comment explains nearby logic, constraints, or intent: `Computes a multiply with alternating add/subtract of 256-bit vectors of`.
  **L736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a multiply with alternating add/subtract of 256-bit vectors of`。
- **L737 EN**: Comment explains nearby logic, constraints, or intent: `[4 x double].`.
  **L737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x double].`。
- **L738 EN**: Separator comment used for visual grouping.
  **L738 CN**: 用于视觉分组的分隔注释。
- **L739 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L740 EN**: Comment explains nearby logic, constraints, or intent: `result[63:0] (__A[63:0] * __B[63:0]) - __C[63:0]`.
  **L740 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:0] (__A[63:0] * __B[63:0]) - __C[63:0]`。
- **L741 EN**: Comment explains nearby logic, constraints, or intent: `result[127:64] (__A[127:64] * __B[127:64]) + __C[127:64]`.
  **L741 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:64] (__A[127:64] * __B[127:64]) + __C[127:64]`。
- **L742 EN**: Comment explains nearby logic, constraints, or intent: `result[191:128] (__A[191:128] * __B[191:128]) - __C[191:128]`.
  **L742 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[191:128] (__A[191:128] * __B[191:128]) - __C[191:128]`。
- **L743 EN**: Comment explains nearby logic, constraints, or intent: `result[255:192] (__A[255:192] * __B[255:192]) + __C[255:192]`.
  **L743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[255:192] (__A[255:192] * __B[255:192]) + __C[255:192]`。
- **L744 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L744 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。

### Lines 745-768

````c
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMADDSUB213PD instruction.
///
/// \param __A
///    A 256-bit vector of [4 x double] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [4 x double] containing the multiplier.
/// \param __C
///    A 256-bit vector of [4 x double] containing the addend/subtrahend.
/// \returns A 256-bit vector of [4 x double] containing the result.
static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_fmaddsub_pd(__m256d __A, __m256d __B, __m256d __C)
{
  return (__m256d)__builtin_ia32_vfmaddsubpd256((__v4df)__A, (__v4df)__B, (__v4df)__C);
}

/// Computes a vector multiply with alternating add/subtract of 256-bit
///    vectors of [8 x float].
///
/// \code{.operation}
/// result[31:0] = (__A[31:0] * __B[31:0]) + __C[31:0]
/// result[63:32] = (__A[63:32] * __B[63:32]) - __C[63:32]
````
- **L745 EN**: Separator comment used for visual grouping.
  **L745 CN**: 用于视觉分组的分隔注释。
- **L746 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L747 EN**: Separator comment used for visual grouping.
  **L747 CN**: 用于视觉分组的分隔注释。
- **L748 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMADDSUB213PD instruction.`.
  **L748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMADDSUB213PD instruction.`。
- **L749 EN**: Separator comment used for visual grouping.
  **L749 CN**: 用于视觉分组的分隔注释。
- **L750 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L750 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L751 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplicand.`.
  **L751 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplicand.`。
- **L752 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L752 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L753 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplier.`.
  **L753 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplier.`。
- **L754 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L754 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L755 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the addend/subtrahend.`.
  **L755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the addend/subtrahend.`。
- **L756 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [4 x double] containing the result.`.
  **L756 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [4 x double] containing the result.`。
- **L757 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L757 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L758 EN**: Continues logic associated with callable symbol `_mm256_fmaddsub_pd`.
  **L758 CN**: 继续与可调用符号 `_mm256_fmaddsub_pd` 相关的逻辑。
- **L759 EN**: Opens a new lexical scope or compound statement.
  **L759 CN**: 打开一个新的词法作用域或复合语句块。
- **L760 EN**: Returns from the current function with `(__m256d)__builtin_ia32_vfmaddsubpd256((__v4df)__A, (__v4df)__B, (__v4df)__C)`.
  **L760 CN**: 以 `(__m256d)__builtin_ia32_vfmaddsubpd256((__v4df)__A, (__v4df)__B, (__v4df)__C)` 从当前函数返回。
- **L761 EN**: Closes the current lexical scope or compound statement.
  **L761 CN**: 结束当前词法作用域或复合语句块。
- **L762 EN**: Blank line separating nearby declarations or logic blocks.
  **L762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L763 EN**: Comment explains nearby logic, constraints, or intent: `Computes a vector multiply with alternating add/subtract of 256-bit`.
  **L763 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a vector multiply with alternating add/subtract of 256-bit`。
- **L764 EN**: Comment explains nearby logic, constraints, or intent: `vectors of [8 x float].`.
  **L764 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vectors of [8 x float].`。
- **L765 EN**: Separator comment used for visual grouping.
  **L765 CN**: 用于视觉分组的分隔注释。
- **L766 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L766 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L767 EN**: Comment explains nearby logic, constraints, or intent: `result[31:0] (__A[31:0] * __B[31:0]) + __C[31:0]`.
  **L767 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[31:0] (__A[31:0] * __B[31:0]) + __C[31:0]`。
- **L768 EN**: Comment explains nearby logic, constraints, or intent: `result[63:32] (__A[63:32] * __B[63:32]) - __C[63:32]`.
  **L768 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:32] (__A[63:32] * __B[63:32]) - __C[63:32]`。

### Lines 769-792

````c
/// result[95:64] = (__A[95:64] * __B[95:64]) + __C[95:64]
/// result[127:96] = (__A[127:96] * __B[127:96]) - __C[127:96]
/// result[159:128] = (__A[159:128] * __B[159:128]) + __C[159:128]
/// result[191:160] = (__A[191:160] * __B[191:160]) - __C[191:160]
/// result[223:192] = (__A[223:192] * __B[223:192]) + __C[223:192]
/// result[255:224] = (__A[255:224] * __B[255:224]) - __C[255:224]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUBADD213PS instruction.
///
/// \param __A
///    A 256-bit vector of [8 x float] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [8 x float] containing the multiplier.
/// \param __C
///    A 256-bit vector of [8 x float] containing the addend/subtrahend.
/// \returns A 256-bit vector of [8 x float] containing the result.
static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_fmsubadd_ps(__m256 __A, __m256 __B, __m256 __C)
{
  return (__m256)__builtin_ia32_vfmaddsubps256((__v8sf)__A, (__v8sf)__B, -(__v8sf)__C);
}
````
- **L769 EN**: Comment explains nearby logic, constraints, or intent: `result[95:64] (__A[95:64] * __B[95:64]) + __C[95:64]`.
  **L769 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[95:64] (__A[95:64] * __B[95:64]) + __C[95:64]`。
- **L770 EN**: Comment explains nearby logic, constraints, or intent: `result[127:96] (__A[127:96] * __B[127:96]) - __C[127:96]`.
  **L770 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:96] (__A[127:96] * __B[127:96]) - __C[127:96]`。
- **L771 EN**: Comment explains nearby logic, constraints, or intent: `result[159:128] (__A[159:128] * __B[159:128]) + __C[159:128]`.
  **L771 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[159:128] (__A[159:128] * __B[159:128]) + __C[159:128]`。
- **L772 EN**: Comment explains nearby logic, constraints, or intent: `result[191:160] (__A[191:160] * __B[191:160]) - __C[191:160]`.
  **L772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[191:160] (__A[191:160] * __B[191:160]) - __C[191:160]`。
- **L773 EN**: Comment explains nearby logic, constraints, or intent: `result[223:192] (__A[223:192] * __B[223:192]) + __C[223:192]`.
  **L773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[223:192] (__A[223:192] * __B[223:192]) + __C[223:192]`。
- **L774 EN**: Comment explains nearby logic, constraints, or intent: `result[255:224] (__A[255:224] * __B[255:224]) - __C[255:224]`.
  **L774 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[255:224] (__A[255:224] * __B[255:224]) - __C[255:224]`。
- **L775 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L775 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L776 EN**: Separator comment used for visual grouping.
  **L776 CN**: 用于视觉分组的分隔注释。
- **L777 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L777 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L778 EN**: Separator comment used for visual grouping.
  **L778 CN**: 用于视觉分组的分隔注释。
- **L779 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUBADD213PS instruction.`.
  **L779 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUBADD213PS instruction.`。
- **L780 EN**: Separator comment used for visual grouping.
  **L780 CN**: 用于视觉分组的分隔注释。
- **L781 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L781 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L782 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplicand.`.
  **L782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplicand.`。
- **L783 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L784 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the multiplier.`.
  **L784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the multiplier.`。
- **L785 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L785 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L786 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float] containing the addend/subtrahend.`.
  **L786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float] containing the addend/subtrahend.`。
- **L787 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [8 x float] containing the result.`.
  **L787 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [8 x float] containing the result.`。
- **L788 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L788 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L789 EN**: Continues logic associated with callable symbol `_mm256_fmsubadd_ps`.
  **L789 CN**: 继续与可调用符号 `_mm256_fmsubadd_ps` 相关的逻辑。
- **L790 EN**: Opens a new lexical scope or compound statement.
  **L790 CN**: 打开一个新的词法作用域或复合语句块。
- **L791 EN**: Returns from the current function with `(__m256)__builtin_ia32_vfmaddsubps256((__v8sf)__A, (__v8sf)__B, -(__v8sf)__C)`.
  **L791 CN**: 以 `(__m256)__builtin_ia32_vfmaddsubps256((__v8sf)__A, (__v8sf)__B, -(__v8sf)__C)` 从当前函数返回。
- **L792 EN**: Closes the current lexical scope or compound statement.
  **L792 CN**: 结束当前词法作用域或复合语句块。

### Lines 793-816

````c

/// Computes a vector multiply with alternating add/subtract of 256-bit
///    vectors of [4 x double].
///
/// \code{.operation}
/// result[63:0] = (__A[63:0] * __B[63:0]) + __C[63:0]
/// result[127:64] = (__A[127:64] * __B[127:64]) - __C[127:64]
/// result[191:128] = (__A[191:128] * __B[191:128]) + __C[191:128]
/// result[255:192] = (__A[255:192] * __B[255:192]) - __C[255:192]
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VFMSUBADD213PD instruction.
///
/// \param __A
///    A 256-bit vector of [4 x double] containing the multiplicand.
/// \param __B
///    A 256-bit vector of [4 x double] containing the multiplier.
/// \param __C
///    A 256-bit vector of [4 x double] containing the addend/subtrahend.
/// \returns A 256-bit vector of [4 x double] containing the result.
static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_fmsubadd_pd(__m256d __A, __m256d __B, __m256d __C)
````
- **L793 EN**: Blank line separating nearby declarations or logic blocks.
  **L793 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L794 EN**: Comment explains nearby logic, constraints, or intent: `Computes a vector multiply with alternating add/subtract of 256-bit`.
  **L794 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Computes a vector multiply with alternating add/subtract of 256-bit`。
- **L795 EN**: Comment explains nearby logic, constraints, or intent: `vectors of [4 x double].`.
  **L795 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vectors of [4 x double].`。
- **L796 EN**: Separator comment used for visual grouping.
  **L796 CN**: 用于视觉分组的分隔注释。
- **L797 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L797 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L798 EN**: Comment explains nearby logic, constraints, or intent: `result[63:0] (__A[63:0] * __B[63:0]) + __C[63:0]`.
  **L798 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[63:0] (__A[63:0] * __B[63:0]) + __C[63:0]`。
- **L799 EN**: Comment explains nearby logic, constraints, or intent: `result[127:64] (__A[127:64] * __B[127:64]) - __C[127:64]`.
  **L799 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[127:64] (__A[127:64] * __B[127:64]) - __C[127:64]`。
- **L800 EN**: Comment explains nearby logic, constraints, or intent: `result[191:128] (__A[191:128] * __B[191:128]) + __C[191:128]`.
  **L800 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[191:128] (__A[191:128] * __B[191:128]) + __C[191:128]`。
- **L801 EN**: Comment explains nearby logic, constraints, or intent: `result[255:192] (__A[255:192] * __B[255:192]) - __C[255:192]`.
  **L801 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result[255:192] (__A[255:192] * __B[255:192]) - __C[255:192]`。
- **L802 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L802 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L803 EN**: Separator comment used for visual grouping.
  **L803 CN**: 用于视觉分组的分隔注释。
- **L804 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L804 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L805 EN**: Separator comment used for visual grouping.
  **L805 CN**: 用于视觉分组的分隔注释。
- **L806 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VFMSUBADD213PD instruction.`.
  **L806 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VFMSUBADD213PD instruction.`。
- **L807 EN**: Separator comment used for visual grouping.
  **L807 CN**: 用于视觉分组的分隔注释。
- **L808 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L808 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L809 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplicand.`.
  **L809 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplicand.`。
- **L810 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L810 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L811 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the multiplier.`.
  **L811 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the multiplier.`。
- **L812 EN**: Comment explains nearby logic, constraints, or intent: `param __C`.
  **L812 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __C`。
- **L813 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [4 x double] containing the addend/subtrahend.`.
  **L813 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [4 x double] containing the addend/subtrahend.`。
- **L814 EN**: Comment explains nearby logic, constraints, or intent: `returns A 256-bit vector of [4 x double] containing the result.`.
  **L814 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 256-bit vector of [4 x double] containing the result.`。
- **L815 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L815 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L816 EN**: Continues logic associated with callable symbol `_mm256_fmsubadd_pd`.
  **L816 CN**: 继续与可调用符号 `_mm256_fmsubadd_pd` 相关的逻辑。

### Lines 817-826

````c
{
  return (__m256d)__builtin_ia32_vfmaddsubpd256((__v4df)__A, (__v4df)__B, -(__v4df)__C);
}

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256
#undef __DEFAULT_FN_ATTRS128_CONSTEXPR
#undef __DEFAULT_FN_ATTRS256_CONSTEXPR

#endif /* __FMAINTRIN_H */
````
- **L817 EN**: Opens a new lexical scope or compound statement.
  **L817 CN**: 打开一个新的词法作用域或复合语句块。
- **L818 EN**: Returns from the current function with `(__m256d)__builtin_ia32_vfmaddsubpd256((__v4df)__A, (__v4df)__B, -(__v4df)__C)`.
  **L818 CN**: 以 `(__m256d)__builtin_ia32_vfmaddsubpd256((__v4df)__A, (__v4df)__B, -(__v4df)__C)` 从当前函数返回。
- **L819 EN**: Closes the current lexical scope or compound statement.
  **L819 CN**: 结束当前词法作用域或复合语句块。
- **L820 EN**: Blank line separating nearby declarations or logic blocks.
  **L820 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L821 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L821 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L822 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L822 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。
- **L823 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L823 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L824 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L824 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Closes the current preprocessor conditional block.
  **L826 CN**: 结束当前预处理条件块。

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
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__FMAINTRIN_H`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_elementwise_fma`, `__builtin_ia32_vfmaddsubps`, `__builtin_ia32_vfmaddsubpd`, `__builtin_ia32_vfmaddsubps256`, `__builtin_ia32_vfmaddsubpd256`
