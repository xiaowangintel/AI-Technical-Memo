# avx10_2convertintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx10_2convertintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: AVX10_2CONVERT.
- **Purpose (CN)**: 该头文件主要作用是：AVX10_2CONVERT。
- **Line Count / 行数**: 3257

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===--------------- avx10_2convertintrin.h - AVX10_2CONVERT ---------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */
#ifndef __IMMINTRIN_H
#error                                                                         \
    "Never use <avx10_2convertintrin.h> directly; include <immintrin.h> instead."
#endif // __IMMINTRIN_H

#ifdef __SSE2__

#ifndef __AVX10_2CONVERTINTRIN_H
#define __AVX10_2CONVERTINTRIN_H

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS128                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \
                 __min_vector_width__(128)))
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
- **L11 EN**: Continues the surrounding expression or declaration: `"Never use <avx10_2convertintrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 继续构造周围的表达式或声明：`"Never use <avx10_2convertintrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifdef __SSE2__`.
  **L14 CN**: 开始一个预处理条件块：`#ifdef __SSE2__`。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Starts a preprocessor conditional block: `#ifndef __AVX10_2CONVERTINTRIN_H`.
  **L16 CN**: 开始一个预处理条件块：`#ifndef __AVX10_2CONVERTINTRIN_H`。
- **L17 EN**: Defines macro `__AVX10_2CONVERTINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L17 CN**: 定义宏 `__AVX10_2CONVERTINTRIN_H`，用于条件编译、简写或 API 生成。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L20 EN**: Defines macro `__DEFAULT_FN_ATTRS128` for conditional compilation, shorthand, or API generation.
  **L20 CN**: 定义宏 `__DEFAULT_FN_ATTRS128`，用于条件编译、简写或 API 生成。
- **L21 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`.
  **L21 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`。
- **L22 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L22 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L23 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L23 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L24 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`.
  **L24 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`。

### Lines 25-48

````c
                 __min_vector_width__(256)))

// clang-format off

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed
///    single-precision (32-bit) floating-point elements to a 128-bit vector
///    containing FP16 elements.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF i < 4
/// 		dst.fp16[i] := convert_fp32_to_fp16(__B.fp32[i])
/// 	ELSE
/// 		dst.fp16[i] := convert_fp32_to_fp16(__A.fp32[i - 4])
/// 	FI
///
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PS2PHX instruction.
````
- **L25 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L25 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L27 EN**: Comment explains nearby logic, constraints, or intent: `clang-format off`.
  **L27 CN**: 注释解释附近代码的逻辑、约束或设计意图：`clang-format off`。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L29 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed`.
  **L29 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed`。
- **L30 EN**: Comment explains nearby logic, constraints, or intent: `single-precision (32-bit) floating-point elements to a 128-bit vector`.
  **L30 CN**: 注释解释附近代码的逻辑、约束或设计意图：`single-precision (32-bit) floating-point elements to a 128-bit vector`。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `containing FP16 elements.`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`containing FP16 elements.`。
- **L32 EN**: Separator comment used for visual grouping.
  **L32 CN**: 用于视觉分组的分隔注释。
- **L33 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L33 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L34 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L34 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L35 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 4`.
  **L35 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 4`。
- **L36 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`.
  **L36 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`。
- **L37 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L37 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L38 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 4])`.
  **L38 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 4])`。
- **L39 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L39 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L40 EN**: Separator comment used for visual grouping.
  **L40 CN**: 用于视觉分组的分隔注释。
- **L41 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L41 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L42 EN**: Separator comment used for visual grouping.
  **L42 CN**: 用于视觉分组的分隔注释。
- **L43 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L43 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L44 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L44 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L45 EN**: Separator comment used for visual grouping.
  **L45 CN**: 用于视觉分组的分隔注释。
- **L46 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L46 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L47 EN**: Separator comment used for visual grouping.
  **L47 CN**: 用于视觉分组的分隔注释。
- **L48 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PS2PHX instruction.`.
  **L48 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PS2PHX instruction.`。

### Lines 49-72

````c
///
/// \param __A
///    A 128-bit vector of [4 x float].
/// \param __B
///    A 128-bit vector of [4 x float].
/// \returns
///    A 128-bit vector of [8 x fp16]. Lower 4 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtx2ps_ph(__m128 __A,
                                                               __m128 __B) {
  return (__m128h)__builtin_ia32_vcvt2ps2phx128_mask(
      (__v4sf)__A, (__v4sf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)(-1));
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed
///    single-precision (32-bit) floating-point elements to a 128-bit vector
///    containing FP16 elements. Merging mask \a __U is used to determine if given
///    element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		IF i < 4
````
- **L49 EN**: Separator comment used for visual grouping.
  **L49 CN**: 用于视觉分组的分隔注释。
- **L50 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L50 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L51 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L51 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L52 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L52 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L53 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L53 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L54 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L54 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L55 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Lower 4 elements correspond to the`.
  **L55 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Lower 4 elements correspond to the`。
- **L56 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L56 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtx2ps_ph(__m128 __A,`.
  **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtx2ps_ph(__m128 __A,`。
- **L59 EN**: Continues the surrounding expression or declaration: `__m128 __B) {`.
  **L59 CN**: 继续构造周围的表达式或声明：`__m128 __B) {`。
- **L60 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvt2ps2phx128_mask(`.
  **L60 CN**: 以 `(__m128h)__builtin_ia32_vcvt2ps2phx128_mask(` 从当前函数返回。
- **L61 EN**: Executes a call or declaration centered on `statement`.
  **L61 CN**: 执行以 `statement` 为核心的调用或声明。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed`。
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `single-precision (32-bit) floating-point elements to a 128-bit vector`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`single-precision (32-bit) floating-point elements to a 128-bit vector`。
- **L66 EN**: Comment explains nearby logic, constraints, or intent: `containing FP16 elements. Merging mask a __U is used to determine if given`.
  **L66 CN**: 注释解释附近代码的逻辑、约束或设计意图：`containing FP16 elements. Merging mask a __U is used to determine if given`。
- **L67 EN**: Comment explains nearby logic, constraints, or intent: `element should be taken from a __W instead.`.
  **L67 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element should be taken from a __W instead.`。
- **L68 EN**: Separator comment used for visual grouping.
  **L68 CN**: 用于视觉分组的分隔注释。
- **L69 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L69 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L71 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L71 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L72 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 4`.
  **L72 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 4`。

### Lines 73-96

````c
/// 			dst.fp16[i] := convert_fp32_to_fp16(__B.fp32[i])
/// 		ELSE
/// 			dst.fp16[i] := convert_fp32_to_fp16(__A.fp32[i - 4])
/// 		FI
/// 	ELSE
/// 		dst.fp16[i] := __W.fp16[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PS2PHX instruction.
///
/// \param __W
///    A 128-bit vector of [8 x fp16].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [4 x float].
/// \param __B
///    A 128-bit vector of [4 x float].
````
- **L73 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`.
  **L73 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`。
- **L74 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L74 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 4])`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 4])`。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L77 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L77 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L78 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : __W.fp16[i]`.
  **L78 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : __W.fp16[i]`。
- **L79 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L79 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L81 EN**: Separator comment used for visual grouping.
  **L81 CN**: 用于视觉分组的分隔注释。
- **L82 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L82 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L83 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L83 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L84 EN**: Separator comment used for visual grouping.
  **L84 CN**: 用于视觉分组的分隔注释。
- **L85 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L85 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L86 EN**: Separator comment used for visual grouping.
  **L86 CN**: 用于视觉分组的分隔注释。
- **L87 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PS2PHX instruction.`.
  **L87 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PS2PHX instruction.`。
- **L88 EN**: Separator comment used for visual grouping.
  **L88 CN**: 用于视觉分组的分隔注释。
- **L89 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L89 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L90 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L90 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L92 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L92 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L93 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L93 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L94 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L94 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。

### Lines 97-120

````c
/// \returns
///    A 128-bit vector of [8 x fp16]. Lower elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_cvtx2ps_ph(__m128h __W, __mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128h)__builtin_ia32_vcvt2ps2phx128_mask(
      (__v4sf)__A, (__v4sf)__B, (__v8hf)__W, (__mmask8)__U);
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed
///    single-precision (32-bit) floating-point elements to a 128-bit vector
///    containing FP16 elements. Zeroing mask \a __U is used to determine if given
///    element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		IF i < 4
/// 			dst.fp16[i] := convert_fp32_to_fp16(__B.fp32[i])
/// 		ELSE
/// 			dst.fp16[i] := convert_fp32_to_fp16(__A.fp32[i - 4])
/// 		FI
````
- **L97 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L97 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L98 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Lower elements correspond to the`.
  **L98 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Lower elements correspond to the`。
- **L99 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L99 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L100 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L101 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L102 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L102 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L103 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtx2ps_ph(__m128h __W, __mmask8 __U, __m128 __A, __m128 __B) {`.
  **L103 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtx2ps_ph(__m128h __W, __mmask8 __U, __m128 __A, __m128 __B) {`。
- **L104 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvt2ps2phx128_mask(`.
  **L104 CN**: 以 `(__m128h)__builtin_ia32_vcvt2ps2phx128_mask(` 从当前函数返回。
- **L105 EN**: Executes a call or declaration centered on `statement`.
  **L105 CN**: 执行以 `statement` 为核心的调用或声明。
- **L106 EN**: Closes the current lexical scope or compound statement.
  **L106 CN**: 结束当前词法作用域或复合语句块。
- **L107 EN**: Blank line separating nearby declarations or logic blocks.
  **L107 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L108 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed`.
  **L108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed`。
- **L109 EN**: Comment explains nearby logic, constraints, or intent: `single-precision (32-bit) floating-point elements to a 128-bit vector`.
  **L109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`single-precision (32-bit) floating-point elements to a 128-bit vector`。
- **L110 EN**: Comment explains nearby logic, constraints, or intent: `containing FP16 elements. Zeroing mask a __U is used to determine if given`.
  **L110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`containing FP16 elements. Zeroing mask a __U is used to determine if given`。
- **L111 EN**: Comment explains nearby logic, constraints, or intent: `element should be zeroed instead.`.
  **L111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element should be zeroed instead.`。
- **L112 EN**: Separator comment used for visual grouping.
  **L112 CN**: 用于视觉分组的分隔注释。
- **L113 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L114 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L115 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L116 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 4`.
  **L116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 4`。
- **L117 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`.
  **L117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`。
- **L118 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L119 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 4])`.
  **L119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 4])`。
- **L120 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。

### Lines 121-144

````c
/// 	ELSE
/// 		dst.fp16[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PS2PHX instruction.
///
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [4 x float].
/// \param __B
///    A 128-bit vector of [4 x float].
/// \returns
///    A 128-bit vector of [8 x fp16]. Lower elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    then zero is taken instead.
static __inline__ __m128h __DEFAULT_FN_ATTRS128
````
- **L121 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L122 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : 0`.
  **L122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : 0`。
- **L123 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L124 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L125 EN**: Separator comment used for visual grouping.
  **L125 CN**: 用于视觉分组的分隔注释。
- **L126 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L127 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L128 EN**: Separator comment used for visual grouping.
  **L128 CN**: 用于视觉分组的分隔注释。
- **L129 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L130 EN**: Separator comment used for visual grouping.
  **L130 CN**: 用于视觉分组的分隔注释。
- **L131 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PS2PHX instruction.`.
  **L131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PS2PHX instruction.`。
- **L132 EN**: Separator comment used for visual grouping.
  **L132 CN**: 用于视觉分组的分隔注释。
- **L133 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L134 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L135 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L136 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L136 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L137 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L138 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x float].`.
  **L138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x float].`。
- **L139 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L140 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Lower elements correspond to the`.
  **L140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Lower elements correspond to the`。
- **L141 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L142 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `then zero is taken instead.`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`then zero is taken instead.`。
- **L144 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L144 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。

### Lines 145-168

````c
_mm_maskz_cvtx2ps_ph(__mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128h)__builtin_ia32_vcvt2ps2phx128_mask(
      (__v4sf)__A, (__v4sf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed
///    single-precision (32-bit) floating-point elements to a 256-bit vector
///    containing FP16 elements.
///   
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF i < 8
/// 		dst.fp16[i] := convert_fp32_to_fp16(__B.fp32[i])
/// 	ELSE
/// 		dst.fp16[i] := convert_fp32_to_fp16(__A.fp32[i - 8])
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PS2PHX instruction.
````
- **L145 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtx2ps_ph(__mmask8 __U, __m128 __A, __m128 __B) {`.
  **L145 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtx2ps_ph(__mmask8 __U, __m128 __A, __m128 __B) {`。
- **L146 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvt2ps2phx128_mask(`.
  **L146 CN**: 以 `(__m128h)__builtin_ia32_vcvt2ps2phx128_mask(` 从当前函数返回。
- **L147 EN**: Executes a call or declaration centered on `statement`.
  **L147 CN**: 执行以 `statement` 为核心的调用或声明。
- **L148 EN**: Closes the current lexical scope or compound statement.
  **L148 CN**: 结束当前词法作用域或复合语句块。
- **L149 EN**: Blank line separating nearby declarations or logic blocks.
  **L149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L150 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed`.
  **L150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed`。
- **L151 EN**: Comment explains nearby logic, constraints, or intent: `single-precision (32-bit) floating-point elements to a 256-bit vector`.
  **L151 CN**: 注释解释附近代码的逻辑、约束或设计意图：`single-precision (32-bit) floating-point elements to a 256-bit vector`。
- **L152 EN**: Comment explains nearby logic, constraints, or intent: `containing FP16 elements.`.
  **L152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`containing FP16 elements.`。
- **L153 EN**: Separator comment used for visual grouping.
  **L153 CN**: 用于视觉分组的分隔注释。
- **L154 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L155 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L156 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L156 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L157 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`.
  **L157 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`。
- **L158 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L159 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 8])`.
  **L159 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 8])`。
- **L160 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L161 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L162 EN**: Separator comment used for visual grouping.
  **L162 CN**: 用于视觉分组的分隔注释。
- **L163 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L163 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L164 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L164 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L165 EN**: Separator comment used for visual grouping.
  **L165 CN**: 用于视觉分组的分隔注释。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L167 EN**: Separator comment used for visual grouping.
  **L167 CN**: 用于视觉分组的分隔注释。
- **L168 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PS2PHX instruction.`.
  **L168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PS2PHX instruction.`。

### Lines 169-192

````c
///
/// \param __A
///    A 256-bit vector of [8 x float].
/// \param __B
///    A 256-bit vector of [8 x float].
/// \returns
///    A 256-bit vector of [16 x fp16]. Lower elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvtx2ps_ph(__m256 __A,
                                                                  __m256 __B) {
  return (__m256h)__builtin_ia32_vcvt2ps2phx256_mask(
      (__v8sf)__A, (__v8sf)__B, (__v16hf)_mm256_setzero_ph(), (__mmask16)(-1));
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed
///    single-precision (32-bit) floating-point elements to a 256-bit vector
///    containing FP16 elements. Merging mask \a __U is used to determine if given
///    element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		IF i < 8
````
- **L169 EN**: Separator comment used for visual grouping.
  **L169 CN**: 用于视觉分组的分隔注释。
- **L170 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L170 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L171 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float].`.
  **L171 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float].`。
- **L172 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L173 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float].`.
  **L173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float].`。
- **L174 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L175 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Lower elements correspond to the`.
  **L175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Lower elements correspond to the`。
- **L176 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L176 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L177 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvtx2ps_ph(__m256 __A,`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvtx2ps_ph(__m256 __A,`。
- **L179 EN**: Continues the surrounding expression or declaration: `__m256 __B) {`.
  **L179 CN**: 继续构造周围的表达式或声明：`__m256 __B) {`。
- **L180 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvt2ps2phx256_mask(`.
  **L180 CN**: 以 `(__m256h)__builtin_ia32_vcvt2ps2phx256_mask(` 从当前函数返回。
- **L181 EN**: Executes a call or declaration centered on `statement`.
  **L181 CN**: 执行以 `statement` 为核心的调用或声明。
- **L182 EN**: Closes the current lexical scope or compound statement.
  **L182 CN**: 结束当前词法作用域或复合语句块。
- **L183 EN**: Blank line separating nearby declarations or logic blocks.
  **L183 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L184 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed`.
  **L184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed`。
- **L185 EN**: Comment explains nearby logic, constraints, or intent: `single-precision (32-bit) floating-point elements to a 256-bit vector`.
  **L185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`single-precision (32-bit) floating-point elements to a 256-bit vector`。
- **L186 EN**: Comment explains nearby logic, constraints, or intent: `containing FP16 elements. Merging mask a __U is used to determine if given`.
  **L186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`containing FP16 elements. Merging mask a __U is used to determine if given`。
- **L187 EN**: Comment explains nearby logic, constraints, or intent: `element should be taken from a __W instead.`.
  **L187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element should be taken from a __W instead.`。
- **L188 EN**: Separator comment used for visual grouping.
  **L188 CN**: 用于视觉分组的分隔注释。
- **L189 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L190 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L191 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L192 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。

### Lines 193-216

````c
/// 			dst.fp16[i] := convert_fp32_to_fp16(__B.fp32[i])
/// 		ELSE
/// 			dst.fp16[i] := convert_fp32_to_fp16(__A.fp32[i - 8])
/// 		FI
/// 	ELSE
/// 		dst.fp16[i] := __W.fp16[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PS2PHX instruction.
///
/// \param __W
///    A 256-bit vector of [16 x fp16].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [8 x float].
/// \param __B
///    A 256-bit vector of [8 x float].
````
- **L193 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`.
  **L193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L195 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 8])`.
  **L195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 8])`。
- **L196 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L197 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L198 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : __W.fp16[i]`.
  **L198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : __W.fp16[i]`。
- **L199 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L199 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L200 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L201 EN**: Separator comment used for visual grouping.
  **L201 CN**: 用于视觉分组的分隔注释。
- **L202 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L203 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L204 EN**: Separator comment used for visual grouping.
  **L204 CN**: 用于视觉分组的分隔注释。
- **L205 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L206 EN**: Separator comment used for visual grouping.
  **L206 CN**: 用于视觉分组的分隔注释。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PS2PHX instruction.`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PS2PHX instruction.`。
- **L208 EN**: Separator comment used for visual grouping.
  **L208 CN**: 用于视觉分组的分隔注释。
- **L209 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L210 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L211 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L212 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L213 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L213 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L214 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float].`.
  **L214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float].`。
- **L215 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L215 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L216 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float].`.
  **L216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float].`。

### Lines 217-240

````c
/// \returns
///    A 256-bit vector of [16 x fp16]. Lower elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_cvtx2ps_ph(__m256h __W, __mmask16 __U, __m256 __A, __m256 __B) {
  return (__m256h)__builtin_ia32_vcvt2ps2phx256_mask(
      (__v8sf)__A, (__v8sf)__B, (__v16hf)__W, (__mmask16)__U);
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed
///    single-precision (32-bit) floating-point elements to a 256-bit vector
///    containing FP16 elements. Zeroing mask \a __U is used to determine if given
///    element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		IF i < 8
/// 			dst.fp16[i] := convert_fp32_to_fp16(__B.fp32[i])
/// 		ELSE
/// 			dst.fp16[i] := convert_fp32_to_fp16(__A.fp32[i - 8])
/// 		FI
````
- **L217 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L218 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Lower elements correspond to the`.
  **L218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Lower elements correspond to the`。
- **L219 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L220 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L221 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L222 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L222 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L223 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtx2ps_ph(__m256h __W, __mmask16 __U, __m256 __A, __m256 __B) {`.
  **L223 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtx2ps_ph(__m256h __W, __mmask16 __U, __m256 __A, __m256 __B) {`。
- **L224 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvt2ps2phx256_mask(`.
  **L224 CN**: 以 `(__m256h)__builtin_ia32_vcvt2ps2phx256_mask(` 从当前函数返回。
- **L225 EN**: Executes a call or declaration centered on `statement`.
  **L225 CN**: 执行以 `statement` 为核心的调用或声明。
- **L226 EN**: Closes the current lexical scope or compound statement.
  **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Blank line separating nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L228 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed`.
  **L228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed`。
- **L229 EN**: Comment explains nearby logic, constraints, or intent: `single-precision (32-bit) floating-point elements to a 256-bit vector`.
  **L229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`single-precision (32-bit) floating-point elements to a 256-bit vector`。
- **L230 EN**: Comment explains nearby logic, constraints, or intent: `containing FP16 elements. Zeroing mask a __U is used to determine if given`.
  **L230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`containing FP16 elements. Zeroing mask a __U is used to determine if given`。
- **L231 EN**: Comment explains nearby logic, constraints, or intent: `element should be zeroed instead.`.
  **L231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element should be zeroed instead.`。
- **L232 EN**: Separator comment used for visual grouping.
  **L232 CN**: 用于视觉分组的分隔注释。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L234 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L235 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L236 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L237 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`.
  **L237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__B.fp32[i])`。
- **L238 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L239 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 8])`.
  **L239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_fp32_to_fp16(__A.fp32[i - 8])`。
- **L240 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。

### Lines 241-264

````c
/// 	ELSE
/// 		dst.fp16[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PS2PHX instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [8 x float].
/// \param __B
///    A 256-bit vector of [8 x float].
/// \returns
///    A 256-bit vector of [16 x fp16]. Lower elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    then zero is taken instead.
static __inline__ __m256h __DEFAULT_FN_ATTRS256
````
- **L241 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L242 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : 0`.
  **L242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : 0`。
- **L243 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L244 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L245 EN**: Separator comment used for visual grouping.
  **L245 CN**: 用于视觉分组的分隔注释。
- **L246 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L247 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L247 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L248 EN**: Separator comment used for visual grouping.
  **L248 CN**: 用于视觉分组的分隔注释。
- **L249 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L250 EN**: Separator comment used for visual grouping.
  **L250 CN**: 用于视觉分组的分隔注释。
- **L251 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PS2PHX instruction.`.
  **L251 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PS2PHX instruction.`。
- **L252 EN**: Separator comment used for visual grouping.
  **L252 CN**: 用于视觉分组的分隔注释。
- **L253 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L254 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L254 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L255 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L256 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float].`.
  **L256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float].`。
- **L257 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L258 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x float].`.
  **L258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x float].`。
- **L259 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L260 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Lower elements correspond to the`.
  **L260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Lower elements correspond to the`。
- **L261 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L262 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L263 EN**: Comment explains nearby logic, constraints, or intent: `then zero is taken instead.`.
  **L263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`then zero is taken instead.`。
- **L264 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L264 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。

### Lines 265-288

````c
_mm256_maskz_cvtx2ps_ph(__mmask16 __U, __m256 __A, __m256 __B) {
  return (__m256h)__builtin_ia32_vcvt2ps2phx256_mask(
      (__v8sf)__A, (__v8sf)__B, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.bf8[i] := convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8 instruction.
///
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
````
- **L265 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtx2ps_ph(__mmask16 __U, __m256 __A, __m256 __B) {`.
  **L265 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtx2ps_ph(__mmask16 __U, __m256 __A, __m256 __B) {`。
- **L266 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvt2ps2phx256_mask(`.
  **L266 CN**: 以 `(__m256h)__builtin_ia32_vcvt2ps2phx256_mask(` 从当前函数返回。
- **L267 EN**: Executes a call or declaration centered on `statement`.
  **L267 CN**: 执行以 `statement` 为核心的调用或声明。
- **L268 EN**: Closes the current lexical scope or compound statement.
  **L268 CN**: 结束当前词法作用域或复合语句块。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L270 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L270 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L271 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L271 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L272 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B.`.
  **L272 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B.`。
- **L273 EN**: Separator comment used for visual grouping.
  **L273 CN**: 用于视觉分组的分隔注释。
- **L274 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L274 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L275 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L275 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L276 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L277 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L278 EN**: Separator comment used for visual grouping.
  **L278 CN**: 用于视觉分组的分隔注释。
- **L279 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L280 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L281 EN**: Separator comment used for visual grouping.
  **L281 CN**: 用于视觉分组的分隔注释。
- **L282 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L282 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L283 EN**: Separator comment used for visual grouping.
  **L283 CN**: 用于视觉分组的分隔注释。
- **L284 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`.
  **L284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`。
- **L285 EN**: Separator comment used for visual grouping.
  **L285 CN**: 用于视觉分组的分隔注释。
- **L286 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L286 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L287 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L288 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。

### Lines 289-312

````c
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    converted elements from \a __B using biases from \a __A; higher order
///    elements are zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtbiasph_bf8(__m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(
      (__v16qi)__A, (__v8hf)__B, (__v16qi)_mm_undefined_si128(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Merging mask \a __U is used to determine if
///    given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
````
- **L289 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L289 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L290 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L290 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L291 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L291 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L292 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B using biases from a __A; higher order`.
  **L292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B using biases from a __A; higher order`。
- **L293 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed.`.
  **L293 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed.`。
- **L294 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L294 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L295 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtbiasph_bf8(__m128i __A, __m128h __B) {`.
  **L295 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtbiasph_bf8(__m128i __A, __m128h __B) {`。
- **L296 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(`.
  **L296 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(` 从当前函数返回。
- **L297 EN**: Executes a call or declaration centered on `statement`.
  **L297 CN**: 执行以 `statement` 为核心的调用或声明。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L300 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L300 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L301 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L302 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Merging mask a __U is used to determine if`.
  **L302 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Merging mask a __U is used to determine if`。
- **L303 EN**: Comment explains nearby logic, constraints, or intent: `given element should be taken from a __W instead.`.
  **L303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be taken from a __W instead.`。
- **L304 EN**: Separator comment used for visual grouping.
  **L304 CN**: 用于视觉分组的分隔注释。
- **L305 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L306 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L307 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L308 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L309 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L310 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L311 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L311 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L312 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。

### Lines 313-336

````c
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    converted elements from \a __B, using biases from \a __A; higher order
///    elements are zeroed. If corresponding mask bit is not set, then element
///    from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtbiasph_bf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(
````
- **L313 EN**: Separator comment used for visual grouping.
  **L313 CN**: 用于视觉分组的分隔注释。
- **L314 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L314 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L315 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L316 EN**: Separator comment used for visual grouping.
  **L316 CN**: 用于视觉分组的分隔注释。
- **L317 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L317 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L318 EN**: Separator comment used for visual grouping.
  **L318 CN**: 用于视觉分组的分隔注释。
- **L319 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`.
  **L319 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`。
- **L320 EN**: Separator comment used for visual grouping.
  **L320 CN**: 用于视觉分组的分隔注释。
- **L321 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L321 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L322 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L322 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L323 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L323 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L324 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L325 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L326 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L326 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L327 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L328 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L328 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L329 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L329 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L330 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L330 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L331 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B, using biases from a __A; higher order`.
  **L331 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B, using biases from a __A; higher order`。
- **L332 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed. If corresponding mask bit is not set, then element`.
  **L332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed. If corresponding mask bit is not set, then element`。
- **L333 EN**: Comment explains nearby logic, constraints, or intent: `from a __W is taken instead.`.
  **L333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W is taken instead.`。
- **L334 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L334 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L335 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtbiasph_bf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {`.
  **L335 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtbiasph_bf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {`。
- **L336 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(`.
  **L336 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(` 从当前函数返回。

### Lines 337-360

````c
      (__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)__W, (__mmask8)__U);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Zeroing mask \a __U is used to determine if
///    given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
///	 	dst.bf8[i] := convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])
///	 ELSE
///	 	dst.bf8[i] := 0
///	 FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8 instruction.
///
````
- **L337 EN**: Executes a call or declaration centered on `statement`.
  **L337 CN**: 执行以 `statement` 为核心的调用或声明。
- **L338 EN**: Closes the current lexical scope or compound statement.
  **L338 CN**: 结束当前词法作用域或复合语句块。
- **L339 EN**: Blank line separating nearby declarations or logic blocks.
  **L339 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L340 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L340 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L341 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L342 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Zeroing mask a __U is used to determine if`.
  **L342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Zeroing mask a __U is used to determine if`。
- **L343 EN**: Comment explains nearby logic, constraints, or intent: `given element should be zeroed instead.`.
  **L343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be zeroed instead.`。
- **L344 EN**: Separator comment used for visual grouping.
  **L344 CN**: 用于视觉分组的分隔注释。
- **L345 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L346 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L346 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L347 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L347 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L348 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L348 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L349 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L349 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L350 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L350 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L351 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L351 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L352 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L352 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L353 EN**: Separator comment used for visual grouping.
  **L353 CN**: 用于视觉分组的分隔注释。
- **L354 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L355 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L356 EN**: Separator comment used for visual grouping.
  **L356 CN**: 用于视觉分组的分隔注释。
- **L357 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L357 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L358 EN**: Separator comment used for visual grouping.
  **L358 CN**: 用于视觉分组的分隔注释。
- **L359 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`.
  **L359 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`。
- **L360 EN**: Separator comment used for visual grouping.
  **L360 CN**: 用于视觉分组的分隔注释。

### Lines 361-384

````c
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    converted elements from \a __B, using biases from \a __A; higher order
///    elements are zeroed. If corresponding mask bit is not set, then element
///    is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtbiasph_bf8(__mmask8 __U, __m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(
      (__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),
      (__mmask8)__U);
}

/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B.
///
/// \code{.operation}
/// FOR i := 0 to 15
````
- **L361 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L361 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L362 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L362 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L363 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L363 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L364 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L364 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L365 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L366 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L366 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L367 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L368 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L368 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L369 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B, using biases from a __A; higher order`.
  **L369 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B, using biases from a __A; higher order`。
- **L370 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed. If corresponding mask bit is not set, then element`.
  **L370 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed. If corresponding mask bit is not set, then element`。
- **L371 EN**: Comment explains nearby logic, constraints, or intent: `is zeroed.`.
  **L371 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is zeroed.`。
- **L372 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L372 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L373 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtbiasph_bf8(__mmask8 __U, __m128i __A, __m128h __B) {`.
  **L373 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtbiasph_bf8(__mmask8 __U, __m128i __A, __m128h __B) {`。
- **L374 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(`.
  **L374 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8_128_mask(` 从当前函数返回。
- **L375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`.
  **L375 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`。
- **L376 EN**: Executes a call or declaration centered on `statement`.
  **L376 CN**: 执行以 `statement` 为核心的调用或声明。
- **L377 EN**: Closes the current lexical scope or compound statement.
  **L377 CN**: 结束当前词法作用域或复合语句块。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L379 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L379 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L380 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L380 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L381 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B.`.
  **L381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B.`。
- **L382 EN**: Separator comment used for visual grouping.
  **L382 CN**: 用于视觉分组的分隔注释。
- **L383 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L383 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L384 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L384 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。

### Lines 385-408

````c
/// 	dst.bf8[i] := convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8 instruction.
///
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Elements correspond to the
///    converted elements from \a __B using biases from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvtbiasph_bf8(__m256i __A, __m256h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),
      (__mmask16)-1);
}

````
- **L385 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L385 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L386 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L386 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L387 EN**: Separator comment used for visual grouping.
  **L387 CN**: 用于视觉分组的分隔注释。
- **L388 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L388 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L389 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L389 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L390 EN**: Separator comment used for visual grouping.
  **L390 CN**: 用于视觉分组的分隔注释。
- **L391 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L391 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L392 EN**: Separator comment used for visual grouping.
  **L392 CN**: 用于视觉分组的分隔注释。
- **L393 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`.
  **L393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`。
- **L394 EN**: Separator comment used for visual grouping.
  **L394 CN**: 用于视觉分组的分隔注释。
- **L395 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L395 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L396 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L397 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L397 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L398 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L399 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L399 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L400 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Elements correspond to the`.
  **L400 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Elements correspond to the`。
- **L401 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B using biases from a __A.`.
  **L401 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B using biases from a __A.`。
- **L402 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L402 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L403 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtbiasph_bf8(__m256i __A, __m256h __B) {`.
  **L403 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtbiasph_bf8(__m256i __A, __m256h __B) {`。
- **L404 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(`.
  **L404 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(` 从当前函数返回。
- **L405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),`.
  **L405 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),`。
- **L406 EN**: Executes a call or declaration centered on `statement`.
  **L406 CN**: 执行以 `statement` 为核心的调用或声明。
- **L407 EN**: Closes the current lexical scope or compound statement.
  **L407 CN**: 结束当前词法作用域或复合语句块。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 409-432

````c
/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Merging mask \a __U is used to determine if
///    given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
````
- **L409 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L409 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L410 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L410 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L411 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Merging mask a __U is used to determine if`.
  **L411 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Merging mask a __U is used to determine if`。
- **L412 EN**: Comment explains nearby logic, constraints, or intent: `given element should be taken from a __W instead.`.
  **L412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be taken from a __W instead.`。
- **L413 EN**: Separator comment used for visual grouping.
  **L413 CN**: 用于视觉分组的分隔注释。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L415 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L415 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L416 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L416 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L417 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L418 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L418 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L419 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L419 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L420 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L420 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L421 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L421 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L422 EN**: Separator comment used for visual grouping.
  **L422 CN**: 用于视觉分组的分隔注释。
- **L423 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L424 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L425 EN**: Separator comment used for visual grouping.
  **L425 CN**: 用于视觉分组的分隔注释。
- **L426 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L427 EN**: Separator comment used for visual grouping.
  **L427 CN**: 用于视觉分组的分隔注释。
- **L428 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`.
  **L428 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`。
- **L429 EN**: Separator comment used for visual grouping.
  **L429 CN**: 用于视觉分组的分隔注释。
- **L430 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L430 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L431 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L432 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L432 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。

### Lines 433-456

````c
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Elements correspond to the converted
///    elements from \a __B, using biases from \a __A. If corresponding mask bit
///    is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256 _mm256_mask_cvtbiasph_bf8(
    __m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)__W, (__mmask16)__U);
}

/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Zeroing mask \a __U is used to determine if
///    given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
///	 	dst.bf8[i] := convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])
````
- **L433 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L434 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L434 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L435 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L435 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L436 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L437 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L439 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Elements correspond to the converted`.
  **L439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Elements correspond to the converted`。
- **L440 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __B, using biases from a __A. If corresponding mask bit`.
  **L440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __B, using biases from a __A. If corresponding mask bit`。
- **L441 EN**: Comment explains nearby logic, constraints, or intent: `is not set, then element from a __W is taken instead.`.
  **L441 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is not set, then element from a __W is taken instead.`。
- **L442 EN**: Continues logic associated with callable symbol `_mm256_mask_cvtbiasph_bf8`.
  **L442 CN**: 继续与可调用符号 `_mm256_mask_cvtbiasph_bf8` 相关的逻辑。
- **L443 EN**: Continues the surrounding expression or declaration: `__m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {`.
  **L443 CN**: 继续构造周围的表达式或声明：`__m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {`。
- **L444 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(`.
  **L444 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(` 从当前函数返回。
- **L445 EN**: Executes a call or declaration centered on `statement`.
  **L445 CN**: 执行以 `statement` 为核心的调用或声明。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L448 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L449 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L450 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Zeroing mask a __U is used to determine if`.
  **L450 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Zeroing mask a __U is used to determine if`。
- **L451 EN**: Comment explains nearby logic, constraints, or intent: `given element should be zeroed instead.`.
  **L451 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be zeroed instead.`。
- **L452 EN**: Separator comment used for visual grouping.
  **L452 CN**: 用于视觉分组的分隔注释。
- **L453 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L454 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L454 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L455 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L455 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L456 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L456 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。

### Lines 457-480

````c
///	 ELSE
///	 	dst.bf8[i] := 0
///	 FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8 instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Elements correspond to the converted
///    elements from \a __B, using biases from \a __A. If corresponding mask bit
///    is not set, then element is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtbiasph_bf8(__mmask16 __U, __m256i __A, __m256h __B) {
````
- **L457 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L458 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L458 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L459 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L459 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L460 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L461 EN**: Separator comment used for visual grouping.
  **L461 CN**: 用于视觉分组的分隔注释。
- **L462 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L463 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L464 EN**: Separator comment used for visual grouping.
  **L464 CN**: 用于视觉分组的分隔注释。
- **L465 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L465 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L466 EN**: Separator comment used for visual grouping.
  **L466 CN**: 用于视觉分组的分隔注释。
- **L467 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`.
  **L467 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`。
- **L468 EN**: Separator comment used for visual grouping.
  **L468 CN**: 用于视觉分组的分隔注释。
- **L469 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L469 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L470 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L470 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L471 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L471 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L472 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L472 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L473 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L473 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L474 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L474 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L475 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L476 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Elements correspond to the converted`.
  **L476 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Elements correspond to the converted`。
- **L477 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __B, using biases from a __A. If corresponding mask bit`.
  **L477 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __B, using biases from a __A. If corresponding mask bit`。
- **L478 EN**: Comment explains nearby logic, constraints, or intent: `is not set, then element is zeroed.`.
  **L478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is not set, then element is zeroed.`。
- **L479 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L479 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L480 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtbiasph_bf8(__mmask16 __U, __m256i __A, __m256h __B) {`.
  **L480 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtbiasph_bf8(__mmask16 __U, __m256i __A, __m256h __B) {`。

### Lines 481-504

````c
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),
      (__mmask16)__U);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.bf8[i] := convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8 instruction.
///
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
````
- **L481 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(`.
  **L481 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8_256_mask(` 从当前函数返回。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`.
  **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`。
- **L483 EN**: Executes a call or declaration centered on `statement`.
  **L483 CN**: 执行以 `statement` 为核心的调用或声明。
- **L484 EN**: Closes the current lexical scope or compound statement.
  **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Blank line separating nearby declarations or logic blocks.
  **L485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L486 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L487 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L487 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L488 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated.`.
  **L488 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated.`。
- **L489 EN**: Separator comment used for visual grouping.
  **L489 CN**: 用于视觉分组的分隔注释。
- **L490 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L490 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L491 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L492 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L492 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L493 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L493 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L494 EN**: Separator comment used for visual grouping.
  **L494 CN**: 用于视觉分组的分隔注释。
- **L495 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L495 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L496 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L496 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L497 EN**: Separator comment used for visual grouping.
  **L497 CN**: 用于视觉分组的分隔注释。
- **L498 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L498 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L499 EN**: Separator comment used for visual grouping.
  **L499 CN**: 用于视觉分组的分隔注释。
- **L500 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`.
  **L500 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8 instruction.`。
- **L501 EN**: Separator comment used for visual grouping.
  **L501 CN**: 用于视觉分组的分隔注释。
- **L502 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L502 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L503 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L503 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L504 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。

### Lines 505-528

````c
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    converted elements from \a __B using biases from \a __A; higher order
///    elements are zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvts_biasph_bf8(__m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask(
      (__v16qi)__A, (__v8hf)__B, (__v16qi)_mm_undefined_si128(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated. Merging mask \a __U
///    is used to determine if given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
````
- **L505 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L505 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L506 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L506 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L507 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L507 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L508 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B using biases from a __A; higher order`.
  **L508 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B using biases from a __A; higher order`。
- **L509 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed.`.
  **L509 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed.`。
- **L510 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L510 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L511 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvts_biasph_bf8(__m128i __A, __m128h __B) {`.
  **L511 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvts_biasph_bf8(__m128i __A, __m128h __B) {`。
- **L512 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask(`.
  **L512 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask(` 从当前函数返回。
- **L513 EN**: Executes a call or declaration centered on `statement`.
  **L513 CN**: 执行以 `statement` 为核心的调用或声明。
- **L514 EN**: Closes the current lexical scope or compound statement.
  **L514 CN**: 结束当前词法作用域或复合语句块。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L516 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L516 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L517 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L518 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated. Merging mask a __U`.
  **L518 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated. Merging mask a __U`。
- **L519 EN**: Comment explains nearby logic, constraints, or intent: `is used to determine if given element should be taken from a __W instead.`.
  **L519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is used to determine if given element should be taken from a __W instead.`。
- **L520 EN**: Separator comment used for visual grouping.
  **L520 CN**: 用于视觉分组的分隔注释。
- **L521 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L521 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L522 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L522 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L523 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L523 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L524 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L524 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L525 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L525 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L526 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L526 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L527 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L527 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L528 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L528 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。

### Lines 529-552

````c
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    converted elements from \a __B, using biases from \a __A; higher order
///    elements are zeroed. If corresponding mask bit is not set, then element
///    from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_mask_cvts_biasph_bf8(__m128i
		__W, __mmask8 __U, __m128i __A, __m128h __B) { return
	(__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask( (__v16qi)__A,
````
- **L529 EN**: Separator comment used for visual grouping.
  **L529 CN**: 用于视觉分组的分隔注释。
- **L530 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L530 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L531 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L531 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L532 EN**: Separator comment used for visual grouping.
  **L532 CN**: 用于视觉分组的分隔注释。
- **L533 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L533 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L534 EN**: Separator comment used for visual grouping.
  **L534 CN**: 用于视觉分组的分隔注释。
- **L535 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`.
  **L535 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`。
- **L536 EN**: Separator comment used for visual grouping.
  **L536 CN**: 用于视觉分组的分隔注释。
- **L537 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L537 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L538 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L538 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L539 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L539 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L540 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L540 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L541 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L541 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L542 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L542 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L543 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L543 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L544 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L544 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L545 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L545 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L546 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L546 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L547 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B, using biases from a __A; higher order`.
  **L547 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B, using biases from a __A; higher order`。
- **L548 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed. If corresponding mask bit is not set, then element`.
  **L548 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed. If corresponding mask bit is not set, then element`。
- **L549 EN**: Comment explains nearby logic, constraints, or intent: `from a __W is taken instead.`.
  **L549 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W is taken instead.`。
- **L550 EN**: Continues logic associated with callable symbol `_mm_mask_cvts_biasph_bf8`.
  **L550 CN**: 继续与可调用符号 `_mm_mask_cvts_biasph_bf8` 相关的逻辑。
- **L551 EN**: Continues the surrounding expression or declaration: `__W, __mmask8 __U, __m128i __A, __m128h __B) { return`.
  **L551 CN**: 继续构造周围的表达式或声明：`__W, __mmask8 __U, __m128i __A, __m128h __B) { return`。
- **L552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask( (__v16qi)__A,`.
  **L552 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask( (__v16qi)__A,`。

### Lines 553-576

````c
			(__v8hf)__B, (__v16qi)(__m128i)__W, (__mmask8)__U); }

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated. Zeroing mask \a __U
///    is used to determine if given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
///	 	dst.bf8[i] := convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
///	 ELSE
///	 	dst.bf8[i] := 0
///	 FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8S instruction.
///
/// \param __U
````
- **L553 EN**: Continues the surrounding expression or declaration: `(__v8hf)__B, (__v16qi)(__m128i)__W, (__mmask8)__U); }`.
  **L553 CN**: 继续构造周围的表达式或声明：`(__v8hf)__B, (__v16qi)(__m128i)__W, (__mmask8)__U); }`。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L555 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L556 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L556 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L557 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated. Zeroing mask a __U`.
  **L557 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated. Zeroing mask a __U`。
- **L558 EN**: Comment explains nearby logic, constraints, or intent: `is used to determine if given element should be zeroed instead.`.
  **L558 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is used to determine if given element should be zeroed instead.`。
- **L559 EN**: Separator comment used for visual grouping.
  **L559 CN**: 用于视觉分组的分隔注释。
- **L560 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L560 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L561 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L561 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L562 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L562 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L563 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L563 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L564 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L564 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L565 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L565 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L566 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L566 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L567 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L567 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L568 EN**: Separator comment used for visual grouping.
  **L568 CN**: 用于视觉分组的分隔注释。
- **L569 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L569 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L570 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L570 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L571 EN**: Separator comment used for visual grouping.
  **L571 CN**: 用于视觉分组的分隔注释。
- **L572 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L572 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L573 EN**: Separator comment used for visual grouping.
  **L573 CN**: 用于视觉分组的分隔注释。
- **L574 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`.
  **L574 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`。
- **L575 EN**: Separator comment used for visual grouping.
  **L575 CN**: 用于视觉分组的分隔注释。
- **L576 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L576 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。

### Lines 577-600

````c
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    converted elements from \a __B, using biases from \a __A; higher order
///    elements are zeroed. If corresponding mask bit is not set, then element
///    is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvts_biasph_bf8(__mmask8 __U, __m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask(
      (__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),
      (__mmask8)__U);
}


/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated.
///
/// \code{.operation}
/// FOR i := 0 to 15
````
- **L577 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L577 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L578 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L578 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L579 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L579 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L580 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L580 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L581 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L581 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L582 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L582 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L583 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L583 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L584 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B, using biases from a __A; higher order`.
  **L584 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B, using biases from a __A; higher order`。
- **L585 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed. If corresponding mask bit is not set, then element`.
  **L585 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed. If corresponding mask bit is not set, then element`。
- **L586 EN**: Comment explains nearby logic, constraints, or intent: `is zeroed.`.
  **L586 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is zeroed.`。
- **L587 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L587 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L588 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvts_biasph_bf8(__mmask8 __U, __m128i __A, __m128h __B) {`.
  **L588 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvts_biasph_bf8(__mmask8 __U, __m128i __A, __m128h __B) {`。
- **L589 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask(`.
  **L589 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_128_mask(` 从当前函数返回。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`。
- **L591 EN**: Executes a call or declaration centered on `statement`.
  **L591 CN**: 执行以 `statement` 为核心的调用或声明。
- **L592 EN**: Closes the current lexical scope or compound statement.
  **L592 CN**: 结束当前词法作用域或复合语句块。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Blank line separating nearby declarations or logic blocks.
  **L594 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L595 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L595 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L596 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L596 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L597 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated.`.
  **L597 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated.`。
- **L598 EN**: Separator comment used for visual grouping.
  **L598 CN**: 用于视觉分组的分隔注释。
- **L599 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L599 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L600 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L600 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。

### Lines 601-624

````c
/// 	dst.bf8[i] := convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8S instruction.
///
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Elements correspond to the
///    converted elements from \a __B using biases from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvts_biasph_bf8(__m256i __A, __m256h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),
      (__mmask16)-1);
}

````
- **L601 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L602 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L603 EN**: Separator comment used for visual grouping.
  **L603 CN**: 用于视觉分组的分隔注释。
- **L604 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L604 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L605 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L605 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L606 EN**: Separator comment used for visual grouping.
  **L606 CN**: 用于视觉分组的分隔注释。
- **L607 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L607 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L608 EN**: Separator comment used for visual grouping.
  **L608 CN**: 用于视觉分组的分隔注释。
- **L609 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`.
  **L609 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`。
- **L610 EN**: Separator comment used for visual grouping.
  **L610 CN**: 用于视觉分组的分隔注释。
- **L611 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L611 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L612 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L613 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L613 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L614 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L614 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L615 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L615 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L616 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Elements correspond to the`.
  **L616 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Elements correspond to the`。
- **L617 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B using biases from a __A.`.
  **L617 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B using biases from a __A.`。
- **L618 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L618 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L619 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvts_biasph_bf8(__m256i __A, __m256h __B) {`.
  **L619 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvts_biasph_bf8(__m256i __A, __m256h __B) {`。
- **L620 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(`.
  **L620 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(` 从当前函数返回。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),`。
- **L622 EN**: Executes a call or declaration centered on `statement`.
  **L622 CN**: 执行以 `statement` 为核心的调用或声明。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 625-648

````c
/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated. Merging mask \a __U
///    is used to determine if given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
````
- **L625 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L625 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L626 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L626 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L627 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated. Merging mask a __U`.
  **L627 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated. Merging mask a __U`。
- **L628 EN**: Comment explains nearby logic, constraints, or intent: `is used to determine if given element should be taken from a __W instead.`.
  **L628 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is used to determine if given element should be taken from a __W instead.`。
- **L629 EN**: Separator comment used for visual grouping.
  **L629 CN**: 用于视觉分组的分隔注释。
- **L630 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L630 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L631 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L632 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L632 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L633 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L633 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L634 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L634 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L635 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L635 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L636 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L636 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L637 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L637 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L638 EN**: Separator comment used for visual grouping.
  **L638 CN**: 用于视觉分组的分隔注释。
- **L639 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L639 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L640 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L640 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L641 EN**: Separator comment used for visual grouping.
  **L641 CN**: 用于视觉分组的分隔注释。
- **L642 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L642 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L643 EN**: Separator comment used for visual grouping.
  **L643 CN**: 用于视觉分组的分隔注释。
- **L644 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`.
  **L644 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`。
- **L645 EN**: Separator comment used for visual grouping.
  **L645 CN**: 用于视觉分组的分隔注释。
- **L646 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L646 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L647 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L647 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L648 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L648 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。

### Lines 649-672

````c
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Elements correspond to the converted
///    elements from \a __B, using biases from \a __A. If corresponding mask bit
///    is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256 _mm256_mask_cvts_biasph_bf8(
    __m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)__W, (__mmask16)__U);
}

/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated. Zeroing mask \a __U
///    is used to determine if given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
///	 	dst.bf8[i] := convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
````
- **L649 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L649 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L650 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L650 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L651 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L651 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L652 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L653 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L653 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L654 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L654 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L655 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Elements correspond to the converted`.
  **L655 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Elements correspond to the converted`。
- **L656 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __B, using biases from a __A. If corresponding mask bit`.
  **L656 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __B, using biases from a __A. If corresponding mask bit`。
- **L657 EN**: Comment explains nearby logic, constraints, or intent: `is not set, then element from a __W is taken instead.`.
  **L657 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is not set, then element from a __W is taken instead.`。
- **L658 EN**: Continues logic associated with callable symbol `_mm256_mask_cvts_biasph_bf8`.
  **L658 CN**: 继续与可调用符号 `_mm256_mask_cvts_biasph_bf8` 相关的逻辑。
- **L659 EN**: Continues the surrounding expression or declaration: `__m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {`.
  **L659 CN**: 继续构造周围的表达式或声明：`__m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {`。
- **L660 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(`.
  **L660 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(` 从当前函数返回。
- **L661 EN**: Executes a call or declaration centered on `statement`.
  **L661 CN**: 执行以 `statement` 为核心的调用或声明。
- **L662 EN**: Closes the current lexical scope or compound statement.
  **L662 CN**: 结束当前词法作用域或复合语句块。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L664 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L665 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`.
  **L665 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E5M2 numbers, using conversion biases stored in lower 8 bits of each`。
- **L666 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated. Zeroing mask a __U`.
  **L666 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated. Zeroing mask a __U`。
- **L667 EN**: Comment explains nearby logic, constraints, or intent: `is used to determine if given element should be zeroed instead.`.
  **L667 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is used to determine if given element should be zeroed instead.`。
- **L668 EN**: Separator comment used for visual grouping.
  **L668 CN**: 用于视觉分组的分隔注释。
- **L669 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L669 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L670 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L670 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L671 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L671 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L672 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L672 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。

### Lines 673-696

````c
///	 ELSE
///	 	dst.bf8[i] := 0
///	 FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2BF8S instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Elements correspond to the converted
///    elements from \a __B, using biases from \a __A. If corresponding mask bit
///    is not set, then element is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvts_biasph_bf8(__mmask16 __U, __m256i __A, __m256h __B) {
````
- **L673 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L673 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L674 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L675 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L675 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L676 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L676 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L677 EN**: Separator comment used for visual grouping.
  **L677 CN**: 用于视觉分组的分隔注释。
- **L678 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L679 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L679 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L680 EN**: Separator comment used for visual grouping.
  **L680 CN**: 用于视觉分组的分隔注释。
- **L681 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L681 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L682 EN**: Separator comment used for visual grouping.
  **L682 CN**: 用于视觉分组的分隔注释。
- **L683 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`.
  **L683 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2BF8S instruction.`。
- **L684 EN**: Separator comment used for visual grouping.
  **L684 CN**: 用于视觉分组的分隔注释。
- **L685 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L685 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L686 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L686 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L687 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L687 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L688 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L688 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L689 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L689 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L690 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L690 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L691 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L691 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L692 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Elements correspond to the converted`.
  **L692 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Elements correspond to the converted`。
- **L693 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __B, using biases from a __A. If corresponding mask bit`.
  **L693 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __B, using biases from a __A. If corresponding mask bit`。
- **L694 EN**: Comment explains nearby logic, constraints, or intent: `is not set, then element is zeroed.`.
  **L694 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is not set, then element is zeroed.`。
- **L695 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L695 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L696 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvts_biasph_bf8(__mmask16 __U, __m256i __A, __m256h __B) {`.
  **L696 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvts_biasph_bf8(__mmask16 __U, __m256i __A, __m256h __B) {`。

### Lines 697-720

````c
  return (__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),
      (__mmask16)__U);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.hf8[i] := convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8 instruction.
///
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
````
- **L697 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(`.
  **L697 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2bf8s_256_mask(` 从当前函数返回。
- **L698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`.
  **L698 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`。
- **L699 EN**: Executes a call or declaration centered on `statement`.
  **L699 CN**: 执行以 `statement` 为核心的调用或声明。
- **L700 EN**: Closes the current lexical scope or compound statement.
  **L700 CN**: 结束当前词法作用域或复合语句块。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L702 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L703 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L703 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L704 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B.`.
  **L704 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B.`。
- **L705 EN**: Separator comment used for visual grouping.
  **L705 CN**: 用于视觉分组的分隔注释。
- **L706 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L706 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L707 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L707 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L708 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L708 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L709 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L709 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L710 EN**: Separator comment used for visual grouping.
  **L710 CN**: 用于视觉分组的分隔注释。
- **L711 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L711 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L712 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L712 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L713 EN**: Separator comment used for visual grouping.
  **L713 CN**: 用于视觉分组的分隔注释。
- **L714 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L715 EN**: Separator comment used for visual grouping.
  **L715 CN**: 用于视觉分组的分隔注释。
- **L716 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`.
  **L716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`。
- **L717 EN**: Separator comment used for visual grouping.
  **L717 CN**: 用于视觉分组的分隔注释。
- **L718 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L718 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L719 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L719 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L720 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L720 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。

### Lines 721-744

````c
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    converted elements from \a __B using biases from \a __A; higher order
///    elements are zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtbiasph_hf8(__m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(
      (__v16qi)__A, (__v8hf)__B, (__v16qi)_mm_undefined_si128(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Merging mask \a __U is used to determine if
///    given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
````
- **L721 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L722 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L722 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L723 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L723 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L724 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B using biases from a __A; higher order`.
  **L724 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B using biases from a __A; higher order`。
- **L725 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed.`.
  **L725 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed.`。
- **L726 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L726 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L727 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtbiasph_hf8(__m128i __A, __m128h __B) {`.
  **L727 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtbiasph_hf8(__m128i __A, __m128h __B) {`。
- **L728 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(`.
  **L728 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(` 从当前函数返回。
- **L729 EN**: Executes a call or declaration centered on `statement`.
  **L729 CN**: 执行以 `statement` 为核心的调用或声明。
- **L730 EN**: Closes the current lexical scope or compound statement.
  **L730 CN**: 结束当前词法作用域或复合语句块。
- **L731 EN**: Blank line separating nearby declarations or logic blocks.
  **L731 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L732 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L732 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L733 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L733 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L734 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Merging mask a __U is used to determine if`.
  **L734 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Merging mask a __U is used to determine if`。
- **L735 EN**: Comment explains nearby logic, constraints, or intent: `given element should be taken from a __W instead.`.
  **L735 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be taken from a __W instead.`。
- **L736 EN**: Separator comment used for visual grouping.
  **L736 CN**: 用于视觉分组的分隔注释。
- **L737 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L738 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L738 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L739 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L740 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L740 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L741 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L741 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L742 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L742 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L743 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L744 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L744 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。

### Lines 745-768

````c
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    converted elements from \a __B, using biases from \a __A; higher order
///    elements are zeroed. If corresponding mask bit is not set, then element
///    from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtbiasph_hf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(
````
- **L745 EN**: Separator comment used for visual grouping.
  **L745 CN**: 用于视觉分组的分隔注释。
- **L746 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L747 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L747 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L748 EN**: Separator comment used for visual grouping.
  **L748 CN**: 用于视觉分组的分隔注释。
- **L749 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L749 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L750 EN**: Separator comment used for visual grouping.
  **L750 CN**: 用于视觉分组的分隔注释。
- **L751 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`.
  **L751 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`。
- **L752 EN**: Separator comment used for visual grouping.
  **L752 CN**: 用于视觉分组的分隔注释。
- **L753 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L753 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L754 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L754 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L755 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L756 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L756 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L757 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L758 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L758 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L759 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L759 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L760 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L760 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L761 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L761 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L762 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L762 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L763 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B, using biases from a __A; higher order`.
  **L763 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B, using biases from a __A; higher order`。
- **L764 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed. If corresponding mask bit is not set, then element`.
  **L764 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed. If corresponding mask bit is not set, then element`。
- **L765 EN**: Comment explains nearby logic, constraints, or intent: `from a __W is taken instead.`.
  **L765 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W is taken instead.`。
- **L766 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L766 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L767 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtbiasph_hf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {`.
  **L767 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtbiasph_hf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {`。
- **L768 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(`.
  **L768 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(` 从当前函数返回。

### Lines 769-792

````c
      (__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)__W, (__mmask8)__U);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Zeroing mask \a __U is used to determine if
///    given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
///	 	dst.hf8[i] := convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])
///	 ELSE
///	 	dst.hf8[i] := 0
///	 FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8 instruction.
///
````
- **L769 EN**: Executes a call or declaration centered on `statement`.
  **L769 CN**: 执行以 `statement` 为核心的调用或声明。
- **L770 EN**: Closes the current lexical scope or compound statement.
  **L770 CN**: 结束当前词法作用域或复合语句块。
- **L771 EN**: Blank line separating nearby declarations or logic blocks.
  **L771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L772 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L773 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L774 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Zeroing mask a __U is used to determine if`.
  **L774 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Zeroing mask a __U is used to determine if`。
- **L775 EN**: Comment explains nearby logic, constraints, or intent: `given element should be zeroed instead.`.
  **L775 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be zeroed instead.`。
- **L776 EN**: Separator comment used for visual grouping.
  **L776 CN**: 用于视觉分组的分隔注释。
- **L777 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L777 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L778 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L778 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L779 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L779 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L780 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L780 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L781 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L781 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L782 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L783 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L784 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L785 EN**: Separator comment used for visual grouping.
  **L785 CN**: 用于视觉分组的分隔注释。
- **L786 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L787 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L787 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L788 EN**: Separator comment used for visual grouping.
  **L788 CN**: 用于视觉分组的分隔注释。
- **L789 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L789 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L790 EN**: Separator comment used for visual grouping.
  **L790 CN**: 用于视觉分组的分隔注释。
- **L791 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`.
  **L791 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`。
- **L792 EN**: Separator comment used for visual grouping.
  **L792 CN**: 用于视觉分组的分隔注释。

### Lines 793-816

````c
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    converted elements from \a __B, using biases from \a __A; higher order
///    elements are zeroed. If corresponding mask bit is not set, then element
///    is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtbiasph_hf8(__mmask8 __U, __m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(
      (__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),
      (__mmask8)__U);
}

/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B.
///
/// \code{.operation}
/// FOR i := 0 to 15
````
- **L793 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L793 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L794 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L794 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L795 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L795 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L796 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L796 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L797 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L797 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L798 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L798 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L799 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L799 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L800 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L800 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L801 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B, using biases from a __A; higher order`.
  **L801 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B, using biases from a __A; higher order`。
- **L802 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed. If corresponding mask bit is not set, then element`.
  **L802 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed. If corresponding mask bit is not set, then element`。
- **L803 EN**: Comment explains nearby logic, constraints, or intent: `is zeroed.`.
  **L803 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is zeroed.`。
- **L804 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L804 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L805 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtbiasph_hf8(__mmask8 __U, __m128i __A, __m128h __B) {`.
  **L805 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtbiasph_hf8(__mmask8 __U, __m128i __A, __m128h __B) {`。
- **L806 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(`.
  **L806 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8_128_mask(` 从当前函数返回。
- **L807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`.
  **L807 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`。
- **L808 EN**: Executes a call or declaration centered on `statement`.
  **L808 CN**: 执行以 `statement` 为核心的调用或声明。
- **L809 EN**: Closes the current lexical scope or compound statement.
  **L809 CN**: 结束当前词法作用域或复合语句块。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L811 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L811 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L812 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L812 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L813 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B.`.
  **L813 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B.`。
- **L814 EN**: Separator comment used for visual grouping.
  **L814 CN**: 用于视觉分组的分隔注释。
- **L815 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L816 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L816 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。

### Lines 817-840

````c
/// 	dst.hf8[i] := convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8 instruction.
///
/// \param __A
///    A 256-bit vector of [16 x half].
/// \param __B
///    A 256-bit vector of [16 x i16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Elements correspond to the
///    converted elements from \a __B using biases from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvtbiasph_hf8(__m256i __A, __m256h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),
      (__mmask16)-1);
}

````
- **L817 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L817 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L818 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L818 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L819 EN**: Separator comment used for visual grouping.
  **L819 CN**: 用于视觉分组的分隔注释。
- **L820 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L821 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L821 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L822 EN**: Separator comment used for visual grouping.
  **L822 CN**: 用于视觉分组的分隔注释。
- **L823 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L823 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L824 EN**: Separator comment used for visual grouping.
  **L824 CN**: 用于视觉分组的分隔注释。
- **L825 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`.
  **L825 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`。
- **L826 EN**: Separator comment used for visual grouping.
  **L826 CN**: 用于视觉分组的分隔注释。
- **L827 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L828 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x half].`.
  **L828 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x half].`。
- **L829 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L830 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x i16].`.
  **L830 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x i16].`。
- **L831 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L831 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L832 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Elements correspond to the`.
  **L832 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Elements correspond to the`。
- **L833 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B using biases from a __A.`.
  **L833 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B using biases from a __A.`。
- **L834 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L834 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L835 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtbiasph_hf8(__m256i __A, __m256h __B) {`.
  **L835 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtbiasph_hf8(__m256i __A, __m256h __B) {`。
- **L836 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(`.
  **L836 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(` 从当前函数返回。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),`.
  **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),`。
- **L838 EN**: Executes a call or declaration centered on `statement`.
  **L838 CN**: 执行以 `statement` 为核心的调用或声明。
- **L839 EN**: Closes the current lexical scope or compound statement.
  **L839 CN**: 结束当前词法作用域或复合语句块。
- **L840 EN**: Blank line separating nearby declarations or logic blocks.
  **L840 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 841-864

````c
/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Merging mask \a __U is used to determine if
///    given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
````
- **L841 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L841 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L842 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L842 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L843 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Merging mask a __U is used to determine if`.
  **L843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Merging mask a __U is used to determine if`。
- **L844 EN**: Comment explains nearby logic, constraints, or intent: `given element should be taken from a __W instead.`.
  **L844 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be taken from a __W instead.`。
- **L845 EN**: Separator comment used for visual grouping.
  **L845 CN**: 用于视觉分组的分隔注释。
- **L846 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L846 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L847 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L847 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L848 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L848 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L849 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L849 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。
- **L850 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L850 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L851 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L851 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L852 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L852 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L853 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L853 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L854 EN**: Separator comment used for visual grouping.
  **L854 CN**: 用于视觉分组的分隔注释。
- **L855 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L855 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L856 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L857 EN**: Separator comment used for visual grouping.
  **L857 CN**: 用于视觉分组的分隔注释。
- **L858 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L858 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L859 EN**: Separator comment used for visual grouping.
  **L859 CN**: 用于视觉分组的分隔注释。
- **L860 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`.
  **L860 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`。
- **L861 EN**: Separator comment used for visual grouping.
  **L861 CN**: 用于视觉分组的分隔注释。
- **L862 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L862 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L863 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L864 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L864 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。

### Lines 865-888

````c
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Elements correspond to the converted
///    elements from \a __B, using biases from \a __A. If corresponding mask bit
///    is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256 _mm256_mask_cvtbiasph_hf8(
    __m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)__W, (__mmask16)__U);
}

/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Zeroing mask \a __U is used to determine if
///    given element should be taken zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
///	 	dst.hf8[i] := convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])
````
- **L865 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L865 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L866 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L866 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L867 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L867 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L868 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L868 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L869 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L869 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L870 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L870 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L871 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Elements correspond to the converted`.
  **L871 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Elements correspond to the converted`。
- **L872 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __B, using biases from a __A. If corresponding mask bit`.
  **L872 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __B, using biases from a __A. If corresponding mask bit`。
- **L873 EN**: Comment explains nearby logic, constraints, or intent: `is not set, then element from a __W is taken instead.`.
  **L873 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is not set, then element from a __W is taken instead.`。
- **L874 EN**: Continues logic associated with callable symbol `_mm256_mask_cvtbiasph_hf8`.
  **L874 CN**: 继续与可调用符号 `_mm256_mask_cvtbiasph_hf8` 相关的逻辑。
- **L875 EN**: Continues the surrounding expression or declaration: `__m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {`.
  **L875 CN**: 继续构造周围的表达式或声明：`__m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {`。
- **L876 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(`.
  **L876 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(` 从当前函数返回。
- **L877 EN**: Executes a call or declaration centered on `statement`.
  **L877 CN**: 执行以 `statement` 为核心的调用或声明。
- **L878 EN**: Closes the current lexical scope or compound statement.
  **L878 CN**: 结束当前词法作用域或复合语句块。
- **L879 EN**: Blank line separating nearby declarations or logic blocks.
  **L879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L880 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L880 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L881 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L881 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L882 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Zeroing mask a __U is used to determine if`.
  **L882 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Zeroing mask a __U is used to determine if`。
- **L883 EN**: Comment explains nearby logic, constraints, or intent: `given element should be taken zeroed instead.`.
  **L883 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be taken zeroed instead.`。
- **L884 EN**: Separator comment used for visual grouping.
  **L884 CN**: 用于视觉分组的分隔注释。
- **L885 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L885 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L886 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L886 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L887 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L887 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L888 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`.
  **L888 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias(__A.int8[2 * i], __B.fp16[i])`。

### Lines 889-912

````c
///	 ELSE
///	 	dst.hf8[i] := 0
///	 FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8 instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x half].
/// \param __B
///    A 256-bit vector of [16 x i16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Elements correspond to the converted
///    elements from \a __B, using biases from \a __A. If corresponding mask bit
///    is not set, then element is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtbiasph_hf8(__mmask16 __U, __m256i __A, __m256h __B) {
````
- **L889 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L889 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L890 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L890 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L891 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L891 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L892 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L892 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L893 EN**: Separator comment used for visual grouping.
  **L893 CN**: 用于视觉分组的分隔注释。
- **L894 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L894 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L895 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L895 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L896 EN**: Separator comment used for visual grouping.
  **L896 CN**: 用于视觉分组的分隔注释。
- **L897 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L897 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L898 EN**: Separator comment used for visual grouping.
  **L898 CN**: 用于视觉分组的分隔注释。
- **L899 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`.
  **L899 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8 instruction.`。
- **L900 EN**: Separator comment used for visual grouping.
  **L900 CN**: 用于视觉分组的分隔注释。
- **L901 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L901 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L902 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L902 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L903 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L903 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L904 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x half].`.
  **L904 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x half].`。
- **L905 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L905 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L906 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x i16].`.
  **L906 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x i16].`。
- **L907 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L907 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L908 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Elements correspond to the converted`.
  **L908 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Elements correspond to the converted`。
- **L909 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __B, using biases from a __A. If corresponding mask bit`.
  **L909 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __B, using biases from a __A. If corresponding mask bit`。
- **L910 EN**: Comment explains nearby logic, constraints, or intent: `is not set, then element is zeroed.`.
  **L910 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is not set, then element is zeroed.`。
- **L911 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L911 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L912 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtbiasph_hf8(__mmask16 __U, __m256i __A, __m256h __B) {`.
  **L912 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtbiasph_hf8(__mmask16 __U, __m256i __A, __m256h __B) {`。

### Lines 913-936

````c
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),
      (__mmask16)__U);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.hf8[i] := convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8S`instruction.
///
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
````
- **L913 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(`.
  **L913 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8_256_mask(` 从当前函数返回。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`。
- **L915 EN**: Executes a call or declaration centered on `statement`.
  **L915 CN**: 执行以 `statement` 为核心的调用或声明。
- **L916 EN**: Closes the current lexical scope or compound statement.
  **L916 CN**: 结束当前词法作用域或复合语句块。
- **L917 EN**: Blank line separating nearby declarations or logic blocks.
  **L917 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L918 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L918 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L919 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L919 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L920 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated.`.
  **L920 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated.`。
- **L921 EN**: Separator comment used for visual grouping.
  **L921 CN**: 用于视觉分组的分隔注释。
- **L922 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L922 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L923 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L923 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L924 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L924 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L925 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L925 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L926 EN**: Separator comment used for visual grouping.
  **L926 CN**: 用于视觉分组的分隔注释。
- **L927 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L927 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L928 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L928 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L929 EN**: Separator comment used for visual grouping.
  **L929 CN**: 用于视觉分组的分隔注释。
- **L930 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L930 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L931 EN**: Separator comment used for visual grouping.
  **L931 CN**: 用于视觉分组的分隔注释。
- **L932 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8S`instruction.`.
  **L932 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8S`instruction.`。
- **L933 EN**: Separator comment used for visual grouping.
  **L933 CN**: 用于视觉分组的分隔注释。
- **L934 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L934 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L935 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L935 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L936 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L936 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。

### Lines 937-960

````c
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    converted elements from \a __B using biases from \a __A; higher order
///    elements are zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvts_biasph_hf8(__m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(
      (__v16qi)__A, (__v8hf)__B, (__v16qi)_mm_undefined_si128(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated. Merging mask \a __U
///    is used to determine if given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
````
- **L937 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L937 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L938 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L938 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L939 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L939 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L940 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B using biases from a __A; higher order`.
  **L940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B using biases from a __A; higher order`。
- **L941 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed.`.
  **L941 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed.`。
- **L942 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L942 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L943 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvts_biasph_hf8(__m128i __A, __m128h __B) {`.
  **L943 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvts_biasph_hf8(__m128i __A, __m128h __B) {`。
- **L944 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(`.
  **L944 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(` 从当前函数返回。
- **L945 EN**: Executes a call or declaration centered on `statement`.
  **L945 CN**: 执行以 `statement` 为核心的调用或声明。
- **L946 EN**: Closes the current lexical scope or compound statement.
  **L946 CN**: 结束当前词法作用域或复合语句块。
- **L947 EN**: Blank line separating nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L948 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L948 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L949 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L949 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L950 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated. Merging mask a __U`.
  **L950 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated. Merging mask a __U`。
- **L951 EN**: Comment explains nearby logic, constraints, or intent: `is used to determine if given element should be taken from a __W instead.`.
  **L951 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is used to determine if given element should be taken from a __W instead.`。
- **L952 EN**: Separator comment used for visual grouping.
  **L952 CN**: 用于视觉分组的分隔注释。
- **L953 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L953 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L954 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L954 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L955 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L955 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L956 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L956 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L957 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L957 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L958 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L958 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L959 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L959 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L960 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L960 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。

### Lines 961-984

````c
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    converted elements from \a __B, using biases from \a __A; higher order
///    elements are zeroed. If corresponding mask bit is not set, then element
///    from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvts_biasph_hf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(
````
- **L961 EN**: Separator comment used for visual grouping.
  **L961 CN**: 用于视觉分组的分隔注释。
- **L962 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L962 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L963 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L963 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L964 EN**: Separator comment used for visual grouping.
  **L964 CN**: 用于视觉分组的分隔注释。
- **L965 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L965 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L966 EN**: Separator comment used for visual grouping.
  **L966 CN**: 用于视觉分组的分隔注释。
- **L967 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`.
  **L967 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`。
- **L968 EN**: Separator comment used for visual grouping.
  **L968 CN**: 用于视觉分组的分隔注释。
- **L969 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L969 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L970 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L970 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L971 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L971 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L972 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L972 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L973 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L973 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L974 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L974 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L975 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L975 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L976 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L976 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L977 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L977 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L978 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L978 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L979 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B, using biases from a __A; higher order`.
  **L979 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B, using biases from a __A; higher order`。
- **L980 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed. If corresponding mask bit is not set, then element`.
  **L980 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed. If corresponding mask bit is not set, then element`。
- **L981 EN**: Comment explains nearby logic, constraints, or intent: `from a __W is taken instead.`.
  **L981 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W is taken instead.`。
- **L982 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L982 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L983 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvts_biasph_hf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {`.
  **L983 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvts_biasph_hf8(__m128i __W, __mmask8 __U, __m128i __A, __m128h __B) {`。
- **L984 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(`.
  **L984 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(` 从当前函数返回。

### Lines 985-1008

````c
      (__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)__W, (__mmask8)__U);
}

/// Convert 128-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated. Zeroing mask \a __U
///    is used to determine if given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
///	 	dst.hf8[i] := convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
///	 ELSE
///	 	dst.hf8[i] := 0
///	 FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8S instruction.
///
````
- **L985 EN**: Executes a call or declaration centered on `statement`.
  **L985 CN**: 执行以 `statement` 为核心的调用或声明。
- **L986 EN**: Closes the current lexical scope or compound statement.
  **L986 CN**: 结束当前词法作用域或复合语句块。
- **L987 EN**: Blank line separating nearby declarations or logic blocks.
  **L987 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L988 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __B containing packed FP16 floating-point elements`.
  **L988 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __B containing packed FP16 floating-point elements`。
- **L989 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L989 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L990 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated. Zeroing mask a __U`.
  **L990 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated. Zeroing mask a __U`。
- **L991 EN**: Comment explains nearby logic, constraints, or intent: `is used to determine if given element should be zeroed instead.`.
  **L991 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is used to determine if given element should be zeroed instead.`。
- **L992 EN**: Separator comment used for visual grouping.
  **L992 CN**: 用于视觉分组的分隔注释。
- **L993 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L993 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L994 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L994 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L995 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L995 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L996 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L996 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L997 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L997 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L998 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L998 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L999 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L999 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1000 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1000 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1001 EN**: Separator comment used for visual grouping.
  **L1001 CN**: 用于视觉分组的分隔注释。
- **L1002 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L1002 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L1003 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1003 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1004 EN**: Separator comment used for visual grouping.
  **L1004 CN**: 用于视觉分组的分隔注释。
- **L1005 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1005 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1006 EN**: Separator comment used for visual grouping.
  **L1006 CN**: 用于视觉分组的分隔注释。
- **L1007 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`.
  **L1007 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`。
- **L1008 EN**: Separator comment used for visual grouping.
  **L1008 CN**: 用于视觉分组的分隔注释。

### Lines 1009-1032

````c
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x int16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    converted elements from \a __B, using biases from \a __A; higher order
///    elements are zeroed. If corresponding mask bit is not set, then element
///    is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvts_biasph_hf8(__mmask8 __U, __m128i __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(
      (__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),
      (__mmask8)__U);
}

/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated.
///
/// \code{.operation}
/// FOR i := 0 to 15
````
- **L1009 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1009 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1010 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L1010 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L1011 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1011 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1012 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x int16].`.
  **L1012 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x int16].`。
- **L1013 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1013 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1014 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1014 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1015 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1015 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1016 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L1016 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L1017 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B, using biases from a __A; higher order`.
  **L1017 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B, using biases from a __A; higher order`。
- **L1018 EN**: Comment explains nearby logic, constraints, or intent: `elements are zeroed. If corresponding mask bit is not set, then element`.
  **L1018 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements are zeroed. If corresponding mask bit is not set, then element`。
- **L1019 EN**: Comment explains nearby logic, constraints, or intent: `is zeroed.`.
  **L1019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is zeroed.`。
- **L1020 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1020 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1021 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvts_biasph_hf8(__mmask8 __U, __m128i __A, __m128h __B) {`.
  **L1021 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvts_biasph_hf8(__mmask8 __U, __m128i __A, __m128h __B) {`。
- **L1022 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(`.
  **L1022 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_128_mask(` 从当前函数返回。
- **L1023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`.
  **L1023 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)__A, (__v8hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`。
- **L1024 EN**: Executes a call or declaration centered on `statement`.
  **L1024 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1025 EN**: Closes the current lexical scope or compound statement.
  **L1025 CN**: 结束当前词法作用域或复合语句块。
- **L1026 EN**: Blank line separating nearby declarations or logic blocks.
  **L1026 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1027 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L1027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L1028 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L1028 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L1029 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated.`.
  **L1029 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated.`。
- **L1030 EN**: Separator comment used for visual grouping.
  **L1030 CN**: 用于视觉分组的分隔注释。
- **L1031 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1031 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1032 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1032 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。

### Lines 1033-1056

````c
/// 	dst.hf8[i] := convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8S instruction.
///
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Elements correspond to the
///    converted elements from \a __B using biases from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvts_biasph_hf8(__m256i __A, __m256h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),
      (__mmask16)-1);
}

````
- **L1033 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L1033 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L1034 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1034 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1035 EN**: Separator comment used for visual grouping.
  **L1035 CN**: 用于视觉分组的分隔注释。
- **L1036 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1037 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1037 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1038 EN**: Separator comment used for visual grouping.
  **L1038 CN**: 用于视觉分组的分隔注释。
- **L1039 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1039 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1040 EN**: Separator comment used for visual grouping.
  **L1040 CN**: 用于视觉分组的分隔注释。
- **L1041 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`.
  **L1041 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`。
- **L1042 EN**: Separator comment used for visual grouping.
  **L1042 CN**: 用于视觉分组的分隔注释。
- **L1043 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1043 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1044 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L1044 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L1045 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1045 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1046 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1046 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1047 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1047 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1048 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Elements correspond to the`.
  **L1048 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Elements correspond to the`。
- **L1049 EN**: Comment explains nearby logic, constraints, or intent: `converted elements from a __B using biases from a __A.`.
  **L1049 CN**: 注释解释附近代码的逻辑、约束或设计意图：`converted elements from a __B using biases from a __A.`。
- **L1050 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L1050 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L1051 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvts_biasph_hf8(__m256i __A, __m256h __B) {`.
  **L1051 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvts_biasph_hf8(__m256i __A, __m256h __B) {`。
- **L1052 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(`.
  **L1052 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(` 从当前函数返回。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_undefined_si128(),`。
- **L1054 EN**: Executes a call or declaration centered on `statement`.
  **L1054 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1055 EN**: Closes the current lexical scope or compound statement.
  **L1055 CN**: 结束当前词法作用域或复合语句块。
- **L1056 EN**: Blank line separating nearby declarations or logic blocks.
  **L1056 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1057-1080

````c
/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated. Merging mask \a __U
///    is used to determine if given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
````
- **L1057 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L1057 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L1058 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L1058 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L1059 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated. Merging mask a __U`.
  **L1059 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated. Merging mask a __U`。
- **L1060 EN**: Comment explains nearby logic, constraints, or intent: `is used to determine if given element should be taken from a __W instead.`.
  **L1060 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is used to determine if given element should be taken from a __W instead.`。
- **L1061 EN**: Separator comment used for visual grouping.
  **L1061 CN**: 用于视觉分组的分隔注释。
- **L1062 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1062 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1063 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1063 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1064 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1064 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1065 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L1065 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。
- **L1066 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1066 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1067 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L1067 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L1068 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1068 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1069 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1069 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1070 EN**: Separator comment used for visual grouping.
  **L1070 CN**: 用于视觉分组的分隔注释。
- **L1071 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1071 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1072 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1072 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1073 EN**: Separator comment used for visual grouping.
  **L1073 CN**: 用于视觉分组的分隔注释。
- **L1074 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1074 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1075 EN**: Separator comment used for visual grouping.
  **L1075 CN**: 用于视觉分组的分隔注释。
- **L1076 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`.
  **L1076 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`。
- **L1077 EN**: Separator comment used for visual grouping.
  **L1077 CN**: 用于视觉分组的分隔注释。
- **L1078 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L1078 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L1079 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L1079 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L1080 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1080 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。

### Lines 1081-1104

````c
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Elements correspond to the converted
///    elements from \a __B, using biases from \a __A. If corresponding mask bit
///    is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256 _mm256_mask_cvts_biasph_hf8(
    __m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)__W, (__mmask16)__U);
}

/// Convert 256-bit vector \a __B containing packed FP16 floating-point elements
///    to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each
///    16-bit integer stored in \a __B. Results are saturated. Zeroing mask \a __U
///    is used to determine if given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
///	 	dst.hf8[i] := convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])
````
- **L1081 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L1081 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L1082 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1082 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1083 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L1083 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L1084 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1084 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1085 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1085 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1086 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1086 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1087 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Elements correspond to the converted`.
  **L1087 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Elements correspond to the converted`。
- **L1088 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __B, using biases from a __A. If corresponding mask bit`.
  **L1088 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __B, using biases from a __A. If corresponding mask bit`。
- **L1089 EN**: Comment explains nearby logic, constraints, or intent: `is not set, then element from a __W is taken instead.`.
  **L1089 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is not set, then element from a __W is taken instead.`。
- **L1090 EN**: Continues logic associated with callable symbol `_mm256_mask_cvts_biasph_hf8`.
  **L1090 CN**: 继续与可调用符号 `_mm256_mask_cvts_biasph_hf8` 相关的逻辑。
- **L1091 EN**: Continues the surrounding expression or declaration: `__m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {`.
  **L1091 CN**: 继续构造周围的表达式或声明：`__m128i __W, __mmask16 __U, __m256i __A, __m256h __B) {`。
- **L1092 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(`.
  **L1092 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(` 从当前函数返回。
- **L1093 EN**: Executes a call or declaration centered on `statement`.
  **L1093 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1094 EN**: Closes the current lexical scope or compound statement.
  **L1094 CN**: 结束当前词法作用域或复合语句块。
- **L1095 EN**: Blank line separating nearby declarations or logic blocks.
  **L1095 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1096 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __B containing packed FP16 floating-point elements`.
  **L1096 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __B containing packed FP16 floating-point elements`。
- **L1097 EN**: Comment explains nearby logic, constraints, or intent: `to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`.
  **L1097 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to FP8 E4M3 numbers, using conversion biases stored in lower 8 bits of each`。
- **L1098 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer stored in a __B. Results are saturated. Zeroing mask a __U`.
  **L1098 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer stored in a __B. Results are saturated. Zeroing mask a __U`。
- **L1099 EN**: Comment explains nearby logic, constraints, or intent: `is used to determine if given element should be zeroed instead.`.
  **L1099 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is used to determine if given element should be zeroed instead.`。
- **L1100 EN**: Separator comment used for visual grouping.
  **L1100 CN**: 用于视觉分组的分隔注释。
- **L1101 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1102 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1103 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1104 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`.
  **L1104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_with_bias_saturate(__A.int8[2 * i], __B.fp16[i])`。

### Lines 1105-1128

````c
///	 ELSE
///	 	dst.hf8[i] := 0
///	 FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTBIASPH2HF8S instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x int16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Elements correspond to the converted
///    elements from \a __B, using biases from \a __A. If corresponding mask bit
///    is not set, then element is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvts_biasph_hf8(__mmask16 __U, __m256i __A, __m256h __B) {
````
- **L1105 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1106 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L1106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L1107 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1108 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1109 EN**: Separator comment used for visual grouping.
  **L1109 CN**: 用于视觉分组的分隔注释。
- **L1110 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1111 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1112 EN**: Separator comment used for visual grouping.
  **L1112 CN**: 用于视觉分组的分隔注释。
- **L1113 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1114 EN**: Separator comment used for visual grouping.
  **L1114 CN**: 用于视觉分组的分隔注释。
- **L1115 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`.
  **L1115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTBIASPH2HF8S instruction.`。
- **L1116 EN**: Separator comment used for visual grouping.
  **L1116 CN**: 用于视觉分组的分隔注释。
- **L1117 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1118 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L1118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L1119 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1120 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x int16].`.
  **L1120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x int16].`。
- **L1121 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1122 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1123 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1124 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Elements correspond to the converted`.
  **L1124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Elements correspond to the converted`。
- **L1125 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __B, using biases from a __A. If corresponding mask bit`.
  **L1125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __B, using biases from a __A. If corresponding mask bit`。
- **L1126 EN**: Comment explains nearby logic, constraints, or intent: `is not set, then element is zeroed.`.
  **L1126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is not set, then element is zeroed.`。
- **L1127 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L1127 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L1128 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvts_biasph_hf8(__mmask16 __U, __m256i __A, __m256h __B) {`.
  **L1128 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvts_biasph_hf8(__mmask16 __U, __m256i __A, __m256h __B) {`。

### Lines 1129-1152

````c
  return (__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(
      (__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),
      (__mmask16)__U);
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E5M2 FP8 elements.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF i < 8
/// 		dst.bf8[i] := convert_fp16_to_bf8(__B.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i - 8])
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8 instruction.
///
````
- **L1129 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(`.
  **L1129 CN**: 以 `(__m128i)__builtin_ia32_vcvtbiasph2hf8s_256_mask(` 从当前函数返回。
- **L1130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`.
  **L1130 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)__A, (__v16hf)__B, (__v16qi)(__m128i)_mm_setzero_si128(),`。
- **L1131 EN**: Executes a call or declaration centered on `statement`.
  **L1131 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1132 EN**: Closes the current lexical scope or compound statement.
  **L1132 CN**: 结束当前词法作用域或复合语句块。
- **L1133 EN**: Blank line separating nearby declarations or logic blocks.
  **L1133 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1134 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1135 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`.
  **L1135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`。
- **L1136 EN**: Separator comment used for visual grouping.
  **L1136 CN**: 用于视觉分组的分隔注释。
- **L1137 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1138 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1139 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1140 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`.
  **L1140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`。
- **L1141 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1142 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 8])`.
  **L1142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 8])`。
- **L1143 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1144 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1145 EN**: Separator comment used for visual grouping.
  **L1145 CN**: 用于视觉分组的分隔注释。
- **L1146 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1147 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1148 EN**: Separator comment used for visual grouping.
  **L1148 CN**: 用于视觉分组的分隔注释。
- **L1149 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1149 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1150 EN**: Separator comment used for visual grouping.
  **L1150 CN**: 用于视觉分组的分隔注释。
- **L1151 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`.
  **L1151 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`。
- **L1152 EN**: Separator comment used for visual grouping.
  **L1152 CN**: 用于视觉分组的分隔注释。

### Lines 1153-1176

````c
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvt2ph_bf8(__m128h __A,
                                                                  __m128h __B) {
  return (__m128i)__builtin_ia32_vcvt2ph2bf8_128((__v8hf)(__A),
                                                   (__v8hf)(__B));
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E5M2 FP8 elements.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		IF i < 8
/// 			dst.bf8[i] := convert_fp16_to_bf8(__B.fp16[i])
````
- **L1153 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1154 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1155 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1156 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1156 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1157 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1157 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1158 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`.
  **L1158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`。
- **L1159 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1159 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1160 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L1160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvt2ph_bf8(__m128h __A,`.
  **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvt2ph_bf8(__m128h __A,`。
- **L1162 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1162 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1163 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvt2ph2bf8_128((__v8hf)(__A),`.
  **L1163 CN**: 以 `(__m128i)__builtin_ia32_vcvt2ph2bf8_128((__v8hf)(__A),` 从当前函数返回。
- **L1164 EN**: Executes a call or declaration centered on `statement`.
  **L1164 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1167 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1168 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`.
  **L1168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`。
- **L1169 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L1169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L1170 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead.`.
  **L1170 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead.`。
- **L1171 EN**: Separator comment used for visual grouping.
  **L1171 CN**: 用于视觉分组的分隔注释。
- **L1172 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1173 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1174 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1175 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1176 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`.
  **L1176 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`。

### Lines 1177-1200

````c
/// 		ELSE
/// 			dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i - 8])
/// 		FI
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
````
- **L1177 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1178 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 8])`.
  **L1178 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 8])`。
- **L1179 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1180 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1181 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L1181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L1182 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1183 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1184 EN**: Separator comment used for visual grouping.
  **L1184 CN**: 用于视觉分组的分隔注释。
- **L1185 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1186 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1187 EN**: Separator comment used for visual grouping.
  **L1187 CN**: 用于视觉分组的分隔注释。
- **L1188 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1189 EN**: Separator comment used for visual grouping.
  **L1189 CN**: 用于视觉分组的分隔注释。
- **L1190 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`.
  **L1190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`。
- **L1191 EN**: Separator comment used for visual grouping.
  **L1191 CN**: 用于视觉分组的分隔注释。
- **L1192 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L1192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L1193 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L1193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L1194 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1195 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L1195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L1196 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1197 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1198 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1199 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1199 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1200 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。

### Lines 1201-1224

````c
///    A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvt2ph_bf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_cvt2ph_bf8(__A, __B), (__v16qi)__W);
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E5M2 FP8 elements.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		IF i < 8
/// 			dst.bf8[i] := convert_fp16_to_bf8(__B.fp16[i])
/// 		ELSE
/// 			dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i - 8])
/// 		FI
/// 	ELSE
````
- **L1201 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`.
  **L1201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`。
- **L1202 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1203 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1204 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L1204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L1205 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1205 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1206 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvt2ph_bf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {`.
  **L1206 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvt2ph_bf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {`。
- **L1207 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1207 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1208 EN**: Executes a call or declaration centered on `statement`.
  **L1208 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1209 EN**: Closes the current lexical scope or compound statement.
  **L1209 CN**: 结束当前词法作用域或复合语句块。
- **L1210 EN**: Blank line separating nearby declarations or logic blocks.
  **L1210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1211 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1212 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`.
  **L1212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`。
- **L1213 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L1213 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L1214 EN**: Comment explains nearby logic, constraints, or intent: `instead.`.
  **L1214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead.`。
- **L1215 EN**: Separator comment used for visual grouping.
  **L1215 CN**: 用于视觉分组的分隔注释。
- **L1216 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1217 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1218 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1219 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1220 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`.
  **L1220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`。
- **L1221 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1222 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 8])`.
  **L1222 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 8])`。
- **L1223 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1224 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。

### Lines 1225-1248

````c
/// 		dst.bf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8 instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    zero is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvt2ph_bf8(__mmask16 __U, __m128h __A, __m128h __B) {
````
- **L1225 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L1225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L1226 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1227 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1227 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1228 EN**: Separator comment used for visual grouping.
  **L1228 CN**: 用于视觉分组的分隔注释。
- **L1229 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1230 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1231 EN**: Separator comment used for visual grouping.
  **L1231 CN**: 用于视觉分组的分隔注释。
- **L1232 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1232 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1233 EN**: Separator comment used for visual grouping.
  **L1233 CN**: 用于视觉分组的分隔注释。
- **L1234 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`.
  **L1234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`。
- **L1235 EN**: Separator comment used for visual grouping.
  **L1235 CN**: 用于视觉分组的分隔注释。
- **L1236 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1237 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L1237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L1238 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1239 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1240 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1241 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1242 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1243 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`.
  **L1243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`。
- **L1244 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1245 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1246 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L1246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L1247 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1247 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1248 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvt2ph_bf8(__mmask16 __U, __m128h __A, __m128h __B) {`.
  **L1248 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvt2ph_bf8(__mmask16 __U, __m128h __A, __m128h __B) {`。

### Lines 1249-1272

````c
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_cvt2ph_bf8(__A, __B),
      (__v16qi)(__m128i)_mm_setzero_si128());
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E5M2 FP8 elements.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF i < 16 
/// 		dst.bf8[i] := convert_fp16_to_bf8(__B.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i - 16])
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8 instruction.
///
````
- **L1249 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1249 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16qi)_mm_cvt2ph_bf8(__A, __B),`.
  **L1250 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16qi)_mm_cvt2ph_bf8(__A, __B),`。
- **L1251 EN**: Executes a call or declaration centered on `statement`.
  **L1251 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1252 EN**: Closes the current lexical scope or compound statement.
  **L1252 CN**: 结束当前词法作用域或复合语句块。
- **L1253 EN**: Blank line separating nearby declarations or logic blocks.
  **L1253 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1254 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1254 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1255 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`.
  **L1255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`。
- **L1256 EN**: Separator comment used for visual grouping.
  **L1256 CN**: 用于视觉分组的分隔注释。
- **L1257 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1258 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1259 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1260 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`.
  **L1260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`。
- **L1261 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1262 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 16])`.
  **L1262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 16])`。
- **L1263 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1264 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1265 EN**: Separator comment used for visual grouping.
  **L1265 CN**: 用于视觉分组的分隔注释。
- **L1266 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1267 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1268 EN**: Separator comment used for visual grouping.
  **L1268 CN**: 用于视觉分组的分隔注释。
- **L1269 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1269 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1270 EN**: Separator comment used for visual grouping.
  **L1270 CN**: 用于视觉分组的分隔注释。
- **L1271 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`.
  **L1271 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`。
- **L1272 EN**: Separator comment used for visual grouping.
  **L1272 CN**: 用于视觉分组的分隔注释。

### Lines 1273-1296

````c
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvt2ph_bf8(__m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_vcvt2ph2bf8_256((__v16hf)(__A),
                                                   (__v16hf)(__B));
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E5M2 FP8 elements.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF __U[i]
/// 		IF i < 16 
/// 			dst.bf8[i] := convert_fp16_to_bf8(__B.fp16[i])
````
- **L1273 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1273 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1274 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1274 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1275 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1275 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1276 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1277 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1278 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`.
  **L1278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`。
- **L1279 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1280 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L1280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L1281 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1281 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1282 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvt2ph_bf8(__m256h __A, __m256h __B) {`.
  **L1282 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvt2ph_bf8(__m256h __A, __m256h __B) {`。
- **L1283 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvt2ph2bf8_256((__v16hf)(__A),`.
  **L1283 CN**: 以 `(__m256i)__builtin_ia32_vcvt2ph2bf8_256((__v16hf)(__A),` 从当前函数返回。
- **L1284 EN**: Executes a call or declaration centered on `statement`.
  **L1284 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1285 EN**: Closes the current lexical scope or compound statement.
  **L1285 CN**: 结束当前词法作用域或复合语句块。
- **L1286 EN**: Blank line separating nearby declarations or logic blocks.
  **L1286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1287 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1288 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`.
  **L1288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`。
- **L1289 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L1289 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L1290 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead.`.
  **L1290 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead.`。
- **L1291 EN**: Separator comment used for visual grouping.
  **L1291 CN**: 用于视觉分组的分隔注释。
- **L1292 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1293 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1293 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1294 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1294 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1295 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1295 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1296 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`.
  **L1296 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`。

### Lines 1297-1320

````c
/// 		ELSE
/// 			dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i - 16])
/// 		FI
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8 instruction.
///
/// \param __W
///    A 256-bit vector of [32 x bf8].
/// \param __U
///    A 32-bit merging mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
````
- **L1297 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1298 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 16])`.
  **L1298 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 16])`。
- **L1299 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1299 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1300 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1300 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1301 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L1301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L1302 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1302 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1303 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1304 EN**: Separator comment used for visual grouping.
  **L1304 CN**: 用于视觉分组的分隔注释。
- **L1305 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1306 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1307 EN**: Separator comment used for visual grouping.
  **L1307 CN**: 用于视觉分组的分隔注释。
- **L1308 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1309 EN**: Separator comment used for visual grouping.
  **L1309 CN**: 用于视觉分组的分隔注释。
- **L1310 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`.
  **L1310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`。
- **L1311 EN**: Separator comment used for visual grouping.
  **L1311 CN**: 用于视觉分组的分隔注释。
- **L1312 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L1312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L1313 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8].`.
  **L1313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8].`。
- **L1314 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1314 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1315 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit merging mask.`.
  **L1315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit merging mask.`。
- **L1316 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1317 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1317 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1318 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1318 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1319 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1319 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1320 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1320 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。

### Lines 1321-1344

````c
///    A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m256i __DEFAULT_FN_ATTRS256 _mm256_mask_cvt2ph_bf8(
    __m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_cvt2ph_bf8(__A, __B), (__v32qi)__W);
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E5M2 FP8 elements.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF __U[i]
/// 		IF i < 16 
/// 			dst.bf8[i] := convert_fp16_to_bf8(__B.fp16[i])
/// 		ELSE
/// 			dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i - 16])
/// 		FI
/// 	ELSE
````
- **L1321 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`.
  **L1321 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`。
- **L1322 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1322 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1323 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1323 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1324 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L1324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L1325 EN**: Continues logic associated with callable symbol `_mm256_mask_cvt2ph_bf8`.
  **L1325 CN**: 继续与可调用符号 `_mm256_mask_cvt2ph_bf8` 相关的逻辑。
- **L1326 EN**: Continues the surrounding expression or declaration: `__m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {`.
  **L1326 CN**: 继续构造周围的表达式或声明：`__m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {`。
- **L1327 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L1327 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L1328 EN**: Executes a call or declaration centered on `statement`.
  **L1328 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1329 EN**: Closes the current lexical scope or compound statement.
  **L1329 CN**: 结束当前词法作用域或复合语句块。
- **L1330 EN**: Blank line separating nearby declarations or logic blocks.
  **L1330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1331 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1331 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1332 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`.
  **L1332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`。
- **L1333 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L1333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L1334 EN**: Comment explains nearby logic, constraints, or intent: `instead.`.
  **L1334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead.`。
- **L1335 EN**: Separator comment used for visual grouping.
  **L1335 CN**: 用于视觉分组的分隔注释。
- **L1336 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1336 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1337 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1337 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1338 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1339 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1339 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1340 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`.
  **L1340 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__B.fp16[i])`。
- **L1341 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1342 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 16])`.
  **L1342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i - 16])`。
- **L1343 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1344 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。

### Lines 1345-1368

````c
/// 		dst.bf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8 instruction.
///
/// \param __U
///    A 32-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    zero is taken instead.
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvt2ph_bf8(__mmask32 __U, __m256h __A, __m256h __B) {
````
- **L1345 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L1345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L1346 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1346 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1347 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1347 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1348 EN**: Separator comment used for visual grouping.
  **L1348 CN**: 用于视觉分组的分隔注释。
- **L1349 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1349 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1350 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1350 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1351 EN**: Separator comment used for visual grouping.
  **L1351 CN**: 用于视觉分组的分隔注释。
- **L1352 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1352 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1353 EN**: Separator comment used for visual grouping.
  **L1353 CN**: 用于视觉分组的分隔注释。
- **L1354 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`.
  **L1354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8 instruction.`。
- **L1355 EN**: Separator comment used for visual grouping.
  **L1355 CN**: 用于视觉分组的分隔注释。
- **L1356 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1356 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1357 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit zeroing mask.`.
  **L1357 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit zeroing mask.`。
- **L1358 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1358 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1359 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1359 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1360 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1360 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1361 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1361 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1362 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1362 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1363 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`.
  **L1363 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`。
- **L1364 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1364 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1365 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L1365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L1366 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L1366 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L1367 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1367 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1368 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvt2ph_bf8(__mmask32 __U, __m256h __A, __m256h __B) {`.
  **L1368 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvt2ph_bf8(__mmask32 __U, __m256h __A, __m256h __B) {`。

### Lines 1369-1392

````c
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_cvt2ph_bf8(__A, __B),
      (__v32qi)(__m256i)_mm256_setzero_si256());
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E5M2 FP8 elements.
///    Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF i < 8
/// 		dst.bf8[i] := convert_fp16_to_bf8_saturate(__B.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i - 8])
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8S instruction.
````
- **L1369 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L1369 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L1370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask32)__U, (__v32qi)_mm256_cvt2ph_bf8(__A, __B),`.
  **L1370 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask32)__U, (__v32qi)_mm256_cvt2ph_bf8(__A, __B),`。
- **L1371 EN**: Executes a call or declaration centered on `statement`.
  **L1371 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1372 EN**: Closes the current lexical scope or compound statement.
  **L1372 CN**: 结束当前词法作用域或复合语句块。
- **L1373 EN**: Blank line separating nearby declarations or logic blocks.
  **L1373 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1374 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1375 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`.
  **L1375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`。
- **L1376 EN**: Comment explains nearby logic, constraints, or intent: `Resulting elements are saturated in case of overflow.`.
  **L1376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Resulting elements are saturated in case of overflow.`。
- **L1377 EN**: Separator comment used for visual grouping.
  **L1377 CN**: 用于视觉分组的分隔注释。
- **L1378 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1378 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1379 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1379 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1380 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1380 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1381 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`.
  **L1381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`。
- **L1382 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1382 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1383 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 8])`.
  **L1383 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 8])`。
- **L1384 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1384 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1385 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1385 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1386 EN**: Separator comment used for visual grouping.
  **L1386 CN**: 用于视觉分组的分隔注释。
- **L1387 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1387 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1388 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1388 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1389 EN**: Separator comment used for visual grouping.
  **L1389 CN**: 用于视觉分组的分隔注释。
- **L1390 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1390 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1391 EN**: Separator comment used for visual grouping.
  **L1391 CN**: 用于视觉分组的分隔注释。
- **L1392 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`.
  **L1392 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`。

### Lines 1393-1416

````c
///
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvts_2ph_bf8(__m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvt2ph2bf8s_128((__v8hf)(__A),
                                                    (__v8hf)(__B));
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E5M2 FP8 elements.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead. Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		IF i < 8
````
- **L1393 EN**: Separator comment used for visual grouping.
  **L1393 CN**: 用于视觉分组的分隔注释。
- **L1394 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1394 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1395 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1395 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1396 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1397 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1397 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1398 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1399 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`.
  **L1399 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`。
- **L1400 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1400 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1401 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L1401 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L1402 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1402 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1403 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvts_2ph_bf8(__m128h __A, __m128h __B) {`.
  **L1403 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvts_2ph_bf8(__m128h __A, __m128h __B) {`。
- **L1404 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvt2ph2bf8s_128((__v8hf)(__A),`.
  **L1404 CN**: 以 `(__m128i)__builtin_ia32_vcvt2ph2bf8s_128((__v8hf)(__A),` 从当前函数返回。
- **L1405 EN**: Executes a call or declaration centered on `statement`.
  **L1405 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1406 EN**: Closes the current lexical scope or compound statement.
  **L1406 CN**: 结束当前词法作用域或复合语句块。
- **L1407 EN**: Blank line separating nearby declarations or logic blocks.
  **L1407 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1408 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1408 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1409 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`.
  **L1409 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`。
- **L1410 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L1410 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L1411 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead. Resulting elements are saturated in case of overflow.`.
  **L1411 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead. Resulting elements are saturated in case of overflow.`。
- **L1412 EN**: Separator comment used for visual grouping.
  **L1412 CN**: 用于视觉分组的分隔注释。
- **L1413 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1413 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1414 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1415 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1415 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1416 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1416 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。

### Lines 1417-1440

````c
/// 			dst.bf8[i] := convert_fp16_to_bf8_saturate(__B.fp16[i])
/// 		ELSE
/// 			dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i - 8])
/// 		FI
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
````
- **L1417 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`.
  **L1417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`。
- **L1418 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1418 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1419 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 8])`.
  **L1419 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 8])`。
- **L1420 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1420 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1421 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1421 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1422 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L1422 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L1423 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1424 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1425 EN**: Separator comment used for visual grouping.
  **L1425 CN**: 用于视觉分组的分隔注释。
- **L1426 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1427 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1427 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1428 EN**: Separator comment used for visual grouping.
  **L1428 CN**: 用于视觉分组的分隔注释。
- **L1429 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1429 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1430 EN**: Separator comment used for visual grouping.
  **L1430 CN**: 用于视觉分组的分隔注释。
- **L1431 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`.
  **L1431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`。
- **L1432 EN**: Separator comment used for visual grouping.
  **L1432 CN**: 用于视觉分组的分隔注释。
- **L1433 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L1433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L1434 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L1434 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L1435 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1435 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1436 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L1436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L1437 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1438 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1439 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1440 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。

### Lines 1441-1464

````c
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvts_2ph_bf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_cvts_2ph_bf8(__A, __B), (__v16qi)__W);
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E5M2 FP8 elements.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead. Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		IF i < 8
/// 			dst.bf8[i] := convert_fp16_to_bf8_saturate(__B.fp16[i])
/// 		ELSE
/// 			dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i - 8])
/// 		FI
````
- **L1441 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1441 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1442 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`.
  **L1442 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`。
- **L1443 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1443 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1444 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1444 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1445 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L1445 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L1446 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1446 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1447 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvts_2ph_bf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {`.
  **L1447 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvts_2ph_bf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {`。
- **L1448 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1448 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1449 EN**: Executes a call or declaration centered on `statement`.
  **L1449 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1450 EN**: Closes the current lexical scope or compound statement.
  **L1450 CN**: 结束当前词法作用域或复合语句块。
- **L1451 EN**: Blank line separating nearby declarations or logic blocks.
  **L1451 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1452 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1452 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1453 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`.
  **L1453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E5M2 FP8 elements.`。
- **L1454 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L1454 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L1455 EN**: Comment explains nearby logic, constraints, or intent: `instead. Resulting elements are saturated in case of overflow.`.
  **L1455 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead. Resulting elements are saturated in case of overflow.`。
- **L1456 EN**: Separator comment used for visual grouping.
  **L1456 CN**: 用于视觉分组的分隔注释。
- **L1457 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1458 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1458 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1459 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1459 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1460 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1461 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`.
  **L1461 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`。
- **L1462 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1463 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 8])`.
  **L1463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 8])`。
- **L1464 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。

### Lines 1465-1488

````c
/// 	ELSE
/// 		dst.bf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8S instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    zero is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
````
- **L1465 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1465 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1466 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L1466 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L1467 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1467 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1468 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1468 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1469 EN**: Separator comment used for visual grouping.
  **L1469 CN**: 用于视觉分组的分隔注释。
- **L1470 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1470 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1471 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1471 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1472 EN**: Separator comment used for visual grouping.
  **L1472 CN**: 用于视觉分组的分隔注释。
- **L1473 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1473 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1474 EN**: Separator comment used for visual grouping.
  **L1474 CN**: 用于视觉分组的分隔注释。
- **L1475 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`.
  **L1475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`。
- **L1476 EN**: Separator comment used for visual grouping.
  **L1476 CN**: 用于视觉分组的分隔注释。
- **L1477 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1477 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1478 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L1478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L1479 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1479 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1480 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1480 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1481 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1481 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1482 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1482 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1483 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1483 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1484 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`.
  **L1484 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower 8 elements correspond to the`。
- **L1485 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1485 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1486 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1487 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L1487 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L1488 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1488 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。

### Lines 1489-1512

````c
_mm_maskz_cvts_2ph_bf8(__mmask16 __U, __m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_cvts_2ph_bf8(__A, __B),
      (__v16qi)(__m128i)_mm_setzero_si128());
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E5M2 FP8 elements.
///    Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF i < 16 
/// 		dst.bf8[i] := convert_fp16_to_bf8_saturate(__B.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i - 16])
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
````
- **L1489 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvts_2ph_bf8(__mmask16 __U, __m128h __A, __m128h __B) {`.
  **L1489 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvts_2ph_bf8(__mmask16 __U, __m128h __A, __m128h __B) {`。
- **L1490 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1490 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16qi)_mm_cvts_2ph_bf8(__A, __B),`.
  **L1491 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16qi)_mm_cvts_2ph_bf8(__A, __B),`。
- **L1492 EN**: Executes a call or declaration centered on `statement`.
  **L1492 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1493 EN**: Closes the current lexical scope or compound statement.
  **L1493 CN**: 结束当前词法作用域或复合语句块。
- **L1494 EN**: Blank line separating nearby declarations or logic blocks.
  **L1494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1495 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1495 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1496 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`.
  **L1496 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`。
- **L1497 EN**: Comment explains nearby logic, constraints, or intent: `Resulting elements are saturated in case of overflow.`.
  **L1497 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Resulting elements are saturated in case of overflow.`。
- **L1498 EN**: Separator comment used for visual grouping.
  **L1498 CN**: 用于视觉分组的分隔注释。
- **L1499 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1499 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1500 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1500 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1501 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1501 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1502 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`.
  **L1502 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`。
- **L1503 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1503 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1504 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 16])`.
  **L1504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 16])`。
- **L1505 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1505 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1506 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1506 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1507 EN**: Separator comment used for visual grouping.
  **L1507 CN**: 用于视觉分组的分隔注释。
- **L1508 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1508 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1509 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1509 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1510 EN**: Separator comment used for visual grouping.
  **L1510 CN**: 用于视觉分组的分隔注释。
- **L1511 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1511 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1512 EN**: Separator comment used for visual grouping.
  **L1512 CN**: 用于视觉分组的分隔注释。

### Lines 1513-1536

````c
/// This intrinsic corresponds to the \c VCVT2PH2BF8S instruction.
///
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvts_2ph_bf8(__m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_vcvt2ph2bf8s_256((__v16hf)(__A),
                                                    (__v16hf)(__B));
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E5M2 FP8 elements.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead. Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF __U[i]
````
- **L1513 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`.
  **L1513 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`。
- **L1514 EN**: Separator comment used for visual grouping.
  **L1514 CN**: 用于视觉分组的分隔注释。
- **L1515 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1516 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1516 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1517 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1518 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1518 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1519 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1520 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`.
  **L1520 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`。
- **L1521 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1521 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1522 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L1522 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L1523 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1523 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1524 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvts_2ph_bf8(__m256h __A, __m256h __B) {`.
  **L1524 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvts_2ph_bf8(__m256h __A, __m256h __B) {`。
- **L1525 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvt2ph2bf8s_256((__v16hf)(__A),`.
  **L1525 CN**: 以 `(__m256i)__builtin_ia32_vcvt2ph2bf8s_256((__v16hf)(__A),` 从当前函数返回。
- **L1526 EN**: Executes a call or declaration centered on `statement`.
  **L1526 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1527 EN**: Closes the current lexical scope or compound statement.
  **L1527 CN**: 结束当前词法作用域或复合语句块。
- **L1528 EN**: Blank line separating nearby declarations or logic blocks.
  **L1528 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1529 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1529 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1530 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`.
  **L1530 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`。
- **L1531 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L1531 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L1532 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead. Resulting elements are saturated in case of overflow.`.
  **L1532 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead. Resulting elements are saturated in case of overflow.`。
- **L1533 EN**: Separator comment used for visual grouping.
  **L1533 CN**: 用于视觉分组的分隔注释。
- **L1534 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1534 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1535 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1535 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1536 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1536 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。

### Lines 1537-1560

````c
/// 		IF i < 16 
/// 			dst.bf8[i] := convert_fp16_to_bf8_saturate(__B.fp16[i])
/// 		ELSE
/// 			dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i - 16])
/// 		FI
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8S instruction.
///
/// \param __W
///    A 256-bit vector of [32 x bf8].
/// \param __U
///    A 32-bit merging mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
````
- **L1537 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1537 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1538 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`.
  **L1538 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`。
- **L1539 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1539 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1540 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 16])`.
  **L1540 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 16])`。
- **L1541 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1541 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1542 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1542 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1543 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L1543 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L1544 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1544 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1545 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1545 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1546 EN**: Separator comment used for visual grouping.
  **L1546 CN**: 用于视觉分组的分隔注释。
- **L1547 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1547 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1548 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1548 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1549 EN**: Separator comment used for visual grouping.
  **L1549 CN**: 用于视觉分组的分隔注释。
- **L1550 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1550 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1551 EN**: Separator comment used for visual grouping.
  **L1551 CN**: 用于视觉分组的分隔注释。
- **L1552 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`.
  **L1552 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`。
- **L1553 EN**: Separator comment used for visual grouping.
  **L1553 CN**: 用于视觉分组的分隔注释。
- **L1554 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L1554 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L1555 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8].`.
  **L1555 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8].`。
- **L1556 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1556 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1557 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit merging mask.`.
  **L1557 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit merging mask.`。
- **L1558 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1558 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1559 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1559 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1560 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1560 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。

### Lines 1561-1584

````c
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m256i __DEFAULT_FN_ATTRS256 _mm256_mask_cvts_2ph_bf8(
    __m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_cvts_2ph_bf8(__A, __B), (__v32qi)__W);
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E5M2 FP8 elements.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead. Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF __U[i]
/// 		IF i < 16 
/// 			dst.bf8[i] := convert_fp16_to_bf8_saturate(__B.fp16[i])
/// 		ELSE
/// 			dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i - 16])
````
- **L1561 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1561 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1562 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1562 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1563 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`.
  **L1563 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`。
- **L1564 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1564 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1565 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1565 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1566 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L1566 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L1567 EN**: Continues logic associated with callable symbol `_mm256_mask_cvts_2ph_bf8`.
  **L1567 CN**: 继续与可调用符号 `_mm256_mask_cvts_2ph_bf8` 相关的逻辑。
- **L1568 EN**: Continues the surrounding expression or declaration: `__m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {`.
  **L1568 CN**: 继续构造周围的表达式或声明：`__m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {`。
- **L1569 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L1569 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L1570 EN**: Executes a call or declaration centered on `statement`.
  **L1570 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1571 EN**: Closes the current lexical scope or compound statement.
  **L1571 CN**: 结束当前词法作用域或复合语句块。
- **L1572 EN**: Blank line separating nearby declarations or logic blocks.
  **L1572 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1573 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1573 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1574 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`.
  **L1574 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E5M2 FP8 elements.`。
- **L1575 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L1575 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L1576 EN**: Comment explains nearby logic, constraints, or intent: `instead. Resulting elements are saturated in case of overflow.`.
  **L1576 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead. Resulting elements are saturated in case of overflow.`。
- **L1577 EN**: Separator comment used for visual grouping.
  **L1577 CN**: 用于视觉分组的分隔注释。
- **L1578 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1578 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1579 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1579 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1580 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1580 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1581 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1581 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1582 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`.
  **L1582 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__B.fp16[i])`。
- **L1583 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1583 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1584 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 16])`.
  **L1584 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i - 16])`。

### Lines 1585-1608

````c
/// 		FI
/// 	ELSE
/// 		dst.bf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2BF8S instruction.
///
/// \param __U
///    A 32-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    zero is taken instead.
````
- **L1585 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1585 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1586 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1586 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1587 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L1587 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L1588 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1588 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1589 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1589 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1590 EN**: Separator comment used for visual grouping.
  **L1590 CN**: 用于视觉分组的分隔注释。
- **L1591 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1591 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1592 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1592 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1593 EN**: Separator comment used for visual grouping.
  **L1593 CN**: 用于视觉分组的分隔注释。
- **L1594 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1594 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1595 EN**: Separator comment used for visual grouping.
  **L1595 CN**: 用于视觉分组的分隔注释。
- **L1596 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`.
  **L1596 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2BF8S instruction.`。
- **L1597 EN**: Separator comment used for visual grouping.
  **L1597 CN**: 用于视觉分组的分隔注释。
- **L1598 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1598 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1599 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit zeroing mask.`.
  **L1599 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit zeroing mask.`。
- **L1600 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1600 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1601 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1602 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1603 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1603 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1604 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1604 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1605 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`.
  **L1605 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8]. Lower 16 elements correspond to the`。
- **L1606 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1606 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1607 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L1607 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L1608 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L1608 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。

### Lines 1609-1632

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvts_2ph_bf8(__mmask32 __U, __m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_cvts_2ph_bf8(__A, __B),
      (__v32qi)(__m256i)_mm256_setzero_si256());
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E4M3 FP8 elements.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF i < 8
/// 		dst.hf8[i] := convert_fp16_to_hf8(__B.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i - 8])
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
````
- **L1609 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1609 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1610 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvts_2ph_bf8(__mmask32 __U, __m256h __A, __m256h __B) {`.
  **L1610 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvts_2ph_bf8(__mmask32 __U, __m256h __A, __m256h __B) {`。
- **L1611 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L1611 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L1612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask32)__U, (__v32qi)_mm256_cvts_2ph_bf8(__A, __B),`.
  **L1612 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask32)__U, (__v32qi)_mm256_cvts_2ph_bf8(__A, __B),`。
- **L1613 EN**: Executes a call or declaration centered on `statement`.
  **L1613 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1614 EN**: Closes the current lexical scope or compound statement.
  **L1614 CN**: 结束当前词法作用域或复合语句块。
- **L1615 EN**: Blank line separating nearby declarations or logic blocks.
  **L1615 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1616 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1616 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1617 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`.
  **L1617 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`。
- **L1618 EN**: Separator comment used for visual grouping.
  **L1618 CN**: 用于视觉分组的分隔注释。
- **L1619 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1619 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1620 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1620 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1621 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1621 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1622 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`.
  **L1622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`。
- **L1623 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1623 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1624 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 8])`.
  **L1624 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 8])`。
- **L1625 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1625 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1626 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1626 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1627 EN**: Separator comment used for visual grouping.
  **L1627 CN**: 用于视觉分组的分隔注释。
- **L1628 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1628 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1629 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1629 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1630 EN**: Separator comment used for visual grouping.
  **L1630 CN**: 用于视觉分组的分隔注释。
- **L1631 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1632 EN**: Separator comment used for visual grouping.
  **L1632 CN**: 用于视觉分组的分隔注释。

### Lines 1633-1656

````c
/// This intrinsic corresponds to the \c VCVT2PH2HF8 instruction.
///
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvt2ph_hf8(__m128h __A,
                                                                  __m128h __B) {
  return (__m128i)__builtin_ia32_vcvt2ph2hf8_128((__v8hf)(__A),
                                                   (__v8hf)(__B));
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E4M3 FP8 elements.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
````
- **L1633 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`.
  **L1633 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`。
- **L1634 EN**: Separator comment used for visual grouping.
  **L1634 CN**: 用于视觉分组的分隔注释。
- **L1635 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1635 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1636 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1636 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1637 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1637 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1638 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1639 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1639 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1640 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`.
  **L1640 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`。
- **L1641 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1641 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1642 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L1642 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L1643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvt2ph_hf8(__m128h __A,`.
  **L1643 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvt2ph_hf8(__m128h __A,`。
- **L1644 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1644 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1645 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvt2ph2hf8_128((__v8hf)(__A),`.
  **L1645 CN**: 以 `(__m128i)__builtin_ia32_vcvt2ph2hf8_128((__v8hf)(__A),` 从当前函数返回。
- **L1646 EN**: Executes a call or declaration centered on `statement`.
  **L1646 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1647 EN**: Closes the current lexical scope or compound statement.
  **L1647 CN**: 结束当前词法作用域或复合语句块。
- **L1648 EN**: Blank line separating nearby declarations or logic blocks.
  **L1648 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1649 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1649 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1650 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`.
  **L1650 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`。
- **L1651 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L1651 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L1652 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead.`.
  **L1652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead.`。
- **L1653 EN**: Separator comment used for visual grouping.
  **L1653 CN**: 用于视觉分组的分隔注释。
- **L1654 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1654 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1655 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1655 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1656 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1656 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。

### Lines 1657-1680

````c
/// 		IF i < 8
/// 			dst.hf8[i] := convert_fp16_to_hf8(__B.fp16[i])
/// 		ELSE
/// 			dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i - 8])
/// 		FI
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
````
- **L1657 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1657 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1658 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`.
  **L1658 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`。
- **L1659 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1659 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1660 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 8])`.
  **L1660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 8])`。
- **L1661 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1661 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1662 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1662 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1663 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L1663 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L1664 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1665 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1665 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1666 EN**: Separator comment used for visual grouping.
  **L1666 CN**: 用于视觉分组的分隔注释。
- **L1667 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1667 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1668 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1668 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1669 EN**: Separator comment used for visual grouping.
  **L1669 CN**: 用于视觉分组的分隔注释。
- **L1670 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1670 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1671 EN**: Separator comment used for visual grouping.
  **L1671 CN**: 用于视觉分组的分隔注释。
- **L1672 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`.
  **L1672 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`。
- **L1673 EN**: Separator comment used for visual grouping.
  **L1673 CN**: 用于视觉分组的分隔注释。
- **L1674 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L1674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L1675 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L1675 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L1676 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1676 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1677 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L1677 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L1678 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1679 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1679 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1680 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1680 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。

### Lines 1681-1704

````c
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvt2ph_hf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_cvt2ph_hf8(__A, __B), (__v16qi)__W);
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E4M3 FP8 elements.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		IF i < 8
/// 			dst.hf8[i] := convert_fp16_to_hf8(__B.fp16[i])
/// 		ELSE
/// 			dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i - 8])
````
- **L1681 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1681 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1682 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1682 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1683 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`.
  **L1683 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`。
- **L1684 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1685 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1685 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1686 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L1686 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L1687 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1687 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1688 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvt2ph_hf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {`.
  **L1688 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvt2ph_hf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {`。
- **L1689 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1689 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1690 EN**: Executes a call or declaration centered on `statement`.
  **L1690 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1691 EN**: Closes the current lexical scope or compound statement.
  **L1691 CN**: 结束当前词法作用域或复合语句块。
- **L1692 EN**: Blank line separating nearby declarations or logic blocks.
  **L1692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1693 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1693 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1694 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`.
  **L1694 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`。
- **L1695 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L1695 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L1696 EN**: Comment explains nearby logic, constraints, or intent: `instead.`.
  **L1696 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead.`。
- **L1697 EN**: Separator comment used for visual grouping.
  **L1697 CN**: 用于视觉分组的分隔注释。
- **L1698 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1698 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1699 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1699 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1700 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1700 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1701 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1701 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1702 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`.
  **L1702 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`。
- **L1703 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1703 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1704 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 8])`.
  **L1704 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 8])`。

### Lines 1705-1728

````c
/// 		FI
/// 	ELSE
/// 		dst.hf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8 instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    zero is taken instead.
````
- **L1705 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1706 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1706 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1707 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L1707 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L1708 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1708 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1709 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1709 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1710 EN**: Separator comment used for visual grouping.
  **L1710 CN**: 用于视觉分组的分隔注释。
- **L1711 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1711 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1712 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1712 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1713 EN**: Separator comment used for visual grouping.
  **L1713 CN**: 用于视觉分组的分隔注释。
- **L1714 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1715 EN**: Separator comment used for visual grouping.
  **L1715 CN**: 用于视觉分组的分隔注释。
- **L1716 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`.
  **L1716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`。
- **L1717 EN**: Separator comment used for visual grouping.
  **L1717 CN**: 用于视觉分组的分隔注释。
- **L1718 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1718 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1719 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L1719 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L1720 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1720 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1721 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1722 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1722 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1723 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1723 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1724 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1724 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1725 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`.
  **L1725 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`。
- **L1726 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1726 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1727 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1727 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1728 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L1728 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。

### Lines 1729-1752

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvt2ph_hf8(__mmask16 __U, __m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_cvt2ph_hf8(__A, __B),
      (__v16qi)(__m128i)_mm_setzero_si128());
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E4M3 FP8 elements.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF i < 16 
/// 		dst.hf8[i] := convert_fp16_to_hf8(__B.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i - 16])
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
````
- **L1729 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1729 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1730 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvt2ph_hf8(__mmask16 __U, __m128h __A, __m128h __B) {`.
  **L1730 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvt2ph_hf8(__mmask16 __U, __m128h __A, __m128h __B) {`。
- **L1731 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1731 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16qi)_mm_cvt2ph_hf8(__A, __B),`.
  **L1732 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16qi)_mm_cvt2ph_hf8(__A, __B),`。
- **L1733 EN**: Executes a call or declaration centered on `statement`.
  **L1733 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1734 EN**: Closes the current lexical scope or compound statement.
  **L1734 CN**: 结束当前词法作用域或复合语句块。
- **L1735 EN**: Blank line separating nearby declarations or logic blocks.
  **L1735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1736 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1737 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`.
  **L1737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`。
- **L1738 EN**: Separator comment used for visual grouping.
  **L1738 CN**: 用于视觉分组的分隔注释。
- **L1739 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1740 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1740 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1741 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1741 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1742 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`.
  **L1742 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`。
- **L1743 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1744 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 16])`.
  **L1744 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 16])`。
- **L1745 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1745 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1746 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1747 EN**: Separator comment used for visual grouping.
  **L1747 CN**: 用于视觉分组的分隔注释。
- **L1748 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1749 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1749 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1750 EN**: Separator comment used for visual grouping.
  **L1750 CN**: 用于视觉分组的分隔注释。
- **L1751 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1751 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1752 EN**: Separator comment used for visual grouping.
  **L1752 CN**: 用于视觉分组的分隔注释。

### Lines 1753-1776

````c
/// This intrinsic corresponds to the \c VCVT2PH2HF8 instruction.
///
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvt2ph_hf8(__m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_vcvt2ph2hf8_256((__v16hf)(__A),
                                                   (__v16hf)(__B));
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E4M3 FP8 elements.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF __U[i]
````
- **L1753 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`.
  **L1753 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`。
- **L1754 EN**: Separator comment used for visual grouping.
  **L1754 CN**: 用于视觉分组的分隔注释。
- **L1755 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1756 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1756 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1757 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1758 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1758 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1759 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1759 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1760 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`.
  **L1760 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`。
- **L1761 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1761 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1762 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L1762 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L1763 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1763 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1764 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvt2ph_hf8(__m256h __A, __m256h __B) {`.
  **L1764 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvt2ph_hf8(__m256h __A, __m256h __B) {`。
- **L1765 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvt2ph2hf8_256((__v16hf)(__A),`.
  **L1765 CN**: 以 `(__m256i)__builtin_ia32_vcvt2ph2hf8_256((__v16hf)(__A),` 从当前函数返回。
- **L1766 EN**: Executes a call or declaration centered on `statement`.
  **L1766 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1767 EN**: Closes the current lexical scope or compound statement.
  **L1767 CN**: 结束当前词法作用域或复合语句块。
- **L1768 EN**: Blank line separating nearby declarations or logic blocks.
  **L1768 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1769 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1769 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1770 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`.
  **L1770 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`。
- **L1771 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L1771 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L1772 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead.`.
  **L1772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead.`。
- **L1773 EN**: Separator comment used for visual grouping.
  **L1773 CN**: 用于视觉分组的分隔注释。
- **L1774 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1774 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1775 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1775 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1776 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1776 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。

### Lines 1777-1800

````c
/// 		IF i < 16 
/// 			dst.hf8[i] := convert_fp16_to_hf8(__B.fp16[i])
/// 		ELSE
/// 			dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i - 16])
/// 		FI
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8 instruction.
///
/// \param __W
///    A 256-bit vector of [32 x hf8].
/// \param __U
///    A 32-bit merging mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
````
- **L1777 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1777 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1778 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`.
  **L1778 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`。
- **L1779 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1779 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1780 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 16])`.
  **L1780 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 16])`。
- **L1781 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1781 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1782 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1783 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L1783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L1784 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1785 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1785 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1786 EN**: Separator comment used for visual grouping.
  **L1786 CN**: 用于视觉分组的分隔注释。
- **L1787 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1787 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1788 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1788 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1789 EN**: Separator comment used for visual grouping.
  **L1789 CN**: 用于视觉分组的分隔注释。
- **L1790 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1790 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1791 EN**: Separator comment used for visual grouping.
  **L1791 CN**: 用于视觉分组的分隔注释。
- **L1792 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`.
  **L1792 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`。
- **L1793 EN**: Separator comment used for visual grouping.
  **L1793 CN**: 用于视觉分组的分隔注释。
- **L1794 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L1794 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L1795 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8].`.
  **L1795 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8].`。
- **L1796 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1796 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1797 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit merging mask.`.
  **L1797 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit merging mask.`。
- **L1798 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1798 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1799 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1799 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1800 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1800 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。

### Lines 1801-1824

````c
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m256i __DEFAULT_FN_ATTRS256 _mm256_mask_cvt2ph_hf8(
    __m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_cvt2ph_hf8(__A, __B), (__v32qi)__W);
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E4M3 FP8 elements.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF __U[i]
/// 		IF i < 16 
/// 			dst.hf8[i] := convert_fp16_to_hf8(__B.fp16[i])
/// 		ELSE
/// 			dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i - 16])
````
- **L1801 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1801 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1802 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1802 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1803 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`.
  **L1803 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`。
- **L1804 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1804 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1805 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1805 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1806 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L1806 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L1807 EN**: Continues logic associated with callable symbol `_mm256_mask_cvt2ph_hf8`.
  **L1807 CN**: 继续与可调用符号 `_mm256_mask_cvt2ph_hf8` 相关的逻辑。
- **L1808 EN**: Continues the surrounding expression or declaration: `__m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {`.
  **L1808 CN**: 继续构造周围的表达式或声明：`__m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {`。
- **L1809 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L1809 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L1810 EN**: Executes a call or declaration centered on `statement`.
  **L1810 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1811 EN**: Closes the current lexical scope or compound statement.
  **L1811 CN**: 结束当前词法作用域或复合语句块。
- **L1812 EN**: Blank line separating nearby declarations or logic blocks.
  **L1812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1813 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1813 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1814 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`.
  **L1814 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`。
- **L1815 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L1815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L1816 EN**: Comment explains nearby logic, constraints, or intent: `instead.`.
  **L1816 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead.`。
- **L1817 EN**: Separator comment used for visual grouping.
  **L1817 CN**: 用于视觉分组的分隔注释。
- **L1818 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1818 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1819 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1819 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1820 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1821 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1821 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1822 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`.
  **L1822 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__B.fp16[i])`。
- **L1823 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1823 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1824 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 16])`.
  **L1824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i - 16])`。

### Lines 1825-1848

````c
/// 		FI
/// 	ELSE
/// 		dst.hf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8 instruction.
///
/// \param __U
///    A 32-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    zero is taken instead.
````
- **L1825 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1825 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1826 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1826 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1827 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L1827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L1828 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1828 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1829 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1830 EN**: Separator comment used for visual grouping.
  **L1830 CN**: 用于视觉分组的分隔注释。
- **L1831 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1831 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1832 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1832 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1833 EN**: Separator comment used for visual grouping.
  **L1833 CN**: 用于视觉分组的分隔注释。
- **L1834 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1834 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1835 EN**: Separator comment used for visual grouping.
  **L1835 CN**: 用于视觉分组的分隔注释。
- **L1836 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`.
  **L1836 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8 instruction.`。
- **L1837 EN**: Separator comment used for visual grouping.
  **L1837 CN**: 用于视觉分组的分隔注释。
- **L1838 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1838 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1839 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit zeroing mask.`.
  **L1839 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit zeroing mask.`。
- **L1840 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1840 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1841 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1841 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1842 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1842 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1843 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1844 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1844 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1845 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`.
  **L1845 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`。
- **L1846 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1846 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1847 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L1847 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L1848 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L1848 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。

### Lines 1849-1872

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvt2ph_hf8(__mmask32 __U, __m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_cvt2ph_hf8(__A, __B),
      (__v32qi)(__m256i)_mm256_setzero_si256());
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E4M3 FP8 elements.
///    Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF i < 8
/// 		dst.hf8[i] := convert_fp16_to_hf8_saturate(__B.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i - 8])
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
````
- **L1849 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1849 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1850 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvt2ph_hf8(__mmask32 __U, __m256h __A, __m256h __B) {`.
  **L1850 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvt2ph_hf8(__mmask32 __U, __m256h __A, __m256h __B) {`。
- **L1851 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L1851 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L1852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask32)__U, (__v32qi)_mm256_cvt2ph_hf8(__A, __B),`.
  **L1852 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask32)__U, (__v32qi)_mm256_cvt2ph_hf8(__A, __B),`。
- **L1853 EN**: Executes a call or declaration centered on `statement`.
  **L1853 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1854 EN**: Closes the current lexical scope or compound statement.
  **L1854 CN**: 结束当前词法作用域或复合语句块。
- **L1855 EN**: Blank line separating nearby declarations or logic blocks.
  **L1855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1856 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1857 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`.
  **L1857 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`。
- **L1858 EN**: Comment explains nearby logic, constraints, or intent: `Resulting elements are saturated in case of overflow.`.
  **L1858 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Resulting elements are saturated in case of overflow.`。
- **L1859 EN**: Separator comment used for visual grouping.
  **L1859 CN**: 用于视觉分组的分隔注释。
- **L1860 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1860 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1861 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1861 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1862 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1862 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1863 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`.
  **L1863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`。
- **L1864 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1864 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1865 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 8])`.
  **L1865 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 8])`。
- **L1866 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1866 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1867 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1867 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1868 EN**: Separator comment used for visual grouping.
  **L1868 CN**: 用于视觉分组的分隔注释。
- **L1869 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1869 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1870 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1870 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1871 EN**: Separator comment used for visual grouping.
  **L1871 CN**: 用于视觉分组的分隔注释。
- **L1872 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1872 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。

### Lines 1873-1896

````c
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8S instruction.
///
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvts_2ph_hf8(__m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_vcvt2ph2hf8s_128((__v8hf)(__A),
                                                    (__v8hf)(__B));
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E4M3 FP8 elements.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead. Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 15 
````
- **L1873 EN**: Separator comment used for visual grouping.
  **L1873 CN**: 用于视觉分组的分隔注释。
- **L1874 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`.
  **L1874 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`。
- **L1875 EN**: Separator comment used for visual grouping.
  **L1875 CN**: 用于视觉分组的分隔注释。
- **L1876 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1876 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1877 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1877 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1878 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1878 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1879 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1879 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1880 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1880 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1881 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`.
  **L1881 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`。
- **L1882 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1882 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1883 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L1883 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L1884 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1884 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1885 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvts_2ph_hf8(__m128h __A, __m128h __B) {`.
  **L1885 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvts_2ph_hf8(__m128h __A, __m128h __B) {`。
- **L1886 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvt2ph2hf8s_128((__v8hf)(__A),`.
  **L1886 CN**: 以 `(__m128i)__builtin_ia32_vcvt2ph2hf8s_128((__v8hf)(__A),` 从当前函数返回。
- **L1887 EN**: Executes a call or declaration centered on `statement`.
  **L1887 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1888 EN**: Closes the current lexical scope or compound statement.
  **L1888 CN**: 结束当前词法作用域或复合语句块。
- **L1889 EN**: Blank line separating nearby declarations or logic blocks.
  **L1889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1890 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1890 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1891 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`.
  **L1891 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`。
- **L1892 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L1892 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L1893 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead. Resulting elements are saturated in case of overflow.`.
  **L1893 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead. Resulting elements are saturated in case of overflow.`。
- **L1894 EN**: Separator comment used for visual grouping.
  **L1894 CN**: 用于视觉分组的分隔注释。
- **L1895 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1895 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1896 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1896 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。

### Lines 1897-1920

````c
/// 	IF __U[i]
/// 		IF i < 8
/// 			dst.hf8[i] := convert_fp16_to_hf8_saturate(__B.fp16[i])
/// 		ELSE
/// 			dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i - 8])
/// 		FI
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
````
- **L1897 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1897 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1898 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1898 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1899 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`.
  **L1899 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`。
- **L1900 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1900 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1901 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 8])`.
  **L1901 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 8])`。
- **L1902 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1902 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1903 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1903 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1904 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L1904 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L1905 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1905 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1906 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1906 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1907 EN**: Separator comment used for visual grouping.
  **L1907 CN**: 用于视觉分组的分隔注释。
- **L1908 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1908 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1909 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1909 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1910 EN**: Separator comment used for visual grouping.
  **L1910 CN**: 用于视觉分组的分隔注释。
- **L1911 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1911 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1912 EN**: Separator comment used for visual grouping.
  **L1912 CN**: 用于视觉分组的分隔注释。
- **L1913 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`.
  **L1913 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`。
- **L1914 EN**: Separator comment used for visual grouping.
  **L1914 CN**: 用于视觉分组的分隔注释。
- **L1915 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L1915 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L1916 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L1916 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L1917 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1917 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1918 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L1918 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L1919 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1919 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1920 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1920 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。

### Lines 1921-1944

````c
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvts_2ph_hf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_cvts_2ph_hf8(__A, __B), (__v16qi)__W);
}

/// Convert two 128-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 128-bit vector containing E4M3 FP8 elements.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead. Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		IF i < 8
/// 			dst.hf8[i] := convert_fp16_to_hf8_saturate(__B.fp16[i])
/// 		ELSE
````
- **L1921 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1921 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1922 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1922 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1923 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1923 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1924 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`.
  **L1924 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`。
- **L1925 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1925 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1926 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1926 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L1927 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L1927 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L1928 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1928 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1929 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvts_2ph_hf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {`.
  **L1929 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvts_2ph_hf8(__m128i __W, __mmask16 __U, __m128h __A, __m128h __B) {`。
- **L1930 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1930 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1931 EN**: Executes a call or declaration centered on `statement`.
  **L1931 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1932 EN**: Closes the current lexical scope or compound statement.
  **L1932 CN**: 结束当前词法作用域或复合语句块。
- **L1933 EN**: Blank line separating nearby declarations or logic blocks.
  **L1933 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1934 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 128-bit vectors, a __A and a __B, containing packed FP16`.
  **L1934 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 128-bit vectors, a __A and a __B, containing packed FP16`。
- **L1935 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`.
  **L1935 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 128-bit vector containing E4M3 FP8 elements.`。
- **L1936 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L1936 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L1937 EN**: Comment explains nearby logic, constraints, or intent: `instead. Resulting elements are saturated in case of overflow.`.
  **L1937 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead. Resulting elements are saturated in case of overflow.`。
- **L1938 EN**: Separator comment used for visual grouping.
  **L1938 CN**: 用于视觉分组的分隔注释。
- **L1939 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1939 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1940 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L1940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L1941 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L1941 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L1942 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 8`.
  **L1942 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 8`。
- **L1943 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`.
  **L1943 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`。
- **L1944 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1944 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。

### Lines 1945-1968

````c
/// 			dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i - 8])
/// 		FI
/// 	ELSE
/// 		dst.hf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8S instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \param __B
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
````
- **L1945 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 8])`.
  **L1945 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 8])`。
- **L1946 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1946 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1947 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1947 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1948 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L1948 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L1949 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1949 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1950 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1950 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1951 EN**: Separator comment used for visual grouping.
  **L1951 CN**: 用于视觉分组的分隔注释。
- **L1952 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L1952 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L1953 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1953 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1954 EN**: Separator comment used for visual grouping.
  **L1954 CN**: 用于视觉分组的分隔注释。
- **L1955 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1955 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1956 EN**: Separator comment used for visual grouping.
  **L1956 CN**: 用于视觉分组的分隔注释。
- **L1957 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`.
  **L1957 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`。
- **L1958 EN**: Separator comment used for visual grouping.
  **L1958 CN**: 用于视觉分组的分隔注释。
- **L1959 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L1959 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L1960 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L1960 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L1961 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1961 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1962 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1962 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1963 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1963 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L1964 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L1964 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L1965 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L1965 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L1966 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`.
  **L1966 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower 8 elements correspond to the`。
- **L1967 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L1967 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L1968 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L1968 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。

### Lines 1969-1992

````c
///    zero is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvts_2ph_hf8(__mmask16 __U, __m128h __A, __m128h __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_cvts_2ph_hf8(__A, __B),
      (__v16qi)(__m128i)_mm_setzero_si128());
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E4M3 FP8 elements.
///    Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF i < 16 
/// 		dst.hf8[i] := convert_fp16_to_hf8_saturate(__B.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i - 16])
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
````
- **L1969 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L1969 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L1970 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1970 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1971 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvts_2ph_hf8(__mmask16 __U, __m128h __A, __m128h __B) {`.
  **L1971 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvts_2ph_hf8(__mmask16 __U, __m128h __A, __m128h __B) {`。
- **L1972 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1972 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16qi)_mm_cvts_2ph_hf8(__A, __B),`.
  **L1973 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16qi)_mm_cvts_2ph_hf8(__A, __B),`。
- **L1974 EN**: Executes a call or declaration centered on `statement`.
  **L1974 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1975 EN**: Closes the current lexical scope or compound statement.
  **L1975 CN**: 结束当前词法作用域或复合语句块。
- **L1976 EN**: Blank line separating nearby declarations or logic blocks.
  **L1976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1977 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L1977 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L1978 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`.
  **L1978 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`。
- **L1979 EN**: Comment explains nearby logic, constraints, or intent: `Resulting elements are saturated in case of overflow.`.
  **L1979 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Resulting elements are saturated in case of overflow.`。
- **L1980 EN**: Separator comment used for visual grouping.
  **L1980 CN**: 用于视觉分组的分隔注释。
- **L1981 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L1981 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L1982 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L1982 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L1983 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L1983 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L1984 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`.
  **L1984 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`。
- **L1985 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L1985 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L1986 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 16])`.
  **L1986 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 16])`。
- **L1987 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L1987 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L1988 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L1988 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L1989 EN**: Separator comment used for visual grouping.
  **L1989 CN**: 用于视觉分组的分隔注释。
- **L1990 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L1990 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L1991 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L1991 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L1992 EN**: Separator comment used for visual grouping.
  **L1992 CN**: 用于视觉分组的分隔注释。

### Lines 1993-2016

````c
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8S instruction.
///
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvts_2ph_hf8(__m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_vcvt2ph2hf8s_256((__v16hf)(__A),
                                                    (__v16hf)(__B));
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E4M3 FP8 elements.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead. Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
````
- **L1993 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L1993 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L1994 EN**: Separator comment used for visual grouping.
  **L1994 CN**: 用于视觉分组的分隔注释。
- **L1995 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`.
  **L1995 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`。
- **L1996 EN**: Separator comment used for visual grouping.
  **L1996 CN**: 用于视觉分组的分隔注释。
- **L1997 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L1997 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L1998 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L1998 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L1999 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L1999 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L2000 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2000 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2001 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2001 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2002 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`.
  **L2002 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`。
- **L2003 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L2003 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L2004 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L2004 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L2005 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2005 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2006 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvts_2ph_hf8(__m256h __A, __m256h __B) {`.
  **L2006 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvts_2ph_hf8(__m256h __A, __m256h __B) {`。
- **L2007 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vcvt2ph2hf8s_256((__v16hf)(__A),`.
  **L2007 CN**: 以 `(__m256i)__builtin_ia32_vcvt2ph2hf8s_256((__v16hf)(__A),` 从当前函数返回。
- **L2008 EN**: Executes a call or declaration centered on `statement`.
  **L2008 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2009 EN**: Closes the current lexical scope or compound statement.
  **L2009 CN**: 结束当前词法作用域或复合语句块。
- **L2010 EN**: Blank line separating nearby declarations or logic blocks.
  **L2010 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2011 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L2011 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L2012 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`.
  **L2012 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`。
- **L2013 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L2013 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L2014 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead. Resulting elements are saturated in case of overflow.`.
  **L2014 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead. Resulting elements are saturated in case of overflow.`。
- **L2015 EN**: Separator comment used for visual grouping.
  **L2015 CN**: 用于视觉分组的分隔注释。
- **L2016 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2016 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。

### Lines 2017-2040

````c
/// FOR i := 0 to 31 
/// 	IF __U[i]
/// 		IF i < 16 
/// 			dst.hf8[i] := convert_fp16_to_hf8_saturate(__B.fp16[i])
/// 		ELSE
/// 			dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i - 16])
/// 		FI
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8S instruction.
///
/// \param __W
///    A 256-bit vector of [32 x hf8].
/// \param __U
///    A 32-bit merging mask.
/// \param __A
````
- **L2017 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L2017 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L2018 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2018 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2019 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L2019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L2020 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`.
  **L2020 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`。
- **L2021 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2021 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2022 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 16])`.
  **L2022 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 16])`。
- **L2023 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2023 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2024 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2024 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2025 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L2025 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L2026 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2026 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2027 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2028 EN**: Separator comment used for visual grouping.
  **L2028 CN**: 用于视觉分组的分隔注释。
- **L2029 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L2029 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L2030 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2030 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2031 EN**: Separator comment used for visual grouping.
  **L2031 CN**: 用于视觉分组的分隔注释。
- **L2032 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2032 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2033 EN**: Separator comment used for visual grouping.
  **L2033 CN**: 用于视觉分组的分隔注释。
- **L2034 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`.
  **L2034 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`。
- **L2035 EN**: Separator comment used for visual grouping.
  **L2035 CN**: 用于视觉分组的分隔注释。
- **L2036 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2037 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8].`.
  **L2037 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8].`。
- **L2038 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2038 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2039 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit merging mask.`.
  **L2039 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit merging mask.`。
- **L2040 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。

### Lines 2041-2064

````c
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m256i __DEFAULT_FN_ATTRS256 _mm256_mask_cvts_2ph_hf8(
    __m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_cvts_2ph_hf8(__A, __B), (__v32qi)__W);
}

/// Convert two 256-bit vectors, \a __A and \a __B, containing packed FP16
///    floating-point elements to a 256-bit vector containing E4M3 FP8 elements.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead. Resulting elements are saturated in case of overflow.
///
/// \code{.operation}
/// FOR i := 0 to 31 
/// 	IF __U[i]
/// 		IF i < 16 
/// 			dst.hf8[i] := convert_fp16_to_hf8_saturate(__B.fp16[i])
````
- **L2041 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2041 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2042 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L2042 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L2043 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2043 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2044 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2044 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2045 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`.
  **L2045 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`。
- **L2046 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L2046 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。
- **L2047 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L2047 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L2048 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L2048 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L2049 EN**: Continues logic associated with callable symbol `_mm256_mask_cvts_2ph_hf8`.
  **L2049 CN**: 继续与可调用符号 `_mm256_mask_cvts_2ph_hf8` 相关的逻辑。
- **L2050 EN**: Continues the surrounding expression or declaration: `__m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {`.
  **L2050 CN**: 继续构造周围的表达式或声明：`__m256i __W, __mmask32 __U, __m256h __A, __m256h __B) {`。
- **L2051 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L2051 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L2052 EN**: Executes a call or declaration centered on `statement`.
  **L2052 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2053 EN**: Closes the current lexical scope or compound statement.
  **L2053 CN**: 结束当前词法作用域或复合语句块。
- **L2054 EN**: Blank line separating nearby declarations or logic blocks.
  **L2054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2055 EN**: Comment explains nearby logic, constraints, or intent: `Convert two 256-bit vectors, a __A and a __B, containing packed FP16`.
  **L2055 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert two 256-bit vectors, a __A and a __B, containing packed FP16`。
- **L2056 EN**: Comment explains nearby logic, constraints, or intent: `floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`.
  **L2056 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating-point elements to a 256-bit vector containing E4M3 FP8 elements.`。
- **L2057 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L2057 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L2058 EN**: Comment explains nearby logic, constraints, or intent: `instead. Resulting elements are saturated in case of overflow.`.
  **L2058 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead. Resulting elements are saturated in case of overflow.`。
- **L2059 EN**: Separator comment used for visual grouping.
  **L2059 CN**: 用于视觉分组的分隔注释。
- **L2060 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2060 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2061 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 31`.
  **L2061 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 31`。
- **L2062 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2062 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2063 EN**: Comment explains nearby logic, constraints, or intent: `IF i < 16`.
  **L2063 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF i < 16`。
- **L2064 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`.
  **L2064 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__B.fp16[i])`。

### Lines 2065-2088

````c
/// 		ELSE
/// 			dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i - 16])
/// 		FI
/// 	ELSE
/// 		dst.hf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVT2PH2HF8S instruction.
///
/// \param __U
///    A 32-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \param __B
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the
///    (converted) elements from \a __B; higher order elements correspond to the
````
- **L2065 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2065 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2066 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 16])`.
  **L2066 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i - 16])`。
- **L2067 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2067 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2068 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2068 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2069 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L2069 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L2070 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2070 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2071 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2071 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2072 EN**: Separator comment used for visual grouping.
  **L2072 CN**: 用于视觉分组的分隔注释。
- **L2073 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L2073 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L2074 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2074 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2075 EN**: Separator comment used for visual grouping.
  **L2075 CN**: 用于视觉分组的分隔注释。
- **L2076 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2076 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2077 EN**: Separator comment used for visual grouping.
  **L2077 CN**: 用于视觉分组的分隔注释。
- **L2078 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`.
  **L2078 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVT2PH2HF8S instruction.`。
- **L2079 EN**: Separator comment used for visual grouping.
  **L2079 CN**: 用于视觉分组的分隔注释。
- **L2080 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2080 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2081 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit zeroing mask.`.
  **L2081 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit zeroing mask.`。
- **L2082 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2082 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2083 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2083 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2084 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L2084 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L2085 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2085 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2086 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2086 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2087 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`.
  **L2087 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8]. Lower 16 elements correspond to the`。
- **L2088 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __B; higher order elements correspond to the`.
  **L2088 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __B; higher order elements correspond to the`。

### Lines 2089-2112

````c
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    zero is taken instead.
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvts_2ph_hf8(__mmask32 __U, __m256h __A, __m256h __B) {
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_cvts_2ph_hf8(__A, __B),
      (__v32qi)(__m256i)_mm256_setzero_si256());
}

/// Convert 128-bit vector \a __A, containing packed FP8 E4M3 floating-point
///    elements to a 128-bit vector containing FP16 elements. The conversion is exact.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.fp16[i] := convert_hf8_to_fp16(__A.hf8[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTHF82PH instruction.
///
````
- **L2089 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L2089 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L2090 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L2090 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L2091 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2091 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2092 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvts_2ph_hf8(__mmask32 __U, __m256h __A, __m256h __B) {`.
  **L2092 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvts_2ph_hf8(__mmask32 __U, __m256h __A, __m256h __B) {`。
- **L2093 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L2093 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L2094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask32)__U, (__v32qi)_mm256_cvts_2ph_hf8(__A, __B),`.
  **L2094 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask32)__U, (__v32qi)_mm256_cvts_2ph_hf8(__A, __B),`。
- **L2095 EN**: Executes a call or declaration centered on `statement`.
  **L2095 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2096 EN**: Closes the current lexical scope or compound statement.
  **L2096 CN**: 结束当前词法作用域或复合语句块。
- **L2097 EN**: Blank line separating nearby declarations or logic blocks.
  **L2097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2098 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A, containing packed FP8 E4M3 floating-point`.
  **L2098 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A, containing packed FP8 E4M3 floating-point`。
- **L2099 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 128-bit vector containing FP16 elements. The conversion is exact.`.
  **L2099 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 128-bit vector containing FP16 elements. The conversion is exact.`。
- **L2100 EN**: Separator comment used for visual grouping.
  **L2100 CN**: 用于视觉分组的分隔注释。
- **L2101 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2102 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2103 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`.
  **L2103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`。
- **L2104 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2105 EN**: Separator comment used for visual grouping.
  **L2105 CN**: 用于视觉分组的分隔注释。
- **L2106 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2107 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2108 EN**: Separator comment used for visual grouping.
  **L2108 CN**: 用于视觉分组的分隔注释。
- **L2109 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2110 EN**: Separator comment used for visual grouping.
  **L2110 CN**: 用于视觉分组的分隔注释。
- **L2111 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTHF82PH instruction.`.
  **L2111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTHF82PH instruction.`。
- **L2112 EN**: Separator comment used for visual grouping.
  **L2112 CN**: 用于视觉分组的分隔注释。

### Lines 2113-2136

````c
/// \param __A
///    A 128-bit vector of [16 x hf8].
/// \returns
///    A 128-bit vector of [8 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvthf8_ph(__m128i __A) {
  return (__m128h)__builtin_ia32_vcvthf8_2ph128_mask(
      (__v16qi)__A, (__v8hf)(__m128h)_mm_undefined_ph(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __A, containing packed FP8 E4M3 floating-point
///    elements to a 128-bit vector containing FP16 elements. The conversion is
///    exact. Merging mask \a __U is used to determine if given element should be
///    taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.fp16[i] := convert_hf8_to_fp16(__A.hf8[i])
/// 	ELSE
/// 		dst.fp16[i] := __W.fp16[i]
/// 	FI
/// ENDFOR
///
````
- **L2113 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2114 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L2114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L2115 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2116 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`.
  **L2116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`。
- **L2117 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L2117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L2118 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvthf8_ph(__m128i __A) {`.
  **L2118 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvthf8_ph(__m128i __A) {`。
- **L2119 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvthf8_2ph128_mask(`.
  **L2119 CN**: 以 `(__m128h)__builtin_ia32_vcvthf8_2ph128_mask(` 从当前函数返回。
- **L2120 EN**: Executes a call or declaration centered on `statement`.
  **L2120 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2121 EN**: Closes the current lexical scope or compound statement.
  **L2121 CN**: 结束当前词法作用域或复合语句块。
- **L2122 EN**: Blank line separating nearby declarations or logic blocks.
  **L2122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2123 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A, containing packed FP8 E4M3 floating-point`.
  **L2123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A, containing packed FP8 E4M3 floating-point`。
- **L2124 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 128-bit vector containing FP16 elements. The conversion is`.
  **L2124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 128-bit vector containing FP16 elements. The conversion is`。
- **L2125 EN**: Comment explains nearby logic, constraints, or intent: `exact. Merging mask a __U is used to determine if given element should be`.
  **L2125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exact. Merging mask a __U is used to determine if given element should be`。
- **L2126 EN**: Comment explains nearby logic, constraints, or intent: `taken from a __W instead.`.
  **L2126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`taken from a __W instead.`。
- **L2127 EN**: Separator comment used for visual grouping.
  **L2127 CN**: 用于视觉分组的分隔注释。
- **L2128 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2129 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2130 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2131 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`.
  **L2131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`。
- **L2132 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2133 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : __W.fp16[i]`.
  **L2133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : __W.fp16[i]`。
- **L2134 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2135 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2136 EN**: Separator comment used for visual grouping.
  **L2136 CN**: 用于视觉分组的分隔注释。

### Lines 2137-2160

````c
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTHF82PH instruction.
///
/// \param __W
///    A 128-bit vector of [8 x fp16].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [16 x hf8].
/// \returns
///    A 128-bit vector of [8 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_cvthf8_ph(__m128h __W, __mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvthf8_2ph128_mask(
      (__v16qi)__A, (__v8hf)(__m128h)__W, (__mmask8)__U);
}

/// Convert 128-bit vector \a __A, containing packed FP8 E4M3 floating-point
````
- **L2137 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2138 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2139 EN**: Separator comment used for visual grouping.
  **L2139 CN**: 用于视觉分组的分隔注释。
- **L2140 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2141 EN**: Separator comment used for visual grouping.
  **L2141 CN**: 用于视觉分组的分隔注释。
- **L2142 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTHF82PH instruction.`.
  **L2142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTHF82PH instruction.`。
- **L2143 EN**: Separator comment used for visual grouping.
  **L2143 CN**: 用于视觉分组的分隔注释。
- **L2144 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2145 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2145 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2146 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2147 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L2147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L2148 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2149 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L2149 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L2150 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2151 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`.
  **L2151 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`。
- **L2152 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L2152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L2153 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L2153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L2154 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2154 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2155 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvthf8_ph(__m128h __W, __mmask8 __U, __m128i __A) {`.
  **L2155 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvthf8_ph(__m128h __W, __mmask8 __U, __m128i __A) {`。
- **L2156 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvthf8_2ph128_mask(`.
  **L2156 CN**: 以 `(__m128h)__builtin_ia32_vcvthf8_2ph128_mask(` 从当前函数返回。
- **L2157 EN**: Executes a call or declaration centered on `statement`.
  **L2157 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2158 EN**: Closes the current lexical scope or compound statement.
  **L2158 CN**: 结束当前词法作用域或复合语句块。
- **L2159 EN**: Blank line separating nearby declarations or logic blocks.
  **L2159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2160 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A, containing packed FP8 E4M3 floating-point`.
  **L2160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A, containing packed FP8 E4M3 floating-point`。

### Lines 2161-2184

````c
///    elements to a 128-bit vector containing FP16 elements. The conversion is
///    exact. Zeroing mask \a __U is used to determine if given element should be
///    zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.fp16[i] := convert_hf8_to_fp16(__A.hf8[i])
/// 	ELSE
/// 		dst.fp16[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTHF82PH instruction.
///
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [16 x hf8].
````
- **L2161 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 128-bit vector containing FP16 elements. The conversion is`.
  **L2161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 128-bit vector containing FP16 elements. The conversion is`。
- **L2162 EN**: Comment explains nearby logic, constraints, or intent: `exact. Zeroing mask a __U is used to determine if given element should be`.
  **L2162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exact. Zeroing mask a __U is used to determine if given element should be`。
- **L2163 EN**: Comment explains nearby logic, constraints, or intent: `zeroed instead.`.
  **L2163 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zeroed instead.`。
- **L2164 EN**: Separator comment used for visual grouping.
  **L2164 CN**: 用于视觉分组的分隔注释。
- **L2165 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2166 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2167 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2168 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`.
  **L2168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`。
- **L2169 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2170 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : 0`.
  **L2170 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : 0`。
- **L2171 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2171 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2172 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2173 EN**: Separator comment used for visual grouping.
  **L2173 CN**: 用于视觉分组的分隔注释。
- **L2174 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2175 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2176 EN**: Separator comment used for visual grouping.
  **L2176 CN**: 用于视觉分组的分隔注释。
- **L2177 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2178 EN**: Separator comment used for visual grouping.
  **L2178 CN**: 用于视觉分组的分隔注释。
- **L2179 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTHF82PH instruction.`.
  **L2179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTHF82PH instruction.`。
- **L2180 EN**: Separator comment used for visual grouping.
  **L2180 CN**: 用于视觉分组的分隔注释。
- **L2181 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2182 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L2182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L2183 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2184 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L2184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。

### Lines 2185-2208

````c
/// \returns
///    A 128-bit vector of [8 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    zero is taken instead.
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvthf8_ph(__mmask8 __U, __m128i __A) {
  return (__m128h)__builtin_ia32_vcvthf8_2ph128_mask(
      (__v16qi)__A, (__v8hf)(__m128h)_mm_setzero_ph(), (__mmask8)__U);
}

/// Convert 256-bit vector \a __A, containing packed FP8 E4M3 floating-point
///    elements to a 256-bit vector containing FP16 elements. The conversion is exact.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	dst.fp16[i] := convert_hf8_to_fp16(__A.hf8[i])
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTHF82PH instruction.
````
- **L2185 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2186 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`.
  **L2186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`。
- **L2187 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L2187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L2188 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L2188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L2189 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2189 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2190 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvthf8_ph(__mmask8 __U, __m128i __A) {`.
  **L2190 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvthf8_ph(__mmask8 __U, __m128i __A) {`。
- **L2191 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvthf8_2ph128_mask(`.
  **L2191 CN**: 以 `(__m128h)__builtin_ia32_vcvthf8_2ph128_mask(` 从当前函数返回。
- **L2192 EN**: Executes a call or declaration centered on `statement`.
  **L2192 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2193 EN**: Closes the current lexical scope or compound statement.
  **L2193 CN**: 结束当前词法作用域或复合语句块。
- **L2194 EN**: Blank line separating nearby declarations or logic blocks.
  **L2194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2195 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A, containing packed FP8 E4M3 floating-point`.
  **L2195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A, containing packed FP8 E4M3 floating-point`。
- **L2196 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 256-bit vector containing FP16 elements. The conversion is exact.`.
  **L2196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 256-bit vector containing FP16 elements. The conversion is exact.`。
- **L2197 EN**: Separator comment used for visual grouping.
  **L2197 CN**: 用于视觉分组的分隔注释。
- **L2198 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2199 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2199 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2200 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`.
  **L2200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`。
- **L2201 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2202 EN**: Separator comment used for visual grouping.
  **L2202 CN**: 用于视觉分组的分隔注释。
- **L2203 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L2203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L2204 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2205 EN**: Separator comment used for visual grouping.
  **L2205 CN**: 用于视觉分组的分隔注释。
- **L2206 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2207 EN**: Separator comment used for visual grouping.
  **L2207 CN**: 用于视觉分组的分隔注释。
- **L2208 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTHF82PH instruction.`.
  **L2208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTHF82PH instruction.`。

### Lines 2209-2232

````c
///
/// \param __A
///    A 256-bit vector of [32 x hf8].
/// \returns
///    A 256-bit vector of [16 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvthf8_ph(__m128i __A) {
  return (__m256h)__builtin_ia32_vcvthf8_2ph256_mask(
      (__v16qi)__A, (__v16hf)(__m256h)_mm256_undefined_ph(), (__mmask16)-1);
}

/// Convert 256-bit vector \a __A, containing packed FP8 E4M3 floating-point
///    elements to a 256-bit vector containing FP16 elements. The conversion is
///    exact. Merging mask \a __U is used to determine if given element should be
///    taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		dst.fp16[i] := convert_hf8_to_fp16(__A.hf8[i])
/// 	ELSE
/// 		dst.fp16[i] := __W.fp16[i]
/// 	FI
/// ENDFOR
````
- **L2209 EN**: Separator comment used for visual grouping.
  **L2209 CN**: 用于视觉分组的分隔注释。
- **L2210 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2211 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8].`.
  **L2211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8].`。
- **L2212 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2213 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`.
  **L2213 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`。
- **L2214 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L2214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L2215 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvthf8_ph(__m128i __A) {`.
  **L2215 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvthf8_ph(__m128i __A) {`。
- **L2216 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvthf8_2ph256_mask(`.
  **L2216 CN**: 以 `(__m256h)__builtin_ia32_vcvthf8_2ph256_mask(` 从当前函数返回。
- **L2217 EN**: Executes a call or declaration centered on `statement`.
  **L2217 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2218 EN**: Closes the current lexical scope or compound statement.
  **L2218 CN**: 结束当前词法作用域或复合语句块。
- **L2219 EN**: Blank line separating nearby declarations or logic blocks.
  **L2219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2220 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A, containing packed FP8 E4M3 floating-point`.
  **L2220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A, containing packed FP8 E4M3 floating-point`。
- **L2221 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 256-bit vector containing FP16 elements. The conversion is`.
  **L2221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 256-bit vector containing FP16 elements. The conversion is`。
- **L2222 EN**: Comment explains nearby logic, constraints, or intent: `exact. Merging mask a __U is used to determine if given element should be`.
  **L2222 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exact. Merging mask a __U is used to determine if given element should be`。
- **L2223 EN**: Comment explains nearby logic, constraints, or intent: `taken from a __W instead.`.
  **L2223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`taken from a __W instead.`。
- **L2224 EN**: Separator comment used for visual grouping.
  **L2224 CN**: 用于视觉分组的分隔注释。
- **L2225 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2226 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2227 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2227 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2228 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`.
  **L2228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`。
- **L2229 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2230 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : __W.fp16[i]`.
  **L2230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : __W.fp16[i]`。
- **L2231 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2232 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2232 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。

### Lines 2233-2256

````c
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTHF82PH instruction.
///
/// \param __W
///    A 256-bit vector of [16 x fp16].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [32 x hf8].
/// \returns
///    A 256-bit vector of [16 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_cvthf8_ph(__m256h __W, __mmask16 __U, __m128i __A) {
  return (__m256h)__builtin_ia32_vcvthf8_2ph256_mask(
      (__v16qi)__A, (__v16hf)(__m256h)__W, (__mmask16)__U);
}

````
- **L2233 EN**: Separator comment used for visual grouping.
  **L2233 CN**: 用于视觉分组的分隔注释。
- **L2234 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L2234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L2235 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2236 EN**: Separator comment used for visual grouping.
  **L2236 CN**: 用于视觉分组的分隔注释。
- **L2237 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2238 EN**: Separator comment used for visual grouping.
  **L2238 CN**: 用于视觉分组的分隔注释。
- **L2239 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTHF82PH instruction.`.
  **L2239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTHF82PH instruction.`。
- **L2240 EN**: Separator comment used for visual grouping.
  **L2240 CN**: 用于视觉分组的分隔注释。
- **L2241 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2242 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2243 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2244 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L2244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L2245 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2246 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8].`.
  **L2246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8].`。
- **L2247 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2247 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2248 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`.
  **L2248 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`。
- **L2249 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L2249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L2250 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L2250 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L2251 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L2251 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L2252 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvthf8_ph(__m256h __W, __mmask16 __U, __m128i __A) {`.
  **L2252 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvthf8_ph(__m256h __W, __mmask16 __U, __m128i __A) {`。
- **L2253 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvthf8_2ph256_mask(`.
  **L2253 CN**: 以 `(__m256h)__builtin_ia32_vcvthf8_2ph256_mask(` 从当前函数返回。
- **L2254 EN**: Executes a call or declaration centered on `statement`.
  **L2254 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2255 EN**: Closes the current lexical scope or compound statement.
  **L2255 CN**: 结束当前词法作用域或复合语句块。
- **L2256 EN**: Blank line separating nearby declarations or logic blocks.
  **L2256 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2257-2280

````c
/// Convert 256-bit vector \a __A, containing packed FP8 E4M3 floating-point
///    elements to a 256-bit vector containing FP16 elements. The conversion is
///    exact. Zeroing mask \a __U is used to determine if given element should be
///    zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		dst.fp16[i] := convert_hf8_to_fp16(__A.hf8[i])
/// 	ELSE
/// 		dst.fp16[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:256] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTHF82PH instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
````
- **L2257 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A, containing packed FP8 E4M3 floating-point`.
  **L2257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A, containing packed FP8 E4M3 floating-point`。
- **L2258 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 256-bit vector containing FP16 elements. The conversion is`.
  **L2258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 256-bit vector containing FP16 elements. The conversion is`。
- **L2259 EN**: Comment explains nearby logic, constraints, or intent: `exact. Zeroing mask a __U is used to determine if given element should be`.
  **L2259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exact. Zeroing mask a __U is used to determine if given element should be`。
- **L2260 EN**: Comment explains nearby logic, constraints, or intent: `zeroed instead.`.
  **L2260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zeroed instead.`。
- **L2261 EN**: Separator comment used for visual grouping.
  **L2261 CN**: 用于视觉分组的分隔注释。
- **L2262 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2263 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2264 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2265 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`.
  **L2265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_hf8_to_fp16(__A.hf8[i])`。
- **L2266 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2267 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : 0`.
  **L2267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : 0`。
- **L2268 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2268 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2269 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2269 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2270 EN**: Separator comment used for visual grouping.
  **L2270 CN**: 用于视觉分组的分隔注释。
- **L2271 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:256] : 0`.
  **L2271 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:256] : 0`。
- **L2272 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2272 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2273 EN**: Separator comment used for visual grouping.
  **L2273 CN**: 用于视觉分组的分隔注释。
- **L2274 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2274 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2275 EN**: Separator comment used for visual grouping.
  **L2275 CN**: 用于视觉分组的分隔注释。
- **L2276 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTHF82PH instruction.`.
  **L2276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTHF82PH instruction.`。
- **L2277 EN**: Separator comment used for visual grouping.
  **L2277 CN**: 用于视觉分组的分隔注释。
- **L2278 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2279 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L2279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L2280 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。

### Lines 2281-2304

````c
///    A 256-bit vector of [32 x hf8].
/// \returns
///    A 256-bit vector of [16 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    zero is taken instead.
static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_cvthf8_ph(__mmask16 __U, __m128i __A) {
  return (__m256h)__builtin_ia32_vcvthf8_2ph256_mask(
      (__v16qi)__A, (__v16hf)(__m256h)_mm256_setzero_ph(), (__mmask16)__U);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Upper elements of
///    resulting vector are zeroed.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i])
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
````
- **L2281 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x hf8].`.
  **L2281 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x hf8].`。
- **L2282 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2282 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2283 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`.
  **L2283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`。
- **L2284 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L2284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L2285 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L2285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L2286 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L2286 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L2287 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvthf8_ph(__mmask16 __U, __m128i __A) {`.
  **L2287 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvthf8_ph(__mmask16 __U, __m128i __A) {`。
- **L2288 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvthf8_2ph256_mask(`.
  **L2288 CN**: 以 `(__m256h)__builtin_ia32_vcvthf8_2ph256_mask(` 从当前函数返回。
- **L2289 EN**: Executes a call or declaration centered on `statement`.
  **L2289 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2290 EN**: Closes the current lexical scope or compound statement.
  **L2290 CN**: 结束当前词法作用域或复合语句块。
- **L2291 EN**: Blank line separating nearby declarations or logic blocks.
  **L2291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2292 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2293 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`.
  **L2293 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`。
- **L2294 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed.`.
  **L2294 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed.`。
- **L2295 EN**: Separator comment used for visual grouping.
  **L2295 CN**: 用于视觉分组的分隔注释。
- **L2296 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2296 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2297 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2298 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`.
  **L2298 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`。
- **L2299 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2299 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2300 EN**: Separator comment used for visual grouping.
  **L2300 CN**: 用于视觉分组的分隔注释。
- **L2301 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2302 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2302 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2303 EN**: Separator comment used for visual grouping.
  **L2303 CN**: 用于视觉分组的分隔注释。
- **L2304 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2304 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。

### Lines 2305-2328

````c
///
/// This intrinsic corresponds to the \c VCVTPH2BF8 instruction.
///
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the (converted)
///    elements from \a __A; upper elements are zeroed. 
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_bf8(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)_mm_undefined_si128(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Upper elements of
///    resulting vector are zeroed. Merging mask \a __U is used to determine if
///    given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
````
- **L2305 EN**: Separator comment used for visual grouping.
  **L2305 CN**: 用于视觉分组的分隔注释。
- **L2306 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8 instruction.`.
  **L2306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8 instruction.`。
- **L2307 EN**: Separator comment used for visual grouping.
  **L2307 CN**: 用于视觉分组的分隔注释。
- **L2308 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2309 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2310 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2311 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the (converted)`.
  **L2311 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the (converted)`。
- **L2312 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __A; upper elements are zeroed.`.
  **L2312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __A; upper elements are zeroed.`。
- **L2313 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_bf8(__m128h __A) {`.
  **L2313 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_bf8(__m128h __A) {`。
- **L2314 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8_128_mask(`.
  **L2314 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8_128_mask(` 从当前函数返回。
- **L2315 EN**: Executes a call or declaration centered on `statement`.
  **L2315 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2316 EN**: Closes the current lexical scope or compound statement.
  **L2316 CN**: 结束当前词法作用域或复合语句块。
- **L2317 EN**: Blank line separating nearby declarations or logic blocks.
  **L2317 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2318 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2318 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2319 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`.
  **L2319 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`。
- **L2320 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Merging mask a __U is used to determine if`.
  **L2320 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Merging mask a __U is used to determine if`。
- **L2321 EN**: Comment explains nearby logic, constraints, or intent: `given element should be taken from a __W instead.`.
  **L2321 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be taken from a __W instead.`。
- **L2322 EN**: Separator comment used for visual grouping.
  **L2322 CN**: 用于视觉分组的分隔注释。
- **L2323 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2323 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2324 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2325 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2326 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`.
  **L2326 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`。
- **L2327 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2328 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L2328 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。

### Lines 2329-2352

````c
/// 	FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    (converted) elements from \a __A; upper elements are zeroed. If
///    corresponding mask bit is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtph_bf8(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)__W, (__mmask8)__U);
````
- **L2329 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2329 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2330 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2330 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2331 EN**: Separator comment used for visual grouping.
  **L2331 CN**: 用于视觉分组的分隔注释。
- **L2332 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2333 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2334 EN**: Separator comment used for visual grouping.
  **L2334 CN**: 用于视觉分组的分隔注释。
- **L2335 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2335 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2336 EN**: Separator comment used for visual grouping.
  **L2336 CN**: 用于视觉分组的分隔注释。
- **L2337 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8 instruction.`.
  **L2337 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8 instruction.`。
- **L2338 EN**: Separator comment used for visual grouping.
  **L2338 CN**: 用于视觉分组的分隔注释。
- **L2339 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2339 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2340 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L2340 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L2341 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2342 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L2342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L2343 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2344 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2345 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2346 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L2346 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L2347 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A; upper elements are zeroed. If`.
  **L2347 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A; upper elements are zeroed. If`。
- **L2348 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element from a __W is taken instead.`.
  **L2348 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element from a __W is taken instead.`。
- **L2349 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2349 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2350 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtph_bf8(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L2350 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtph_bf8(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L2351 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8_128_mask(`.
  **L2351 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8_128_mask(` 从当前函数返回。
- **L2352 EN**: Executes a call or declaration centered on `statement`.
  **L2352 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 2353-2376

````c
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Upper elements of
///    resulting vector are zeroed. Zeroing mask \a __U is used to determine if
///    given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8 instruction.
///
/// \param __U
````
- **L2353 EN**: Closes the current lexical scope or compound statement.
  **L2353 CN**: 结束当前词法作用域或复合语句块。
- **L2354 EN**: Blank line separating nearby declarations or logic blocks.
  **L2354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2355 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2356 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`.
  **L2356 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`。
- **L2357 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Zeroing mask a __U is used to determine if`.
  **L2357 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Zeroing mask a __U is used to determine if`。
- **L2358 EN**: Comment explains nearby logic, constraints, or intent: `given element should be zeroed instead.`.
  **L2358 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be zeroed instead.`。
- **L2359 EN**: Separator comment used for visual grouping.
  **L2359 CN**: 用于视觉分组的分隔注释。
- **L2360 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2360 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2361 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2361 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2362 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2362 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2363 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`.
  **L2363 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`。
- **L2364 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2364 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2365 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L2365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L2366 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2366 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2367 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2368 EN**: Separator comment used for visual grouping.
  **L2368 CN**: 用于视觉分组的分隔注释。
- **L2369 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2369 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2370 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2370 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2371 EN**: Separator comment used for visual grouping.
  **L2371 CN**: 用于视觉分组的分隔注释。
- **L2372 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2372 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2373 EN**: Separator comment used for visual grouping.
  **L2373 CN**: 用于视觉分组的分隔注释。
- **L2374 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8 instruction.`.
  **L2374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8 instruction.`。
- **L2375 EN**: Separator comment used for visual grouping.
  **L2375 CN**: 用于视觉分组的分隔注释。
- **L2376 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。

### Lines 2377-2400

````c
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    (converted) elements from \a __A; upper elements are zeroed. If
///    corresponding mask bit is not set, then element is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtph_bf8(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)_mm_setzero_si128(), (__mmask8)__U);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
````
- **L2377 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L2377 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L2378 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2378 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2379 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2379 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2380 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2380 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2381 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L2381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L2382 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A; upper elements are zeroed. If`.
  **L2382 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A; upper elements are zeroed. If`。
- **L2383 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element is zeroed.`.
  **L2383 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element is zeroed.`。
- **L2384 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2384 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_bf8(__mmask8 __U, __m128h __A) {`.
  **L2385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_bf8(__mmask8 __U, __m128h __A) {`。
- **L2386 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8_128_mask(`.
  **L2386 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8_128_mask(` 从当前函数返回。
- **L2387 EN**: Executes a call or declaration centered on `statement`.
  **L2387 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2388 EN**: Closes the current lexical scope or compound statement.
  **L2388 CN**: 结束当前词法作用域或复合语句块。
- **L2389 EN**: Blank line separating nearby declarations or logic blocks.
  **L2389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2390 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2390 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2391 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements.`.
  **L2391 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements.`。
- **L2392 EN**: Separator comment used for visual grouping.
  **L2392 CN**: 用于视觉分组的分隔注释。
- **L2393 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2394 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2394 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2395 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`.
  **L2395 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`。
- **L2396 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2397 EN**: Separator comment used for visual grouping.
  **L2397 CN**: 用于视觉分组的分隔注释。
- **L2398 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2399 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2399 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2400 EN**: Separator comment used for visual grouping.
  **L2400 CN**: 用于视觉分组的分隔注释。

### Lines 2401-2424

````c
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8 instruction.
///
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Resulting elements correspond to the (converted)
///    elements from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvtph_bf8(__m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)_mm_undefined_si128(), (__mmask16)-1);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Merging mask \a __U is
///    used to determine if given element should be taken from \a __W instead.
///   
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i])
/// 	ELSE
````
- **L2401 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2401 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2402 EN**: Separator comment used for visual grouping.
  **L2402 CN**: 用于视觉分组的分隔注释。
- **L2403 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8 instruction.`.
  **L2403 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8 instruction.`。
- **L2404 EN**: Separator comment used for visual grouping.
  **L2404 CN**: 用于视觉分组的分隔注释。
- **L2405 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2405 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2406 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2406 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2407 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2407 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2408 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Resulting elements correspond to the (converted)`.
  **L2408 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Resulting elements correspond to the (converted)`。
- **L2409 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __A.`.
  **L2409 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __A.`。
- **L2410 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2410 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2411 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtph_bf8(__m256h __A) {`.
  **L2411 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtph_bf8(__m256h __A) {`。
- **L2412 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8_256_mask(`.
  **L2412 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8_256_mask(` 从当前函数返回。
- **L2413 EN**: Executes a call or declaration centered on `statement`.
  **L2413 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2414 EN**: Closes the current lexical scope or compound statement.
  **L2414 CN**: 结束当前词法作用域或复合语句块。
- **L2415 EN**: Blank line separating nearby declarations or logic blocks.
  **L2415 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2416 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2416 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2417 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Merging mask a __U is`.
  **L2417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Merging mask a __U is`。
- **L2418 EN**: Comment explains nearby logic, constraints, or intent: `used to determine if given element should be taken from a __W instead.`.
  **L2418 CN**: 注释解释附近代码的逻辑、约束或设计意图：`used to determine if given element should be taken from a __W instead.`。
- **L2419 EN**: Separator comment used for visual grouping.
  **L2419 CN**: 用于视觉分组的分隔注释。
- **L2420 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2420 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2421 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2421 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2422 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2422 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2423 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`.
  **L2423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`。
- **L2424 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。

### Lines 2425-2448

````c
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Resulting elements correspond to the
///    (converted) elements from \a __A. If
///    corresponding mask bit is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_bf8(__m128i __W, __mmask16 __U, __m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8_256_mask(
````
- **L2425 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L2425 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L2426 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2427 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2427 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2428 EN**: Separator comment used for visual grouping.
  **L2428 CN**: 用于视觉分组的分隔注释。
- **L2429 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2429 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2430 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2430 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2431 EN**: Separator comment used for visual grouping.
  **L2431 CN**: 用于视觉分组的分隔注释。
- **L2432 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2432 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2433 EN**: Separator comment used for visual grouping.
  **L2433 CN**: 用于视觉分组的分隔注释。
- **L2434 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8 instruction.`.
  **L2434 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8 instruction.`。
- **L2435 EN**: Separator comment used for visual grouping.
  **L2435 CN**: 用于视觉分组的分隔注释。
- **L2436 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2437 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L2437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L2438 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2439 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L2439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L2440 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2441 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x fp16].`.
  **L2441 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x fp16].`。
- **L2442 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2442 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2443 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Resulting elements correspond to the`.
  **L2443 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Resulting elements correspond to the`。
- **L2444 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If`.
  **L2444 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If`。
- **L2445 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element from a __W is taken instead.`.
  **L2445 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element from a __W is taken instead.`。
- **L2446 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2446 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2447 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_bf8(__m128i __W, __mmask16 __U, __m256h __A) {`.
  **L2447 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_bf8(__m128i __W, __mmask16 __U, __m256h __A) {`。
- **L2448 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8_256_mask(`.
  **L2448 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8_256_mask(` 从当前函数返回。

### Lines 2449-2472

````c
      (__v16hf)__A, (__v16qi)(__m128i)__W, (__mmask16)__U);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Zeroing mask \a __U is
///    used to determine if given element should be zeroed instead.
///   
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8(__A.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8 instruction.
///
/// \param __U
````
- **L2449 EN**: Executes a call or declaration centered on `statement`.
  **L2449 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2450 EN**: Closes the current lexical scope or compound statement.
  **L2450 CN**: 结束当前词法作用域或复合语句块。
- **L2451 EN**: Blank line separating nearby declarations or logic blocks.
  **L2451 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2452 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2452 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2453 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Zeroing mask a __U is`.
  **L2453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Zeroing mask a __U is`。
- **L2454 EN**: Comment explains nearby logic, constraints, or intent: `used to determine if given element should be zeroed instead.`.
  **L2454 CN**: 注释解释附近代码的逻辑、约束或设计意图：`used to determine if given element should be zeroed instead.`。
- **L2455 EN**: Separator comment used for visual grouping.
  **L2455 CN**: 用于视觉分组的分隔注释。
- **L2456 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2456 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2457 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2458 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2458 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2459 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`.
  **L2459 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8(__A.fp16[i])`。
- **L2460 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2461 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L2461 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L2462 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2463 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2464 EN**: Separator comment used for visual grouping.
  **L2464 CN**: 用于视觉分组的分隔注释。
- **L2465 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2465 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2466 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2466 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2467 EN**: Separator comment used for visual grouping.
  **L2467 CN**: 用于视觉分组的分隔注释。
- **L2468 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2468 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2469 EN**: Separator comment used for visual grouping.
  **L2469 CN**: 用于视觉分组的分隔注释。
- **L2470 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8 instruction.`.
  **L2470 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8 instruction.`。
- **L2471 EN**: Separator comment used for visual grouping.
  **L2471 CN**: 用于视觉分组的分隔注释。
- **L2472 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2472 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。

### Lines 2473-2496

````c
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    then element is zeroed instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtph_bf8(__mmask16 __U, __m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)_mm_setzero_si128(), (__mmask16)__U);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Upper elements of
///    resulting vector are zeroed. Results are saturated.
///   
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i])
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
````
- **L2473 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L2473 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L2474 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2474 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2475 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2476 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2476 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2477 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Resulting elements correspond to the`.
  **L2477 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Resulting elements correspond to the`。
- **L2478 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L2478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L2479 EN**: Comment explains nearby logic, constraints, or intent: `then element is zeroed instead.`.
  **L2479 CN**: 注释解释附近代码的逻辑、约束或设计意图：`then element is zeroed instead.`。
- **L2480 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2480 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2481 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_bf8(__mmask16 __U, __m256h __A) {`.
  **L2481 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_bf8(__mmask16 __U, __m256h __A) {`。
- **L2482 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8_256_mask(`.
  **L2482 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8_256_mask(` 从当前函数返回。
- **L2483 EN**: Executes a call or declaration centered on `statement`.
  **L2483 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2484 EN**: Closes the current lexical scope or compound statement.
  **L2484 CN**: 结束当前词法作用域或复合语句块。
- **L2485 EN**: Blank line separating nearby declarations or logic blocks.
  **L2485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2486 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2487 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`.
  **L2487 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`。
- **L2488 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Results are saturated.`.
  **L2488 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Results are saturated.`。
- **L2489 EN**: Separator comment used for visual grouping.
  **L2489 CN**: 用于视觉分组的分隔注释。
- **L2490 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2490 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2491 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2492 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`.
  **L2492 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`。
- **L2493 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2493 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2494 EN**: Separator comment used for visual grouping.
  **L2494 CN**: 用于视觉分组的分隔注释。
- **L2495 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2495 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2496 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2496 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。

### Lines 2497-2520

````c
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8S instruction.
///
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the (converted)
///    elements from \a __A; upper elements are zeroed. 
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvts_ph_bf8(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)_mm_undefined_si128(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Upper elements of
///    resulting vector are zeroed. Results are saturated. Merging mask \a __U is
///    used to determine if given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i])
````
- **L2497 EN**: Separator comment used for visual grouping.
  **L2497 CN**: 用于视觉分组的分隔注释。
- **L2498 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2498 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2499 EN**: Separator comment used for visual grouping.
  **L2499 CN**: 用于视觉分组的分隔注释。
- **L2500 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8S instruction.`.
  **L2500 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8S instruction.`。
- **L2501 EN**: Separator comment used for visual grouping.
  **L2501 CN**: 用于视觉分组的分隔注释。
- **L2502 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2502 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2503 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2503 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2504 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2505 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the (converted)`.
  **L2505 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the (converted)`。
- **L2506 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __A; upper elements are zeroed.`.
  **L2506 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __A; upper elements are zeroed.`。
- **L2507 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvts_ph_bf8(__m128h __A) {`.
  **L2507 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvts_ph_bf8(__m128h __A) {`。
- **L2508 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(`.
  **L2508 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(` 从当前函数返回。
- **L2509 EN**: Executes a call or declaration centered on `statement`.
  **L2509 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2510 EN**: Closes the current lexical scope or compound statement.
  **L2510 CN**: 结束当前词法作用域或复合语句块。
- **L2511 EN**: Blank line separating nearby declarations or logic blocks.
  **L2511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2512 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2512 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2513 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`.
  **L2513 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`。
- **L2514 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Results are saturated. Merging mask a __U is`.
  **L2514 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Results are saturated. Merging mask a __U is`。
- **L2515 EN**: Comment explains nearby logic, constraints, or intent: `used to determine if given element should be taken from a __W instead.`.
  **L2515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`used to determine if given element should be taken from a __W instead.`。
- **L2516 EN**: Separator comment used for visual grouping.
  **L2516 CN**: 用于视觉分组的分隔注释。
- **L2517 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2518 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2518 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2519 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2520 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`.
  **L2520 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`。

### Lines 2521-2544

````c
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    (converted) elements from \a __A; upper elements are zeroed. If
///    corresponding mask bit is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvts_ph_bf8(__m128i __W, __mmask8 __U, __m128h __A) {
````
- **L2521 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2521 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2522 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L2522 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L2523 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2523 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2524 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2524 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2525 EN**: Separator comment used for visual grouping.
  **L2525 CN**: 用于视觉分组的分隔注释。
- **L2526 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2526 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2527 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2527 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2528 EN**: Separator comment used for visual grouping.
  **L2528 CN**: 用于视觉分组的分隔注释。
- **L2529 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2529 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2530 EN**: Separator comment used for visual grouping.
  **L2530 CN**: 用于视觉分组的分隔注释。
- **L2531 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8S instruction.`.
  **L2531 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8S instruction.`。
- **L2532 EN**: Separator comment used for visual grouping.
  **L2532 CN**: 用于视觉分组的分隔注释。
- **L2533 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2533 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2534 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L2534 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L2535 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2535 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2536 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L2536 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L2537 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2537 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2538 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2538 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2539 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2539 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2540 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L2540 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L2541 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A; upper elements are zeroed. If`.
  **L2541 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A; upper elements are zeroed. If`。
- **L2542 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element from a __W is taken instead.`.
  **L2542 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element from a __W is taken instead.`。
- **L2543 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2543 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2544 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvts_ph_bf8(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L2544 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvts_ph_bf8(__m128i __W, __mmask8 __U, __m128h __A) {`。

### Lines 2545-2568

````c
  return (__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)__W, (__mmask8)__U);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Upper elements of
///    resulting vector are zeroed. Results are saturated. Zeroing mask \a __U is
///    used to determine if given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8S instruction.
````
- **L2545 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(`.
  **L2545 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(` 从当前函数返回。
- **L2546 EN**: Executes a call or declaration centered on `statement`.
  **L2546 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2547 EN**: Closes the current lexical scope or compound statement.
  **L2547 CN**: 结束当前词法作用域或复合语句块。
- **L2548 EN**: Blank line separating nearby declarations or logic blocks.
  **L2548 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2549 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2549 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2550 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`.
  **L2550 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`。
- **L2551 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Results are saturated. Zeroing mask a __U is`.
  **L2551 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Results are saturated. Zeroing mask a __U is`。
- **L2552 EN**: Comment explains nearby logic, constraints, or intent: `used to determine if given element should be zeroed instead.`.
  **L2552 CN**: 注释解释附近代码的逻辑、约束或设计意图：`used to determine if given element should be zeroed instead.`。
- **L2553 EN**: Separator comment used for visual grouping.
  **L2553 CN**: 用于视觉分组的分隔注释。
- **L2554 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2554 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2555 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2555 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2556 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2556 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2557 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`.
  **L2557 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`。
- **L2558 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2558 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2559 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L2559 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L2560 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2560 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2561 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2561 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2562 EN**: Separator comment used for visual grouping.
  **L2562 CN**: 用于视觉分组的分隔注释。
- **L2563 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2563 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2564 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2564 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2565 EN**: Separator comment used for visual grouping.
  **L2565 CN**: 用于视觉分组的分隔注释。
- **L2566 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2566 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2567 EN**: Separator comment used for visual grouping.
  **L2567 CN**: 用于视觉分组的分隔注释。
- **L2568 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8S instruction.`.
  **L2568 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8S instruction.`。

### Lines 2569-2592

````c
///
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Lower elements correspond to the
///    (converted) elements from \a __A; upper elements are zeroed. If
///    corresponding mask bit is not set, then element is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvts_ph_bf8(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)_mm_setzero_si128(), (__mmask8)__U);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
````
- **L2569 EN**: Separator comment used for visual grouping.
  **L2569 CN**: 用于视觉分组的分隔注释。
- **L2570 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2570 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2571 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L2571 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L2572 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2572 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2573 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2573 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2574 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2574 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2575 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Lower elements correspond to the`.
  **L2575 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Lower elements correspond to the`。
- **L2576 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A; upper elements are zeroed. If`.
  **L2576 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A; upper elements are zeroed. If`。
- **L2577 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element is zeroed.`.
  **L2577 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element is zeroed.`。
- **L2578 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2578 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2579 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvts_ph_bf8(__mmask8 __U, __m128h __A) {`.
  **L2579 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvts_ph_bf8(__mmask8 __U, __m128h __A) {`。
- **L2580 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(`.
  **L2580 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8s_128_mask(` 从当前函数返回。
- **L2581 EN**: Executes a call or declaration centered on `statement`.
  **L2581 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2582 EN**: Closes the current lexical scope or compound statement.
  **L2582 CN**: 结束当前词法作用域或复合语句块。
- **L2583 EN**: Blank line separating nearby declarations or logic blocks.
  **L2583 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2584 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2584 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2585 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.`.
  **L2585 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.`。
- **L2586 EN**: Separator comment used for visual grouping.
  **L2586 CN**: 用于视觉分组的分隔注释。
- **L2587 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2587 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2588 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2588 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2589 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`.
  **L2589 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`。
- **L2590 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2590 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2591 EN**: Separator comment used for visual grouping.
  **L2591 CN**: 用于视觉分组的分隔注释。
- **L2592 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2592 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。

### Lines 2593-2616

````c
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8S instruction.
///
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Resulting elements correspond to the (converted)
///    elements from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvts_ph_bf8(__m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)_mm_undefined_si128(), (__mmask16)-1);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
````
- **L2593 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2593 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2594 EN**: Separator comment used for visual grouping.
  **L2594 CN**: 用于视觉分组的分隔注释。
- **L2595 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2595 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2596 EN**: Separator comment used for visual grouping.
  **L2596 CN**: 用于视觉分组的分隔注释。
- **L2597 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8S instruction.`.
  **L2597 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8S instruction.`。
- **L2598 EN**: Separator comment used for visual grouping.
  **L2598 CN**: 用于视觉分组的分隔注释。
- **L2599 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2599 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2600 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2600 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2601 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2602 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Resulting elements correspond to the (converted)`.
  **L2602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Resulting elements correspond to the (converted)`。
- **L2603 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __A.`.
  **L2603 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __A.`。
- **L2604 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2604 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2605 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvts_ph_bf8(__m256h __A) {`.
  **L2605 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvts_ph_bf8(__m256h __A) {`。
- **L2606 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(`.
  **L2606 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(` 从当前函数返回。
- **L2607 EN**: Executes a call or declaration centered on `statement`.
  **L2607 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2608 EN**: Closes the current lexical scope or compound statement.
  **L2608 CN**: 结束当前词法作用域或复合语句块。
- **L2609 EN**: Blank line separating nearby declarations or logic blocks.
  **L2609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2610 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2610 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2611 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.`.
  **L2611 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.`。
- **L2612 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L2612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L2613 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead.`.
  **L2613 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead.`。
- **L2614 EN**: Separator comment used for visual grouping.
  **L2614 CN**: 用于视觉分组的分隔注释。
- **L2615 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2615 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2616 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2616 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。

### Lines 2617-2640

````c
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := __W.bf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2BF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x bf8].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Resulting elements correspond to the
///    (converted) elements from \a __A. If
///    corresponding mask bit is not set, then element from \a __W is taken instead.
````
- **L2617 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2617 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2618 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`.
  **L2618 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`。
- **L2619 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2619 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2620 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : __W.bf8[i]`.
  **L2620 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : __W.bf8[i]`。
- **L2621 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2621 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2622 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2623 EN**: Separator comment used for visual grouping.
  **L2623 CN**: 用于视觉分组的分隔注释。
- **L2624 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2624 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2625 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2625 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2626 EN**: Separator comment used for visual grouping.
  **L2626 CN**: 用于视觉分组的分隔注释。
- **L2627 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2627 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2628 EN**: Separator comment used for visual grouping.
  **L2628 CN**: 用于视觉分组的分隔注释。
- **L2629 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8S instruction.`.
  **L2629 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8S instruction.`。
- **L2630 EN**: Separator comment used for visual grouping.
  **L2630 CN**: 用于视觉分组的分隔注释。
- **L2631 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2632 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L2632 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L2633 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2633 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2634 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L2634 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L2635 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2635 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2636 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x fp16].`.
  **L2636 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x fp16].`。
- **L2637 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2637 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2638 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Resulting elements correspond to the`.
  **L2638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Resulting elements correspond to the`。
- **L2639 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If`.
  **L2639 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If`。
- **L2640 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element from a __W is taken instead.`.
  **L2640 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element from a __W is taken instead.`。

### Lines 2641-2664

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_mask_cvts_ph_bf8(__m128i __W, __mmask16 __U, __m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)__W, (__mmask16)__U);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		dst.bf8[i] := convert_fp16_to_bf8_saturate(__A.fp16[i])
/// 	ELSE
/// 		dst.bf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
````
- **L2641 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2641 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2642 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvts_ph_bf8(__m128i __W, __mmask16 __U, __m256h __A) {`.
  **L2642 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvts_ph_bf8(__m128i __W, __mmask16 __U, __m256h __A) {`。
- **L2643 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(`.
  **L2643 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(` 从当前函数返回。
- **L2644 EN**: Executes a call or declaration centered on `statement`.
  **L2644 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2645 EN**: Closes the current lexical scope or compound statement.
  **L2645 CN**: 结束当前词法作用域或复合语句块。
- **L2646 EN**: Blank line separating nearby declarations or logic blocks.
  **L2646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2647 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2647 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2648 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.`.
  **L2648 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Results are saturated.`。
- **L2649 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L2649 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L2650 EN**: Comment explains nearby logic, constraints, or intent: `instead.`.
  **L2650 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead.`。
- **L2651 EN**: Separator comment used for visual grouping.
  **L2651 CN**: 用于视觉分组的分隔注释。
- **L2652 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2653 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2653 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2654 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2654 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2655 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`.
  **L2655 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : convert_fp16_to_bf8_saturate(__A.fp16[i])`。
- **L2656 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2656 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2657 EN**: Comment explains nearby logic, constraints, or intent: `dst.bf8[i] : 0`.
  **L2657 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.bf8[i] : 0`。
- **L2658 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2658 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2659 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2659 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2660 EN**: Separator comment used for visual grouping.
  **L2660 CN**: 用于视觉分组的分隔注释。
- **L2661 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2661 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2662 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2662 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2663 EN**: Separator comment used for visual grouping.
  **L2663 CN**: 用于视觉分组的分隔注释。
- **L2664 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。

### Lines 2665-2688

````c
///
/// This intrinsic corresponds to the \c VCVTPH2BF8S instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x bf8]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    then element is zeroed instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvts_ph_bf8(__mmask16 __U, __m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)_mm_setzero_si128(), (__mmask16)__U);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E5M2 FP8 elements. Upper elements of
///    resulting vector are zeroed.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i])
````
- **L2665 EN**: Separator comment used for visual grouping.
  **L2665 CN**: 用于视觉分组的分隔注释。
- **L2666 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2BF8S instruction.`.
  **L2666 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2BF8S instruction.`。
- **L2667 EN**: Separator comment used for visual grouping.
  **L2667 CN**: 用于视觉分组的分隔注释。
- **L2668 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2668 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2669 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L2669 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L2670 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2670 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2671 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2671 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2672 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2672 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2673 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8]. Resulting elements correspond to the`.
  **L2673 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8]. Resulting elements correspond to the`。
- **L2674 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L2674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L2675 EN**: Comment explains nearby logic, constraints, or intent: `then element is zeroed instead.`.
  **L2675 CN**: 注释解释附近代码的逻辑、约束或设计意图：`then element is zeroed instead.`。
- **L2676 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2676 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2677 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvts_ph_bf8(__mmask16 __U, __m256h __A) {`.
  **L2677 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvts_ph_bf8(__mmask16 __U, __m256h __A) {`。
- **L2678 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(`.
  **L2678 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2bf8s_256_mask(` 从当前函数返回。
- **L2679 EN**: Executes a call or declaration centered on `statement`.
  **L2679 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2680 EN**: Closes the current lexical scope or compound statement.
  **L2680 CN**: 结束当前词法作用域或复合语句块。
- **L2681 EN**: Blank line separating nearby declarations or logic blocks.
  **L2681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2682 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2682 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2683 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`.
  **L2683 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E5M2 FP8 elements. Upper elements of`。
- **L2684 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed.`.
  **L2684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed.`。
- **L2685 EN**: Separator comment used for visual grouping.
  **L2685 CN**: 用于视觉分组的分隔注释。
- **L2686 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2686 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2687 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2687 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2688 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`.
  **L2688 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`。

### Lines 2689-2712

````c
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8 instruction.
///
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the (converted)
///    elements from \a __A; upper elements are zeroed. 
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_hf8(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)_mm_undefined_si128(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Upper elements of
///    resulting vector are zeroed. Merging mask \a __U is used to determine if
///    given element should be taken from \a __W instead.
///
````
- **L2689 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2689 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2690 EN**: Separator comment used for visual grouping.
  **L2690 CN**: 用于视觉分组的分隔注释。
- **L2691 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2691 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2692 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2692 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2693 EN**: Separator comment used for visual grouping.
  **L2693 CN**: 用于视觉分组的分隔注释。
- **L2694 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2694 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2695 EN**: Separator comment used for visual grouping.
  **L2695 CN**: 用于视觉分组的分隔注释。
- **L2696 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8 instruction.`.
  **L2696 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8 instruction.`。
- **L2697 EN**: Separator comment used for visual grouping.
  **L2697 CN**: 用于视觉分组的分隔注释。
- **L2698 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2698 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2699 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2699 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2700 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2700 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2701 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the (converted)`.
  **L2701 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the (converted)`。
- **L2702 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __A; upper elements are zeroed.`.
  **L2702 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __A; upper elements are zeroed.`。
- **L2703 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_hf8(__m128h __A) {`.
  **L2703 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtph_hf8(__m128h __A) {`。
- **L2704 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8_128_mask(`.
  **L2704 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8_128_mask(` 从当前函数返回。
- **L2705 EN**: Executes a call or declaration centered on `statement`.
  **L2705 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2706 EN**: Closes the current lexical scope or compound statement.
  **L2706 CN**: 结束当前词法作用域或复合语句块。
- **L2707 EN**: Blank line separating nearby declarations or logic blocks.
  **L2707 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2708 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2708 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2709 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`.
  **L2709 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`。
- **L2710 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Merging mask a __U is used to determine if`.
  **L2710 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Merging mask a __U is used to determine if`。
- **L2711 EN**: Comment explains nearby logic, constraints, or intent: `given element should be taken from a __W instead.`.
  **L2711 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be taken from a __W instead.`。
- **L2712 EN**: Separator comment used for visual grouping.
  **L2712 CN**: 用于视觉分组的分隔注释。

### Lines 2713-2736

````c
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
````
- **L2713 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2713 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2714 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2715 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2715 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2716 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`.
  **L2716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`。
- **L2717 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2717 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2718 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L2718 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L2719 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2719 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2720 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2720 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2721 EN**: Separator comment used for visual grouping.
  **L2721 CN**: 用于视觉分组的分隔注释。
- **L2722 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2722 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2723 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2723 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2724 EN**: Separator comment used for visual grouping.
  **L2724 CN**: 用于视觉分组的分隔注释。
- **L2725 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2725 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2726 EN**: Separator comment used for visual grouping.
  **L2726 CN**: 用于视觉分组的分隔注释。
- **L2727 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8 instruction.`.
  **L2727 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8 instruction.`。
- **L2728 EN**: Separator comment used for visual grouping.
  **L2728 CN**: 用于视觉分组的分隔注释。
- **L2729 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2729 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2730 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L2730 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L2731 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2731 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2732 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L2732 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L2733 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2733 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2734 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2734 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2735 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2735 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2736 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L2736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。

### Lines 2737-2760

````c
///    (converted) elements from \a __A; upper elements are zeroed. If
///    corresponding mask bit is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtph_hf8(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)__W, (__mmask8)__U);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Upper elements of
///    resulting vector are zeroed. Zeroing mask \a __U is used to determine if
///    given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
````
- **L2737 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A; upper elements are zeroed. If`.
  **L2737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A; upper elements are zeroed. If`。
- **L2738 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element from a __W is taken instead.`.
  **L2738 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element from a __W is taken instead.`。
- **L2739 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2739 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2740 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtph_hf8(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L2740 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtph_hf8(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L2741 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8_128_mask(`.
  **L2741 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8_128_mask(` 从当前函数返回。
- **L2742 EN**: Executes a call or declaration centered on `statement`.
  **L2742 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2743 EN**: Closes the current lexical scope or compound statement.
  **L2743 CN**: 结束当前词法作用域或复合语句块。
- **L2744 EN**: Blank line separating nearby declarations or logic blocks.
  **L2744 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2745 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2745 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2746 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`.
  **L2746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`。
- **L2747 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Zeroing mask a __U is used to determine if`.
  **L2747 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Zeroing mask a __U is used to determine if`。
- **L2748 EN**: Comment explains nearby logic, constraints, or intent: `given element should be zeroed instead.`.
  **L2748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element should be zeroed instead.`。
- **L2749 EN**: Separator comment used for visual grouping.
  **L2749 CN**: 用于视觉分组的分隔注释。
- **L2750 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2750 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2751 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2751 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2752 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2752 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2753 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`.
  **L2753 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`。
- **L2754 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2754 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2755 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L2755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L2756 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2756 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2757 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2758 EN**: Separator comment used for visual grouping.
  **L2758 CN**: 用于视觉分组的分隔注释。
- **L2759 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2759 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2760 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2760 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。

### Lines 2761-2784

````c
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8 instruction.
///
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    (converted) elements from \a __A; upper elements are zeroed. If
///    corresponding mask bit is not set, then element is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtph_hf8(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)_mm_setzero_si128(), (__mmask8)__U);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements.
///
/// \code{.operation}
/// FOR i := 0 to 15
````
- **L2761 EN**: Separator comment used for visual grouping.
  **L2761 CN**: 用于视觉分组的分隔注释。
- **L2762 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2762 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2763 EN**: Separator comment used for visual grouping.
  **L2763 CN**: 用于视觉分组的分隔注释。
- **L2764 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8 instruction.`.
  **L2764 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8 instruction.`。
- **L2765 EN**: Separator comment used for visual grouping.
  **L2765 CN**: 用于视觉分组的分隔注释。
- **L2766 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2766 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2767 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L2767 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L2768 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2768 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2769 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2769 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2770 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2770 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2771 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L2771 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L2772 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A; upper elements are zeroed. If`.
  **L2772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A; upper elements are zeroed. If`。
- **L2773 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element is zeroed.`.
  **L2773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element is zeroed.`。
- **L2774 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2774 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2775 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtph_hf8(__mmask8 __U, __m128h __A) {`.
  **L2775 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtph_hf8(__mmask8 __U, __m128h __A) {`。
- **L2776 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8_128_mask(`.
  **L2776 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8_128_mask(` 从当前函数返回。
- **L2777 EN**: Executes a call or declaration centered on `statement`.
  **L2777 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2778 EN**: Closes the current lexical scope or compound statement.
  **L2778 CN**: 结束当前词法作用域或复合语句块。
- **L2779 EN**: Blank line separating nearby declarations or logic blocks.
  **L2779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2780 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2780 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2781 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements.`.
  **L2781 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements.`。
- **L2782 EN**: Separator comment used for visual grouping.
  **L2782 CN**: 用于视觉分组的分隔注释。
- **L2783 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2784 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。

### Lines 2785-2808

````c
/// 	dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8 instruction.
///
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Resulting elements correspond to the (converted)
///    elements from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvtph_hf8(__m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)_mm_undefined_si128(), (__mmask16)-1);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Merging mask \a __U is
///    used to determine if given element should be taken from \a __W instead.
````
- **L2785 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`.
  **L2785 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`。
- **L2786 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2787 EN**: Separator comment used for visual grouping.
  **L2787 CN**: 用于视觉分组的分隔注释。
- **L2788 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2788 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2789 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2789 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2790 EN**: Separator comment used for visual grouping.
  **L2790 CN**: 用于视觉分组的分隔注释。
- **L2791 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2791 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2792 EN**: Separator comment used for visual grouping.
  **L2792 CN**: 用于视觉分组的分隔注释。
- **L2793 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8 instruction.`.
  **L2793 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8 instruction.`。
- **L2794 EN**: Separator comment used for visual grouping.
  **L2794 CN**: 用于视觉分组的分隔注释。
- **L2795 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2795 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2796 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2796 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2797 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2797 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2798 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Resulting elements correspond to the (converted)`.
  **L2798 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Resulting elements correspond to the (converted)`。
- **L2799 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __A.`.
  **L2799 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __A.`。
- **L2800 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2800 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2801 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtph_hf8(__m256h __A) {`.
  **L2801 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtph_hf8(__m256h __A) {`。
- **L2802 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8_256_mask(`.
  **L2802 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8_256_mask(` 从当前函数返回。
- **L2803 EN**: Executes a call or declaration centered on `statement`.
  **L2803 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2804 EN**: Closes the current lexical scope or compound statement.
  **L2804 CN**: 结束当前词法作用域或复合语句块。
- **L2805 EN**: Blank line separating nearby declarations or logic blocks.
  **L2805 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2806 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2806 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2807 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Merging mask a __U is`.
  **L2807 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Merging mask a __U is`。
- **L2808 EN**: Comment explains nearby logic, constraints, or intent: `used to determine if given element should be taken from a __W instead.`.
  **L2808 CN**: 注释解释附近代码的逻辑、约束或设计意图：`used to determine if given element should be taken from a __W instead.`。

### Lines 2809-2832

````c
///   
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8 instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [8 x fp16].
/// \returns
````
- **L2809 EN**: Separator comment used for visual grouping.
  **L2809 CN**: 用于视觉分组的分隔注释。
- **L2810 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2810 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2811 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2811 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2812 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2812 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2813 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`.
  **L2813 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`。
- **L2814 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2814 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2815 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L2815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L2816 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2816 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2817 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2817 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2818 EN**: Separator comment used for visual grouping.
  **L2818 CN**: 用于视觉分组的分隔注释。
- **L2819 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2819 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2820 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2821 EN**: Separator comment used for visual grouping.
  **L2821 CN**: 用于视觉分组的分隔注释。
- **L2822 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2822 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2823 EN**: Separator comment used for visual grouping.
  **L2823 CN**: 用于视觉分组的分隔注释。
- **L2824 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8 instruction.`.
  **L2824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8 instruction.`。
- **L2825 EN**: Separator comment used for visual grouping.
  **L2825 CN**: 用于视觉分组的分隔注释。
- **L2826 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2826 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2827 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L2827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L2828 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2828 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2829 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L2829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L2830 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2830 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2831 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x fp16].`.
  **L2831 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x fp16].`。
- **L2832 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2832 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。

### Lines 2833-2856

````c
///    A 128-bit vector of [16 x hf8]. Resulting elements correspond to the
///    (converted) elements from \a __A. If
///    corresponding mask bit is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtph_hf8(__m128i __W, __mmask16 __U, __m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)__W, (__mmask16)__U);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Zeroing mask \a __U is
///    used to determine if given element should be zeroed instead.
///   
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8(__A.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := 0
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
````
- **L2833 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Resulting elements correspond to the`.
  **L2833 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Resulting elements correspond to the`。
- **L2834 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If`.
  **L2834 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If`。
- **L2835 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element from a __W is taken instead.`.
  **L2835 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element from a __W is taken instead.`。
- **L2836 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2836 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2837 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtph_hf8(__m128i __W, __mmask16 __U, __m256h __A) {`.
  **L2837 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtph_hf8(__m128i __W, __mmask16 __U, __m256h __A) {`。
- **L2838 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8_256_mask(`.
  **L2838 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8_256_mask(` 从当前函数返回。
- **L2839 EN**: Executes a call or declaration centered on `statement`.
  **L2839 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2840 EN**: Closes the current lexical scope or compound statement.
  **L2840 CN**: 结束当前词法作用域或复合语句块。
- **L2841 EN**: Blank line separating nearby declarations or logic blocks.
  **L2841 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2842 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2842 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2843 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Zeroing mask a __U is`.
  **L2843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Zeroing mask a __U is`。
- **L2844 EN**: Comment explains nearby logic, constraints, or intent: `used to determine if given element should be zeroed instead.`.
  **L2844 CN**: 注释解释附近代码的逻辑、约束或设计意图：`used to determine if given element should be zeroed instead.`。
- **L2845 EN**: Separator comment used for visual grouping.
  **L2845 CN**: 用于视觉分组的分隔注释。
- **L2846 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2846 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2847 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2847 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2848 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2848 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2849 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`.
  **L2849 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8(__A.fp16[i])`。
- **L2850 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2850 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2851 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L2851 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L2852 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2852 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2853 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2853 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2854 EN**: Separator comment used for visual grouping.
  **L2854 CN**: 用于视觉分组的分隔注释。
- **L2855 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2855 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2856 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。

### Lines 2857-2880

````c
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8 instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    then element is zeroed instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtph_hf8(__mmask16 __U, __m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)_mm_setzero_si128(), (__mmask16)__U);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Upper elements of
///    resulting vector are zeroed. Results are saturated.
///   
/// \code{.operation}
````
- **L2857 EN**: Separator comment used for visual grouping.
  **L2857 CN**: 用于视觉分组的分隔注释。
- **L2858 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2858 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2859 EN**: Separator comment used for visual grouping.
  **L2859 CN**: 用于视觉分组的分隔注释。
- **L2860 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8 instruction.`.
  **L2860 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8 instruction.`。
- **L2861 EN**: Separator comment used for visual grouping.
  **L2861 CN**: 用于视觉分组的分隔注释。
- **L2862 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2862 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2863 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L2863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L2864 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2864 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2865 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2865 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2866 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2866 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2867 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Resulting elements correspond to the`.
  **L2867 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Resulting elements correspond to the`。
- **L2868 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L2868 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L2869 EN**: Comment explains nearby logic, constraints, or intent: `then element is zeroed instead.`.
  **L2869 CN**: 注释解释附近代码的逻辑、约束或设计意图：`then element is zeroed instead.`。
- **L2870 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2870 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2871 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtph_hf8(__mmask16 __U, __m256h __A) {`.
  **L2871 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtph_hf8(__mmask16 __U, __m256h __A) {`。
- **L2872 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8_256_mask(`.
  **L2872 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8_256_mask(` 从当前函数返回。
- **L2873 EN**: Executes a call or declaration centered on `statement`.
  **L2873 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2874 EN**: Closes the current lexical scope or compound statement.
  **L2874 CN**: 结束当前词法作用域或复合语句块。
- **L2875 EN**: Blank line separating nearby declarations or logic blocks.
  **L2875 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2876 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2876 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2877 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`.
  **L2877 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`。
- **L2878 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Results are saturated.`.
  **L2878 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Results are saturated.`。
- **L2879 EN**: Separator comment used for visual grouping.
  **L2879 CN**: 用于视觉分组的分隔注释。
- **L2880 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2880 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。

### Lines 2881-2904

````c
/// FOR i := 0 to 7
/// 	dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i])
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8S instruction.
///
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the (converted)
///    elements from \a __A; upper elements are zeroed. 
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvts_ph_hf8(__m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)_mm_undefined_si128(), (__mmask8)-1);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Upper elements of
///    resulting vector are zeroed. Results are saturated. Merging mask \a __U is
````
- **L2881 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2881 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2882 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`.
  **L2882 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`。
- **L2883 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2883 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2884 EN**: Separator comment used for visual grouping.
  **L2884 CN**: 用于视觉分组的分隔注释。
- **L2885 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2885 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2886 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2886 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2887 EN**: Separator comment used for visual grouping.
  **L2887 CN**: 用于视觉分组的分隔注释。
- **L2888 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2888 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2889 EN**: Separator comment used for visual grouping.
  **L2889 CN**: 用于视觉分组的分隔注释。
- **L2890 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8S instruction.`.
  **L2890 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8S instruction.`。
- **L2891 EN**: Separator comment used for visual grouping.
  **L2891 CN**: 用于视觉分组的分隔注释。
- **L2892 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2892 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2893 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2893 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2894 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2894 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2895 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the (converted)`.
  **L2895 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the (converted)`。
- **L2896 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __A; upper elements are zeroed.`.
  **L2896 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __A; upper elements are zeroed.`。
- **L2897 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvts_ph_hf8(__m128h __A) {`.
  **L2897 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvts_ph_hf8(__m128h __A) {`。
- **L2898 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(`.
  **L2898 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(` 从当前函数返回。
- **L2899 EN**: Executes a call or declaration centered on `statement`.
  **L2899 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2900 EN**: Closes the current lexical scope or compound statement.
  **L2900 CN**: 结束当前词法作用域或复合语句块。
- **L2901 EN**: Blank line separating nearby declarations or logic blocks.
  **L2901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2902 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2902 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2903 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`.
  **L2903 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`。
- **L2904 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Results are saturated. Merging mask a __U is`.
  **L2904 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Results are saturated. Merging mask a __U is`。

### Lines 2905-2928

````c
///    used to determine if given element should be taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
````
- **L2905 EN**: Comment explains nearby logic, constraints, or intent: `used to determine if given element should be taken from a __W instead.`.
  **L2905 CN**: 注释解释附近代码的逻辑、约束或设计意图：`used to determine if given element should be taken from a __W instead.`。
- **L2906 EN**: Separator comment used for visual grouping.
  **L2906 CN**: 用于视觉分组的分隔注释。
- **L2907 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2907 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2908 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2908 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2909 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2909 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2910 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`.
  **L2910 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`。
- **L2911 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2911 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2912 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L2912 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L2913 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2913 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2914 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2914 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2915 EN**: Separator comment used for visual grouping.
  **L2915 CN**: 用于视觉分组的分隔注释。
- **L2916 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2916 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2917 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2917 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2918 EN**: Separator comment used for visual grouping.
  **L2918 CN**: 用于视觉分组的分隔注释。
- **L2919 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2919 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2920 EN**: Separator comment used for visual grouping.
  **L2920 CN**: 用于视觉分组的分隔注释。
- **L2921 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8S instruction.`.
  **L2921 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8S instruction.`。
- **L2922 EN**: Separator comment used for visual grouping.
  **L2922 CN**: 用于视觉分组的分隔注释。
- **L2923 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L2923 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L2924 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L2924 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L2925 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2925 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2926 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L2926 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L2927 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2927 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2928 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2928 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。

### Lines 2929-2952

````c
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    (converted) elements from \a __A; upper elements are zeroed. If
///    corresponding mask bit is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvts_ph_hf8(__m128i __W, __mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)__W, (__mmask8)__U);
}

/// Convert 128-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Upper elements of
///    resulting vector are zeroed. Results are saturated. Zeroing mask \a __U is
///    used to determine if given element should be zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := 0
/// 	FI
/// ENDFOR
///
````
- **L2929 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2929 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2930 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L2930 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L2931 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A; upper elements are zeroed. If`.
  **L2931 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A; upper elements are zeroed. If`。
- **L2932 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element from a __W is taken instead.`.
  **L2932 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element from a __W is taken instead.`。
- **L2933 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2933 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2934 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvts_ph_hf8(__m128i __W, __mmask8 __U, __m128h __A) {`.
  **L2934 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvts_ph_hf8(__m128i __W, __mmask8 __U, __m128h __A) {`。
- **L2935 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(`.
  **L2935 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(` 从当前函数返回。
- **L2936 EN**: Executes a call or declaration centered on `statement`.
  **L2936 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2937 EN**: Closes the current lexical scope or compound statement.
  **L2937 CN**: 结束当前词法作用域或复合语句块。
- **L2938 EN**: Blank line separating nearby declarations or logic blocks.
  **L2938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2939 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A containing packed FP16 floating-point elements`.
  **L2939 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A containing packed FP16 floating-point elements`。
- **L2940 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`.
  **L2940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Upper elements of`。
- **L2941 EN**: Comment explains nearby logic, constraints, or intent: `resulting vector are zeroed. Results are saturated. Zeroing mask a __U is`.
  **L2941 CN**: 注释解释附近代码的逻辑、约束或设计意图：`resulting vector are zeroed. Results are saturated. Zeroing mask a __U is`。
- **L2942 EN**: Comment explains nearby logic, constraints, or intent: `used to determine if given element should be zeroed instead.`.
  **L2942 CN**: 注释解释附近代码的逻辑、约束或设计意图：`used to determine if given element should be zeroed instead.`。
- **L2943 EN**: Separator comment used for visual grouping.
  **L2943 CN**: 用于视觉分组的分隔注释。
- **L2944 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2944 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2945 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L2945 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L2946 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L2946 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L2947 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`.
  **L2947 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`。
- **L2948 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L2948 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L2949 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L2949 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L2950 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L2950 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L2951 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2951 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2952 EN**: Separator comment used for visual grouping.
  **L2952 CN**: 用于视觉分组的分隔注释。

### Lines 2953-2976

````c
/// dst[MAX:64] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8S instruction.
///
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Lower elements correspond to the
///    (converted) elements from \a __A; upper elements are zeroed. If
///    corresponding mask bit is not set, then element is zeroed.
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvts_ph_hf8(__mmask8 __U, __m128h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(
      (__v8hf)__A, (__v16qi)(__m128i)_mm_setzero_si128(), (__mmask8)__U);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.
///
````
- **L2953 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:64] : 0`.
  **L2953 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:64] : 0`。
- **L2954 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2954 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2955 EN**: Separator comment used for visual grouping.
  **L2955 CN**: 用于视觉分组的分隔注释。
- **L2956 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2956 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2957 EN**: Separator comment used for visual grouping.
  **L2957 CN**: 用于视觉分组的分隔注释。
- **L2958 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8S instruction.`.
  **L2958 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8S instruction.`。
- **L2959 EN**: Separator comment used for visual grouping.
  **L2959 CN**: 用于视觉分组的分隔注释。
- **L2960 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L2960 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L2961 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L2961 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L2962 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2962 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2963 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L2963 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L2964 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2964 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2965 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Lower elements correspond to the`.
  **L2965 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Lower elements correspond to the`。
- **L2966 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A; upper elements are zeroed. If`.
  **L2966 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A; upper elements are zeroed. If`。
- **L2967 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element is zeroed.`.
  **L2967 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element is zeroed.`。
- **L2968 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2968 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2969 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvts_ph_hf8(__mmask8 __U, __m128h __A) {`.
  **L2969 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvts_ph_hf8(__mmask8 __U, __m128h __A) {`。
- **L2970 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(`.
  **L2970 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8s_128_mask(` 从当前函数返回。
- **L2971 EN**: Executes a call or declaration centered on `statement`.
  **L2971 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2972 EN**: Closes the current lexical scope or compound statement.
  **L2972 CN**: 结束当前词法作用域或复合语句块。
- **L2973 EN**: Blank line separating nearby declarations or logic blocks.
  **L2973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2974 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L2974 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L2975 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.`.
  **L2975 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.`。
- **L2976 EN**: Separator comment used for visual grouping.
  **L2976 CN**: 用于视觉分组的分隔注释。

### Lines 2977-3000

````c
/// \code{.operation}
/// FOR i := 0 to 15
/// 	dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i])
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8S instruction.
///
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Resulting elements correspond to the (converted)
///    elements from \a __A.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvts_ph_hf8(__m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)_mm_undefined_si128(), (__mmask16)-1);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
````
- **L2977 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L2977 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L2978 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L2978 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L2979 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`.
  **L2979 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`。
- **L2980 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L2980 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L2981 EN**: Separator comment used for visual grouping.
  **L2981 CN**: 用于视觉分组的分隔注释。
- **L2982 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L2982 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L2983 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L2983 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L2984 EN**: Separator comment used for visual grouping.
  **L2984 CN**: 用于视觉分组的分隔注释。
- **L2985 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L2985 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L2986 EN**: Separator comment used for visual grouping.
  **L2986 CN**: 用于视觉分组的分隔注释。
- **L2987 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8S instruction.`.
  **L2987 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8S instruction.`。
- **L2988 EN**: Separator comment used for visual grouping.
  **L2988 CN**: 用于视觉分组的分隔注释。
- **L2989 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L2989 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L2990 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L2990 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L2991 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L2991 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L2992 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Resulting elements correspond to the (converted)`.
  **L2992 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Resulting elements correspond to the (converted)`。
- **L2993 EN**: Comment explains nearby logic, constraints, or intent: `elements from a __A.`.
  **L2993 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements from a __A.`。
- **L2994 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L2994 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L2995 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvts_ph_hf8(__m256h __A) {`.
  **L2995 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvts_ph_hf8(__m256h __A) {`。
- **L2996 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(`.
  **L2996 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(` 从当前函数返回。
- **L2997 EN**: Executes a call or declaration centered on `statement`.
  **L2997 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2998 EN**: Closes the current lexical scope or compound statement.
  **L2998 CN**: 结束当前词法作用域或复合语句块。
- **L2999 EN**: Blank line separating nearby declarations or logic blocks.
  **L2999 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3000 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L3000 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。

### Lines 3001-3024

````c
///    to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.
///    Merging mask \a __U is used to determine if given element should be taken
///    from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := __W.hf8[i]
/// 	FI
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8S instruction.
///
/// \param __W
///    A 128-bit vector of [16 x hf8].
/// \param __U
///    A 16-bit merging mask.
````
- **L3001 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.`.
  **L3001 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.`。
- **L3002 EN**: Comment explains nearby logic, constraints, or intent: `Merging mask a __U is used to determine if given element should be taken`.
  **L3002 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Merging mask a __U is used to determine if given element should be taken`。
- **L3003 EN**: Comment explains nearby logic, constraints, or intent: `from a __W instead.`.
  **L3003 CN**: 注释解释附近代码的逻辑、约束或设计意图：`from a __W instead.`。
- **L3004 EN**: Separator comment used for visual grouping.
  **L3004 CN**: 用于视觉分组的分隔注释。
- **L3005 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L3005 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L3006 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L3006 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L3007 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L3007 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L3008 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`.
  **L3008 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`。
- **L3009 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L3009 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L3010 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : __W.hf8[i]`.
  **L3010 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : __W.hf8[i]`。
- **L3011 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L3011 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L3012 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L3012 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L3013 EN**: Separator comment used for visual grouping.
  **L3013 CN**: 用于视觉分组的分隔注释。
- **L3014 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L3014 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L3015 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L3015 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L3016 EN**: Separator comment used for visual grouping.
  **L3016 CN**: 用于视觉分组的分隔注释。
- **L3017 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L3017 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L3018 EN**: Separator comment used for visual grouping.
  **L3018 CN**: 用于视觉分组的分隔注释。
- **L3019 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8S instruction.`.
  **L3019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8S instruction.`。
- **L3020 EN**: Separator comment used for visual grouping.
  **L3020 CN**: 用于视觉分组的分隔注释。
- **L3021 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L3021 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L3022 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8].`.
  **L3022 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8].`。
- **L3023 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L3023 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L3024 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L3024 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。

### Lines 3025-3048

````c
/// \param __A
///    A 256-bit vector of [8 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Resulting elements correspond to the
///    (converted) elements from \a __A. If
///    corresponding mask bit is not set, then element from \a __W is taken instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_mask_cvts_ph_hf8(__m128i __W, __mmask16 __U, __m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)__W, (__mmask16)__U);
}

/// Convert 256-bit vector \a __A containing packed FP16 floating-point elements
///    to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.
///    Zeroing mask \a __U is used to determine if given element should be zeroed
///    instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		dst.hf8[i] := convert_fp16_to_hf8_saturate(__A.fp16[i])
/// 	ELSE
/// 		dst.hf8[i] := 0
/// 	FI
````
- **L3025 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L3025 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L3026 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x fp16].`.
  **L3026 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x fp16].`。
- **L3027 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L3027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L3028 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Resulting elements correspond to the`.
  **L3028 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Resulting elements correspond to the`。
- **L3029 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If`.
  **L3029 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If`。
- **L3030 EN**: Comment explains nearby logic, constraints, or intent: `corresponding mask bit is not set, then element from a __W is taken instead.`.
  **L3030 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding mask bit is not set, then element from a __W is taken instead.`。
- **L3031 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L3031 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L3032 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvts_ph_hf8(__m128i __W, __mmask16 __U, __m256h __A) {`.
  **L3032 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvts_ph_hf8(__m128i __W, __mmask16 __U, __m256h __A) {`。
- **L3033 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(`.
  **L3033 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(` 从当前函数返回。
- **L3034 EN**: Executes a call or declaration centered on `statement`.
  **L3034 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3035 EN**: Closes the current lexical scope or compound statement.
  **L3035 CN**: 结束当前词法作用域或复合语句块。
- **L3036 EN**: Blank line separating nearby declarations or logic blocks.
  **L3036 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3037 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A containing packed FP16 floating-point elements`.
  **L3037 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A containing packed FP16 floating-point elements`。
- **L3038 EN**: Comment explains nearby logic, constraints, or intent: `to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.`.
  **L3038 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to a 128-bit vector containing E4M3 FP8 elements. Results are saturated.`。
- **L3039 EN**: Comment explains nearby logic, constraints, or intent: `Zeroing mask a __U is used to determine if given element should be zeroed`.
  **L3039 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zeroing mask a __U is used to determine if given element should be zeroed`。
- **L3040 EN**: Comment explains nearby logic, constraints, or intent: `instead.`.
  **L3040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instead.`。
- **L3041 EN**: Separator comment used for visual grouping.
  **L3041 CN**: 用于视觉分组的分隔注释。
- **L3042 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L3042 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L3043 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L3043 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L3044 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L3044 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L3045 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`.
  **L3045 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : convert_fp16_to_hf8_saturate(__A.fp16[i])`。
- **L3046 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L3046 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L3047 EN**: Comment explains nearby logic, constraints, or intent: `dst.hf8[i] : 0`.
  **L3047 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.hf8[i] : 0`。
- **L3048 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L3048 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。

### Lines 3049-3072

````c
/// ENDFOR
///
/// dst[MAX:128] := 0
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c VCVTPH2HF8S instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [16 x fp16].
/// \returns
///    A 128-bit vector of [16 x hf8]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set,
///    then element is zeroed instead.
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvts_ph_hf8(__mmask16 __U, __m256h __A) {
  return (__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(
      (__v16hf)__A, (__v16qi)(__m128i)_mm_setzero_si128(), (__mmask16)__U);
}

/// Convert 128-bit vector \a __A, containing packed FP8 E5M2 floating-point
````
- **L3049 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L3049 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L3050 EN**: Separator comment used for visual grouping.
  **L3050 CN**: 用于视觉分组的分隔注释。
- **L3051 EN**: Comment explains nearby logic, constraints, or intent: `dst[MAX:128] : 0`.
  **L3051 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst[MAX:128] : 0`。
- **L3052 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L3052 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L3053 EN**: Separator comment used for visual grouping.
  **L3053 CN**: 用于视觉分组的分隔注释。
- **L3054 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L3054 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L3055 EN**: Separator comment used for visual grouping.
  **L3055 CN**: 用于视觉分组的分隔注释。
- **L3056 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VCVTPH2HF8S instruction.`.
  **L3056 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VCVTPH2HF8S instruction.`。
- **L3057 EN**: Separator comment used for visual grouping.
  **L3057 CN**: 用于视觉分组的分隔注释。
- **L3058 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L3058 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L3059 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L3059 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L3060 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L3060 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L3061 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L3061 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L3062 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L3062 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L3063 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x hf8]. Resulting elements correspond to the`.
  **L3063 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x hf8]. Resulting elements correspond to the`。
- **L3064 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set,`.
  **L3064 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set,`。
- **L3065 EN**: Comment explains nearby logic, constraints, or intent: `then element is zeroed instead.`.
  **L3065 CN**: 注释解释附近代码的逻辑、约束或设计意图：`then element is zeroed instead.`。
- **L3066 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L3066 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L3067 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvts_ph_hf8(__mmask16 __U, __m256h __A) {`.
  **L3067 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvts_ph_hf8(__mmask16 __U, __m256h __A) {`。
- **L3068 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(`.
  **L3068 CN**: 以 `(__m128i)__builtin_ia32_vcvtph2hf8s_256_mask(` 从当前函数返回。
- **L3069 EN**: Executes a call or declaration centered on `statement`.
  **L3069 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3070 EN**: Closes the current lexical scope or compound statement.
  **L3070 CN**: 结束当前词法作用域或复合语句块。
- **L3071 EN**: Blank line separating nearby declarations or logic blocks.
  **L3071 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3072 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A, containing packed FP8 E5M2 floating-point`.
  **L3072 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A, containing packed FP8 E5M2 floating-point`。

### Lines 3073-3096

````c
///    elements to a 128-bit vector containing FP16 elements. The conversion is exact.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	dst.fp16[i] := convert_bf8_to_fp16(__A.bf8[i])
/// ENDFOR
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic does not correspond to a single instruction.
///
/// \param __A
///    A 128-bit vector of [16 x bf8].
/// \returns
///    A 128-bit vector of [8 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtbf8_ph(__m128i __A) {
  return _mm_castsi128_ph(_mm_slli_epi16(_mm_cvtepi8_epi16(__A), 8));
}

/// Convert 128-bit vector \a __A, containing packed FP8 E5M2 floating-point
///    elements to a 128-bit vector containing FP16 elements. The conversion is
///    exact. Merging mask \a __U is used to determine if given element should be
````
- **L3073 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 128-bit vector containing FP16 elements. The conversion is exact.`.
  **L3073 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 128-bit vector containing FP16 elements. The conversion is exact.`。
- **L3074 EN**: Separator comment used for visual grouping.
  **L3074 CN**: 用于视觉分组的分隔注释。
- **L3075 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L3075 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L3076 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L3076 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L3077 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`.
  **L3077 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`。
- **L3078 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L3078 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L3079 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L3079 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L3080 EN**: Separator comment used for visual grouping.
  **L3080 CN**: 用于视觉分组的分隔注释。
- **L3081 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L3081 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L3082 EN**: Separator comment used for visual grouping.
  **L3082 CN**: 用于视觉分组的分隔注释。
- **L3083 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic does not correspond to a single instruction.`.
  **L3083 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic does not correspond to a single instruction.`。
- **L3084 EN**: Separator comment used for visual grouping.
  **L3084 CN**: 用于视觉分组的分隔注释。
- **L3085 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L3085 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L3086 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L3086 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L3087 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L3087 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L3088 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`.
  **L3088 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`。
- **L3089 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L3089 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L3090 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtbf8_ph(__m128i __A) {`.
  **L3090 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtbf8_ph(__m128i __A) {`。
- **L3091 EN**: Returns from the current function with `_mm_castsi128_ph(_mm_slli_epi16(_mm_cvtepi8_epi16(__A), 8))`.
  **L3091 CN**: 以 `_mm_castsi128_ph(_mm_slli_epi16(_mm_cvtepi8_epi16(__A), 8))` 从当前函数返回。
- **L3092 EN**: Closes the current lexical scope or compound statement.
  **L3092 CN**: 结束当前词法作用域或复合语句块。
- **L3093 EN**: Blank line separating nearby declarations or logic blocks.
  **L3093 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3094 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A, containing packed FP8 E5M2 floating-point`.
  **L3094 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A, containing packed FP8 E5M2 floating-point`。
- **L3095 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 128-bit vector containing FP16 elements. The conversion is`.
  **L3095 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 128-bit vector containing FP16 elements. The conversion is`。
- **L3096 EN**: Comment explains nearby logic, constraints, or intent: `exact. Merging mask a __U is used to determine if given element should be`.
  **L3096 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exact. Merging mask a __U is used to determine if given element should be`。

### Lines 3097-3120

````c
///    taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.fp16[i] := convert_bf8_to_fp16(__A.bf8[i])
/// 	ELSE
/// 		dst.fp16[i] := __W.fp16[i]
/// 	FI
/// ENDFOR
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic does not correspond to a single instruction.
///
/// \param __W
///    A 128-bit vector of [8 x fp16].
/// \param __U
///    A 8-bit merging mask.
/// \param __A
///    A 128-bit vector of [16 x bf8].
/// \returns
///    A 128-bit vector of [8 x fp16]. Resulting elements correspond to the
````
- **L3097 EN**: Comment explains nearby logic, constraints, or intent: `taken from a __W instead.`.
  **L3097 CN**: 注释解释附近代码的逻辑、约束或设计意图：`taken from a __W instead.`。
- **L3098 EN**: Separator comment used for visual grouping.
  **L3098 CN**: 用于视觉分组的分隔注释。
- **L3099 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L3099 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L3100 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L3100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L3101 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L3101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L3102 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`.
  **L3102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`。
- **L3103 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L3103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L3104 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : __W.fp16[i]`.
  **L3104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : __W.fp16[i]`。
- **L3105 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L3105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L3106 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L3106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L3107 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L3107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L3108 EN**: Separator comment used for visual grouping.
  **L3108 CN**: 用于视觉分组的分隔注释。
- **L3109 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L3109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L3110 EN**: Separator comment used for visual grouping.
  **L3110 CN**: 用于视觉分组的分隔注释。
- **L3111 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic does not correspond to a single instruction.`.
  **L3111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic does not correspond to a single instruction.`。
- **L3112 EN**: Separator comment used for visual grouping.
  **L3112 CN**: 用于视觉分组的分隔注释。
- **L3113 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L3113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L3114 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16].`.
  **L3114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16].`。
- **L3115 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L3115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L3116 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit merging mask.`.
  **L3116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit merging mask.`。
- **L3117 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L3117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L3118 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L3118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L3119 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L3119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L3120 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`.
  **L3120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`。

### Lines 3121-3144

````c
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_cvtbf8_ph(__m128h __W, __mmask8 __U, __m128i __A) {
  return _mm_castsi128_ph(
      _mm_mask_slli_epi16((__m128i)__W, __U, _mm_cvtepi8_epi16(__A), 8));
}

/// Convert 128-bit vector \a __A, containing packed FP8 E5M2 floating-point
///    elements to a 128-bit vector containing FP16 elements. The conversion is
///    exact. Zeroing mask \a __U is used to determine if given element should be
///    zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 7
/// 	IF __U[i]
/// 		dst.fp16[i] := convert_bf8_to_fp16(__A.bf8[i])
/// 	ELSE
/// 		dst.fp16[i] := 0
/// 	FI
/// ENDFOR
/// \endcode
///
/// \headerfile <immintrin.h>
````
- **L3121 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L3121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L3122 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L3122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L3123 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L3123 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L3124 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtbf8_ph(__m128h __W, __mmask8 __U, __m128i __A) {`.
  **L3124 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtbf8_ph(__m128h __W, __mmask8 __U, __m128i __A) {`。
- **L3125 EN**: Returns from the current function with `_mm_castsi128_ph(`.
  **L3125 CN**: 以 `_mm_castsi128_ph(` 从当前函数返回。
- **L3126 EN**: Executes a call or declaration centered on `_mm_mask_slli_epi16`.
  **L3126 CN**: 执行以 `_mm_mask_slli_epi16` 为核心的调用或声明。
- **L3127 EN**: Closes the current lexical scope or compound statement.
  **L3127 CN**: 结束当前词法作用域或复合语句块。
- **L3128 EN**: Blank line separating nearby declarations or logic blocks.
  **L3128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3129 EN**: Comment explains nearby logic, constraints, or intent: `Convert 128-bit vector a __A, containing packed FP8 E5M2 floating-point`.
  **L3129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 128-bit vector a __A, containing packed FP8 E5M2 floating-point`。
- **L3130 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 128-bit vector containing FP16 elements. The conversion is`.
  **L3130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 128-bit vector containing FP16 elements. The conversion is`。
- **L3131 EN**: Comment explains nearby logic, constraints, or intent: `exact. Zeroing mask a __U is used to determine if given element should be`.
  **L3131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exact. Zeroing mask a __U is used to determine if given element should be`。
- **L3132 EN**: Comment explains nearby logic, constraints, or intent: `zeroed instead.`.
  **L3132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zeroed instead.`。
- **L3133 EN**: Separator comment used for visual grouping.
  **L3133 CN**: 用于视觉分组的分隔注释。
- **L3134 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L3134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L3135 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 7`.
  **L3135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 7`。
- **L3136 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L3136 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L3137 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`.
  **L3137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`。
- **L3138 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L3138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L3139 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : 0`.
  **L3139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : 0`。
- **L3140 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L3140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L3141 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L3141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L3142 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L3142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L3143 EN**: Separator comment used for visual grouping.
  **L3143 CN**: 用于视觉分组的分隔注释。
- **L3144 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L3144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。

### Lines 3145-3168

````c
///
/// This intrinsic does not correspond to a single instruction.
///
/// \param __U
///    A 8-bit zeroing mask.
/// \param __A
///    A 128-bit vector of [16 x bf8].
/// \returns
///    A 128-bit vector of [8 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    zero is taken instead.
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvtbf8_ph(__mmask8 __U, __m128i __A) {
  return _mm_castsi128_ph(_mm_slli_epi16(_mm_maskz_cvtepi8_epi16(__U, __A), 8));
}

/// Convert 256-bit vector \a __A, containing packed FP8 E4M3 floating-point
///    elements to a 256-bit vector containing FP16 elements. The conversion is exact.
///
/// \code{.operation}
/// FOR i := 0 to 15
/// 	dst.fp16[i] := convert_bf8_to_fp16(__A.bf8[i])
/// ENDFOR
/// \endcode
````
- **L3145 EN**: Separator comment used for visual grouping.
  **L3145 CN**: 用于视觉分组的分隔注释。
- **L3146 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic does not correspond to a single instruction.`.
  **L3146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic does not correspond to a single instruction.`。
- **L3147 EN**: Separator comment used for visual grouping.
  **L3147 CN**: 用于视觉分组的分隔注释。
- **L3148 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L3148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L3149 EN**: Comment explains nearby logic, constraints, or intent: `A 8-bit zeroing mask.`.
  **L3149 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 8-bit zeroing mask.`。
- **L3150 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L3150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L3151 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [16 x bf8].`.
  **L3151 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [16 x bf8].`。
- **L3152 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L3152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L3153 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`.
  **L3153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x fp16]. Resulting elements correspond to the`。
- **L3154 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L3154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L3155 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L3155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L3156 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L3156 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L3157 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtbf8_ph(__mmask8 __U, __m128i __A) {`.
  **L3157 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtbf8_ph(__mmask8 __U, __m128i __A) {`。
- **L3158 EN**: Returns from the current function with `_mm_castsi128_ph(_mm_slli_epi16(_mm_maskz_cvtepi8_epi16(__U, __A), 8))`.
  **L3158 CN**: 以 `_mm_castsi128_ph(_mm_slli_epi16(_mm_maskz_cvtepi8_epi16(__U, __A), 8))` 从当前函数返回。
- **L3159 EN**: Closes the current lexical scope or compound statement.
  **L3159 CN**: 结束当前词法作用域或复合语句块。
- **L3160 EN**: Blank line separating nearby declarations or logic blocks.
  **L3160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3161 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A, containing packed FP8 E4M3 floating-point`.
  **L3161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A, containing packed FP8 E4M3 floating-point`。
- **L3162 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 256-bit vector containing FP16 elements. The conversion is exact.`.
  **L3162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 256-bit vector containing FP16 elements. The conversion is exact.`。
- **L3163 EN**: Separator comment used for visual grouping.
  **L3163 CN**: 用于视觉分组的分隔注释。
- **L3164 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L3164 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L3165 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L3165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L3166 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`.
  **L3166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`。
- **L3167 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L3167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L3168 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L3168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。

### Lines 3169-3192

````c
///
/// \headerfile <immintrin.h>
///
/// This intrinsic does not correspond to a single instruction.
///
/// \param __A
///    A 256-bit vector of [32 x bf8].
/// \returns
///    A 256-bit vector of [16 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A.
static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvtbf8_ph(__m128i __A) {
  return _mm256_castsi256_ph(_mm256_slli_epi16(_mm256_cvtepi8_epi16(__A), 8));
}

/// Convert 256-bit vector \a __A, containing packed FP8 E5M2 floating-point
///    elements to a 256-bit vector containing FP16 elements. The conversion is
///    exact. Merging mask \a __U is used to determine if given element should be
///    taken from \a __W instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		dst.fp16[i] := convert_bf8_to_fp16(__A.bf8[i])
/// 	ELSE
````
- **L3169 EN**: Separator comment used for visual grouping.
  **L3169 CN**: 用于视觉分组的分隔注释。
- **L3170 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L3170 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L3171 EN**: Separator comment used for visual grouping.
  **L3171 CN**: 用于视觉分组的分隔注释。
- **L3172 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic does not correspond to a single instruction.`.
  **L3172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic does not correspond to a single instruction.`。
- **L3173 EN**: Separator comment used for visual grouping.
  **L3173 CN**: 用于视觉分组的分隔注释。
- **L3174 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L3174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L3175 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8].`.
  **L3175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8].`。
- **L3176 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L3176 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L3177 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`.
  **L3177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`。
- **L3178 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A.`.
  **L3178 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A.`。
- **L3179 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvtbf8_ph(__m128i __A) {`.
  **L3179 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_cvtbf8_ph(__m128i __A) {`。
- **L3180 EN**: Returns from the current function with `_mm256_castsi256_ph(_mm256_slli_epi16(_mm256_cvtepi8_epi16(__A), 8))`.
  **L3180 CN**: 以 `_mm256_castsi256_ph(_mm256_slli_epi16(_mm256_cvtepi8_epi16(__A), 8))` 从当前函数返回。
- **L3181 EN**: Closes the current lexical scope or compound statement.
  **L3181 CN**: 结束当前词法作用域或复合语句块。
- **L3182 EN**: Blank line separating nearby declarations or logic blocks.
  **L3182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3183 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A, containing packed FP8 E5M2 floating-point`.
  **L3183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A, containing packed FP8 E5M2 floating-point`。
- **L3184 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 256-bit vector containing FP16 elements. The conversion is`.
  **L3184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 256-bit vector containing FP16 elements. The conversion is`。
- **L3185 EN**: Comment explains nearby logic, constraints, or intent: `exact. Merging mask a __U is used to determine if given element should be`.
  **L3185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exact. Merging mask a __U is used to determine if given element should be`。
- **L3186 EN**: Comment explains nearby logic, constraints, or intent: `taken from a __W instead.`.
  **L3186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`taken from a __W instead.`。
- **L3187 EN**: Separator comment used for visual grouping.
  **L3187 CN**: 用于视觉分组的分隔注释。
- **L3188 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L3188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L3189 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L3189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L3190 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L3190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L3191 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`.
  **L3191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`。
- **L3192 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L3192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。

### Lines 3193-3216

````c
/// 		dst.fp16[i] := __W.fp16[i]
/// 	FI
/// ENDFOR
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic does not correspond to a single instruction.
///
/// \param __W
///    A 256-bit vector of [16 x fp16].
/// \param __U
///    A 16-bit merging mask.
/// \param __A
///    A 256-bit vector of [32 x bf8].
/// \returns
///    A 256-bit vector of [16 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    element from \a __W is taken instead.
static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_mask_cvtbf8_ph(__m256h __W, __mmask16 __U, __m128i __A) {
  return _mm256_castsi256_ph(
      _mm256_mask_slli_epi16((__m256i)__W, __U, _mm256_cvtepi8_epi16(__A), 8));
}
````
- **L3193 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : __W.fp16[i]`.
  **L3193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : __W.fp16[i]`。
- **L3194 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L3194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L3195 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L3195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L3196 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L3196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L3197 EN**: Separator comment used for visual grouping.
  **L3197 CN**: 用于视觉分组的分隔注释。
- **L3198 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L3198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L3199 EN**: Separator comment used for visual grouping.
  **L3199 CN**: 用于视觉分组的分隔注释。
- **L3200 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic does not correspond to a single instruction.`.
  **L3200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic does not correspond to a single instruction.`。
- **L3201 EN**: Separator comment used for visual grouping.
  **L3201 CN**: 用于视觉分组的分隔注释。
- **L3202 EN**: Comment explains nearby logic, constraints, or intent: `param __W`.
  **L3202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __W`。
- **L3203 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16].`.
  **L3203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16].`。
- **L3204 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L3204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L3205 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit merging mask.`.
  **L3205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit merging mask.`。
- **L3206 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L3206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L3207 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8].`.
  **L3207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8].`。
- **L3208 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L3208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L3209 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`.
  **L3209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`。
- **L3210 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L3210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L3211 EN**: Comment explains nearby logic, constraints, or intent: `element from a __W is taken instead.`.
  **L3211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element from a __W is taken instead.`。
- **L3212 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L3212 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L3213 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtbf8_ph(__m256h __W, __mmask16 __U, __m128i __A) {`.
  **L3213 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtbf8_ph(__m256h __W, __mmask16 __U, __m128i __A) {`。
- **L3214 EN**: Returns from the current function with `_mm256_castsi256_ph(`.
  **L3214 CN**: 以 `_mm256_castsi256_ph(` 从当前函数返回。
- **L3215 EN**: Executes a call or declaration centered on `_mm256_mask_slli_epi16`.
  **L3215 CN**: 执行以 `_mm256_mask_slli_epi16` 为核心的调用或声明。
- **L3216 EN**: Closes the current lexical scope or compound statement.
  **L3216 CN**: 结束当前词法作用域或复合语句块。

### Lines 3217-3240

````c

/// Convert 256-bit vector \a __A, containing packed FP8 E5M2 floating-point
///    elements to a 256-bit vector containing FP16 elements. The conversion is
///    exact. Zeroing mask \a __U is used to determine if given element should be
///    zeroed instead.
///
/// \code{.operation}
/// FOR i := 0 to 15 
/// 	IF __U[i]
/// 		dst.fp16[i] := convert_bf8_to_fp16(__A.bf8[i])
/// 	ELSE
/// 		dst.fp16[i] := 0
/// 	FI
/// ENDFOR
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic does not correspond to a single instruction.
///
/// \param __U
///    A 16-bit zeroing mask.
/// \param __A
///    A 256-bit vector of [32 x bf8].
````
- **L3217 EN**: Blank line separating nearby declarations or logic blocks.
  **L3217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3218 EN**: Comment explains nearby logic, constraints, or intent: `Convert 256-bit vector a __A, containing packed FP8 E5M2 floating-point`.
  **L3218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert 256-bit vector a __A, containing packed FP8 E5M2 floating-point`。
- **L3219 EN**: Comment explains nearby logic, constraints, or intent: `elements to a 256-bit vector containing FP16 elements. The conversion is`.
  **L3219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements to a 256-bit vector containing FP16 elements. The conversion is`。
- **L3220 EN**: Comment explains nearby logic, constraints, or intent: `exact. Zeroing mask a __U is used to determine if given element should be`.
  **L3220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exact. Zeroing mask a __U is used to determine if given element should be`。
- **L3221 EN**: Comment explains nearby logic, constraints, or intent: `zeroed instead.`.
  **L3221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zeroed instead.`。
- **L3222 EN**: Separator comment used for visual grouping.
  **L3222 CN**: 用于视觉分组的分隔注释。
- **L3223 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L3223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L3224 EN**: Comment explains nearby logic, constraints, or intent: `FOR i : 0 to 15`.
  **L3224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i : 0 to 15`。
- **L3225 EN**: Comment explains nearby logic, constraints, or intent: `IF __U[i]`.
  **L3225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF __U[i]`。
- **L3226 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`.
  **L3226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : convert_bf8_to_fp16(__A.bf8[i])`。
- **L3227 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L3227 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L3228 EN**: Comment explains nearby logic, constraints, or intent: `dst.fp16[i] : 0`.
  **L3228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dst.fp16[i] : 0`。
- **L3229 EN**: Comment explains nearby logic, constraints, or intent: `FI`.
  **L3229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FI`。
- **L3230 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L3230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L3231 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L3231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L3232 EN**: Separator comment used for visual grouping.
  **L3232 CN**: 用于视觉分组的分隔注释。
- **L3233 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L3233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L3234 EN**: Separator comment used for visual grouping.
  **L3234 CN**: 用于视觉分组的分隔注释。
- **L3235 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic does not correspond to a single instruction.`.
  **L3235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic does not correspond to a single instruction.`。
- **L3236 EN**: Separator comment used for visual grouping.
  **L3236 CN**: 用于视觉分组的分隔注释。
- **L3237 EN**: Comment explains nearby logic, constraints, or intent: `param __U`.
  **L3237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __U`。
- **L3238 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit zeroing mask.`.
  **L3238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit zeroing mask.`。
- **L3239 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L3239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L3240 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [32 x bf8].`.
  **L3240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [32 x bf8].`。

### Lines 3241-3257

````c
/// \returns
///    A 256-bit vector of [16 x fp16]. Resulting elements correspond to the
///    (converted) elements from \a __A. If corresponding mask bit is not set, then
///    zero is taken instead.
static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtbf8_ph(__mmask16 __U, __m128i __A) {
  return _mm256_castsi256_ph(
      _mm256_slli_epi16(_mm256_maskz_cvtepi8_epi16(__U, __A), 8));
}

// clang-format on

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256

#endif // __AVX10_2CONVERTINTRIN_H
#endif // __SSE2__
````
- **L3241 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L3241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L3242 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`.
  **L3242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x fp16]. Resulting elements correspond to the`。
- **L3243 EN**: Comment explains nearby logic, constraints, or intent: `(converted) elements from a __A. If corresponding mask bit is not set, then`.
  **L3243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(converted) elements from a __A. If corresponding mask bit is not set, then`。
- **L3244 EN**: Comment explains nearby logic, constraints, or intent: `zero is taken instead.`.
  **L3244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zero is taken instead.`。
- **L3245 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L3245 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L3246 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtbf8_ph(__mmask16 __U, __m128i __A) {`.
  **L3246 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtbf8_ph(__mmask16 __U, __m128i __A) {`。
- **L3247 EN**: Returns from the current function with `_mm256_castsi256_ph(`.
  **L3247 CN**: 以 `_mm256_castsi256_ph(` 从当前函数返回。
- **L3248 EN**: Executes a call or declaration centered on `_mm256_slli_epi16`.
  **L3248 CN**: 执行以 `_mm256_slli_epi16` 为核心的调用或声明。
- **L3249 EN**: Closes the current lexical scope or compound statement.
  **L3249 CN**: 结束当前词法作用域或复合语句块。
- **L3250 EN**: Blank line separating nearby declarations or logic blocks.
  **L3250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3251 EN**: Comment explains nearby logic, constraints, or intent: `clang-format on`.
  **L3251 CN**: 注释解释附近代码的逻辑、约束或设计意图：`clang-format on`。
- **L3252 EN**: Blank line separating nearby declarations or logic blocks.
  **L3252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3253 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L3253 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L3254 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L3254 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。
- **L3255 EN**: Blank line separating nearby declarations or logic blocks.
  **L3255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3256 EN**: Closes the current preprocessor conditional block.
  **L3256 CN**: 结束当前预处理条件块。
- **L3257 EN**: Closes the current preprocessor conditional block.
  **L3257 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **x86 SIMD or system intrinsics / x86 SIMD 或系统 intrinsic 接口**
- **VE vector interfaces / VE 向量接口**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Inline wrapper functions / 内联包装函数**
- **Compiler-specific attributes / 编译器特定属性**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__SSE2__`, `__AVX10_2CONVERTINTRIN_H`
- **External builtins / 外部 builtin**: `__builtin_ia32_vcvt2ps2phx128_mask`, `__builtin_ia32_vcvt2ps2phx256_mask`, `__builtin_ia32_vcvtbiasph2bf8_128_mask`, `__builtin_ia32_vcvtbiasph2bf8_256_mask`, `__builtin_ia32_vcvtbiasph2bf8s_128_mask`, `__builtin_ia32_vcvtbiasph2bf8s_256_mask`, `__builtin_ia32_vcvtbiasph2hf8_128_mask`, `__builtin_ia32_vcvtbiasph2hf8_256_mask`, `__builtin_ia32_vcvtbiasph2hf8s_128_mask`, `__builtin_ia32_vcvtbiasph2hf8s_256_mask`, `__builtin_ia32_vcvt2ph2bf8_128`, `__builtin_ia32_selectb_128`
