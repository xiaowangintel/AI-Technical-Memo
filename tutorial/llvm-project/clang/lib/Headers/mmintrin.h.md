# mmintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/mmintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: MMX intrinsics.
- **Purpose (CN)**: 提供 MMX intrinsic 接口。
- **Line Count / 行数**: 1541

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- mmintrin.h - MMX intrinsics --------------------------------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __MMINTRIN_H
#define __MMINTRIN_H

#if !defined(__i386__) && !defined(__x86_64__)
#error "This header is only meant to be used on x86 and x64 architecture"
#endif

typedef long long __m64 __attribute__((__vector_size__(8), __aligned__(8)));

typedef long long __v1di __attribute__((__vector_size__(8)));
typedef int __v2si __attribute__((__vector_size__(8)));
typedef short __v4hi __attribute__((__vector_size__(8)));
typedef char __v8qi __attribute__((__vector_size__(8)));

/* Unsigned types */
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
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __MMINTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __MMINTRIN_H`。
- **L11 EN**: Defines macro `__MMINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `__MMINTRIN_H`，用于条件编译、简写或 API 生成。
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
- **L17 EN**: Introduces an alias or helper declaration: `typedef long long __m64 __attribute__((__vector_size__(8), __aligned__(8)));`.
  **L17 CN**: 引入一条别名或辅助声明：`typedef long long __m64 __attribute__((__vector_size__(8), __aligned__(8)));`。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Introduces an alias or helper declaration: `typedef long long __v1di __attribute__((__vector_size__(8)));`.
  **L19 CN**: 引入一条别名或辅助声明：`typedef long long __v1di __attribute__((__vector_size__(8)));`。
- **L20 EN**: Introduces an alias or helper declaration: `typedef int __v2si __attribute__((__vector_size__(8)));`.
  **L20 CN**: 引入一条别名或辅助声明：`typedef int __v2si __attribute__((__vector_size__(8)));`。
- **L21 EN**: Introduces an alias or helper declaration: `typedef short __v4hi __attribute__((__vector_size__(8)));`.
  **L21 CN**: 引入一条别名或辅助声明：`typedef short __v4hi __attribute__((__vector_size__(8)));`。
- **L22 EN**: Introduces an alias or helper declaration: `typedef char __v8qi __attribute__((__vector_size__(8)));`.
  **L22 CN**: 引入一条别名或辅助声明：`typedef char __v8qi __attribute__((__vector_size__(8)));`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Comment explains nearby logic, constraints, or intent: `Unsigned types`.
  **L24 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unsigned types`。

### Lines 25-48

````c
typedef unsigned long long __v1du __attribute__ ((__vector_size__ (8)));
typedef unsigned int __v2su __attribute__ ((__vector_size__ (8)));
typedef unsigned short __v4hu __attribute__((__vector_size__(8)));
typedef unsigned char __v8qu __attribute__((__vector_size__(8)));

/* We need an explicitly signed variant for char. Note that this shouldn't
 * appear in the interface though. */
typedef signed char __v8qs __attribute__((__vector_size__(8)));

/* SSE/SSE2 types */
typedef long long __m128i __attribute__((__vector_size__(16), __aligned__(16)));
typedef long long __v2di __attribute__ ((__vector_size__ (16)));
typedef int __v4si __attribute__((__vector_size__(16)));
typedef short __v8hi __attribute__((__vector_size__(16)));
typedef char __v16qi __attribute__((__vector_size__(16)));

/* Define the default attributes for the functions in this file. */
#if defined(__cplusplus) && (__cplusplus >= 201103L)
#define __DEFAULT_FN_ATTRS_SSE2                                                \
  __attribute__((__always_inline__, __nodebug__, __target__("sse2"),           \
                 __min_vector_width__(128))) constexpr
#else
#define __DEFAULT_FN_ATTRS_SSE2                                                \
  __attribute__((__always_inline__, __nodebug__, __target__("sse2"),           \
````
- **L25 EN**: Introduces an alias or helper declaration: `typedef unsigned long long __v1du __attribute__ ((__vector_size__ (8)));`.
  **L25 CN**: 引入一条别名或辅助声明：`typedef unsigned long long __v1du __attribute__ ((__vector_size__ (8)));`。
- **L26 EN**: Introduces an alias or helper declaration: `typedef unsigned int __v2su __attribute__ ((__vector_size__ (8)));`.
  **L26 CN**: 引入一条别名或辅助声明：`typedef unsigned int __v2su __attribute__ ((__vector_size__ (8)));`。
- **L27 EN**: Introduces an alias or helper declaration: `typedef unsigned short __v4hu __attribute__((__vector_size__(8)));`.
  **L27 CN**: 引入一条别名或辅助声明：`typedef unsigned short __v4hu __attribute__((__vector_size__(8)));`。
- **L28 EN**: Introduces an alias or helper declaration: `typedef unsigned char __v8qu __attribute__((__vector_size__(8)));`.
  **L28 CN**: 引入一条别名或辅助声明：`typedef unsigned char __v8qu __attribute__((__vector_size__(8)));`。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L30 EN**: Comment highlights an implementation note: `We need an explicitly signed variant for char. Note that this shouldn't`.
  **L30 CN**: 注释强调一条实现说明：`We need an explicitly signed variant for char. Note that this shouldn't`。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `appear in the interface though.`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`appear in the interface though.`。
- **L32 EN**: Introduces an alias or helper declaration: `typedef signed char __v8qs __attribute__((__vector_size__(8)));`.
  **L32 CN**: 引入一条别名或辅助声明：`typedef signed char __v8qs __attribute__((__vector_size__(8)));`。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L34 EN**: Comment explains nearby logic, constraints, or intent: `SSE/SSE2 types`.
  **L34 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE/SSE2 types`。
- **L35 EN**: Introduces an alias or helper declaration: `typedef long long __m128i __attribute__((__vector_size__(16), __aligned__(16)));`.
  **L35 CN**: 引入一条别名或辅助声明：`typedef long long __m128i __attribute__((__vector_size__(16), __aligned__(16)));`。
- **L36 EN**: Introduces an alias or helper declaration: `typedef long long __v2di __attribute__ ((__vector_size__ (16)));`.
  **L36 CN**: 引入一条别名或辅助声明：`typedef long long __v2di __attribute__ ((__vector_size__ (16)));`。
- **L37 EN**: Introduces an alias or helper declaration: `typedef int __v4si __attribute__((__vector_size__(16)));`.
  **L37 CN**: 引入一条别名或辅助声明：`typedef int __v4si __attribute__((__vector_size__(16)));`。
- **L38 EN**: Introduces an alias or helper declaration: `typedef short __v8hi __attribute__((__vector_size__(16)));`.
  **L38 CN**: 引入一条别名或辅助声明：`typedef short __v8hi __attribute__((__vector_size__(16)));`。
- **L39 EN**: Introduces an alias or helper declaration: `typedef char __v16qi __attribute__((__vector_size__(16)));`.
  **L39 CN**: 引入一条别名或辅助声明：`typedef char __v16qi __attribute__((__vector_size__(16)));`。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L41 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L41 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L42 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L42 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L43 EN**: Defines macro `__DEFAULT_FN_ATTRS_SSE2` for conditional compilation, shorthand, or API generation.
  **L43 CN**: 定义宏 `__DEFAULT_FN_ATTRS_SSE2`，用于条件编译、简写或 API 生成。
- **L44 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("sse2"),           \`.
  **L44 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("sse2"),           \`。
- **L45 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L45 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L46 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L46 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L47 EN**: Defines macro `__DEFAULT_FN_ATTRS_SSE2` for conditional compilation, shorthand, or API generation.
  **L47 CN**: 定义宏 `__DEFAULT_FN_ATTRS_SSE2`，用于条件编译、简写或 API 生成。
- **L48 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("sse2"),           \`.
  **L48 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("sse2"),           \`。

### Lines 49-72

````c
                 __min_vector_width__(128)))
#endif

#define __trunc64(x)                                                           \
  (__m64) __builtin_shufflevector((__v2di)(x), __extension__(__v2di){}, 0)
#define __zext128(x)                                                           \
  (__m128i) __builtin_shufflevector((__v2si)(x), __extension__(__v2si){}, 0,   \
                                    1, 2, 3)

/// Clears the MMX state by setting the state of the x87 stack registers
///    to empty.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> EMMS </c> instruction.
///
static __inline__ void
    __attribute__((__always_inline__, __nodebug__, __target__("mmx")))
    _mm_empty(void) {
  __builtin_ia32_emms();
}

/// Constructs a 64-bit integer vector, setting the lower 32 bits to the
///    value of the 32-bit integer parameter and setting the upper 32 bits to 0.
````
- **L49 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L49 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L50 EN**: Closes the current preprocessor conditional block.
  **L50 CN**: 结束当前预处理条件块。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Defines macro `__trunc64(x)` for conditional compilation, shorthand, or API generation.
  **L52 CN**: 定义宏 `__trunc64(x)`，用于条件编译、简写或 API 生成。
- **L53 EN**: Continues logic associated with callable symbol `__builtin_shufflevector`.
  **L53 CN**: 继续与可调用符号 `__builtin_shufflevector` 相关的逻辑。
- **L54 EN**: Defines macro `__zext128(x)` for conditional compilation, shorthand, or API generation.
  **L54 CN**: 定义宏 `__zext128(x)`，用于条件编译、简写或 API 生成。
- **L55 EN**: Continues logic associated with callable symbol `__builtin_shufflevector`.
  **L55 CN**: 继续与可调用符号 `__builtin_shufflevector` 相关的逻辑。
- **L56 EN**: Continues the surrounding expression or declaration: `1, 2, 3)`.
  **L56 CN**: 继续构造周围的表达式或声明：`1, 2, 3)`。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L58 EN**: Comment explains nearby logic, constraints, or intent: `Clears the MMX state by setting the state of the x87 stack registers`.
  **L58 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Clears the MMX state by setting the state of the x87 stack registers`。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `to empty.`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to empty.`。
- **L60 EN**: Separator comment used for visual grouping.
  **L60 CN**: 用于视觉分组的分隔注释。
- **L61 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L61 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L62 EN**: Separator comment used for visual grouping.
  **L62 CN**: 用于视觉分组的分隔注释。
- **L63 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> EMMS </c> instruction.`.
  **L63 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> EMMS </c> instruction.`。
- **L64 EN**: Separator comment used for visual grouping.
  **L64 CN**: 用于视觉分组的分隔注释。
- **L65 EN**: Continues the surrounding expression or declaration: `static __inline__ void`.
  **L65 CN**: 继续构造周围的表达式或声明：`static __inline__ void`。
- **L66 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("mmx")))`.
  **L66 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("mmx")))`。
- **L67 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_empty(void) {`.
  **L67 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_empty(void) {`。
- **L68 EN**: Executes a call or declaration centered on `__builtin_ia32_emms`.
  **L68 CN**: 执行以 `__builtin_ia32_emms` 为核心的调用或声明。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L71 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector, setting the lower 32 bits to the`.
  **L71 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector, setting the lower 32 bits to the`。
- **L72 EN**: Comment explains nearby logic, constraints, or intent: `value of the 32-bit integer parameter and setting the upper 32 bits to 0.`.
  **L72 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value of the 32-bit integer parameter and setting the upper 32 bits to 0.`。

### Lines 73-96

````c
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> MOVD </c> instruction.
///
/// \param __i
///    A 32-bit integer value.
/// \returns A 64-bit integer vector. The lower 32 bits contain the value of the
///    parameter. The upper 32 bits are set to 0.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi32_si64(int __i) {
  return __extension__(__m64)(__v2si){__i, 0};
}

/// Returns the lower 32 bits of a 64-bit integer vector as a 32-bit
///    signed integer.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> MOVD </c> instruction.
///
/// \param __m
///    A 64-bit integer vector.
/// \returns A 32-bit signed integer value containing the lower 32 bits of the
///    parameter.
````
- **L73 EN**: Separator comment used for visual grouping.
  **L73 CN**: 用于视觉分组的分隔注释。
- **L74 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L74 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L75 EN**: Separator comment used for visual grouping.
  **L75 CN**: 用于视觉分组的分隔注释。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> MOVD </c> instruction.`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> MOVD </c> instruction.`。
- **L77 EN**: Separator comment used for visual grouping.
  **L77 CN**: 用于视觉分组的分隔注释。
- **L78 EN**: Comment explains nearby logic, constraints, or intent: `param __i`.
  **L78 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __i`。
- **L79 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L79 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector. The lower 32 bits contain the value of the`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector. The lower 32 bits contain the value of the`。
- **L81 EN**: Comment explains nearby logic, constraints, or intent: `parameter. The upper 32 bits are set to 0.`.
  **L81 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter. The upper 32 bits are set to 0.`。
- **L82 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi32_si64(int __i) {`.
  **L82 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi32_si64(int __i) {`。
- **L83 EN**: Returns from the current function with `__extension__(__m64)(__v2si){__i, 0}`.
  **L83 CN**: 以 `__extension__(__m64)(__v2si){__i, 0}` 从当前函数返回。
- **L84 EN**: Closes the current lexical scope or compound statement.
  **L84 CN**: 结束当前词法作用域或复合语句块。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L86 EN**: Comment explains nearby logic, constraints, or intent: `Returns the lower 32 bits of a 64-bit integer vector as a 32-bit`.
  **L86 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Returns the lower 32 bits of a 64-bit integer vector as a 32-bit`。
- **L87 EN**: Comment explains nearby logic, constraints, or intent: `signed integer.`.
  **L87 CN**: 注释解释附近代码的逻辑、约束或设计意图：`signed integer.`。
- **L88 EN**: Separator comment used for visual grouping.
  **L88 CN**: 用于视觉分组的分隔注释。
- **L89 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L89 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L90 EN**: Separator comment used for visual grouping.
  **L90 CN**: 用于视觉分组的分隔注释。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> MOVD </c> instruction.`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> MOVD </c> instruction.`。
- **L92 EN**: Separator comment used for visual grouping.
  **L92 CN**: 用于视觉分组的分隔注释。
- **L93 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L93 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L94 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L94 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `returns A 32-bit signed integer value containing the lower 32 bits of the`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 32-bit signed integer value containing the lower 32 bits of the`。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `parameter.`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter.`。

### Lines 97-120

````c
static __inline__ int __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi64_si32(__m64 __m) {
  return ((__v2si)__m)[0];
}

/// Casts a 64-bit signed integer value into a 64-bit integer vector.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> MOVQ </c> instruction.
///
/// \param __i
///    A 64-bit signed integer.
/// \returns A 64-bit integer vector containing the same bitwise pattern as the
///    parameter.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi64_m64(long long __i) {
  return __extension__(__m64)(__v1di){__i};
}

/// Casts a 64-bit integer vector into a 64-bit signed integer value.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> MOVQ </c> instruction.
///
````
- **L97 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ int __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi64_si32(__m64 __m) {`.
  **L97 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ int __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi64_si32(__m64 __m) {`。
- **L98 EN**: Returns from the current function with `((__v2si)__m)[0]`.
  **L98 CN**: 以 `((__v2si)__m)[0]` 从当前函数返回。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L101 EN**: Comment explains nearby logic, constraints, or intent: `Casts a 64-bit signed integer value into a 64-bit integer vector.`.
  **L101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Casts a 64-bit signed integer value into a 64-bit integer vector.`。
- **L102 EN**: Separator comment used for visual grouping.
  **L102 CN**: 用于视觉分组的分隔注释。
- **L103 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L104 EN**: Separator comment used for visual grouping.
  **L104 CN**: 用于视觉分组的分隔注释。
- **L105 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> MOVQ </c> instruction.`.
  **L105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> MOVQ </c> instruction.`。
- **L106 EN**: Separator comment used for visual grouping.
  **L106 CN**: 用于视觉分组的分隔注释。
- **L107 EN**: Comment explains nearby logic, constraints, or intent: `param __i`.
  **L107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __i`。
- **L108 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit signed integer.`.
  **L108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit signed integer.`。
- **L109 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the same bitwise pattern as the`.
  **L109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the same bitwise pattern as the`。
- **L110 EN**: Comment explains nearby logic, constraints, or intent: `parameter.`.
  **L110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter.`。
- **L111 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi64_m64(long long __i) {`.
  **L111 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cvtsi64_m64(long long __i) {`。
- **L112 EN**: Returns from the current function with `__extension__(__m64)(__v1di){__i}`.
  **L112 CN**: 以 `__extension__(__m64)(__v1di){__i}` 从当前函数返回。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Comment explains nearby logic, constraints, or intent: `Casts a 64-bit integer vector into a 64-bit signed integer value.`.
  **L115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Casts a 64-bit integer vector into a 64-bit signed integer value.`。
- **L116 EN**: Separator comment used for visual grouping.
  **L116 CN**: 用于视觉分组的分隔注释。
- **L117 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L118 EN**: Separator comment used for visual grouping.
  **L118 CN**: 用于视觉分组的分隔注释。
- **L119 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> MOVQ </c> instruction.`.
  **L119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> MOVQ </c> instruction.`。
- **L120 EN**: Separator comment used for visual grouping.
  **L120 CN**: 用于视觉分组的分隔注释。

### Lines 121-144

````c
/// \param __m
///    A 64-bit integer vector.
/// \returns A 64-bit signed integer containing the same bitwise pattern as the
///    parameter.
static __inline__ long long __DEFAULT_FN_ATTRS_SSE2 _mm_cvtm64_si64(__m64 __m) {
  return ((__v1di)__m)[0];
}

/// Converts, with saturation, 16-bit signed integers from both 64-bit integer
///    vector parameters of [4 x i16] into 8-bit signed integer values, and
///    constructs a 64-bit integer vector of [8 x i8] as the result.
///
///    Positive values greater than 0x7F are saturated to 0x7F. Negative values
///    less than 0x80 are saturated to 0x80.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PACKSSWB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are
///    written to the lower 32 bits of the result.
/// \param __m2
///    A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are
````
- **L121 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L122 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。
- **L123 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit signed integer containing the same bitwise pattern as the`.
  **L123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit signed integer containing the same bitwise pattern as the`。
- **L124 EN**: Comment explains nearby logic, constraints, or intent: `parameter.`.
  **L124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter.`。
- **L125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ long long __DEFAULT_FN_ATTRS_SSE2 _mm_cvtm64_si64(__m64 __m) {`.
  **L125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ long long __DEFAULT_FN_ATTRS_SSE2 _mm_cvtm64_si64(__m64 __m) {`。
- **L126 EN**: Returns from the current function with `((__v1di)__m)[0]`.
  **L126 CN**: 以 `((__v1di)__m)[0]` 从当前函数返回。
- **L127 EN**: Closes the current lexical scope or compound statement.
  **L127 CN**: 结束当前词法作用域或复合语句块。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L129 EN**: Comment explains nearby logic, constraints, or intent: `Converts, with saturation, 16-bit signed integers from both 64-bit integer`.
  **L129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converts, with saturation, 16-bit signed integers from both 64-bit integer`。
- **L130 EN**: Comment explains nearby logic, constraints, or intent: `vector parameters of [4 x i16] into 8-bit signed integer values, and`.
  **L130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector parameters of [4 x i16] into 8-bit signed integer values, and`。
- **L131 EN**: Comment explains nearby logic, constraints, or intent: `constructs a 64-bit integer vector of [8 x i8] as the result.`.
  **L131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`constructs a 64-bit integer vector of [8 x i8] as the result.`。
- **L132 EN**: Separator comment used for visual grouping.
  **L132 CN**: 用于视觉分组的分隔注释。
- **L133 EN**: Comment explains nearby logic, constraints, or intent: `Positive values greater than 0x7F are saturated to 0x7F. Negative values`.
  **L133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Positive values greater than 0x7F are saturated to 0x7F. Negative values`。
- **L134 EN**: Comment explains nearby logic, constraints, or intent: `less than 0x80 are saturated to 0x80.`.
  **L134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`less than 0x80 are saturated to 0x80.`。
- **L135 EN**: Separator comment used for visual grouping.
  **L135 CN**: 用于视觉分组的分隔注释。
- **L136 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L136 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L137 EN**: Separator comment used for visual grouping.
  **L137 CN**: 用于视觉分组的分隔注释。
- **L138 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PACKSSWB </c> instruction.`.
  **L138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PACKSSWB </c> instruction.`。
- **L139 EN**: Separator comment used for visual grouping.
  **L139 CN**: 用于视觉分组的分隔注释。
- **L140 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L141 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are`.
  **L141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are`。
- **L142 EN**: Comment explains nearby logic, constraints, or intent: `written to the lower 32 bits of the result.`.
  **L142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`written to the lower 32 bits of the result.`。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L144 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are`.
  **L144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are`。

### Lines 145-168

````c
///    written to the upper 32 bits of the result.
/// \returns A 64-bit integer vector of [8 x i8] containing the converted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pi16(__m64 __m1,
                                                               __m64 __m2) {
  return __trunc64(__builtin_ia32_packsswb128(
      (__v8hi)__builtin_shufflevector(__m1, __m2, 0, 1), (__v8hi){}));
}

/// Converts, with saturation, 32-bit signed integers from both 64-bit integer
///    vector parameters of [2 x i32] into 16-bit signed integer values, and
///    constructs a 64-bit integer vector of [4 x i16] as the result.
///
///    Positive values greater than 0x7FFF are saturated to 0x7FFF. Negative
///    values less than 0x8000 are saturated to 0x8000.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PACKSSDW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [2 x i32]. The converted [2 x i16] values are
///    written to the lower 32 bits of the result.
/// \param __m2
````
- **L145 EN**: Comment explains nearby logic, constraints, or intent: `written to the upper 32 bits of the result.`.
  **L145 CN**: 注释解释附近代码的逻辑、约束或设计意图：`written to the upper 32 bits of the result.`。
- **L146 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the converted`.
  **L146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the converted`。
- **L147 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pi16(__m64 __m1,`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pi16(__m64 __m1,`。
- **L149 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L149 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L150 EN**: Returns from the current function with `__trunc64(__builtin_ia32_packsswb128(`.
  **L150 CN**: 以 `__trunc64(__builtin_ia32_packsswb128(` 从当前函数返回。
- **L151 EN**: Executes a call or declaration centered on `statement`.
  **L151 CN**: 执行以 `statement` 为核心的调用或声明。
- **L152 EN**: Closes the current lexical scope or compound statement.
  **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L154 EN**: Comment explains nearby logic, constraints, or intent: `Converts, with saturation, 32-bit signed integers from both 64-bit integer`.
  **L154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converts, with saturation, 32-bit signed integers from both 64-bit integer`。
- **L155 EN**: Comment explains nearby logic, constraints, or intent: `vector parameters of [2 x i32] into 16-bit signed integer values, and`.
  **L155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector parameters of [2 x i32] into 16-bit signed integer values, and`。
- **L156 EN**: Comment explains nearby logic, constraints, or intent: `constructs a 64-bit integer vector of [4 x i16] as the result.`.
  **L156 CN**: 注释解释附近代码的逻辑、约束或设计意图：`constructs a 64-bit integer vector of [4 x i16] as the result.`。
- **L157 EN**: Separator comment used for visual grouping.
  **L157 CN**: 用于视觉分组的分隔注释。
- **L158 EN**: Comment explains nearby logic, constraints, or intent: `Positive values greater than 0x7FFF are saturated to 0x7FFF. Negative`.
  **L158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Positive values greater than 0x7FFF are saturated to 0x7FFF. Negative`。
- **L159 EN**: Comment explains nearby logic, constraints, or intent: `values less than 0x8000 are saturated to 0x8000.`.
  **L159 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values less than 0x8000 are saturated to 0x8000.`。
- **L160 EN**: Separator comment used for visual grouping.
  **L160 CN**: 用于视觉分组的分隔注释。
- **L161 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L162 EN**: Separator comment used for visual grouping.
  **L162 CN**: 用于视觉分组的分隔注释。
- **L163 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PACKSSDW </c> instruction.`.
  **L163 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PACKSSDW </c> instruction.`。
- **L164 EN**: Separator comment used for visual grouping.
  **L164 CN**: 用于视觉分组的分隔注释。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32]. The converted [2 x i16] values are`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32]. The converted [2 x i16] values are`。
- **L167 EN**: Comment explains nearby logic, constraints, or intent: `written to the lower 32 bits of the result.`.
  **L167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`written to the lower 32 bits of the result.`。
- **L168 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。

### Lines 169-192

````c
///    A 64-bit integer vector of [2 x i32]. The converted [2 x i16] values are
///    written to the upper 32 bits of the result.
/// \returns A 64-bit integer vector of [4 x i16] containing the converted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pi32(__m64 __m1,
                                                               __m64 __m2) {
  return __trunc64(__builtin_ia32_packssdw128(
      (__v4si)__builtin_shufflevector(__m1, __m2, 0, 1), (__v4si){}));
}

/// Converts, with saturation, 16-bit signed integers from both 64-bit integer
///    vector parameters of [4 x i16] into 8-bit unsigned integer values, and
///    constructs a 64-bit integer vector of [8 x i8] as the result.
///
///    Values greater than 0xFF are saturated to 0xFF. Values less than 0 are
///    saturated to 0.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PACKUSWB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are
///    written to the lower 32 bits of the result.
````
- **L169 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32]. The converted [2 x i16] values are`.
  **L169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32]. The converted [2 x i16] values are`。
- **L170 EN**: Comment explains nearby logic, constraints, or intent: `written to the upper 32 bits of the result.`.
  **L170 CN**: 注释解释附近代码的逻辑、约束或设计意图：`written to the upper 32 bits of the result.`。
- **L171 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the converted`.
  **L171 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the converted`。
- **L172 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pi32(__m64 __m1,`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pi32(__m64 __m1,`。
- **L174 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L174 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L175 EN**: Returns from the current function with `__trunc64(__builtin_ia32_packssdw128(`.
  **L175 CN**: 以 `__trunc64(__builtin_ia32_packssdw128(` 从当前函数返回。
- **L176 EN**: Executes a call or declaration centered on `statement`.
  **L176 CN**: 执行以 `statement` 为核心的调用或声明。
- **L177 EN**: Closes the current lexical scope or compound statement.
  **L177 CN**: 结束当前词法作用域或复合语句块。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L179 EN**: Comment explains nearby logic, constraints, or intent: `Converts, with saturation, 16-bit signed integers from both 64-bit integer`.
  **L179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converts, with saturation, 16-bit signed integers from both 64-bit integer`。
- **L180 EN**: Comment explains nearby logic, constraints, or intent: `vector parameters of [4 x i16] into 8-bit unsigned integer values, and`.
  **L180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector parameters of [4 x i16] into 8-bit unsigned integer values, and`。
- **L181 EN**: Comment explains nearby logic, constraints, or intent: `constructs a 64-bit integer vector of [8 x i8] as the result.`.
  **L181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`constructs a 64-bit integer vector of [8 x i8] as the result.`。
- **L182 EN**: Separator comment used for visual grouping.
  **L182 CN**: 用于视觉分组的分隔注释。
- **L183 EN**: Comment explains nearby logic, constraints, or intent: `Values greater than 0xFF are saturated to 0xFF. Values less than 0 are`.
  **L183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Values greater than 0xFF are saturated to 0xFF. Values less than 0 are`。
- **L184 EN**: Comment explains nearby logic, constraints, or intent: `saturated to 0.`.
  **L184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`saturated to 0.`。
- **L185 EN**: Separator comment used for visual grouping.
  **L185 CN**: 用于视觉分组的分隔注释。
- **L186 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L187 EN**: Separator comment used for visual grouping.
  **L187 CN**: 用于视觉分组的分隔注释。
- **L188 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PACKUSWB </c> instruction.`.
  **L188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PACKUSWB </c> instruction.`。
- **L189 EN**: Separator comment used for visual grouping.
  **L189 CN**: 用于视觉分组的分隔注释。
- **L190 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L191 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are`.
  **L191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are`。
- **L192 EN**: Comment explains nearby logic, constraints, or intent: `written to the lower 32 bits of the result.`.
  **L192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`written to the lower 32 bits of the result.`。

### Lines 193-216

````c
/// \param __m2
///    A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are
///    written to the upper 32 bits of the result.
/// \returns A 64-bit integer vector of [8 x i8] containing the converted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pu16(__m64 __m1,
                                                               __m64 __m2) {
  return __trunc64(__builtin_ia32_packuswb128(
      (__v8hi)__builtin_shufflevector(__m1, __m2, 0, 1), (__v8hi){}));
}

/// Unpacks the upper 32 bits from two 64-bit integer vectors of [8 x i8]
///    and interleaves them into a 64-bit integer vector of [8 x i8].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PUNPCKHBW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8]. \n
///    Bits [39:32] are written to bits [7:0] of the result. \n
///    Bits [47:40] are written to bits [23:16] of the result. \n
///    Bits [55:48] are written to bits [39:32] of the result. \n
///    Bits [63:56] are written to bits [55:48] of the result.
````
- **L193 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16]. The converted [4 x i8] values are`。
- **L195 EN**: Comment explains nearby logic, constraints, or intent: `written to the upper 32 bits of the result.`.
  **L195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`written to the upper 32 bits of the result.`。
- **L196 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the converted`.
  **L196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the converted`。
- **L197 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pu16(__m64 __m1,`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_packs_pu16(__m64 __m1,`。
- **L199 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L199 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L200 EN**: Returns from the current function with `__trunc64(__builtin_ia32_packuswb128(`.
  **L200 CN**: 以 `__trunc64(__builtin_ia32_packuswb128(` 从当前函数返回。
- **L201 EN**: Executes a call or declaration centered on `statement`.
  **L201 CN**: 执行以 `statement` 为核心的调用或声明。
- **L202 EN**: Closes the current lexical scope or compound statement.
  **L202 CN**: 结束当前词法作用域或复合语句块。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L204 EN**: Comment explains nearby logic, constraints, or intent: `Unpacks the upper 32 bits from two 64-bit integer vectors of [8 x i8]`.
  **L204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unpacks the upper 32 bits from two 64-bit integer vectors of [8 x i8]`。
- **L205 EN**: Comment explains nearby logic, constraints, or intent: `and interleaves them into a 64-bit integer vector of [8 x i8].`.
  **L205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and interleaves them into a 64-bit integer vector of [8 x i8].`。
- **L206 EN**: Separator comment used for visual grouping.
  **L206 CN**: 用于视觉分组的分隔注释。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L208 EN**: Separator comment used for visual grouping.
  **L208 CN**: 用于视觉分组的分隔注释。
- **L209 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PUNPCKHBW </c> instruction.`.
  **L209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PUNPCKHBW </c> instruction.`。
- **L210 EN**: Separator comment used for visual grouping.
  **L210 CN**: 用于视觉分组的分隔注释。
- **L211 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L212 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8]. n`.
  **L212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8]. n`。
- **L213 EN**: Comment explains nearby logic, constraints, or intent: `Bits [39:32] are written to bits [7:0] of the result. n`.
  **L213 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [39:32] are written to bits [7:0] of the result. n`。
- **L214 EN**: Comment explains nearby logic, constraints, or intent: `Bits [47:40] are written to bits [23:16] of the result. n`.
  **L214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [47:40] are written to bits [23:16] of the result. n`。
- **L215 EN**: Comment explains nearby logic, constraints, or intent: `Bits [55:48] are written to bits [39:32] of the result. n`.
  **L215 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [55:48] are written to bits [39:32] of the result. n`。
- **L216 EN**: Comment explains nearby logic, constraints, or intent: `Bits [63:56] are written to bits [55:48] of the result.`.
  **L216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [63:56] are written to bits [55:48] of the result.`。

### Lines 217-240

````c
/// \param __m2
///    A 64-bit integer vector of [8 x i8].
///    Bits [39:32] are written to bits [15:8] of the result. \n
///    Bits [47:40] are written to bits [31:24] of the result. \n
///    Bits [55:48] are written to bits [47:40] of the result. \n
///    Bits [63:56] are written to bits [63:56] of the result.
/// \returns A 64-bit integer vector of [8 x i8] containing the interleaved
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi8(__m64 __m1,
                                                                 __m64 __m2) {
  return (__m64)__builtin_shufflevector((__v8qi)__m1, (__v8qi)__m2, 4, 12, 5,
                                        13, 6, 14, 7, 15);
}

/// Unpacks the upper 32 bits from two 64-bit integer vectors of
///    [4 x i16] and interleaves them into a 64-bit integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PUNPCKHWD </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
///    Bits [47:32] are written to bits [15:0] of the result. \n
````
- **L217 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L218 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L219 EN**: Comment explains nearby logic, constraints, or intent: `Bits [39:32] are written to bits [15:8] of the result. n`.
  **L219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [39:32] are written to bits [15:8] of the result. n`。
- **L220 EN**: Comment explains nearby logic, constraints, or intent: `Bits [47:40] are written to bits [31:24] of the result. n`.
  **L220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [47:40] are written to bits [31:24] of the result. n`。
- **L221 EN**: Comment explains nearby logic, constraints, or intent: `Bits [55:48] are written to bits [47:40] of the result. n`.
  **L221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [55:48] are written to bits [47:40] of the result. n`。
- **L222 EN**: Comment explains nearby logic, constraints, or intent: `Bits [63:56] are written to bits [63:56] of the result.`.
  **L222 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [63:56] are written to bits [63:56] of the result.`。
- **L223 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the interleaved`.
  **L223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the interleaved`。
- **L224 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi8(__m64 __m1,`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi8(__m64 __m1,`。
- **L226 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L226 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L227 EN**: Returns from the current function with `(__m64)__builtin_shufflevector((__v8qi)__m1, (__v8qi)__m2, 4, 12, 5,`.
  **L227 CN**: 以 `(__m64)__builtin_shufflevector((__v8qi)__m1, (__v8qi)__m2, 4, 12, 5,` 从当前函数返回。
- **L228 EN**: Adds a standalone statement or declaration: `13, 6, 14, 7, 15);`.
  **L228 CN**: 添加一条独立语句或声明：`13, 6, 14, 7, 15);`。
- **L229 EN**: Closes the current lexical scope or compound statement.
  **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L231 EN**: Comment explains nearby logic, constraints, or intent: `Unpacks the upper 32 bits from two 64-bit integer vectors of`.
  **L231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unpacks the upper 32 bits from two 64-bit integer vectors of`。
- **L232 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i16] and interleaves them into a 64-bit integer vector of [4 x i16].`.
  **L232 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i16] and interleaves them into a 64-bit integer vector of [4 x i16].`。
- **L233 EN**: Separator comment used for visual grouping.
  **L233 CN**: 用于视觉分组的分隔注释。
- **L234 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L235 EN**: Separator comment used for visual grouping.
  **L235 CN**: 用于视觉分组的分隔注释。
- **L236 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PUNPCKHWD </c> instruction.`.
  **L236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PUNPCKHWD </c> instruction.`。
- **L237 EN**: Separator comment used for visual grouping.
  **L237 CN**: 用于视觉分组的分隔注释。
- **L238 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L239 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L240 EN**: Comment explains nearby logic, constraints, or intent: `Bits [47:32] are written to bits [15:0] of the result. n`.
  **L240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [47:32] are written to bits [15:0] of the result. n`。

### Lines 241-264

````c
///    Bits [63:48] are written to bits [47:32] of the result.
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
///    Bits [47:32] are written to bits [31:16] of the result. \n
///    Bits [63:48] are written to bits [63:48] of the result.
/// \returns A 64-bit integer vector of [4 x i16] containing the interleaved
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi16(__m64 __m1,
                                                                  __m64 __m2) {
  return (__m64)__builtin_shufflevector((__v4hi)__m1, (__v4hi)__m2, 2, 6, 3, 7);
}

/// Unpacks the upper 32 bits from two 64-bit integer vectors of
///    [2 x i32] and interleaves them into a 64-bit integer vector of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PUNPCKHDQ </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [2 x i32]. The upper 32 bits are written to
///    the lower 32 bits of the result.
/// \param __m2
///    A 64-bit integer vector of [2 x i32]. The upper 32 bits are written to
````
- **L241 EN**: Comment explains nearby logic, constraints, or intent: `Bits [63:48] are written to bits [47:32] of the result.`.
  **L241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [63:48] are written to bits [47:32] of the result.`。
- **L242 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L243 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L244 EN**: Comment explains nearby logic, constraints, or intent: `Bits [47:32] are written to bits [31:16] of the result. n`.
  **L244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [47:32] are written to bits [31:16] of the result. n`。
- **L245 EN**: Comment explains nearby logic, constraints, or intent: `Bits [63:48] are written to bits [63:48] of the result.`.
  **L245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [63:48] are written to bits [63:48] of the result.`。
- **L246 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the interleaved`.
  **L246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the interleaved`。
- **L247 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L247 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi16(__m64 __m1,`.
  **L248 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi16(__m64 __m1,`。
- **L249 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L249 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L250 EN**: Returns from the current function with `(__m64)__builtin_shufflevector((__v4hi)__m1, (__v4hi)__m2, 2, 6, 3, 7)`.
  **L250 CN**: 以 `(__m64)__builtin_shufflevector((__v4hi)__m1, (__v4hi)__m2, 2, 6, 3, 7)` 从当前函数返回。
- **L251 EN**: Closes the current lexical scope or compound statement.
  **L251 CN**: 结束当前词法作用域或复合语句块。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L253 EN**: Comment explains nearby logic, constraints, or intent: `Unpacks the upper 32 bits from two 64-bit integer vectors of`.
  **L253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unpacks the upper 32 bits from two 64-bit integer vectors of`。
- **L254 EN**: Comment explains nearby logic, constraints, or intent: `[2 x i32] and interleaves them into a 64-bit integer vector of [2 x i32].`.
  **L254 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x i32] and interleaves them into a 64-bit integer vector of [2 x i32].`。
- **L255 EN**: Separator comment used for visual grouping.
  **L255 CN**: 用于视觉分组的分隔注释。
- **L256 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L257 EN**: Separator comment used for visual grouping.
  **L257 CN**: 用于视觉分组的分隔注释。
- **L258 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PUNPCKHDQ </c> instruction.`.
  **L258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PUNPCKHDQ </c> instruction.`。
- **L259 EN**: Separator comment used for visual grouping.
  **L259 CN**: 用于视觉分组的分隔注释。
- **L260 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L261 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32]. The upper 32 bits are written to`.
  **L261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32]. The upper 32 bits are written to`。
- **L262 EN**: Comment explains nearby logic, constraints, or intent: `the lower 32 bits of the result.`.
  **L262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the lower 32 bits of the result.`。
- **L263 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L264 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32]. The upper 32 bits are written to`.
  **L264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32]. The upper 32 bits are written to`。

### Lines 265-288

````c
///    the upper 32 bits of the result.
/// \returns A 64-bit integer vector of [2 x i32] containing the interleaved
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi32(__m64 __m1,
                                                                  __m64 __m2) {
  return (__m64)__builtin_shufflevector((__v2si)__m1, (__v2si)__m2, 1, 3);
}

/// Unpacks the lower 32 bits from two 64-bit integer vectors of [8 x i8]
///    and interleaves them into a 64-bit integer vector of [8 x i8].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PUNPCKLBW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8].
///    Bits [7:0] are written to bits [7:0] of the result. \n
///    Bits [15:8] are written to bits [23:16] of the result. \n
///    Bits [23:16] are written to bits [39:32] of the result. \n
///    Bits [31:24] are written to bits [55:48] of the result.
/// \param __m2
///    A 64-bit integer vector of [8 x i8].
///    Bits [7:0] are written to bits [15:8] of the result. \n
````
- **L265 EN**: Comment explains nearby logic, constraints, or intent: `the upper 32 bits of the result.`.
  **L265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the upper 32 bits of the result.`。
- **L266 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the interleaved`.
  **L266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the interleaved`。
- **L267 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi32(__m64 __m1,`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpackhi_pi32(__m64 __m1,`。
- **L269 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L269 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L270 EN**: Returns from the current function with `(__m64)__builtin_shufflevector((__v2si)__m1, (__v2si)__m2, 1, 3)`.
  **L270 CN**: 以 `(__m64)__builtin_shufflevector((__v2si)__m1, (__v2si)__m2, 1, 3)` 从当前函数返回。
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L273 EN**: Comment explains nearby logic, constraints, or intent: `Unpacks the lower 32 bits from two 64-bit integer vectors of [8 x i8]`.
  **L273 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unpacks the lower 32 bits from two 64-bit integer vectors of [8 x i8]`。
- **L274 EN**: Comment explains nearby logic, constraints, or intent: `and interleaves them into a 64-bit integer vector of [8 x i8].`.
  **L274 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and interleaves them into a 64-bit integer vector of [8 x i8].`。
- **L275 EN**: Separator comment used for visual grouping.
  **L275 CN**: 用于视觉分组的分隔注释。
- **L276 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L277 EN**: Separator comment used for visual grouping.
  **L277 CN**: 用于视觉分组的分隔注释。
- **L278 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PUNPCKLBW </c> instruction.`.
  **L278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PUNPCKLBW </c> instruction.`。
- **L279 EN**: Separator comment used for visual grouping.
  **L279 CN**: 用于视觉分组的分隔注释。
- **L280 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L281 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L281 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L282 EN**: Comment explains nearby logic, constraints, or intent: `Bits [7:0] are written to bits [7:0] of the result. n`.
  **L282 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [7:0] are written to bits [7:0] of the result. n`。
- **L283 EN**: Comment explains nearby logic, constraints, or intent: `Bits [15:8] are written to bits [23:16] of the result. n`.
  **L283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [15:8] are written to bits [23:16] of the result. n`。
- **L284 EN**: Comment explains nearby logic, constraints, or intent: `Bits [23:16] are written to bits [39:32] of the result. n`.
  **L284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [23:16] are written to bits [39:32] of the result. n`。
- **L285 EN**: Comment explains nearby logic, constraints, or intent: `Bits [31:24] are written to bits [55:48] of the result.`.
  **L285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [31:24] are written to bits [55:48] of the result.`。
- **L286 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L286 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L287 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L288 EN**: Comment explains nearby logic, constraints, or intent: `Bits [7:0] are written to bits [15:8] of the result. n`.
  **L288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [7:0] are written to bits [15:8] of the result. n`。

### Lines 289-312

````c
///    Bits [15:8] are written to bits [31:24] of the result. \n
///    Bits [23:16] are written to bits [47:40] of the result. \n
///    Bits [31:24] are written to bits [63:56] of the result.
/// \returns A 64-bit integer vector of [8 x i8] containing the interleaved
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi8(__m64 __m1,
                                                                 __m64 __m2) {
  return (__m64)__builtin_shufflevector((__v8qi)__m1, (__v8qi)__m2, 0, 8, 1, 9,
                                        2, 10, 3, 11);
}

/// Unpacks the lower 32 bits from two 64-bit integer vectors of
///    [4 x i16] and interleaves them into a 64-bit integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PUNPCKLWD </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
///    Bits [15:0] are written to bits [15:0] of the result. \n
///    Bits [31:16] are written to bits [47:32] of the result.
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
````
- **L289 EN**: Comment explains nearby logic, constraints, or intent: `Bits [15:8] are written to bits [31:24] of the result. n`.
  **L289 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [15:8] are written to bits [31:24] of the result. n`。
- **L290 EN**: Comment explains nearby logic, constraints, or intent: `Bits [23:16] are written to bits [47:40] of the result. n`.
  **L290 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [23:16] are written to bits [47:40] of the result. n`。
- **L291 EN**: Comment explains nearby logic, constraints, or intent: `Bits [31:24] are written to bits [63:56] of the result.`.
  **L291 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [31:24] are written to bits [63:56] of the result.`。
- **L292 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the interleaved`.
  **L292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the interleaved`。
- **L293 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L293 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi8(__m64 __m1,`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi8(__m64 __m1,`。
- **L295 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L295 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L296 EN**: Returns from the current function with `(__m64)__builtin_shufflevector((__v8qi)__m1, (__v8qi)__m2, 0, 8, 1, 9,`.
  **L296 CN**: 以 `(__m64)__builtin_shufflevector((__v8qi)__m1, (__v8qi)__m2, 0, 8, 1, 9,` 从当前函数返回。
- **L297 EN**: Adds a standalone statement or declaration: `2, 10, 3, 11);`.
  **L297 CN**: 添加一条独立语句或声明：`2, 10, 3, 11);`。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L300 EN**: Comment explains nearby logic, constraints, or intent: `Unpacks the lower 32 bits from two 64-bit integer vectors of`.
  **L300 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unpacks the lower 32 bits from two 64-bit integer vectors of`。
- **L301 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i16] and interleaves them into a 64-bit integer vector of [4 x i16].`.
  **L301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i16] and interleaves them into a 64-bit integer vector of [4 x i16].`。
- **L302 EN**: Separator comment used for visual grouping.
  **L302 CN**: 用于视觉分组的分隔注释。
- **L303 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L304 EN**: Separator comment used for visual grouping.
  **L304 CN**: 用于视觉分组的分隔注释。
- **L305 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PUNPCKLWD </c> instruction.`.
  **L305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PUNPCKLWD </c> instruction.`。
- **L306 EN**: Separator comment used for visual grouping.
  **L306 CN**: 用于视觉分组的分隔注释。
- **L307 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L308 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L309 EN**: Comment explains nearby logic, constraints, or intent: `Bits [15:0] are written to bits [15:0] of the result. n`.
  **L309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [15:0] are written to bits [15:0] of the result. n`。
- **L310 EN**: Comment explains nearby logic, constraints, or intent: `Bits [31:16] are written to bits [47:32] of the result.`.
  **L310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [31:16] are written to bits [47:32] of the result.`。
- **L311 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L311 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L312 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。

### Lines 313-336

````c
///    Bits [15:0] are written to bits [31:16] of the result. \n
///    Bits [31:16] are written to bits [63:48] of the result.
/// \returns A 64-bit integer vector of [4 x i16] containing the interleaved
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi16(__m64 __m1,
                                                                  __m64 __m2) {
  return (__m64)__builtin_shufflevector((__v4hi)__m1, (__v4hi)__m2, 0, 4, 1, 5);
}

/// Unpacks the lower 32 bits from two 64-bit integer vectors of
///    [2 x i32] and interleaves them into a 64-bit integer vector of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PUNPCKLDQ </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [2 x i32]. The lower 32 bits are written to
///    the lower 32 bits of the result.
/// \param __m2
///    A 64-bit integer vector of [2 x i32]. The lower 32 bits are written to
///    the upper 32 bits of the result.
/// \returns A 64-bit integer vector of [2 x i32] containing the interleaved
///    values.
````
- **L313 EN**: Comment explains nearby logic, constraints, or intent: `Bits [15:0] are written to bits [31:16] of the result. n`.
  **L313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [15:0] are written to bits [31:16] of the result. n`。
- **L314 EN**: Comment explains nearby logic, constraints, or intent: `Bits [31:16] are written to bits [63:48] of the result.`.
  **L314 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bits [31:16] are written to bits [63:48] of the result.`。
- **L315 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the interleaved`.
  **L315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the interleaved`。
- **L316 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi16(__m64 __m1,`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi16(__m64 __m1,`。
- **L318 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L318 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L319 EN**: Returns from the current function with `(__m64)__builtin_shufflevector((__v4hi)__m1, (__v4hi)__m2, 0, 4, 1, 5)`.
  **L319 CN**: 以 `(__m64)__builtin_shufflevector((__v4hi)__m1, (__v4hi)__m2, 0, 4, 1, 5)` 从当前函数返回。
- **L320 EN**: Closes the current lexical scope or compound statement.
  **L320 CN**: 结束当前词法作用域或复合语句块。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L322 EN**: Comment explains nearby logic, constraints, or intent: `Unpacks the lower 32 bits from two 64-bit integer vectors of`.
  **L322 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unpacks the lower 32 bits from two 64-bit integer vectors of`。
- **L323 EN**: Comment explains nearby logic, constraints, or intent: `[2 x i32] and interleaves them into a 64-bit integer vector of [2 x i32].`.
  **L323 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x i32] and interleaves them into a 64-bit integer vector of [2 x i32].`。
- **L324 EN**: Separator comment used for visual grouping.
  **L324 CN**: 用于视觉分组的分隔注释。
- **L325 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L326 EN**: Separator comment used for visual grouping.
  **L326 CN**: 用于视觉分组的分隔注释。
- **L327 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PUNPCKLDQ </c> instruction.`.
  **L327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PUNPCKLDQ </c> instruction.`。
- **L328 EN**: Separator comment used for visual grouping.
  **L328 CN**: 用于视觉分组的分隔注释。
- **L329 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L329 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L330 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32]. The lower 32 bits are written to`.
  **L330 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32]. The lower 32 bits are written to`。
- **L331 EN**: Comment explains nearby logic, constraints, or intent: `the lower 32 bits of the result.`.
  **L331 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the lower 32 bits of the result.`。
- **L332 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L333 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32]. The lower 32 bits are written to`.
  **L333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32]. The lower 32 bits are written to`。
- **L334 EN**: Comment explains nearby logic, constraints, or intent: `the upper 32 bits of the result.`.
  **L334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the upper 32 bits of the result.`。
- **L335 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the interleaved`.
  **L335 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the interleaved`。
- **L336 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L336 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。

### Lines 337-360

````c
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi32(__m64 __m1,
                                                                  __m64 __m2) {
  return (__m64)__builtin_shufflevector((__v2si)__m1, (__v2si)__m2, 0, 2);
}

/// Adds each 8-bit integer element of the first 64-bit integer vector
///    of [8 x i8] to the corresponding 8-bit integer element of the second
///    64-bit integer vector of [8 x i8]. The lower 8 bits of the results are
///    packed into a 64-bit integer vector of [8 x i8].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PADDB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8].
/// \param __m2
///    A 64-bit integer vector of [8 x i8].
/// \returns A 64-bit integer vector of [8 x i8] containing the sums of both
///    parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi8(__m64 __m1,
                                                            __m64 __m2) {
  return (__m64)(((__v8qu)__m1) + ((__v8qu)__m2));
}
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi32(__m64 __m1,`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_unpacklo_pi32(__m64 __m1,`。
- **L338 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L338 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L339 EN**: Returns from the current function with `(__m64)__builtin_shufflevector((__v2si)__m1, (__v2si)__m2, 0, 2)`.
  **L339 CN**: 以 `(__m64)__builtin_shufflevector((__v2si)__m1, (__v2si)__m2, 0, 2)` 从当前函数返回。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。
- **L341 EN**: Blank line separating nearby declarations or logic blocks.
  **L341 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L342 EN**: Comment explains nearby logic, constraints, or intent: `Adds each 8-bit integer element of the first 64-bit integer vector`.
  **L342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Adds each 8-bit integer element of the first 64-bit integer vector`。
- **L343 EN**: Comment explains nearby logic, constraints, or intent: `of [8 x i8] to the corresponding 8-bit integer element of the second`.
  **L343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [8 x i8] to the corresponding 8-bit integer element of the second`。
- **L344 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [8 x i8]. The lower 8 bits of the results are`.
  **L344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [8 x i8]. The lower 8 bits of the results are`。
- **L345 EN**: Comment explains nearby logic, constraints, or intent: `packed into a 64-bit integer vector of [8 x i8].`.
  **L345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`packed into a 64-bit integer vector of [8 x i8].`。
- **L346 EN**: Separator comment used for visual grouping.
  **L346 CN**: 用于视觉分组的分隔注释。
- **L347 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L347 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L348 EN**: Separator comment used for visual grouping.
  **L348 CN**: 用于视觉分组的分隔注释。
- **L349 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PADDB </c> instruction.`.
  **L349 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PADDB </c> instruction.`。
- **L350 EN**: Separator comment used for visual grouping.
  **L350 CN**: 用于视觉分组的分隔注释。
- **L351 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L351 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L352 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L352 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L353 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L353 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L354 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L355 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the sums of both`.
  **L355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the sums of both`。
- **L356 EN**: Comment explains nearby logic, constraints, or intent: `parameters.`.
  **L356 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameters.`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi8(__m64 __m1,`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi8(__m64 __m1,`。
- **L358 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L358 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L359 EN**: Returns from the current function with `(__m64)(((__v8qu)__m1) + ((__v8qu)__m2))`.
  **L359 CN**: 以 `(__m64)(((__v8qu)__m1) + ((__v8qu)__m2))` 从当前函数返回。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384

````c

/// Adds each 16-bit integer element of the first 64-bit integer vector
///    of [4 x i16] to the corresponding 16-bit integer element of the second
///    64-bit integer vector of [4 x i16]. The lower 16 bits of the results are
///    packed into a 64-bit integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PADDW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
/// \returns A 64-bit integer vector of [4 x i16] containing the sums of both
///    parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi16(__m64 __m1,
                                                             __m64 __m2) {
  return (__m64)(((__v4hu)__m1) + ((__v4hu)__m2));
}

/// Adds each 32-bit integer element of the first 64-bit integer vector
///    of [2 x i32] to the corresponding 32-bit integer element of the second
///    64-bit integer vector of [2 x i32]. The lower 32 bits of the results are
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L362 EN**: Comment explains nearby logic, constraints, or intent: `Adds each 16-bit integer element of the first 64-bit integer vector`.
  **L362 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Adds each 16-bit integer element of the first 64-bit integer vector`。
- **L363 EN**: Comment explains nearby logic, constraints, or intent: `of [4 x i16] to the corresponding 16-bit integer element of the second`.
  **L363 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [4 x i16] to the corresponding 16-bit integer element of the second`。
- **L364 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [4 x i16]. The lower 16 bits of the results are`.
  **L364 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [4 x i16]. The lower 16 bits of the results are`。
- **L365 EN**: Comment explains nearby logic, constraints, or intent: `packed into a 64-bit integer vector of [4 x i16].`.
  **L365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`packed into a 64-bit integer vector of [4 x i16].`。
- **L366 EN**: Separator comment used for visual grouping.
  **L366 CN**: 用于视觉分组的分隔注释。
- **L367 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L368 EN**: Separator comment used for visual grouping.
  **L368 CN**: 用于视觉分组的分隔注释。
- **L369 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PADDW </c> instruction.`.
  **L369 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PADDW </c> instruction.`。
- **L370 EN**: Separator comment used for visual grouping.
  **L370 CN**: 用于视觉分组的分隔注释。
- **L371 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L371 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L372 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L372 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L373 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L373 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L374 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L375 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the sums of both`.
  **L375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the sums of both`。
- **L376 EN**: Comment explains nearby logic, constraints, or intent: `parameters.`.
  **L376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameters.`。
- **L377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi16(__m64 __m1,`.
  **L377 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi16(__m64 __m1,`。
- **L378 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L378 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L379 EN**: Returns from the current function with `(__m64)(((__v4hu)__m1) + ((__v4hu)__m2))`.
  **L379 CN**: 以 `(__m64)(((__v4hu)__m1) + ((__v4hu)__m2))` 从当前函数返回。
- **L380 EN**: Closes the current lexical scope or compound statement.
  **L380 CN**: 结束当前词法作用域或复合语句块。
- **L381 EN**: Blank line separating nearby declarations or logic blocks.
  **L381 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L382 EN**: Comment explains nearby logic, constraints, or intent: `Adds each 32-bit integer element of the first 64-bit integer vector`.
  **L382 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Adds each 32-bit integer element of the first 64-bit integer vector`。
- **L383 EN**: Comment explains nearby logic, constraints, or intent: `of [2 x i32] to the corresponding 32-bit integer element of the second`.
  **L383 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [2 x i32] to the corresponding 32-bit integer element of the second`。
- **L384 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [2 x i32]. The lower 32 bits of the results are`.
  **L384 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [2 x i32]. The lower 32 bits of the results are`。

### Lines 385-408

````c
///    packed into a 64-bit integer vector of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PADDD </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [2 x i32].
/// \param __m2
///    A 64-bit integer vector of [2 x i32].
/// \returns A 64-bit integer vector of [2 x i32] containing the sums of both
///    parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi32(__m64 __m1,
                                                             __m64 __m2) {
  return (__m64)(((__v2su)__m1) + ((__v2su)__m2));
}

/// Adds, with saturation, each 8-bit signed integer element of the first
///    64-bit integer vector of [8 x i8] to the corresponding 8-bit signed
///    integer element of the second 64-bit integer vector of [8 x i8].
///
///    Positive sums greater than 0x7F are saturated to 0x7F. Negative sums
///    less than 0x80 are saturated to 0x80. The results are packed into a
///    64-bit integer vector of [8 x i8].
````
- **L385 EN**: Comment explains nearby logic, constraints, or intent: `packed into a 64-bit integer vector of [2 x i32].`.
  **L385 CN**: 注释解释附近代码的逻辑、约束或设计意图：`packed into a 64-bit integer vector of [2 x i32].`。
- **L386 EN**: Separator comment used for visual grouping.
  **L386 CN**: 用于视觉分组的分隔注释。
- **L387 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L387 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L388 EN**: Separator comment used for visual grouping.
  **L388 CN**: 用于视觉分组的分隔注释。
- **L389 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PADDD </c> instruction.`.
  **L389 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PADDD </c> instruction.`。
- **L390 EN**: Separator comment used for visual grouping.
  **L390 CN**: 用于视觉分组的分隔注释。
- **L391 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L391 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L392 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L392 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L393 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L394 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L394 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L395 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the sums of both`.
  **L395 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the sums of both`。
- **L396 EN**: Comment explains nearby logic, constraints, or intent: `parameters.`.
  **L396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameters.`。
- **L397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi32(__m64 __m1,`.
  **L397 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_add_pi32(__m64 __m1,`。
- **L398 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L398 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L399 EN**: Returns from the current function with `(__m64)(((__v2su)__m1) + ((__v2su)__m2))`.
  **L399 CN**: 以 `(__m64)(((__v2su)__m1) + ((__v2su)__m2))` 从当前函数返回。
- **L400 EN**: Closes the current lexical scope or compound statement.
  **L400 CN**: 结束当前词法作用域或复合语句块。
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L402 EN**: Comment explains nearby logic, constraints, or intent: `Adds, with saturation, each 8-bit signed integer element of the first`.
  **L402 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Adds, with saturation, each 8-bit signed integer element of the first`。
- **L403 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [8 x i8] to the corresponding 8-bit signed`.
  **L403 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [8 x i8] to the corresponding 8-bit signed`。
- **L404 EN**: Comment explains nearby logic, constraints, or intent: `integer element of the second 64-bit integer vector of [8 x i8].`.
  **L404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer element of the second 64-bit integer vector of [8 x i8].`。
- **L405 EN**: Separator comment used for visual grouping.
  **L405 CN**: 用于视觉分组的分隔注释。
- **L406 EN**: Comment explains nearby logic, constraints, or intent: `Positive sums greater than 0x7F are saturated to 0x7F. Negative sums`.
  **L406 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Positive sums greater than 0x7F are saturated to 0x7F. Negative sums`。
- **L407 EN**: Comment explains nearby logic, constraints, or intent: `less than 0x80 are saturated to 0x80. The results are packed into a`.
  **L407 CN**: 注释解释附近代码的逻辑、约束或设计意图：`less than 0x80 are saturated to 0x80. The results are packed into a`。
- **L408 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [8 x i8].`.
  **L408 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [8 x i8].`。

### Lines 409-432

````c
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PADDSB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8].
/// \param __m2
///    A 64-bit integer vector of [8 x i8].
/// \returns A 64-bit integer vector of [8 x i8] containing the saturated sums
///    of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pi8(__m64 __m1,
                                                             __m64 __m2) {
  return (__m64)__builtin_elementwise_add_sat((__v8qs)__m1, (__v8qs)__m2);
}

/// Adds, with saturation, each 16-bit signed integer element of the first
///    64-bit integer vector of [4 x i16] to the corresponding 16-bit signed
///    integer element of the second 64-bit integer vector of [4 x i16].
///
///    Positive sums greater than 0x7FFF are saturated to 0x7FFF. Negative sums
///    less than 0x8000 are saturated to 0x8000. The results are packed into a
///    64-bit integer vector of [4 x i16].
///
````
- **L409 EN**: Separator comment used for visual grouping.
  **L409 CN**: 用于视觉分组的分隔注释。
- **L410 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L410 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L411 EN**: Separator comment used for visual grouping.
  **L411 CN**: 用于视觉分组的分隔注释。
- **L412 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PADDSB </c> instruction.`.
  **L412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PADDSB </c> instruction.`。
- **L413 EN**: Separator comment used for visual grouping.
  **L413 CN**: 用于视觉分组的分隔注释。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L415 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L415 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L416 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L416 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L417 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L418 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the saturated sums`.
  **L418 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the saturated sums`。
- **L419 EN**: Comment explains nearby logic, constraints, or intent: `of both parameters.`.
  **L419 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of both parameters.`。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pi8(__m64 __m1,`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pi8(__m64 __m1,`。
- **L421 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L421 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L422 EN**: Returns from the current function with `(__m64)__builtin_elementwise_add_sat((__v8qs)__m1, (__v8qs)__m2)`.
  **L422 CN**: 以 `(__m64)__builtin_elementwise_add_sat((__v8qs)__m1, (__v8qs)__m2)` 从当前函数返回。
- **L423 EN**: Closes the current lexical scope or compound statement.
  **L423 CN**: 结束当前词法作用域或复合语句块。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L425 EN**: Comment explains nearby logic, constraints, or intent: `Adds, with saturation, each 16-bit signed integer element of the first`.
  **L425 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Adds, with saturation, each 16-bit signed integer element of the first`。
- **L426 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [4 x i16] to the corresponding 16-bit signed`.
  **L426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [4 x i16] to the corresponding 16-bit signed`。
- **L427 EN**: Comment explains nearby logic, constraints, or intent: `integer element of the second 64-bit integer vector of [4 x i16].`.
  **L427 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer element of the second 64-bit integer vector of [4 x i16].`。
- **L428 EN**: Separator comment used for visual grouping.
  **L428 CN**: 用于视觉分组的分隔注释。
- **L429 EN**: Comment explains nearby logic, constraints, or intent: `Positive sums greater than 0x7FFF are saturated to 0x7FFF. Negative sums`.
  **L429 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Positive sums greater than 0x7FFF are saturated to 0x7FFF. Negative sums`。
- **L430 EN**: Comment explains nearby logic, constraints, or intent: `less than 0x8000 are saturated to 0x8000. The results are packed into a`.
  **L430 CN**: 注释解释附近代码的逻辑、约束或设计意图：`less than 0x8000 are saturated to 0x8000. The results are packed into a`。
- **L431 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [4 x i16].`.
  **L431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [4 x i16].`。
- **L432 EN**: Separator comment used for visual grouping.
  **L432 CN**: 用于视觉分组的分隔注释。

### Lines 433-456

````c
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PADDSW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
/// \returns A 64-bit integer vector of [4 x i16] containing the saturated sums
///    of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pi16(__m64 __m1,
                                                              __m64 __m2) {
  return (__m64)__builtin_elementwise_add_sat((__v4hi)__m1, (__v4hi)__m2);
}

/// Adds, with saturation, each 8-bit unsigned integer element of the first
///    64-bit integer vector of [8 x i8] to the corresponding 8-bit unsigned
///    integer element of the second 64-bit integer vector of [8 x i8].
///
///    Sums greater than 0xFF are saturated to 0xFF. The results are packed
///    into a 64-bit integer vector of [8 x i8].
///
/// \headerfile <x86intrin.h>
///
````
- **L433 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L434 EN**: Separator comment used for visual grouping.
  **L434 CN**: 用于视觉分组的分隔注释。
- **L435 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PADDSW </c> instruction.`.
  **L435 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PADDSW </c> instruction.`。
- **L436 EN**: Separator comment used for visual grouping.
  **L436 CN**: 用于视觉分组的分隔注释。
- **L437 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L439 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L440 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L441 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the saturated sums`.
  **L441 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the saturated sums`。
- **L442 EN**: Comment explains nearby logic, constraints, or intent: `of both parameters.`.
  **L442 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of both parameters.`。
- **L443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pi16(__m64 __m1,`.
  **L443 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pi16(__m64 __m1,`。
- **L444 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L444 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L445 EN**: Returns from the current function with `(__m64)__builtin_elementwise_add_sat((__v4hi)__m1, (__v4hi)__m2)`.
  **L445 CN**: 以 `(__m64)__builtin_elementwise_add_sat((__v4hi)__m1, (__v4hi)__m2)` 从当前函数返回。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Comment explains nearby logic, constraints, or intent: `Adds, with saturation, each 8-bit unsigned integer element of the first`.
  **L448 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Adds, with saturation, each 8-bit unsigned integer element of the first`。
- **L449 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [8 x i8] to the corresponding 8-bit unsigned`.
  **L449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [8 x i8] to the corresponding 8-bit unsigned`。
- **L450 EN**: Comment explains nearby logic, constraints, or intent: `integer element of the second 64-bit integer vector of [8 x i8].`.
  **L450 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer element of the second 64-bit integer vector of [8 x i8].`。
- **L451 EN**: Separator comment used for visual grouping.
  **L451 CN**: 用于视觉分组的分隔注释。
- **L452 EN**: Comment explains nearby logic, constraints, or intent: `Sums greater than 0xFF are saturated to 0xFF. The results are packed`.
  **L452 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sums greater than 0xFF are saturated to 0xFF. The results are packed`。
- **L453 EN**: Comment explains nearby logic, constraints, or intent: `into a 64-bit integer vector of [8 x i8].`.
  **L453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`into a 64-bit integer vector of [8 x i8].`。
- **L454 EN**: Separator comment used for visual grouping.
  **L454 CN**: 用于视觉分组的分隔注释。
- **L455 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L455 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L456 EN**: Separator comment used for visual grouping.
  **L456 CN**: 用于视觉分组的分隔注释。

### Lines 457-480

````c
/// This intrinsic corresponds to the <c> PADDUSB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8].
/// \param __m2
///    A 64-bit integer vector of [8 x i8].
/// \returns A 64-bit integer vector of [8 x i8] containing the saturated
///    unsigned sums of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pu8(__m64 __m1,
                                                             __m64 __m2) {
  return (__m64)__builtin_elementwise_add_sat((__v8qu)__m1, (__v8qu)__m2);
}

/// Adds, with saturation, each 16-bit unsigned integer element of the first
///    64-bit integer vector of [4 x i16] to the corresponding 16-bit unsigned
///    integer element of the second 64-bit integer vector of [4 x i16].
///
///    Sums greater than 0xFFFF are saturated to 0xFFFF. The results are packed
///    into a 64-bit integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PADDUSW </c> instruction.
///
````
- **L457 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PADDUSB </c> instruction.`.
  **L457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PADDUSB </c> instruction.`。
- **L458 EN**: Separator comment used for visual grouping.
  **L458 CN**: 用于视觉分组的分隔注释。
- **L459 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L459 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L460 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L461 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L461 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L462 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L463 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the saturated`.
  **L463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the saturated`。
- **L464 EN**: Comment explains nearby logic, constraints, or intent: `unsigned sums of both parameters.`.
  **L464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`unsigned sums of both parameters.`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pu8(__m64 __m1,`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pu8(__m64 __m1,`。
- **L466 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L466 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L467 EN**: Returns from the current function with `(__m64)__builtin_elementwise_add_sat((__v8qu)__m1, (__v8qu)__m2)`.
  **L467 CN**: 以 `(__m64)__builtin_elementwise_add_sat((__v8qu)__m1, (__v8qu)__m2)` 从当前函数返回。
- **L468 EN**: Closes the current lexical scope or compound statement.
  **L468 CN**: 结束当前词法作用域或复合语句块。
- **L469 EN**: Blank line separating nearby declarations or logic blocks.
  **L469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L470 EN**: Comment explains nearby logic, constraints, or intent: `Adds, with saturation, each 16-bit unsigned integer element of the first`.
  **L470 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Adds, with saturation, each 16-bit unsigned integer element of the first`。
- **L471 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [4 x i16] to the corresponding 16-bit unsigned`.
  **L471 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [4 x i16] to the corresponding 16-bit unsigned`。
- **L472 EN**: Comment explains nearby logic, constraints, or intent: `integer element of the second 64-bit integer vector of [4 x i16].`.
  **L472 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer element of the second 64-bit integer vector of [4 x i16].`。
- **L473 EN**: Separator comment used for visual grouping.
  **L473 CN**: 用于视觉分组的分隔注释。
- **L474 EN**: Comment explains nearby logic, constraints, or intent: `Sums greater than 0xFFFF are saturated to 0xFFFF. The results are packed`.
  **L474 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sums greater than 0xFFFF are saturated to 0xFFFF. The results are packed`。
- **L475 EN**: Comment explains nearby logic, constraints, or intent: `into a 64-bit integer vector of [4 x i16].`.
  **L475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`into a 64-bit integer vector of [4 x i16].`。
- **L476 EN**: Separator comment used for visual grouping.
  **L476 CN**: 用于视觉分组的分隔注释。
- **L477 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L477 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L478 EN**: Separator comment used for visual grouping.
  **L478 CN**: 用于视觉分组的分隔注释。
- **L479 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PADDUSW </c> instruction.`.
  **L479 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PADDUSW </c> instruction.`。
- **L480 EN**: Separator comment used for visual grouping.
  **L480 CN**: 用于视觉分组的分隔注释。

### Lines 481-504

````c
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
/// \returns A 64-bit integer vector of [4 x i16] containing the saturated
///    unsigned sums of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pu16(__m64 __m1,
                                                              __m64 __m2) {
  return (__m64)__builtin_elementwise_add_sat((__v4hu)__m1, (__v4hu)__m2);
}

/// Subtracts each 8-bit integer element of the second 64-bit integer
///    vector of [8 x i8] from the corresponding 8-bit integer element of the
///    first 64-bit integer vector of [8 x i8]. The lower 8 bits of the results
///    are packed into a 64-bit integer vector of [8 x i8].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSUBB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8] containing the minuends.
/// \param __m2
///    A 64-bit integer vector of [8 x i8] containing the subtrahends.
````
- **L481 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L481 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L482 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L482 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L483 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L483 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L484 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L484 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L485 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the saturated`.
  **L485 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the saturated`。
- **L486 EN**: Comment explains nearby logic, constraints, or intent: `unsigned sums of both parameters.`.
  **L486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`unsigned sums of both parameters.`。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pu16(__m64 __m1,`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_adds_pu16(__m64 __m1,`。
- **L488 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L488 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L489 EN**: Returns from the current function with `(__m64)__builtin_elementwise_add_sat((__v4hu)__m1, (__v4hu)__m2)`.
  **L489 CN**: 以 `(__m64)__builtin_elementwise_add_sat((__v4hu)__m1, (__v4hu)__m2)` 从当前函数返回。
- **L490 EN**: Closes the current lexical scope or compound statement.
  **L490 CN**: 结束当前词法作用域或复合语句块。
- **L491 EN**: Blank line separating nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L492 EN**: Comment explains nearby logic, constraints, or intent: `Subtracts each 8-bit integer element of the second 64-bit integer`.
  **L492 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtracts each 8-bit integer element of the second 64-bit integer`。
- **L493 EN**: Comment explains nearby logic, constraints, or intent: `vector of [8 x i8] from the corresponding 8-bit integer element of the`.
  **L493 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [8 x i8] from the corresponding 8-bit integer element of the`。
- **L494 EN**: Comment explains nearby logic, constraints, or intent: `first 64-bit integer vector of [8 x i8]. The lower 8 bits of the results`.
  **L494 CN**: 注释解释附近代码的逻辑、约束或设计意图：`first 64-bit integer vector of [8 x i8]. The lower 8 bits of the results`。
- **L495 EN**: Comment explains nearby logic, constraints, or intent: `are packed into a 64-bit integer vector of [8 x i8].`.
  **L495 CN**: 注释解释附近代码的逻辑、约束或设计意图：`are packed into a 64-bit integer vector of [8 x i8].`。
- **L496 EN**: Separator comment used for visual grouping.
  **L496 CN**: 用于视觉分组的分隔注释。
- **L497 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L497 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L498 EN**: Separator comment used for visual grouping.
  **L498 CN**: 用于视觉分组的分隔注释。
- **L499 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSUBB </c> instruction.`.
  **L499 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSUBB </c> instruction.`。
- **L500 EN**: Separator comment used for visual grouping.
  **L500 CN**: 用于视觉分组的分隔注释。
- **L501 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L501 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L502 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8] containing the minuends.`.
  **L502 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8] containing the minuends.`。
- **L503 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L503 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L504 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8] containing the subtrahends.`.
  **L504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8] containing the subtrahends.`。

### Lines 505-528

````c
/// \returns A 64-bit integer vector of [8 x i8] containing the differences of
///    both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi8(__m64 __m1,
                                                            __m64 __m2) {
  return (__m64)(((__v8qu)__m1) - ((__v8qu)__m2));
}

/// Subtracts each 16-bit integer element of the second 64-bit integer
///    vector of [4 x i16] from the corresponding 16-bit integer element of the
///    first 64-bit integer vector of [4 x i16]. The lower 16 bits of the
///    results are packed into a 64-bit integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSUBW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16] containing the minuends.
/// \param __m2
///    A 64-bit integer vector of [4 x i16] containing the subtrahends.
/// \returns A 64-bit integer vector of [4 x i16] containing the differences of
///    both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi16(__m64 __m1,
                                                             __m64 __m2) {
````
- **L505 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the differences of`.
  **L505 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the differences of`。
- **L506 EN**: Comment explains nearby logic, constraints, or intent: `both parameters.`.
  **L506 CN**: 注释解释附近代码的逻辑、约束或设计意图：`both parameters.`。
- **L507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi8(__m64 __m1,`.
  **L507 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi8(__m64 __m1,`。
- **L508 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L508 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L509 EN**: Returns from the current function with `(__m64)(((__v8qu)__m1) - ((__v8qu)__m2))`.
  **L509 CN**: 以 `(__m64)(((__v8qu)__m1) - ((__v8qu)__m2))` 从当前函数返回。
- **L510 EN**: Closes the current lexical scope or compound statement.
  **L510 CN**: 结束当前词法作用域或复合语句块。
- **L511 EN**: Blank line separating nearby declarations or logic blocks.
  **L511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L512 EN**: Comment explains nearby logic, constraints, or intent: `Subtracts each 16-bit integer element of the second 64-bit integer`.
  **L512 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtracts each 16-bit integer element of the second 64-bit integer`。
- **L513 EN**: Comment explains nearby logic, constraints, or intent: `vector of [4 x i16] from the corresponding 16-bit integer element of the`.
  **L513 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [4 x i16] from the corresponding 16-bit integer element of the`。
- **L514 EN**: Comment explains nearby logic, constraints, or intent: `first 64-bit integer vector of [4 x i16]. The lower 16 bits of the`.
  **L514 CN**: 注释解释附近代码的逻辑、约束或设计意图：`first 64-bit integer vector of [4 x i16]. The lower 16 bits of the`。
- **L515 EN**: Comment explains nearby logic, constraints, or intent: `results are packed into a 64-bit integer vector of [4 x i16].`.
  **L515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results are packed into a 64-bit integer vector of [4 x i16].`。
- **L516 EN**: Separator comment used for visual grouping.
  **L516 CN**: 用于视觉分组的分隔注释。
- **L517 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L518 EN**: Separator comment used for visual grouping.
  **L518 CN**: 用于视觉分组的分隔注释。
- **L519 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSUBW </c> instruction.`.
  **L519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSUBW </c> instruction.`。
- **L520 EN**: Separator comment used for visual grouping.
  **L520 CN**: 用于视觉分组的分隔注释。
- **L521 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L521 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L522 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16] containing the minuends.`.
  **L522 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16] containing the minuends.`。
- **L523 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L523 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L524 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16] containing the subtrahends.`.
  **L524 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16] containing the subtrahends.`。
- **L525 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the differences of`.
  **L525 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the differences of`。
- **L526 EN**: Comment explains nearby logic, constraints, or intent: `both parameters.`.
  **L526 CN**: 注释解释附近代码的逻辑、约束或设计意图：`both parameters.`。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi16(__m64 __m1,`.
  **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi16(__m64 __m1,`。
- **L528 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L528 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。

### Lines 529-552

````c
  return (__m64)(((__v4hu)__m1) - ((__v4hu)__m2));
}

/// Subtracts each 32-bit integer element of the second 64-bit integer
///    vector of [2 x i32] from the corresponding 32-bit integer element of the
///    first 64-bit integer vector of [2 x i32]. The lower 32 bits of the
///    results are packed into a 64-bit integer vector of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSUBD </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [2 x i32] containing the minuends.
/// \param __m2
///    A 64-bit integer vector of [2 x i32] containing the subtrahends.
/// \returns A 64-bit integer vector of [2 x i32] containing the differences of
///    both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi32(__m64 __m1,
                                                             __m64 __m2) {
  return (__m64)(((__v2su)__m1) - ((__v2su)__m2));
}

/// Subtracts, with saturation, each 8-bit signed integer element of the second
````
- **L529 EN**: Returns from the current function with `(__m64)(((__v4hu)__m1) - ((__v4hu)__m2))`.
  **L529 CN**: 以 `(__m64)(((__v4hu)__m1) - ((__v4hu)__m2))` 从当前函数返回。
- **L530 EN**: Closes the current lexical scope or compound statement.
  **L530 CN**: 结束当前词法作用域或复合语句块。
- **L531 EN**: Blank line separating nearby declarations or logic blocks.
  **L531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L532 EN**: Comment explains nearby logic, constraints, or intent: `Subtracts each 32-bit integer element of the second 64-bit integer`.
  **L532 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtracts each 32-bit integer element of the second 64-bit integer`。
- **L533 EN**: Comment explains nearby logic, constraints, or intent: `vector of [2 x i32] from the corresponding 32-bit integer element of the`.
  **L533 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [2 x i32] from the corresponding 32-bit integer element of the`。
- **L534 EN**: Comment explains nearby logic, constraints, or intent: `first 64-bit integer vector of [2 x i32]. The lower 32 bits of the`.
  **L534 CN**: 注释解释附近代码的逻辑、约束或设计意图：`first 64-bit integer vector of [2 x i32]. The lower 32 bits of the`。
- **L535 EN**: Comment explains nearby logic, constraints, or intent: `results are packed into a 64-bit integer vector of [2 x i32].`.
  **L535 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results are packed into a 64-bit integer vector of [2 x i32].`。
- **L536 EN**: Separator comment used for visual grouping.
  **L536 CN**: 用于视觉分组的分隔注释。
- **L537 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L537 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L538 EN**: Separator comment used for visual grouping.
  **L538 CN**: 用于视觉分组的分隔注释。
- **L539 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSUBD </c> instruction.`.
  **L539 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSUBD </c> instruction.`。
- **L540 EN**: Separator comment used for visual grouping.
  **L540 CN**: 用于视觉分组的分隔注释。
- **L541 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L541 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L542 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32] containing the minuends.`.
  **L542 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32] containing the minuends.`。
- **L543 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L543 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L544 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32] containing the subtrahends.`.
  **L544 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32] containing the subtrahends.`。
- **L545 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the differences of`.
  **L545 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the differences of`。
- **L546 EN**: Comment explains nearby logic, constraints, or intent: `both parameters.`.
  **L546 CN**: 注释解释附近代码的逻辑、约束或设计意图：`both parameters.`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi32(__m64 __m1,`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_sub_pi32(__m64 __m1,`。
- **L548 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L548 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L549 EN**: Returns from the current function with `(__m64)(((__v2su)__m1) - ((__v2su)__m2))`.
  **L549 CN**: 以 `(__m64)(((__v2su)__m1) - ((__v2su)__m2))` 从当前函数返回。
- **L550 EN**: Closes the current lexical scope or compound statement.
  **L550 CN**: 结束当前词法作用域或复合语句块。
- **L551 EN**: Blank line separating nearby declarations or logic blocks.
  **L551 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L552 EN**: Comment explains nearby logic, constraints, or intent: `Subtracts, with saturation, each 8-bit signed integer element of the second`.
  **L552 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtracts, with saturation, each 8-bit signed integer element of the second`。

### Lines 553-576

````c
///    64-bit integer vector of [8 x i8] from the corresponding 8-bit signed
///    integer element of the first 64-bit integer vector of [8 x i8].
///
///    Positive results greater than 0x7F are saturated to 0x7F. Negative
///    results less than 0x80 are saturated to 0x80. The results are packed
///    into a 64-bit integer vector of [8 x i8].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSUBSB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8] containing the minuends.
/// \param __m2
///    A 64-bit integer vector of [8 x i8] containing the subtrahends.
/// \returns A 64-bit integer vector of [8 x i8] containing the saturated
///    differences of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pi8(__m64 __m1,
                                                             __m64 __m2) {
  return (__m64)__builtin_elementwise_sub_sat((__v8qs)__m1, (__v8qs)__m2);
}

/// Subtracts, with saturation, each 16-bit signed integer element of the
///    second 64-bit integer vector of [4 x i16] from the corresponding 16-bit
````
- **L553 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer vector of [8 x i8] from the corresponding 8-bit signed`.
  **L553 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer vector of [8 x i8] from the corresponding 8-bit signed`。
- **L554 EN**: Comment explains nearby logic, constraints, or intent: `integer element of the first 64-bit integer vector of [8 x i8].`.
  **L554 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer element of the first 64-bit integer vector of [8 x i8].`。
- **L555 EN**: Separator comment used for visual grouping.
  **L555 CN**: 用于视觉分组的分隔注释。
- **L556 EN**: Comment explains nearby logic, constraints, or intent: `Positive results greater than 0x7F are saturated to 0x7F. Negative`.
  **L556 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Positive results greater than 0x7F are saturated to 0x7F. Negative`。
- **L557 EN**: Comment explains nearby logic, constraints, or intent: `results less than 0x80 are saturated to 0x80. The results are packed`.
  **L557 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results less than 0x80 are saturated to 0x80. The results are packed`。
- **L558 EN**: Comment explains nearby logic, constraints, or intent: `into a 64-bit integer vector of [8 x i8].`.
  **L558 CN**: 注释解释附近代码的逻辑、约束或设计意图：`into a 64-bit integer vector of [8 x i8].`。
- **L559 EN**: Separator comment used for visual grouping.
  **L559 CN**: 用于视觉分组的分隔注释。
- **L560 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L560 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L561 EN**: Separator comment used for visual grouping.
  **L561 CN**: 用于视觉分组的分隔注释。
- **L562 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSUBSB </c> instruction.`.
  **L562 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSUBSB </c> instruction.`。
- **L563 EN**: Separator comment used for visual grouping.
  **L563 CN**: 用于视觉分组的分隔注释。
- **L564 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L564 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L565 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8] containing the minuends.`.
  **L565 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8] containing the minuends.`。
- **L566 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L566 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L567 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8] containing the subtrahends.`.
  **L567 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8] containing the subtrahends.`。
- **L568 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the saturated`.
  **L568 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the saturated`。
- **L569 EN**: Comment explains nearby logic, constraints, or intent: `differences of both parameters.`.
  **L569 CN**: 注释解释附近代码的逻辑、约束或设计意图：`differences of both parameters.`。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pi8(__m64 __m1,`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pi8(__m64 __m1,`。
- **L571 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L571 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L572 EN**: Returns from the current function with `(__m64)__builtin_elementwise_sub_sat((__v8qs)__m1, (__v8qs)__m2)`.
  **L572 CN**: 以 `(__m64)__builtin_elementwise_sub_sat((__v8qs)__m1, (__v8qs)__m2)` 从当前函数返回。
- **L573 EN**: Closes the current lexical scope or compound statement.
  **L573 CN**: 结束当前词法作用域或复合语句块。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Comment explains nearby logic, constraints, or intent: `Subtracts, with saturation, each 16-bit signed integer element of the`.
  **L575 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtracts, with saturation, each 16-bit signed integer element of the`。
- **L576 EN**: Comment explains nearby logic, constraints, or intent: `second 64-bit integer vector of [4 x i16] from the corresponding 16-bit`.
  **L576 CN**: 注释解释附近代码的逻辑、约束或设计意图：`second 64-bit integer vector of [4 x i16] from the corresponding 16-bit`。

### Lines 577-600

````c
///    signed integer element of the first 64-bit integer vector of [4 x i16].
///
///    Positive results greater than 0x7FFF are saturated to 0x7FFF. Negative
///    results less than 0x8000 are saturated to 0x8000. The results are packed
///    into a 64-bit integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSUBSW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16] containing the minuends.
/// \param __m2
///    A 64-bit integer vector of [4 x i16] containing the subtrahends.
/// \returns A 64-bit integer vector of [4 x i16] containing the saturated
///    differences of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pi16(__m64 __m1,
                                                              __m64 __m2) {
  return (__m64)__builtin_elementwise_sub_sat((__v4hi)__m1, (__v4hi)__m2);
}

/// Subtracts each 8-bit unsigned integer element of the second 64-bit
///    integer vector of [8 x i8] from the corresponding 8-bit unsigned integer
///    element of the first 64-bit integer vector of [8 x i8].
````
- **L577 EN**: Comment explains nearby logic, constraints, or intent: `signed integer element of the first 64-bit integer vector of [4 x i16].`.
  **L577 CN**: 注释解释附近代码的逻辑、约束或设计意图：`signed integer element of the first 64-bit integer vector of [4 x i16].`。
- **L578 EN**: Separator comment used for visual grouping.
  **L578 CN**: 用于视觉分组的分隔注释。
- **L579 EN**: Comment explains nearby logic, constraints, or intent: `Positive results greater than 0x7FFF are saturated to 0x7FFF. Negative`.
  **L579 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Positive results greater than 0x7FFF are saturated to 0x7FFF. Negative`。
- **L580 EN**: Comment explains nearby logic, constraints, or intent: `results less than 0x8000 are saturated to 0x8000. The results are packed`.
  **L580 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results less than 0x8000 are saturated to 0x8000. The results are packed`。
- **L581 EN**: Comment explains nearby logic, constraints, or intent: `into a 64-bit integer vector of [4 x i16].`.
  **L581 CN**: 注释解释附近代码的逻辑、约束或设计意图：`into a 64-bit integer vector of [4 x i16].`。
- **L582 EN**: Separator comment used for visual grouping.
  **L582 CN**: 用于视觉分组的分隔注释。
- **L583 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L583 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L584 EN**: Separator comment used for visual grouping.
  **L584 CN**: 用于视觉分组的分隔注释。
- **L585 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSUBSW </c> instruction.`.
  **L585 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSUBSW </c> instruction.`。
- **L586 EN**: Separator comment used for visual grouping.
  **L586 CN**: 用于视觉分组的分隔注释。
- **L587 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L587 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L588 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16] containing the minuends.`.
  **L588 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16] containing the minuends.`。
- **L589 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L589 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L590 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16] containing the subtrahends.`.
  **L590 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16] containing the subtrahends.`。
- **L591 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the saturated`.
  **L591 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the saturated`。
- **L592 EN**: Comment explains nearby logic, constraints, or intent: `differences of both parameters.`.
  **L592 CN**: 注释解释附近代码的逻辑、约束或设计意图：`differences of both parameters.`。
- **L593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pi16(__m64 __m1,`.
  **L593 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pi16(__m64 __m1,`。
- **L594 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L594 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L595 EN**: Returns from the current function with `(__m64)__builtin_elementwise_sub_sat((__v4hi)__m1, (__v4hi)__m2)`.
  **L595 CN**: 以 `(__m64)__builtin_elementwise_sub_sat((__v4hi)__m1, (__v4hi)__m2)` 从当前函数返回。
- **L596 EN**: Closes the current lexical scope or compound statement.
  **L596 CN**: 结束当前词法作用域或复合语句块。
- **L597 EN**: Blank line separating nearby declarations or logic blocks.
  **L597 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L598 EN**: Comment explains nearby logic, constraints, or intent: `Subtracts each 8-bit unsigned integer element of the second 64-bit`.
  **L598 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtracts each 8-bit unsigned integer element of the second 64-bit`。
- **L599 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [8 x i8] from the corresponding 8-bit unsigned integer`.
  **L599 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [8 x i8] from the corresponding 8-bit unsigned integer`。
- **L600 EN**: Comment explains nearby logic, constraints, or intent: `element of the first 64-bit integer vector of [8 x i8].`.
  **L600 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element of the first 64-bit integer vector of [8 x i8].`。

### Lines 601-624

````c
///
///    If an element of the first vector is less than the corresponding element
///    of the second vector, the result is saturated to 0. The results are
///    packed into a 64-bit integer vector of [8 x i8].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSUBUSB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8] containing the minuends.
/// \param __m2
///    A 64-bit integer vector of [8 x i8] containing the subtrahends.
/// \returns A 64-bit integer vector of [8 x i8] containing the saturated
///    differences of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pu8(__m64 __m1,
                                                             __m64 __m2) {
  return (__m64)__builtin_elementwise_sub_sat((__v8qu)__m1, (__v8qu)__m2);
}

/// Subtracts each 16-bit unsigned integer element of the second 64-bit
///    integer vector of [4 x i16] from the corresponding 16-bit unsigned
///    integer element of the first 64-bit integer vector of [4 x i16].
///
````
- **L601 EN**: Separator comment used for visual grouping.
  **L601 CN**: 用于视觉分组的分隔注释。
- **L602 EN**: Comment explains nearby logic, constraints, or intent: `If an element of the first vector is less than the corresponding element`.
  **L602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If an element of the first vector is less than the corresponding element`。
- **L603 EN**: Comment explains nearby logic, constraints, or intent: `of the second vector, the result is saturated to 0. The results are`.
  **L603 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the second vector, the result is saturated to 0. The results are`。
- **L604 EN**: Comment explains nearby logic, constraints, or intent: `packed into a 64-bit integer vector of [8 x i8].`.
  **L604 CN**: 注释解释附近代码的逻辑、约束或设计意图：`packed into a 64-bit integer vector of [8 x i8].`。
- **L605 EN**: Separator comment used for visual grouping.
  **L605 CN**: 用于视觉分组的分隔注释。
- **L606 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L606 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L607 EN**: Separator comment used for visual grouping.
  **L607 CN**: 用于视觉分组的分隔注释。
- **L608 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSUBUSB </c> instruction.`.
  **L608 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSUBUSB </c> instruction.`。
- **L609 EN**: Separator comment used for visual grouping.
  **L609 CN**: 用于视觉分组的分隔注释。
- **L610 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L610 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L611 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8] containing the minuends.`.
  **L611 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8] containing the minuends.`。
- **L612 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L613 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8] containing the subtrahends.`.
  **L613 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8] containing the subtrahends.`。
- **L614 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the saturated`.
  **L614 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the saturated`。
- **L615 EN**: Comment explains nearby logic, constraints, or intent: `differences of both parameters.`.
  **L615 CN**: 注释解释附近代码的逻辑、约束或设计意图：`differences of both parameters.`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pu8(__m64 __m1,`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pu8(__m64 __m1,`。
- **L617 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L617 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L618 EN**: Returns from the current function with `(__m64)__builtin_elementwise_sub_sat((__v8qu)__m1, (__v8qu)__m2)`.
  **L618 CN**: 以 `(__m64)__builtin_elementwise_sub_sat((__v8qu)__m1, (__v8qu)__m2)` 从当前函数返回。
- **L619 EN**: Closes the current lexical scope or compound statement.
  **L619 CN**: 结束当前词法作用域或复合语句块。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Comment explains nearby logic, constraints, or intent: `Subtracts each 16-bit unsigned integer element of the second 64-bit`.
  **L621 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtracts each 16-bit unsigned integer element of the second 64-bit`。
- **L622 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [4 x i16] from the corresponding 16-bit unsigned`.
  **L622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [4 x i16] from the corresponding 16-bit unsigned`。
- **L623 EN**: Comment explains nearby logic, constraints, or intent: `integer element of the first 64-bit integer vector of [4 x i16].`.
  **L623 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer element of the first 64-bit integer vector of [4 x i16].`。
- **L624 EN**: Separator comment used for visual grouping.
  **L624 CN**: 用于视觉分组的分隔注释。

### Lines 625-648

````c
///    If an element of the first vector is less than the corresponding element
///    of the second vector, the result is saturated to 0. The results are
///    packed into a 64-bit integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSUBUSW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16] containing the minuends.
/// \param __m2
///    A 64-bit integer vector of [4 x i16] containing the subtrahends.
/// \returns A 64-bit integer vector of [4 x i16] containing the saturated
///    differences of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pu16(__m64 __m1,
                                                              __m64 __m2) {
  return (__m64)__builtin_elementwise_sub_sat((__v4hu)__m1, (__v4hu)__m2);
}

/// Multiplies each 16-bit signed integer element of the first 64-bit
///    integer vector of [4 x i16] by the corresponding 16-bit signed integer
///    element of the second 64-bit integer vector of [4 x i16] and get four
///    32-bit products. Adds adjacent pairs of products to get two 32-bit sums.
///    The lower 32 bits of these two sums are packed into a 64-bit integer
````
- **L625 EN**: Comment explains nearby logic, constraints, or intent: `If an element of the first vector is less than the corresponding element`.
  **L625 CN**: 注释解释附近代码的逻辑、约束或设计意图：`If an element of the first vector is less than the corresponding element`。
- **L626 EN**: Comment explains nearby logic, constraints, or intent: `of the second vector, the result is saturated to 0. The results are`.
  **L626 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the second vector, the result is saturated to 0. The results are`。
- **L627 EN**: Comment explains nearby logic, constraints, or intent: `packed into a 64-bit integer vector of [4 x i16].`.
  **L627 CN**: 注释解释附近代码的逻辑、约束或设计意图：`packed into a 64-bit integer vector of [4 x i16].`。
- **L628 EN**: Separator comment used for visual grouping.
  **L628 CN**: 用于视觉分组的分隔注释。
- **L629 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L629 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L630 EN**: Separator comment used for visual grouping.
  **L630 CN**: 用于视觉分组的分隔注释。
- **L631 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSUBUSW </c> instruction.`.
  **L631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSUBUSW </c> instruction.`。
- **L632 EN**: Separator comment used for visual grouping.
  **L632 CN**: 用于视觉分组的分隔注释。
- **L633 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L633 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L634 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16] containing the minuends.`.
  **L634 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16] containing the minuends.`。
- **L635 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L635 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L636 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16] containing the subtrahends.`.
  **L636 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16] containing the subtrahends.`。
- **L637 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the saturated`.
  **L637 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the saturated`。
- **L638 EN**: Comment explains nearby logic, constraints, or intent: `differences of both parameters.`.
  **L638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`differences of both parameters.`。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pu16(__m64 __m1,`.
  **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_subs_pu16(__m64 __m1,`。
- **L640 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L640 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L641 EN**: Returns from the current function with `(__m64)__builtin_elementwise_sub_sat((__v4hu)__m1, (__v4hu)__m2)`.
  **L641 CN**: 以 `(__m64)__builtin_elementwise_sub_sat((__v4hu)__m1, (__v4hu)__m2)` 从当前函数返回。
- **L642 EN**: Closes the current lexical scope or compound statement.
  **L642 CN**: 结束当前词法作用域或复合语句块。
- **L643 EN**: Blank line separating nearby declarations or logic blocks.
  **L643 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L644 EN**: Comment explains nearby logic, constraints, or intent: `Multiplies each 16-bit signed integer element of the first 64-bit`.
  **L644 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplies each 16-bit signed integer element of the first 64-bit`。
- **L645 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [4 x i16] by the corresponding 16-bit signed integer`.
  **L645 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [4 x i16] by the corresponding 16-bit signed integer`。
- **L646 EN**: Comment explains nearby logic, constraints, or intent: `element of the second 64-bit integer vector of [4 x i16] and get four`.
  **L646 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element of the second 64-bit integer vector of [4 x i16] and get four`。
- **L647 EN**: Comment explains nearby logic, constraints, or intent: `32-bit products. Adds adjacent pairs of products to get two 32-bit sums.`.
  **L647 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit products. Adds adjacent pairs of products to get two 32-bit sums.`。
- **L648 EN**: Comment explains nearby logic, constraints, or intent: `The lower 32 bits of these two sums are packed into a 64-bit integer`.
  **L648 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The lower 32 bits of these two sums are packed into a 64-bit integer`。

### Lines 649-672

````c
///    vector of [2 x i32].
///
///    For example, bits [15:0] of both parameters are multiplied, bits [31:16]
///    of both parameters are multiplied, and the sum of both results is written
///    to bits [31:0] of the result.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PMADDWD </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
/// \returns A 64-bit integer vector of [2 x i32] containing the sums of
///    products of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_madd_pi16(__m64 __m1,
                                                              __m64 __m2) {
  return __trunc64(__builtin_ia32_pmaddwd128((__v8hi)__zext128(__m1),
                                             (__v8hi)__zext128(__m2)));
}

/// Multiplies each 16-bit signed integer element of the first 64-bit
///    integer vector of [4 x i16] by the corresponding 16-bit signed integer
````
- **L649 EN**: Comment explains nearby logic, constraints, or intent: `vector of [2 x i32].`.
  **L649 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [2 x i32].`。
- **L650 EN**: Separator comment used for visual grouping.
  **L650 CN**: 用于视觉分组的分隔注释。
- **L651 EN**: Comment explains nearby logic, constraints, or intent: `For example, bits [15:0] of both parameters are multiplied, bits [31:16]`.
  **L651 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For example, bits [15:0] of both parameters are multiplied, bits [31:16]`。
- **L652 EN**: Comment explains nearby logic, constraints, or intent: `of both parameters are multiplied, and the sum of both results is written`.
  **L652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of both parameters are multiplied, and the sum of both results is written`。
- **L653 EN**: Comment explains nearby logic, constraints, or intent: `to bits [31:0] of the result.`.
  **L653 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to bits [31:0] of the result.`。
- **L654 EN**: Separator comment used for visual grouping.
  **L654 CN**: 用于视觉分组的分隔注释。
- **L655 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L655 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L656 EN**: Separator comment used for visual grouping.
  **L656 CN**: 用于视觉分组的分隔注释。
- **L657 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PMADDWD </c> instruction.`.
  **L657 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PMADDWD </c> instruction.`。
- **L658 EN**: Separator comment used for visual grouping.
  **L658 CN**: 用于视觉分组的分隔注释。
- **L659 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L659 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L660 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L661 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L661 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L662 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L662 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L663 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the sums of`.
  **L663 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the sums of`。
- **L664 EN**: Comment explains nearby logic, constraints, or intent: `products of both parameters.`.
  **L664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`products of both parameters.`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_madd_pi16(__m64 __m1,`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_madd_pi16(__m64 __m1,`。
- **L666 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L666 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L667 EN**: Returns from the current function with `__trunc64(__builtin_ia32_pmaddwd128((__v8hi)__zext128(__m1),`.
  **L667 CN**: 以 `__trunc64(__builtin_ia32_pmaddwd128((__v8hi)__zext128(__m1),` 从当前函数返回。
- **L668 EN**: Executes a call or declaration centered on `statement`.
  **L668 CN**: 执行以 `statement` 为核心的调用或声明。
- **L669 EN**: Closes the current lexical scope or compound statement.
  **L669 CN**: 结束当前词法作用域或复合语句块。
- **L670 EN**: Blank line separating nearby declarations or logic blocks.
  **L670 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L671 EN**: Comment explains nearby logic, constraints, or intent: `Multiplies each 16-bit signed integer element of the first 64-bit`.
  **L671 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplies each 16-bit signed integer element of the first 64-bit`。
- **L672 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [4 x i16] by the corresponding 16-bit signed integer`.
  **L672 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [4 x i16] by the corresponding 16-bit signed integer`。

### Lines 673-696

````c
///    element of the second 64-bit integer vector of [4 x i16]. Packs the upper
///    16 bits of the 32-bit products into a 64-bit integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PMULHW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
/// \returns A 64-bit integer vector of [4 x i16] containing the upper 16 bits
///    of the products of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_mulhi_pi16(__m64 __m1,
                                                               __m64 __m2) {
  return __trunc64(__builtin_ia32_pmulhw128((__v8hi)__zext128(__m1),
                                            (__v8hi)__zext128(__m2)));
}

/// Multiplies each 16-bit signed integer element of the first 64-bit
///    integer vector of [4 x i16] by the corresponding 16-bit signed integer
///    element of the second 64-bit integer vector of [4 x i16]. Packs the lower
///    16 bits of the 32-bit products into a 64-bit integer vector of [4 x i16].
///
````
- **L673 EN**: Comment explains nearby logic, constraints, or intent: `element of the second 64-bit integer vector of [4 x i16]. Packs the upper`.
  **L673 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element of the second 64-bit integer vector of [4 x i16]. Packs the upper`。
- **L674 EN**: Comment explains nearby logic, constraints, or intent: `16 bits of the 32-bit products into a 64-bit integer vector of [4 x i16].`.
  **L674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16 bits of the 32-bit products into a 64-bit integer vector of [4 x i16].`。
- **L675 EN**: Separator comment used for visual grouping.
  **L675 CN**: 用于视觉分组的分隔注释。
- **L676 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L676 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L677 EN**: Separator comment used for visual grouping.
  **L677 CN**: 用于视觉分组的分隔注释。
- **L678 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PMULHW </c> instruction.`.
  **L678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PMULHW </c> instruction.`。
- **L679 EN**: Separator comment used for visual grouping.
  **L679 CN**: 用于视觉分组的分隔注释。
- **L680 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L680 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L681 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L681 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L682 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L682 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L683 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L683 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L684 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the upper 16 bits`.
  **L684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the upper 16 bits`。
- **L685 EN**: Comment explains nearby logic, constraints, or intent: `of the products of both parameters.`.
  **L685 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the products of both parameters.`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_mulhi_pi16(__m64 __m1,`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_mulhi_pi16(__m64 __m1,`。
- **L687 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L687 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L688 EN**: Returns from the current function with `__trunc64(__builtin_ia32_pmulhw128((__v8hi)__zext128(__m1),`.
  **L688 CN**: 以 `__trunc64(__builtin_ia32_pmulhw128((__v8hi)__zext128(__m1),` 从当前函数返回。
- **L689 EN**: Executes a call or declaration centered on `statement`.
  **L689 CN**: 执行以 `statement` 为核心的调用或声明。
- **L690 EN**: Closes the current lexical scope or compound statement.
  **L690 CN**: 结束当前词法作用域或复合语句块。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L692 EN**: Comment explains nearby logic, constraints, or intent: `Multiplies each 16-bit signed integer element of the first 64-bit`.
  **L692 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplies each 16-bit signed integer element of the first 64-bit`。
- **L693 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [4 x i16] by the corresponding 16-bit signed integer`.
  **L693 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [4 x i16] by the corresponding 16-bit signed integer`。
- **L694 EN**: Comment explains nearby logic, constraints, or intent: `element of the second 64-bit integer vector of [4 x i16]. Packs the lower`.
  **L694 CN**: 注释解释附近代码的逻辑、约束或设计意图：`element of the second 64-bit integer vector of [4 x i16]. Packs the lower`。
- **L695 EN**: Comment explains nearby logic, constraints, or intent: `16 bits of the 32-bit products into a 64-bit integer vector of [4 x i16].`.
  **L695 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16 bits of the 32-bit products into a 64-bit integer vector of [4 x i16].`。
- **L696 EN**: Separator comment used for visual grouping.
  **L696 CN**: 用于视觉分组的分隔注释。

### Lines 697-720

````c
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PMULLW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
/// \returns A 64-bit integer vector of [4 x i16] containing the lower 16 bits
///    of the products of both parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_mullo_pi16(__m64 __m1,
                                                               __m64 __m2) {
  return (__m64)(((__v4hu)__m1) * ((__v4hu)__m2));
}

/// Left-shifts each 16-bit signed integer element of the first
///    parameter, which is a 64-bit integer vector of [4 x i16], by the number
///    of bits specified by the second parameter, which is a 64-bit integer. The
///    lower 16 bits of the results are packed into a 64-bit integer vector of
///    [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSLLW </c> instruction.
````
- **L697 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L697 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L698 EN**: Separator comment used for visual grouping.
  **L698 CN**: 用于视觉分组的分隔注释。
- **L699 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PMULLW </c> instruction.`.
  **L699 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PMULLW </c> instruction.`。
- **L700 EN**: Separator comment used for visual grouping.
  **L700 CN**: 用于视觉分组的分隔注释。
- **L701 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L701 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L702 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L702 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L703 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L703 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L704 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L704 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L705 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the lower 16 bits`.
  **L705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the lower 16 bits`。
- **L706 EN**: Comment explains nearby logic, constraints, or intent: `of the products of both parameters.`.
  **L706 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of the products of both parameters.`。
- **L707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_mullo_pi16(__m64 __m1,`.
  **L707 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_mullo_pi16(__m64 __m1,`。
- **L708 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L708 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L709 EN**: Returns from the current function with `(__m64)(((__v4hu)__m1) * ((__v4hu)__m2))`.
  **L709 CN**: 以 `(__m64)(((__v4hu)__m1) * ((__v4hu)__m2))` 从当前函数返回。
- **L710 EN**: Closes the current lexical scope or compound statement.
  **L710 CN**: 结束当前词法作用域或复合语句块。
- **L711 EN**: Blank line separating nearby declarations or logic blocks.
  **L711 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L712 EN**: Comment explains nearby logic, constraints, or intent: `Left-shifts each 16-bit signed integer element of the first`.
  **L712 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Left-shifts each 16-bit signed integer element of the first`。
- **L713 EN**: Comment explains nearby logic, constraints, or intent: `parameter, which is a 64-bit integer vector of [4 x i16], by the number`.
  **L713 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter, which is a 64-bit integer vector of [4 x i16], by the number`。
- **L714 EN**: Comment explains nearby logic, constraints, or intent: `of bits specified by the second parameter, which is a 64-bit integer. The`.
  **L714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of bits specified by the second parameter, which is a 64-bit integer. The`。
- **L715 EN**: Comment explains nearby logic, constraints, or intent: `lower 16 bits of the results are packed into a 64-bit integer vector of`.
  **L715 CN**: 注释解释附近代码的逻辑、约束或设计意图：`lower 16 bits of the results are packed into a 64-bit integer vector of`。
- **L716 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i16].`.
  **L716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i16].`。
- **L717 EN**: Separator comment used for visual grouping.
  **L717 CN**: 用于视觉分组的分隔注释。
- **L718 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L718 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L719 EN**: Separator comment used for visual grouping.
  **L719 CN**: 用于视觉分组的分隔注释。
- **L720 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSLLW </c> instruction.`.
  **L720 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSLLW </c> instruction.`。

### Lines 721-744

````c
///
/// \param __m
///    A 64-bit integer vector of [4 x i16].
/// \param __count
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \returns A 64-bit integer vector of [4 x i16] containing the left-shifted
///    values. If \a __count is greater or equal to 16, the result is set to all
///    0.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_sll_pi16(__m64 __m, __m64 __count)
{
  return __trunc64(__builtin_ia32_psllw128((__v8hi)__zext128(__m),
                                           (__v8hi)__zext128(__count)));
}

/// Left-shifts each 16-bit signed integer element of a 64-bit integer
///    vector of [4 x i16] by the number of bits specified by a 32-bit integer.
///    The lower 16 bits of the results are packed into a 64-bit integer vector
///    of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSLLW </c> instruction.
///
````
- **L721 EN**: Separator comment used for visual grouping.
  **L721 CN**: 用于视觉分组的分隔注释。
- **L722 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L722 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L723 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L723 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L724 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L724 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L725 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L725 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L726 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the left-shifted`.
  **L726 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the left-shifted`。
- **L727 EN**: Comment explains nearby logic, constraints, or intent: `values. If a __count is greater or equal to 16, the result is set to all`.
  **L727 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values. If a __count is greater or equal to 16, the result is set to all`。
- **L728 EN**: Comment explains nearby logic, constraints, or intent: `0.`.
  **L728 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0.`。
- **L729 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L729 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L730 EN**: Continues logic associated with callable symbol `_mm_sll_pi16`.
  **L730 CN**: 继续与可调用符号 `_mm_sll_pi16` 相关的逻辑。
- **L731 EN**: Opens a new lexical scope or compound statement.
  **L731 CN**: 打开一个新的词法作用域或复合语句块。
- **L732 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psllw128((__v8hi)__zext128(__m),`.
  **L732 CN**: 以 `__trunc64(__builtin_ia32_psllw128((__v8hi)__zext128(__m),` 从当前函数返回。
- **L733 EN**: Executes a call or declaration centered on `statement`.
  **L733 CN**: 执行以 `statement` 为核心的调用或声明。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L736 EN**: Comment explains nearby logic, constraints, or intent: `Left-shifts each 16-bit signed integer element of a 64-bit integer`.
  **L736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Left-shifts each 16-bit signed integer element of a 64-bit integer`。
- **L737 EN**: Comment explains nearby logic, constraints, or intent: `vector of [4 x i16] by the number of bits specified by a 32-bit integer.`.
  **L737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [4 x i16] by the number of bits specified by a 32-bit integer.`。
- **L738 EN**: Comment explains nearby logic, constraints, or intent: `The lower 16 bits of the results are packed into a 64-bit integer vector`.
  **L738 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The lower 16 bits of the results are packed into a 64-bit integer vector`。
- **L739 EN**: Comment explains nearby logic, constraints, or intent: `of [4 x i16].`.
  **L739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [4 x i16].`。
- **L740 EN**: Separator comment used for visual grouping.
  **L740 CN**: 用于视觉分组的分隔注释。
- **L741 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L741 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L742 EN**: Separator comment used for visual grouping.
  **L742 CN**: 用于视觉分组的分隔注释。
- **L743 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSLLW </c> instruction.`.
  **L743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSLLW </c> instruction.`。
- **L744 EN**: Separator comment used for visual grouping.
  **L744 CN**: 用于视觉分组的分隔注释。

### Lines 745-768

````c
/// \param __m
///    A 64-bit integer vector of [4 x i16].
/// \param __count
///    A 32-bit integer value.
/// \returns A 64-bit integer vector of [4 x i16] containing the left-shifted
///    values. If \a __count is greater or equal to 16, the result is set to all
///    0.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_pi16(__m64 __m,
                                                              int __count) {
  return __trunc64(__builtin_ia32_psllwi128((__v8hi)__zext128(__m), __count));
}

/// Left-shifts each 32-bit signed integer element of the first
///    parameter, which is a 64-bit integer vector of [2 x i32], by the number
///    of bits specified by the second parameter, which is a 64-bit integer. The
///    lower 32 bits of the results are packed into a 64-bit integer vector of
///    [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSLLD </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [2 x i32].
````
- **L745 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L745 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L746 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L747 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L747 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L748 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L749 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the left-shifted`.
  **L749 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the left-shifted`。
- **L750 EN**: Comment explains nearby logic, constraints, or intent: `values. If a __count is greater or equal to 16, the result is set to all`.
  **L750 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values. If a __count is greater or equal to 16, the result is set to all`。
- **L751 EN**: Comment explains nearby logic, constraints, or intent: `0.`.
  **L751 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0.`。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_pi16(__m64 __m,`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_pi16(__m64 __m,`。
- **L753 EN**: Continues the surrounding expression or declaration: `int __count) {`.
  **L753 CN**: 继续构造周围的表达式或声明：`int __count) {`。
- **L754 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psllwi128((__v8hi)__zext128(__m), __count))`.
  **L754 CN**: 以 `__trunc64(__builtin_ia32_psllwi128((__v8hi)__zext128(__m), __count))` 从当前函数返回。
- **L755 EN**: Closes the current lexical scope or compound statement.
  **L755 CN**: 结束当前词法作用域或复合语句块。
- **L756 EN**: Blank line separating nearby declarations or logic blocks.
  **L756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L757 EN**: Comment explains nearby logic, constraints, or intent: `Left-shifts each 32-bit signed integer element of the first`.
  **L757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Left-shifts each 32-bit signed integer element of the first`。
- **L758 EN**: Comment explains nearby logic, constraints, or intent: `parameter, which is a 64-bit integer vector of [2 x i32], by the number`.
  **L758 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter, which is a 64-bit integer vector of [2 x i32], by the number`。
- **L759 EN**: Comment explains nearby logic, constraints, or intent: `of bits specified by the second parameter, which is a 64-bit integer. The`.
  **L759 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of bits specified by the second parameter, which is a 64-bit integer. The`。
- **L760 EN**: Comment explains nearby logic, constraints, or intent: `lower 32 bits of the results are packed into a 64-bit integer vector of`.
  **L760 CN**: 注释解释附近代码的逻辑、约束或设计意图：`lower 32 bits of the results are packed into a 64-bit integer vector of`。
- **L761 EN**: Comment explains nearby logic, constraints, or intent: `[2 x i32].`.
  **L761 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x i32].`。
- **L762 EN**: Separator comment used for visual grouping.
  **L762 CN**: 用于视觉分组的分隔注释。
- **L763 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L763 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L764 EN**: Separator comment used for visual grouping.
  **L764 CN**: 用于视觉分组的分隔注释。
- **L765 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSLLD </c> instruction.`.
  **L765 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSLLD </c> instruction.`。
- **L766 EN**: Separator comment used for visual grouping.
  **L766 CN**: 用于视觉分组的分隔注释。
- **L767 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L767 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L768 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L768 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。

### Lines 769-792

````c
/// \param __count
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \returns A 64-bit integer vector of [2 x i32] containing the left-shifted
///    values. If \a __count is greater or equal to 32, the result is set to all
///    0.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_sll_pi32(__m64 __m, __m64 __count)
{
  return __trunc64(__builtin_ia32_pslld128((__v4si)__zext128(__m),
                                           (__v4si)__zext128(__count)));
}

/// Left-shifts each 32-bit signed integer element of a 64-bit integer
///    vector of [2 x i32] by the number of bits specified by a 32-bit integer.
///    The lower 32 bits of the results are packed into a 64-bit integer vector
///    of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSLLD </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [2 x i32].
/// \param __count
````
- **L769 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L769 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L770 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L770 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L771 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the left-shifted`.
  **L771 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the left-shifted`。
- **L772 EN**: Comment explains nearby logic, constraints, or intent: `values. If a __count is greater or equal to 32, the result is set to all`.
  **L772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values. If a __count is greater or equal to 32, the result is set to all`。
- **L773 EN**: Comment explains nearby logic, constraints, or intent: `0.`.
  **L773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0.`。
- **L774 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L774 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L775 EN**: Continues logic associated with callable symbol `_mm_sll_pi32`.
  **L775 CN**: 继续与可调用符号 `_mm_sll_pi32` 相关的逻辑。
- **L776 EN**: Opens a new lexical scope or compound statement.
  **L776 CN**: 打开一个新的词法作用域或复合语句块。
- **L777 EN**: Returns from the current function with `__trunc64(__builtin_ia32_pslld128((__v4si)__zext128(__m),`.
  **L777 CN**: 以 `__trunc64(__builtin_ia32_pslld128((__v4si)__zext128(__m),` 从当前函数返回。
- **L778 EN**: Executes a call or declaration centered on `statement`.
  **L778 CN**: 执行以 `statement` 为核心的调用或声明。
- **L779 EN**: Closes the current lexical scope or compound statement.
  **L779 CN**: 结束当前词法作用域或复合语句块。
- **L780 EN**: Blank line separating nearby declarations or logic blocks.
  **L780 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L781 EN**: Comment explains nearby logic, constraints, or intent: `Left-shifts each 32-bit signed integer element of a 64-bit integer`.
  **L781 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Left-shifts each 32-bit signed integer element of a 64-bit integer`。
- **L782 EN**: Comment explains nearby logic, constraints, or intent: `vector of [2 x i32] by the number of bits specified by a 32-bit integer.`.
  **L782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [2 x i32] by the number of bits specified by a 32-bit integer.`。
- **L783 EN**: Comment explains nearby logic, constraints, or intent: `The lower 32 bits of the results are packed into a 64-bit integer vector`.
  **L783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The lower 32 bits of the results are packed into a 64-bit integer vector`。
- **L784 EN**: Comment explains nearby logic, constraints, or intent: `of [2 x i32].`.
  **L784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [2 x i32].`。
- **L785 EN**: Separator comment used for visual grouping.
  **L785 CN**: 用于视觉分组的分隔注释。
- **L786 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L787 EN**: Separator comment used for visual grouping.
  **L787 CN**: 用于视觉分组的分隔注释。
- **L788 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSLLD </c> instruction.`.
  **L788 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSLLD </c> instruction.`。
- **L789 EN**: Separator comment used for visual grouping.
  **L789 CN**: 用于视觉分组的分隔注释。
- **L790 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L790 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L791 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L791 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L792 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L792 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。

### Lines 793-816

````c
///    A 32-bit integer value.
/// \returns A 64-bit integer vector of [2 x i32] containing the left-shifted
///    values. If \a __count is greater or equal to 32, the result is set to all
///    0.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_pi32(__m64 __m,
                                                              int __count) {
  return __trunc64(__builtin_ia32_pslldi128((__v4si)__zext128(__m), __count));
}

/// Left-shifts the first 64-bit integer parameter by the number of bits
///    specified by the second 64-bit integer parameter. The lower 64 bits of
///    result are returned.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSLLQ </c> instruction.
///
/// \param __m
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \param __count
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \returns A 64-bit integer vector containing the left-shifted value. If
///     \a __count is greater or equal to 64, the result is set to 0.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
````
- **L793 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L793 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L794 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the left-shifted`.
  **L794 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the left-shifted`。
- **L795 EN**: Comment explains nearby logic, constraints, or intent: `values. If a __count is greater or equal to 32, the result is set to all`.
  **L795 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values. If a __count is greater or equal to 32, the result is set to all`。
- **L796 EN**: Comment explains nearby logic, constraints, or intent: `0.`.
  **L796 CN**: 注释解释附近代码的逻辑、约束或设计意图：`0.`。
- **L797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_pi32(__m64 __m,`.
  **L797 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_pi32(__m64 __m,`。
- **L798 EN**: Continues the surrounding expression or declaration: `int __count) {`.
  **L798 CN**: 继续构造周围的表达式或声明：`int __count) {`。
- **L799 EN**: Returns from the current function with `__trunc64(__builtin_ia32_pslldi128((__v4si)__zext128(__m), __count))`.
  **L799 CN**: 以 `__trunc64(__builtin_ia32_pslldi128((__v4si)__zext128(__m), __count))` 从当前函数返回。
- **L800 EN**: Closes the current lexical scope or compound statement.
  **L800 CN**: 结束当前词法作用域或复合语句块。
- **L801 EN**: Blank line separating nearby declarations or logic blocks.
  **L801 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L802 EN**: Comment explains nearby logic, constraints, or intent: `Left-shifts the first 64-bit integer parameter by the number of bits`.
  **L802 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Left-shifts the first 64-bit integer parameter by the number of bits`。
- **L803 EN**: Comment explains nearby logic, constraints, or intent: `specified by the second 64-bit integer parameter. The lower 64 bits of`.
  **L803 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified by the second 64-bit integer parameter. The lower 64 bits of`。
- **L804 EN**: Comment explains nearby logic, constraints, or intent: `result are returned.`.
  **L804 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result are returned.`。
- **L805 EN**: Separator comment used for visual grouping.
  **L805 CN**: 用于视觉分组的分隔注释。
- **L806 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L806 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L807 EN**: Separator comment used for visual grouping.
  **L807 CN**: 用于视觉分组的分隔注释。
- **L808 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSLLQ </c> instruction.`.
  **L808 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSLLQ </c> instruction.`。
- **L809 EN**: Separator comment used for visual grouping.
  **L809 CN**: 用于视觉分组的分隔注释。
- **L810 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L810 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L811 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L811 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L812 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L812 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L813 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L813 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L814 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the left-shifted value. If`.
  **L814 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the left-shifted value. If`。
- **L815 EN**: Comment explains nearby logic, constraints, or intent: `a __count is greater or equal to 64, the result is set to 0.`.
  **L815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a __count is greater or equal to 64, the result is set to 0.`。
- **L816 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L816 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。

### Lines 817-840

````c
_mm_sll_si64(__m64 __m, __m64 __count)
{
  return __trunc64(__builtin_ia32_psllq128((__v2di)__zext128(__m),
                                           (__v2di)__zext128(__count)));
}

/// Left-shifts the first parameter, which is a 64-bit integer, by the
///    number of bits specified by the second parameter, which is a 32-bit
///    integer. The lower 64 bits of result are returned.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSLLQ </c> instruction.
///
/// \param __m
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \param __count
///    A 32-bit integer value.
/// \returns A 64-bit integer vector containing the left-shifted value. If
///     \a __count is greater or equal to 64, the result is set to 0.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_si64(__m64 __m,
                                                              int __count) {
  return __trunc64(__builtin_ia32_psllqi128((__v2di)__zext128(__m), __count));
}
````
- **L817 EN**: Continues logic associated with callable symbol `_mm_sll_si64`.
  **L817 CN**: 继续与可调用符号 `_mm_sll_si64` 相关的逻辑。
- **L818 EN**: Opens a new lexical scope or compound statement.
  **L818 CN**: 打开一个新的词法作用域或复合语句块。
- **L819 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psllq128((__v2di)__zext128(__m),`.
  **L819 CN**: 以 `__trunc64(__builtin_ia32_psllq128((__v2di)__zext128(__m),` 从当前函数返回。
- **L820 EN**: Executes a call or declaration centered on `statement`.
  **L820 CN**: 执行以 `statement` 为核心的调用或声明。
- **L821 EN**: Closes the current lexical scope or compound statement.
  **L821 CN**: 结束当前词法作用域或复合语句块。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Comment explains nearby logic, constraints, or intent: `Left-shifts the first parameter, which is a 64-bit integer, by the`.
  **L823 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Left-shifts the first parameter, which is a 64-bit integer, by the`。
- **L824 EN**: Comment explains nearby logic, constraints, or intent: `number of bits specified by the second parameter, which is a 32-bit`.
  **L824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`number of bits specified by the second parameter, which is a 32-bit`。
- **L825 EN**: Comment explains nearby logic, constraints, or intent: `integer. The lower 64 bits of result are returned.`.
  **L825 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer. The lower 64 bits of result are returned.`。
- **L826 EN**: Separator comment used for visual grouping.
  **L826 CN**: 用于视觉分组的分隔注释。
- **L827 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L828 EN**: Separator comment used for visual grouping.
  **L828 CN**: 用于视觉分组的分隔注释。
- **L829 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSLLQ </c> instruction.`.
  **L829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSLLQ </c> instruction.`。
- **L830 EN**: Separator comment used for visual grouping.
  **L830 CN**: 用于视觉分组的分隔注释。
- **L831 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L831 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L832 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L832 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L833 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L833 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L834 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L834 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L835 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the left-shifted value. If`.
  **L835 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the left-shifted value. If`。
- **L836 EN**: Comment explains nearby logic, constraints, or intent: `a __count is greater or equal to 64, the result is set to 0.`.
  **L836 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a __count is greater or equal to 64, the result is set to 0.`。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_si64(__m64 __m,`.
  **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_slli_si64(__m64 __m,`。
- **L838 EN**: Continues the surrounding expression or declaration: `int __count) {`.
  **L838 CN**: 继续构造周围的表达式或声明：`int __count) {`。
- **L839 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psllqi128((__v2di)__zext128(__m), __count))`.
  **L839 CN**: 以 `__trunc64(__builtin_ia32_psllqi128((__v2di)__zext128(__m), __count))` 从当前函数返回。
- **L840 EN**: Closes the current lexical scope or compound statement.
  **L840 CN**: 结束当前词法作用域或复合语句块。

### Lines 841-864

````c

/// Right-shifts each 16-bit integer element of the first parameter,
///    which is a 64-bit integer vector of [4 x i16], by the number of bits
///    specified by the second parameter, which is a 64-bit integer.
///
///    High-order bits are filled with the sign bit of the initial value of each
///    16-bit element. The 16-bit results are packed into a 64-bit integer
///    vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRAW </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [4 x i16].
/// \param __count
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \returns A 64-bit integer vector of [4 x i16] containing the right-shifted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_sra_pi16(__m64 __m, __m64 __count)
{
  return __trunc64(__builtin_ia32_psraw128((__v8hi)__zext128(__m),
                                           (__v8hi)__zext128(__count)));
````
- **L841 EN**: Blank line separating nearby declarations or logic blocks.
  **L841 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L842 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts each 16-bit integer element of the first parameter,`.
  **L842 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts each 16-bit integer element of the first parameter,`。
- **L843 EN**: Comment explains nearby logic, constraints, or intent: `which is a 64-bit integer vector of [4 x i16], by the number of bits`.
  **L843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`which is a 64-bit integer vector of [4 x i16], by the number of bits`。
- **L844 EN**: Comment explains nearby logic, constraints, or intent: `specified by the second parameter, which is a 64-bit integer.`.
  **L844 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified by the second parameter, which is a 64-bit integer.`。
- **L845 EN**: Separator comment used for visual grouping.
  **L845 CN**: 用于视觉分组的分隔注释。
- **L846 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are filled with the sign bit of the initial value of each`.
  **L846 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are filled with the sign bit of the initial value of each`。
- **L847 EN**: Comment explains nearby logic, constraints, or intent: `16-bit element. The 16-bit results are packed into a 64-bit integer`.
  **L847 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit element. The 16-bit results are packed into a 64-bit integer`。
- **L848 EN**: Comment explains nearby logic, constraints, or intent: `vector of [4 x i16].`.
  **L848 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [4 x i16].`。
- **L849 EN**: Separator comment used for visual grouping.
  **L849 CN**: 用于视觉分组的分隔注释。
- **L850 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L850 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L851 EN**: Separator comment used for visual grouping.
  **L851 CN**: 用于视觉分组的分隔注释。
- **L852 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRAW </c> instruction.`.
  **L852 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRAW </c> instruction.`。
- **L853 EN**: Separator comment used for visual grouping.
  **L853 CN**: 用于视觉分组的分隔注释。
- **L854 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L854 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L855 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L855 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L856 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L857 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L857 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L858 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the right-shifted`.
  **L858 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the right-shifted`。
- **L859 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L859 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L860 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L860 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L861 EN**: Continues logic associated with callable symbol `_mm_sra_pi16`.
  **L861 CN**: 继续与可调用符号 `_mm_sra_pi16` 相关的逻辑。
- **L862 EN**: Opens a new lexical scope or compound statement.
  **L862 CN**: 打开一个新的词法作用域或复合语句块。
- **L863 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psraw128((__v8hi)__zext128(__m),`.
  **L863 CN**: 以 `__trunc64(__builtin_ia32_psraw128((__v8hi)__zext128(__m),` 从当前函数返回。
- **L864 EN**: Executes a call or declaration centered on `statement`.
  **L864 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 865-888

````c
}

/// Right-shifts each 16-bit integer element of a 64-bit integer vector
///    of [4 x i16] by the number of bits specified by a 32-bit integer.
///
///    High-order bits are filled with the sign bit of the initial value of each
///    16-bit element. The 16-bit results are packed into a 64-bit integer
///    vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRAW </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [4 x i16].
/// \param __count
///    A 32-bit integer value.
/// \returns A 64-bit integer vector of [4 x i16] containing the right-shifted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srai_pi16(__m64 __m,
                                                              int __count) {
  return __trunc64(__builtin_ia32_psrawi128((__v8hi)__zext128(__m), __count));
}

````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts each 16-bit integer element of a 64-bit integer vector`.
  **L867 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts each 16-bit integer element of a 64-bit integer vector`。
- **L868 EN**: Comment explains nearby logic, constraints, or intent: `of [4 x i16] by the number of bits specified by a 32-bit integer.`.
  **L868 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [4 x i16] by the number of bits specified by a 32-bit integer.`。
- **L869 EN**: Separator comment used for visual grouping.
  **L869 CN**: 用于视觉分组的分隔注释。
- **L870 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are filled with the sign bit of the initial value of each`.
  **L870 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are filled with the sign bit of the initial value of each`。
- **L871 EN**: Comment explains nearby logic, constraints, or intent: `16-bit element. The 16-bit results are packed into a 64-bit integer`.
  **L871 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit element. The 16-bit results are packed into a 64-bit integer`。
- **L872 EN**: Comment explains nearby logic, constraints, or intent: `vector of [4 x i16].`.
  **L872 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [4 x i16].`。
- **L873 EN**: Separator comment used for visual grouping.
  **L873 CN**: 用于视觉分组的分隔注释。
- **L874 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L874 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L875 EN**: Separator comment used for visual grouping.
  **L875 CN**: 用于视觉分组的分隔注释。
- **L876 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRAW </c> instruction.`.
  **L876 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRAW </c> instruction.`。
- **L877 EN**: Separator comment used for visual grouping.
  **L877 CN**: 用于视觉分组的分隔注释。
- **L878 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L878 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L879 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L879 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L880 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L880 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L881 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L881 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L882 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the right-shifted`.
  **L882 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the right-shifted`。
- **L883 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L883 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srai_pi16(__m64 __m,`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srai_pi16(__m64 __m,`。
- **L885 EN**: Continues the surrounding expression or declaration: `int __count) {`.
  **L885 CN**: 继续构造周围的表达式或声明：`int __count) {`。
- **L886 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psrawi128((__v8hi)__zext128(__m), __count))`.
  **L886 CN**: 以 `__trunc64(__builtin_ia32_psrawi128((__v8hi)__zext128(__m), __count))` 从当前函数返回。
- **L887 EN**: Closes the current lexical scope or compound statement.
  **L887 CN**: 结束当前词法作用域或复合语句块。
- **L888 EN**: Blank line separating nearby declarations or logic blocks.
  **L888 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 889-912

````c
/// Right-shifts each 32-bit integer element of the first parameter,
///    which is a 64-bit integer vector of [2 x i32], by the number of bits
///    specified by the second parameter, which is a 64-bit integer.
///
///    High-order bits are filled with the sign bit of the initial value of each
///    32-bit element. The 32-bit results are packed into a 64-bit integer
///    vector of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRAD </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [2 x i32].
/// \param __count
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \returns A 64-bit integer vector of [2 x i32] containing the right-shifted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_sra_pi32(__m64 __m, __m64 __count)
{
  return __trunc64(__builtin_ia32_psrad128((__v4si)__zext128(__m),
                                           (__v4si)__zext128(__count)));
}
````
- **L889 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts each 32-bit integer element of the first parameter,`.
  **L889 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts each 32-bit integer element of the first parameter,`。
- **L890 EN**: Comment explains nearby logic, constraints, or intent: `which is a 64-bit integer vector of [2 x i32], by the number of bits`.
  **L890 CN**: 注释解释附近代码的逻辑、约束或设计意图：`which is a 64-bit integer vector of [2 x i32], by the number of bits`。
- **L891 EN**: Comment explains nearby logic, constraints, or intent: `specified by the second parameter, which is a 64-bit integer.`.
  **L891 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified by the second parameter, which is a 64-bit integer.`。
- **L892 EN**: Separator comment used for visual grouping.
  **L892 CN**: 用于视觉分组的分隔注释。
- **L893 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are filled with the sign bit of the initial value of each`.
  **L893 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are filled with the sign bit of the initial value of each`。
- **L894 EN**: Comment explains nearby logic, constraints, or intent: `32-bit element. The 32-bit results are packed into a 64-bit integer`.
  **L894 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit element. The 32-bit results are packed into a 64-bit integer`。
- **L895 EN**: Comment explains nearby logic, constraints, or intent: `vector of [2 x i32].`.
  **L895 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [2 x i32].`。
- **L896 EN**: Separator comment used for visual grouping.
  **L896 CN**: 用于视觉分组的分隔注释。
- **L897 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L897 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L898 EN**: Separator comment used for visual grouping.
  **L898 CN**: 用于视觉分组的分隔注释。
- **L899 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRAD </c> instruction.`.
  **L899 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRAD </c> instruction.`。
- **L900 EN**: Separator comment used for visual grouping.
  **L900 CN**: 用于视觉分组的分隔注释。
- **L901 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L901 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L902 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L902 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L903 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L903 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L904 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L904 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L905 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the right-shifted`.
  **L905 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the right-shifted`。
- **L906 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L906 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L907 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L907 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L908 EN**: Continues logic associated with callable symbol `_mm_sra_pi32`.
  **L908 CN**: 继续与可调用符号 `_mm_sra_pi32` 相关的逻辑。
- **L909 EN**: Opens a new lexical scope or compound statement.
  **L909 CN**: 打开一个新的词法作用域或复合语句块。
- **L910 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psrad128((__v4si)__zext128(__m),`.
  **L910 CN**: 以 `__trunc64(__builtin_ia32_psrad128((__v4si)__zext128(__m),` 从当前函数返回。
- **L911 EN**: Executes a call or declaration centered on `statement`.
  **L911 CN**: 执行以 `statement` 为核心的调用或声明。
- **L912 EN**: Closes the current lexical scope or compound statement.
  **L912 CN**: 结束当前词法作用域或复合语句块。

### Lines 913-936

````c

/// Right-shifts each 32-bit integer element of a 64-bit integer vector
///    of [2 x i32] by the number of bits specified by a 32-bit integer.
///
///    High-order bits are filled with the sign bit of the initial value of each
///    32-bit element. The 32-bit results are packed into a 64-bit integer
///    vector of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRAD </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [2 x i32].
/// \param __count
///    A 32-bit integer value.
/// \returns A 64-bit integer vector of [2 x i32] containing the right-shifted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srai_pi32(__m64 __m,
                                                              int __count) {
  return __trunc64(__builtin_ia32_psradi128((__v4si)__zext128(__m), __count));
}

/// Right-shifts each 16-bit integer element of the first parameter,
````
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L914 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts each 32-bit integer element of a 64-bit integer vector`.
  **L914 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts each 32-bit integer element of a 64-bit integer vector`。
- **L915 EN**: Comment explains nearby logic, constraints, or intent: `of [2 x i32] by the number of bits specified by a 32-bit integer.`.
  **L915 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [2 x i32] by the number of bits specified by a 32-bit integer.`。
- **L916 EN**: Separator comment used for visual grouping.
  **L916 CN**: 用于视觉分组的分隔注释。
- **L917 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are filled with the sign bit of the initial value of each`.
  **L917 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are filled with the sign bit of the initial value of each`。
- **L918 EN**: Comment explains nearby logic, constraints, or intent: `32-bit element. The 32-bit results are packed into a 64-bit integer`.
  **L918 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit element. The 32-bit results are packed into a 64-bit integer`。
- **L919 EN**: Comment explains nearby logic, constraints, or intent: `vector of [2 x i32].`.
  **L919 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector of [2 x i32].`。
- **L920 EN**: Separator comment used for visual grouping.
  **L920 CN**: 用于视觉分组的分隔注释。
- **L921 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L921 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L922 EN**: Separator comment used for visual grouping.
  **L922 CN**: 用于视觉分组的分隔注释。
- **L923 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRAD </c> instruction.`.
  **L923 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRAD </c> instruction.`。
- **L924 EN**: Separator comment used for visual grouping.
  **L924 CN**: 用于视觉分组的分隔注释。
- **L925 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L925 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L926 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L926 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L927 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L927 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L928 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L928 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L929 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the right-shifted`.
  **L929 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the right-shifted`。
- **L930 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L930 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srai_pi32(__m64 __m,`.
  **L931 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srai_pi32(__m64 __m,`。
- **L932 EN**: Continues the surrounding expression or declaration: `int __count) {`.
  **L932 CN**: 继续构造周围的表达式或声明：`int __count) {`。
- **L933 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psradi128((__v4si)__zext128(__m), __count))`.
  **L933 CN**: 以 `__trunc64(__builtin_ia32_psradi128((__v4si)__zext128(__m), __count))` 从当前函数返回。
- **L934 EN**: Closes the current lexical scope or compound statement.
  **L934 CN**: 结束当前词法作用域或复合语句块。
- **L935 EN**: Blank line separating nearby declarations or logic blocks.
  **L935 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L936 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts each 16-bit integer element of the first parameter,`.
  **L936 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts each 16-bit integer element of the first parameter,`。

### Lines 937-960

````c
///    which is a 64-bit integer vector of [4 x i16], by the number of bits
///    specified by the second parameter, which is a 64-bit integer.
///
///    High-order bits are cleared. The 16-bit results are packed into a 64-bit
///    integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRLW </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [4 x i16].
/// \param __count
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \returns A 64-bit integer vector of [4 x i16] containing the right-shifted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_srl_pi16(__m64 __m, __m64 __count)
{
  return __trunc64(__builtin_ia32_psrlw128((__v8hi)__zext128(__m),
                                           (__v8hi)__zext128(__count)));
}

/// Right-shifts each 16-bit integer element of a 64-bit integer vector
````
- **L937 EN**: Comment explains nearby logic, constraints, or intent: `which is a 64-bit integer vector of [4 x i16], by the number of bits`.
  **L937 CN**: 注释解释附近代码的逻辑、约束或设计意图：`which is a 64-bit integer vector of [4 x i16], by the number of bits`。
- **L938 EN**: Comment explains nearby logic, constraints, or intent: `specified by the second parameter, which is a 64-bit integer.`.
  **L938 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified by the second parameter, which is a 64-bit integer.`。
- **L939 EN**: Separator comment used for visual grouping.
  **L939 CN**: 用于视觉分组的分隔注释。
- **L940 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are cleared. The 16-bit results are packed into a 64-bit`.
  **L940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are cleared. The 16-bit results are packed into a 64-bit`。
- **L941 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [4 x i16].`.
  **L941 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [4 x i16].`。
- **L942 EN**: Separator comment used for visual grouping.
  **L942 CN**: 用于视觉分组的分隔注释。
- **L943 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L943 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L944 EN**: Separator comment used for visual grouping.
  **L944 CN**: 用于视觉分组的分隔注释。
- **L945 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRLW </c> instruction.`.
  **L945 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRLW </c> instruction.`。
- **L946 EN**: Separator comment used for visual grouping.
  **L946 CN**: 用于视觉分组的分隔注释。
- **L947 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L947 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L948 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L948 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L949 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L949 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L950 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L950 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L951 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the right-shifted`.
  **L951 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the right-shifted`。
- **L952 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L952 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L953 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L953 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L954 EN**: Continues logic associated with callable symbol `_mm_srl_pi16`.
  **L954 CN**: 继续与可调用符号 `_mm_srl_pi16` 相关的逻辑。
- **L955 EN**: Opens a new lexical scope or compound statement.
  **L955 CN**: 打开一个新的词法作用域或复合语句块。
- **L956 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psrlw128((__v8hi)__zext128(__m),`.
  **L956 CN**: 以 `__trunc64(__builtin_ia32_psrlw128((__v8hi)__zext128(__m),` 从当前函数返回。
- **L957 EN**: Executes a call or declaration centered on `statement`.
  **L957 CN**: 执行以 `statement` 为核心的调用或声明。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L960 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts each 16-bit integer element of a 64-bit integer vector`.
  **L960 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts each 16-bit integer element of a 64-bit integer vector`。

### Lines 961-984

````c
///    of [4 x i16] by the number of bits specified by a 32-bit integer.
///
///    High-order bits are cleared. The 16-bit results are packed into a 64-bit
///    integer vector of [4 x i16].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRLW </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [4 x i16].
/// \param __count
///    A 32-bit integer value.
/// \returns A 64-bit integer vector of [4 x i16] containing the right-shifted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_pi16(__m64 __m,
                                                              int __count) {
  return __trunc64(__builtin_ia32_psrlwi128((__v8hi)__zext128(__m), __count));
}

/// Right-shifts each 32-bit integer element of the first parameter,
///    which is a 64-bit integer vector of [2 x i32], by the number of bits
///    specified by the second parameter, which is a 64-bit integer.
///
````
- **L961 EN**: Comment explains nearby logic, constraints, or intent: `of [4 x i16] by the number of bits specified by a 32-bit integer.`.
  **L961 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [4 x i16] by the number of bits specified by a 32-bit integer.`。
- **L962 EN**: Separator comment used for visual grouping.
  **L962 CN**: 用于视觉分组的分隔注释。
- **L963 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are cleared. The 16-bit results are packed into a 64-bit`.
  **L963 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are cleared. The 16-bit results are packed into a 64-bit`。
- **L964 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [4 x i16].`.
  **L964 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [4 x i16].`。
- **L965 EN**: Separator comment used for visual grouping.
  **L965 CN**: 用于视觉分组的分隔注释。
- **L966 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L966 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L967 EN**: Separator comment used for visual grouping.
  **L967 CN**: 用于视觉分组的分隔注释。
- **L968 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRLW </c> instruction.`.
  **L968 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRLW </c> instruction.`。
- **L969 EN**: Separator comment used for visual grouping.
  **L969 CN**: 用于视觉分组的分隔注释。
- **L970 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L970 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L971 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L971 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L972 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L972 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L973 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L973 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L974 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the right-shifted`.
  **L974 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the right-shifted`。
- **L975 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L975 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_pi16(__m64 __m,`.
  **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_pi16(__m64 __m,`。
- **L977 EN**: Continues the surrounding expression or declaration: `int __count) {`.
  **L977 CN**: 继续构造周围的表达式或声明：`int __count) {`。
- **L978 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psrlwi128((__v8hi)__zext128(__m), __count))`.
  **L978 CN**: 以 `__trunc64(__builtin_ia32_psrlwi128((__v8hi)__zext128(__m), __count))` 从当前函数返回。
- **L979 EN**: Closes the current lexical scope or compound statement.
  **L979 CN**: 结束当前词法作用域或复合语句块。
- **L980 EN**: Blank line separating nearby declarations or logic blocks.
  **L980 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L981 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts each 32-bit integer element of the first parameter,`.
  **L981 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts each 32-bit integer element of the first parameter,`。
- **L982 EN**: Comment explains nearby logic, constraints, or intent: `which is a 64-bit integer vector of [2 x i32], by the number of bits`.
  **L982 CN**: 注释解释附近代码的逻辑、约束或设计意图：`which is a 64-bit integer vector of [2 x i32], by the number of bits`。
- **L983 EN**: Comment explains nearby logic, constraints, or intent: `specified by the second parameter, which is a 64-bit integer.`.
  **L983 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified by the second parameter, which is a 64-bit integer.`。
- **L984 EN**: Separator comment used for visual grouping.
  **L984 CN**: 用于视觉分组的分隔注释。

### Lines 985-1008

````c
///    High-order bits are cleared. The 32-bit results are packed into a 64-bit
///    integer vector of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRLD </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [2 x i32].
/// \param __count
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \returns A 64-bit integer vector of [2 x i32] containing the right-shifted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_srl_pi32(__m64 __m, __m64 __count)
{
  return __trunc64(__builtin_ia32_psrld128((__v4si)__zext128(__m),
                                           (__v4si)__zext128(__count)));
}

/// Right-shifts each 32-bit integer element of a 64-bit integer vector
///    of [2 x i32] by the number of bits specified by a 32-bit integer.
///
///    High-order bits are cleared. The 32-bit results are packed into a 64-bit
````
- **L985 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are cleared. The 32-bit results are packed into a 64-bit`.
  **L985 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are cleared. The 32-bit results are packed into a 64-bit`。
- **L986 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [2 x i32].`.
  **L986 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [2 x i32].`。
- **L987 EN**: Separator comment used for visual grouping.
  **L987 CN**: 用于视觉分组的分隔注释。
- **L988 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L988 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L989 EN**: Separator comment used for visual grouping.
  **L989 CN**: 用于视觉分组的分隔注释。
- **L990 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRLD </c> instruction.`.
  **L990 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRLD </c> instruction.`。
- **L991 EN**: Separator comment used for visual grouping.
  **L991 CN**: 用于视觉分组的分隔注释。
- **L992 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L992 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L993 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L993 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L994 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L994 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L995 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L995 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L996 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the right-shifted`.
  **L996 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the right-shifted`。
- **L997 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L997 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L998 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L998 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L999 EN**: Continues logic associated with callable symbol `_mm_srl_pi32`.
  **L999 CN**: 继续与可调用符号 `_mm_srl_pi32` 相关的逻辑。
- **L1000 EN**: Opens a new lexical scope or compound statement.
  **L1000 CN**: 打开一个新的词法作用域或复合语句块。
- **L1001 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psrld128((__v4si)__zext128(__m),`.
  **L1001 CN**: 以 `__trunc64(__builtin_ia32_psrld128((__v4si)__zext128(__m),` 从当前函数返回。
- **L1002 EN**: Executes a call or declaration centered on `statement`.
  **L1002 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1003 EN**: Closes the current lexical scope or compound statement.
  **L1003 CN**: 结束当前词法作用域或复合语句块。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1005 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts each 32-bit integer element of a 64-bit integer vector`.
  **L1005 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts each 32-bit integer element of a 64-bit integer vector`。
- **L1006 EN**: Comment explains nearby logic, constraints, or intent: `of [2 x i32] by the number of bits specified by a 32-bit integer.`.
  **L1006 CN**: 注释解释附近代码的逻辑、约束或设计意图：`of [2 x i32] by the number of bits specified by a 32-bit integer.`。
- **L1007 EN**: Separator comment used for visual grouping.
  **L1007 CN**: 用于视觉分组的分隔注释。
- **L1008 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are cleared. The 32-bit results are packed into a 64-bit`.
  **L1008 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are cleared. The 32-bit results are packed into a 64-bit`。

### Lines 1009-1032

````c
///    integer vector of [2 x i32].
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRLD </c> instruction.
///
/// \param __m
///    A 64-bit integer vector of [2 x i32].
/// \param __count
///    A 32-bit integer value.
/// \returns A 64-bit integer vector of [2 x i32] containing the right-shifted
///    values.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_pi32(__m64 __m,
                                                              int __count) {
  return __trunc64(__builtin_ia32_psrldi128((__v4si)__zext128(__m), __count));
}

/// Right-shifts the first 64-bit integer parameter by the number of bits
///    specified by the second 64-bit integer parameter.
///
///    High-order bits are cleared.
///
/// \headerfile <x86intrin.h>
///
````
- **L1009 EN**: Comment explains nearby logic, constraints, or intent: `integer vector of [2 x i32].`.
  **L1009 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector of [2 x i32].`。
- **L1010 EN**: Separator comment used for visual grouping.
  **L1010 CN**: 用于视觉分组的分隔注释。
- **L1011 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1011 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1012 EN**: Separator comment used for visual grouping.
  **L1012 CN**: 用于视觉分组的分隔注释。
- **L1013 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRLD </c> instruction.`.
  **L1013 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRLD </c> instruction.`。
- **L1014 EN**: Separator comment used for visual grouping.
  **L1014 CN**: 用于视觉分组的分隔注释。
- **L1015 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L1015 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L1016 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L1016 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L1017 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L1017 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L1018 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L1018 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L1019 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the right-shifted`.
  **L1019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the right-shifted`。
- **L1020 EN**: Comment explains nearby logic, constraints, or intent: `values.`.
  **L1020 CN**: 注释解释附近代码的逻辑、约束或设计意图：`values.`。
- **L1021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_pi32(__m64 __m,`.
  **L1021 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_pi32(__m64 __m,`。
- **L1022 EN**: Continues the surrounding expression or declaration: `int __count) {`.
  **L1022 CN**: 继续构造周围的表达式或声明：`int __count) {`。
- **L1023 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psrldi128((__v4si)__zext128(__m), __count))`.
  **L1023 CN**: 以 `__trunc64(__builtin_ia32_psrldi128((__v4si)__zext128(__m), __count))` 从当前函数返回。
- **L1024 EN**: Closes the current lexical scope or compound statement.
  **L1024 CN**: 结束当前词法作用域或复合语句块。
- **L1025 EN**: Blank line separating nearby declarations or logic blocks.
  **L1025 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1026 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts the first 64-bit integer parameter by the number of bits`.
  **L1026 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts the first 64-bit integer parameter by the number of bits`。
- **L1027 EN**: Comment explains nearby logic, constraints, or intent: `specified by the second 64-bit integer parameter.`.
  **L1027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`specified by the second 64-bit integer parameter.`。
- **L1028 EN**: Separator comment used for visual grouping.
  **L1028 CN**: 用于视觉分组的分隔注释。
- **L1029 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are cleared.`.
  **L1029 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are cleared.`。
- **L1030 EN**: Separator comment used for visual grouping.
  **L1030 CN**: 用于视觉分组的分隔注释。
- **L1031 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1031 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1032 EN**: Separator comment used for visual grouping.
  **L1032 CN**: 用于视觉分组的分隔注释。

### Lines 1033-1056

````c
/// This intrinsic corresponds to the <c> PSRLQ </c> instruction.
///
/// \param __m
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \param __count
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \returns A 64-bit integer vector containing the right-shifted value.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_srl_si64(__m64 __m, __m64 __count)
{
  return __trunc64(__builtin_ia32_psrlq128((__v2di)__zext128(__m),
                                           (__v2di)__zext128(__count)));
}

/// Right-shifts the first parameter, which is a 64-bit integer, by the
///    number of bits specified by the second parameter, which is a 32-bit
///    integer.
///
///    High-order bits are cleared.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PSRLQ </c> instruction.
///
````
- **L1033 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRLQ </c> instruction.`.
  **L1033 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRLQ </c> instruction.`。
- **L1034 EN**: Separator comment used for visual grouping.
  **L1034 CN**: 用于视觉分组的分隔注释。
- **L1035 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L1035 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L1036 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L1036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L1037 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L1037 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L1038 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L1038 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L1039 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the right-shifted value.`.
  **L1039 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the right-shifted value.`。
- **L1040 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L1040 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L1041 EN**: Continues logic associated with callable symbol `_mm_srl_si64`.
  **L1041 CN**: 继续与可调用符号 `_mm_srl_si64` 相关的逻辑。
- **L1042 EN**: Opens a new lexical scope or compound statement.
  **L1042 CN**: 打开一个新的词法作用域或复合语句块。
- **L1043 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psrlq128((__v2di)__zext128(__m),`.
  **L1043 CN**: 以 `__trunc64(__builtin_ia32_psrlq128((__v2di)__zext128(__m),` 从当前函数返回。
- **L1044 EN**: Executes a call or declaration centered on `statement`.
  **L1044 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1045 EN**: Closes the current lexical scope or compound statement.
  **L1045 CN**: 结束当前词法作用域或复合语句块。
- **L1046 EN**: Blank line separating nearby declarations or logic blocks.
  **L1046 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1047 EN**: Comment explains nearby logic, constraints, or intent: `Right-shifts the first parameter, which is a 64-bit integer, by the`.
  **L1047 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right-shifts the first parameter, which is a 64-bit integer, by the`。
- **L1048 EN**: Comment explains nearby logic, constraints, or intent: `number of bits specified by the second parameter, which is a 32-bit`.
  **L1048 CN**: 注释解释附近代码的逻辑、约束或设计意图：`number of bits specified by the second parameter, which is a 32-bit`。
- **L1049 EN**: Comment explains nearby logic, constraints, or intent: `integer.`.
  **L1049 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer.`。
- **L1050 EN**: Separator comment used for visual grouping.
  **L1050 CN**: 用于视觉分组的分隔注释。
- **L1051 EN**: Comment explains nearby logic, constraints, or intent: `High-order bits are cleared.`.
  **L1051 CN**: 注释解释附近代码的逻辑、约束或设计意图：`High-order bits are cleared.`。
- **L1052 EN**: Separator comment used for visual grouping.
  **L1052 CN**: 用于视觉分组的分隔注释。
- **L1053 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1053 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1054 EN**: Separator comment used for visual grouping.
  **L1054 CN**: 用于视觉分组的分隔注释。
- **L1055 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PSRLQ </c> instruction.`.
  **L1055 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PSRLQ </c> instruction.`。
- **L1056 EN**: Separator comment used for visual grouping.
  **L1056 CN**: 用于视觉分组的分隔注释。

### Lines 1057-1080

````c
/// \param __m
///    A 64-bit integer vector interpreted as a single 64-bit integer.
/// \param __count
///    A 32-bit integer value.
/// \returns A 64-bit integer vector containing the right-shifted value.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_si64(__m64 __m,
                                                              int __count) {
  return __trunc64(__builtin_ia32_psrlqi128((__v2di)__zext128(__m), __count));
}

/// Performs a bitwise AND of two 64-bit integer vectors.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PAND </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector.
/// \param __m2
///    A 64-bit integer vector.
/// \returns A 64-bit integer vector containing the bitwise AND of both
///    parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_and_si64(__m64 __m1,
                                                             __m64 __m2) {
````
- **L1057 EN**: Comment explains nearby logic, constraints, or intent: `param __m`.
  **L1057 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m`。
- **L1058 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector interpreted as a single 64-bit integer.`.
  **L1058 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector interpreted as a single 64-bit integer.`。
- **L1059 EN**: Comment explains nearby logic, constraints, or intent: `param __count`.
  **L1059 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __count`。
- **L1060 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value.`.
  **L1060 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value.`。
- **L1061 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the right-shifted value.`.
  **L1061 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the right-shifted value.`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_si64(__m64 __m,`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_srli_si64(__m64 __m,`。
- **L1063 EN**: Continues the surrounding expression or declaration: `int __count) {`.
  **L1063 CN**: 继续构造周围的表达式或声明：`int __count) {`。
- **L1064 EN**: Returns from the current function with `__trunc64(__builtin_ia32_psrlqi128((__v2di)__zext128(__m), __count))`.
  **L1064 CN**: 以 `__trunc64(__builtin_ia32_psrlqi128((__v2di)__zext128(__m), __count))` 从当前函数返回。
- **L1065 EN**: Closes the current lexical scope or compound statement.
  **L1065 CN**: 结束当前词法作用域或复合语句块。
- **L1066 EN**: Blank line separating nearby declarations or logic blocks.
  **L1066 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1067 EN**: Comment explains nearby logic, constraints, or intent: `Performs a bitwise AND of two 64-bit integer vectors.`.
  **L1067 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Performs a bitwise AND of two 64-bit integer vectors.`。
- **L1068 EN**: Separator comment used for visual grouping.
  **L1068 CN**: 用于视觉分组的分隔注释。
- **L1069 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1069 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1070 EN**: Separator comment used for visual grouping.
  **L1070 CN**: 用于视觉分组的分隔注释。
- **L1071 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PAND </c> instruction.`.
  **L1071 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PAND </c> instruction.`。
- **L1072 EN**: Separator comment used for visual grouping.
  **L1072 CN**: 用于视觉分组的分隔注释。
- **L1073 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1073 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1074 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L1074 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。
- **L1075 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1075 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1076 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L1076 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。
- **L1077 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the bitwise AND of both`.
  **L1077 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the bitwise AND of both`。
- **L1078 EN**: Comment explains nearby logic, constraints, or intent: `parameters.`.
  **L1078 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameters.`。
- **L1079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_and_si64(__m64 __m1,`.
  **L1079 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_and_si64(__m64 __m1,`。
- **L1080 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1080 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。

### Lines 1081-1104

````c
  return (__m64)(((__v1du)__m1) & ((__v1du)__m2));
}

/// Performs a bitwise NOT of the first 64-bit integer vector, and then
///    performs a bitwise AND of the intermediate result and the second 64-bit
///    integer vector.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PANDN </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector. The one's complement of this parameter is used
///    in the bitwise AND.
/// \param __m2
///    A 64-bit integer vector.
/// \returns A 64-bit integer vector containing the bitwise AND of the second
///    parameter and the one's complement of the first parameter.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_andnot_si64(__m64 __m1,
                                                                __m64 __m2) {
  return (__m64)(~((__v1du)__m1) & ((__v1du)__m2));
}

/// Performs a bitwise OR of two 64-bit integer vectors.
````
- **L1081 EN**: Returns from the current function with `(__m64)(((__v1du)__m1) & ((__v1du)__m2))`.
  **L1081 CN**: 以 `(__m64)(((__v1du)__m1) & ((__v1du)__m2))` 从当前函数返回。
- **L1082 EN**: Closes the current lexical scope or compound statement.
  **L1082 CN**: 结束当前词法作用域或复合语句块。
- **L1083 EN**: Blank line separating nearby declarations or logic blocks.
  **L1083 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1084 EN**: Comment explains nearby logic, constraints, or intent: `Performs a bitwise NOT of the first 64-bit integer vector, and then`.
  **L1084 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Performs a bitwise NOT of the first 64-bit integer vector, and then`。
- **L1085 EN**: Comment explains nearby logic, constraints, or intent: `performs a bitwise AND of the intermediate result and the second 64-bit`.
  **L1085 CN**: 注释解释附近代码的逻辑、约束或设计意图：`performs a bitwise AND of the intermediate result and the second 64-bit`。
- **L1086 EN**: Comment explains nearby logic, constraints, or intent: `integer vector.`.
  **L1086 CN**: 注释解释附近代码的逻辑、约束或设计意图：`integer vector.`。
- **L1087 EN**: Separator comment used for visual grouping.
  **L1087 CN**: 用于视觉分组的分隔注释。
- **L1088 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1088 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1089 EN**: Separator comment used for visual grouping.
  **L1089 CN**: 用于视觉分组的分隔注释。
- **L1090 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PANDN </c> instruction.`.
  **L1090 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PANDN </c> instruction.`。
- **L1091 EN**: Separator comment used for visual grouping.
  **L1091 CN**: 用于视觉分组的分隔注释。
- **L1092 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1092 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1093 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector. The one's complement of this parameter is used`.
  **L1093 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector. The one's complement of this parameter is used`。
- **L1094 EN**: Comment explains nearby logic, constraints, or intent: `in the bitwise AND.`.
  **L1094 CN**: 注释解释附近代码的逻辑、约束或设计意图：`in the bitwise AND.`。
- **L1095 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1095 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1096 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L1096 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。
- **L1097 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the bitwise AND of the second`.
  **L1097 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the bitwise AND of the second`。
- **L1098 EN**: Comment explains nearby logic, constraints, or intent: `parameter and the one's complement of the first parameter.`.
  **L1098 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameter and the one's complement of the first parameter.`。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_andnot_si64(__m64 __m1,`.
  **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_andnot_si64(__m64 __m1,`。
- **L1100 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1100 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L1101 EN**: Returns from the current function with `(__m64)(~((__v1du)__m1) & ((__v1du)__m2))`.
  **L1101 CN**: 以 `(__m64)(~((__v1du)__m1) & ((__v1du)__m2))` 从当前函数返回。
- **L1102 EN**: Closes the current lexical scope or compound statement.
  **L1102 CN**: 结束当前词法作用域或复合语句块。
- **L1103 EN**: Blank line separating nearby declarations or logic blocks.
  **L1103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1104 EN**: Comment explains nearby logic, constraints, or intent: `Performs a bitwise OR of two 64-bit integer vectors.`.
  **L1104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Performs a bitwise OR of two 64-bit integer vectors.`。

### Lines 1105-1128

````c
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> POR </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector.
/// \param __m2
///    A 64-bit integer vector.
/// \returns A 64-bit integer vector containing the bitwise OR of both
///    parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_or_si64(__m64 __m1,
                                                            __m64 __m2) {
  return (__m64)(((__v1du)__m1) | ((__v1du)__m2));
}

/// Performs a bitwise exclusive OR of two 64-bit integer vectors.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PXOR </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector.
````
- **L1105 EN**: Separator comment used for visual grouping.
  **L1105 CN**: 用于视觉分组的分隔注释。
- **L1106 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1107 EN**: Separator comment used for visual grouping.
  **L1107 CN**: 用于视觉分组的分隔注释。
- **L1108 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> POR </c> instruction.`.
  **L1108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> POR </c> instruction.`。
- **L1109 EN**: Separator comment used for visual grouping.
  **L1109 CN**: 用于视觉分组的分隔注释。
- **L1110 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1111 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L1111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。
- **L1112 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1112 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1113 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L1113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。
- **L1114 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the bitwise OR of both`.
  **L1114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the bitwise OR of both`。
- **L1115 EN**: Comment explains nearby logic, constraints, or intent: `parameters.`.
  **L1115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameters.`。
- **L1116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_or_si64(__m64 __m1,`.
  **L1116 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_or_si64(__m64 __m1,`。
- **L1117 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1117 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L1118 EN**: Returns from the current function with `(__m64)(((__v1du)__m1) | ((__v1du)__m2))`.
  **L1118 CN**: 以 `(__m64)(((__v1du)__m1) | ((__v1du)__m2))` 从当前函数返回。
- **L1119 EN**: Closes the current lexical scope or compound statement.
  **L1119 CN**: 结束当前词法作用域或复合语句块。
- **L1120 EN**: Blank line separating nearby declarations or logic blocks.
  **L1120 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1121 EN**: Comment explains nearby logic, constraints, or intent: `Performs a bitwise exclusive OR of two 64-bit integer vectors.`.
  **L1121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Performs a bitwise exclusive OR of two 64-bit integer vectors.`。
- **L1122 EN**: Separator comment used for visual grouping.
  **L1122 CN**: 用于视觉分组的分隔注释。
- **L1123 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1124 EN**: Separator comment used for visual grouping.
  **L1124 CN**: 用于视觉分组的分隔注释。
- **L1125 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PXOR </c> instruction.`.
  **L1125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PXOR </c> instruction.`。
- **L1126 EN**: Separator comment used for visual grouping.
  **L1126 CN**: 用于视觉分组的分隔注释。
- **L1127 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1128 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L1128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。

### Lines 1129-1152

````c
/// \param __m2
///    A 64-bit integer vector.
/// \returns A 64-bit integer vector containing the bitwise exclusive OR of both
///    parameters.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_xor_si64(__m64 __m1,
                                                             __m64 __m2) {
  return (__m64)(((__v1du)__m1) ^ ((__v1du)__m2));
}

/// Compares the 8-bit integer elements of two 64-bit integer vectors of
///    [8 x i8] to determine if the element of the first vector is equal to the
///    corresponding element of the second vector.
///
///    Each comparison returns 0 for false, 0xFF for true.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PCMPEQB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8].
/// \param __m2
///    A 64-bit integer vector of [8 x i8].
/// \returns A 64-bit integer vector of [8 x i8] containing the comparison
````
- **L1129 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1130 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector.`.
  **L1130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector.`。
- **L1131 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector containing the bitwise exclusive OR of both`.
  **L1131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector containing the bitwise exclusive OR of both`。
- **L1132 EN**: Comment explains nearby logic, constraints, or intent: `parameters.`.
  **L1132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`parameters.`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_xor_si64(__m64 __m1,`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_xor_si64(__m64 __m1,`。
- **L1134 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1134 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L1135 EN**: Returns from the current function with `(__m64)(((__v1du)__m1) ^ ((__v1du)__m2))`.
  **L1135 CN**: 以 `(__m64)(((__v1du)__m1) ^ ((__v1du)__m2))` 从当前函数返回。
- **L1136 EN**: Closes the current lexical scope or compound statement.
  **L1136 CN**: 结束当前词法作用域或复合语句块。
- **L1137 EN**: Blank line separating nearby declarations or logic blocks.
  **L1137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1138 EN**: Comment explains nearby logic, constraints, or intent: `Compares the 8-bit integer elements of two 64-bit integer vectors of`.
  **L1138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the 8-bit integer elements of two 64-bit integer vectors of`。
- **L1139 EN**: Comment explains nearby logic, constraints, or intent: `[8 x i8] to determine if the element of the first vector is equal to the`.
  **L1139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[8 x i8] to determine if the element of the first vector is equal to the`。
- **L1140 EN**: Comment explains nearby logic, constraints, or intent: `corresponding element of the second vector.`.
  **L1140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding element of the second vector.`。
- **L1141 EN**: Separator comment used for visual grouping.
  **L1141 CN**: 用于视觉分组的分隔注释。
- **L1142 EN**: Comment explains nearby logic, constraints, or intent: `Each comparison returns 0 for false, 0xFF for true.`.
  **L1142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each comparison returns 0 for false, 0xFF for true.`。
- **L1143 EN**: Separator comment used for visual grouping.
  **L1143 CN**: 用于视觉分组的分隔注释。
- **L1144 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1145 EN**: Separator comment used for visual grouping.
  **L1145 CN**: 用于视觉分组的分隔注释。
- **L1146 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PCMPEQB </c> instruction.`.
  **L1146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PCMPEQB </c> instruction.`。
- **L1147 EN**: Separator comment used for visual grouping.
  **L1147 CN**: 用于视觉分组的分隔注释。
- **L1148 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1149 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L1149 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L1150 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1151 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L1151 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L1152 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the comparison`.
  **L1152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the comparison`。

### Lines 1153-1176

````c
///    results.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi8(__m64 __m1,
                                                              __m64 __m2) {
  return (__m64)(((__v8qi)__m1) == ((__v8qi)__m2));
}

/// Compares the 16-bit integer elements of two 64-bit integer vectors of
///    [4 x i16] to determine if the element of the first vector is equal to the
///    corresponding element of the second vector.
///
///    Each comparison returns 0 for false, 0xFFFF for true.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PCMPEQW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
/// \returns A 64-bit integer vector of [4 x i16] containing the comparison
///    results.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi16(__m64 __m1,
                                                               __m64 __m2) {
````
- **L1153 EN**: Comment explains nearby logic, constraints, or intent: `results.`.
  **L1153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results.`。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi8(__m64 __m1,`.
  **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi8(__m64 __m1,`。
- **L1155 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1155 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L1156 EN**: Returns from the current function with `(__m64)(((__v8qi)__m1) == ((__v8qi)__m2))`.
  **L1156 CN**: 以 `(__m64)(((__v8qi)__m1) == ((__v8qi)__m2))` 从当前函数返回。
- **L1157 EN**: Closes the current lexical scope or compound statement.
  **L1157 CN**: 结束当前词法作用域或复合语句块。
- **L1158 EN**: Blank line separating nearby declarations or logic blocks.
  **L1158 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1159 EN**: Comment explains nearby logic, constraints, or intent: `Compares the 16-bit integer elements of two 64-bit integer vectors of`.
  **L1159 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the 16-bit integer elements of two 64-bit integer vectors of`。
- **L1160 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i16] to determine if the element of the first vector is equal to the`.
  **L1160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i16] to determine if the element of the first vector is equal to the`。
- **L1161 EN**: Comment explains nearby logic, constraints, or intent: `corresponding element of the second vector.`.
  **L1161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding element of the second vector.`。
- **L1162 EN**: Separator comment used for visual grouping.
  **L1162 CN**: 用于视觉分组的分隔注释。
- **L1163 EN**: Comment explains nearby logic, constraints, or intent: `Each comparison returns 0 for false, 0xFFFF for true.`.
  **L1163 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each comparison returns 0 for false, 0xFFFF for true.`。
- **L1164 EN**: Separator comment used for visual grouping.
  **L1164 CN**: 用于视觉分组的分隔注释。
- **L1165 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1166 EN**: Separator comment used for visual grouping.
  **L1166 CN**: 用于视觉分组的分隔注释。
- **L1167 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PCMPEQW </c> instruction.`.
  **L1167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PCMPEQW </c> instruction.`。
- **L1168 EN**: Separator comment used for visual grouping.
  **L1168 CN**: 用于视觉分组的分隔注释。
- **L1169 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1170 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L1170 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L1171 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1171 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1172 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L1172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L1173 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the comparison`.
  **L1173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the comparison`。
- **L1174 EN**: Comment explains nearby logic, constraints, or intent: `results.`.
  **L1174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results.`。
- **L1175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi16(__m64 __m1,`.
  **L1175 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi16(__m64 __m1,`。
- **L1176 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1176 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。

### Lines 1177-1200

````c
  return (__m64)(((__v4hi)__m1) == ((__v4hi)__m2));
}

/// Compares the 32-bit integer elements of two 64-bit integer vectors of
///    [2 x i32] to determine if the element of the first vector is equal to the
///    corresponding element of the second vector.
///
///    Each comparison returns 0 for false, 0xFFFFFFFF for true.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PCMPEQD </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [2 x i32].
/// \param __m2
///    A 64-bit integer vector of [2 x i32].
/// \returns A 64-bit integer vector of [2 x i32] containing the comparison
///    results.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi32(__m64 __m1,
                                                               __m64 __m2) {
  return (__m64)(((__v2si)__m1) == ((__v2si)__m2));
}

````
- **L1177 EN**: Returns from the current function with `(__m64)(((__v4hi)__m1) == ((__v4hi)__m2))`.
  **L1177 CN**: 以 `(__m64)(((__v4hi)__m1) == ((__v4hi)__m2))` 从当前函数返回。
- **L1178 EN**: Closes the current lexical scope or compound statement.
  **L1178 CN**: 结束当前词法作用域或复合语句块。
- **L1179 EN**: Blank line separating nearby declarations or logic blocks.
  **L1179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1180 EN**: Comment explains nearby logic, constraints, or intent: `Compares the 32-bit integer elements of two 64-bit integer vectors of`.
  **L1180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the 32-bit integer elements of two 64-bit integer vectors of`。
- **L1181 EN**: Comment explains nearby logic, constraints, or intent: `[2 x i32] to determine if the element of the first vector is equal to the`.
  **L1181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x i32] to determine if the element of the first vector is equal to the`。
- **L1182 EN**: Comment explains nearby logic, constraints, or intent: `corresponding element of the second vector.`.
  **L1182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding element of the second vector.`。
- **L1183 EN**: Separator comment used for visual grouping.
  **L1183 CN**: 用于视觉分组的分隔注释。
- **L1184 EN**: Comment explains nearby logic, constraints, or intent: `Each comparison returns 0 for false, 0xFFFFFFFF for true.`.
  **L1184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each comparison returns 0 for false, 0xFFFFFFFF for true.`。
- **L1185 EN**: Separator comment used for visual grouping.
  **L1185 CN**: 用于视觉分组的分隔注释。
- **L1186 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1187 EN**: Separator comment used for visual grouping.
  **L1187 CN**: 用于视觉分组的分隔注释。
- **L1188 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PCMPEQD </c> instruction.`.
  **L1188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PCMPEQD </c> instruction.`。
- **L1189 EN**: Separator comment used for visual grouping.
  **L1189 CN**: 用于视觉分组的分隔注释。
- **L1190 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1191 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L1191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L1192 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1193 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L1193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L1194 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the comparison`.
  **L1194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the comparison`。
- **L1195 EN**: Comment explains nearby logic, constraints, or intent: `results.`.
  **L1195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results.`。
- **L1196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi32(__m64 __m1,`.
  **L1196 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpeq_pi32(__m64 __m1,`。
- **L1197 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1197 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L1198 EN**: Returns from the current function with `(__m64)(((__v2si)__m1) == ((__v2si)__m2))`.
  **L1198 CN**: 以 `(__m64)(((__v2si)__m1) == ((__v2si)__m2))` 从当前函数返回。
- **L1199 EN**: Closes the current lexical scope or compound statement.
  **L1199 CN**: 结束当前词法作用域或复合语句块。
- **L1200 EN**: Blank line separating nearby declarations or logic blocks.
  **L1200 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1201-1224

````c
/// Compares the 8-bit integer elements of two 64-bit integer vectors of
///    [8 x i8] to determine if the element of the first vector is greater than
///    the corresponding element of the second vector.
///
///    Each comparison returns 0 for false, 0xFF for true.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PCMPGTB </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [8 x i8].
/// \param __m2
///    A 64-bit integer vector of [8 x i8].
/// \returns A 64-bit integer vector of [8 x i8] containing the comparison
///    results.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi8(__m64 __m1,
                                                              __m64 __m2) {
  /* This function always performs a signed comparison, but __v8qi is a char
     which may be signed or unsigned, so use __v8qs. */
    return (__m64)((__v8qs)__m1 > (__v8qs)__m2);
}

/// Compares the 16-bit integer elements of two 64-bit integer vectors of
````
- **L1201 EN**: Comment explains nearby logic, constraints, or intent: `Compares the 8-bit integer elements of two 64-bit integer vectors of`.
  **L1201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the 8-bit integer elements of two 64-bit integer vectors of`。
- **L1202 EN**: Comment explains nearby logic, constraints, or intent: `[8 x i8] to determine if the element of the first vector is greater than`.
  **L1202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[8 x i8] to determine if the element of the first vector is greater than`。
- **L1203 EN**: Comment explains nearby logic, constraints, or intent: `the corresponding element of the second vector.`.
  **L1203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the corresponding element of the second vector.`。
- **L1204 EN**: Separator comment used for visual grouping.
  **L1204 CN**: 用于视觉分组的分隔注释。
- **L1205 EN**: Comment explains nearby logic, constraints, or intent: `Each comparison returns 0 for false, 0xFF for true.`.
  **L1205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each comparison returns 0 for false, 0xFF for true.`。
- **L1206 EN**: Separator comment used for visual grouping.
  **L1206 CN**: 用于视觉分组的分隔注释。
- **L1207 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1208 EN**: Separator comment used for visual grouping.
  **L1208 CN**: 用于视觉分组的分隔注释。
- **L1209 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PCMPGTB </c> instruction.`.
  **L1209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PCMPGTB </c> instruction.`。
- **L1210 EN**: Separator comment used for visual grouping.
  **L1210 CN**: 用于视觉分组的分隔注释。
- **L1211 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1212 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L1212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L1213 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1213 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1214 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [8 x i8].`.
  **L1214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [8 x i8].`。
- **L1215 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [8 x i8] containing the comparison`.
  **L1215 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [8 x i8] containing the comparison`。
- **L1216 EN**: Comment explains nearby logic, constraints, or intent: `results.`.
  **L1216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results.`。
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi8(__m64 __m1,`.
  **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi8(__m64 __m1,`。
- **L1218 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1218 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L1219 EN**: Comment explains nearby logic, constraints, or intent: `This function always performs a signed comparison, but __v8qi is a char`.
  **L1219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This function always performs a signed comparison, but __v8qi is a char`。
- **L1220 EN**: Continues the surrounding expression or declaration: `which may be signed or unsigned, so use __v8qs. */`.
  **L1220 CN**: 继续构造周围的表达式或声明：`which may be signed or unsigned, so use __v8qs. */`。
- **L1221 EN**: Returns from the current function with `(__m64)((__v8qs)__m1 > (__v8qs)__m2)`.
  **L1221 CN**: 以 `(__m64)((__v8qs)__m1 > (__v8qs)__m2)` 从当前函数返回。
- **L1222 EN**: Closes the current lexical scope or compound statement.
  **L1222 CN**: 结束当前词法作用域或复合语句块。
- **L1223 EN**: Blank line separating nearby declarations or logic blocks.
  **L1223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1224 EN**: Comment explains nearby logic, constraints, or intent: `Compares the 16-bit integer elements of two 64-bit integer vectors of`.
  **L1224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the 16-bit integer elements of two 64-bit integer vectors of`。

### Lines 1225-1248

````c
///    [4 x i16] to determine if the element of the first vector is greater than
///    the corresponding element of the second vector.
///
///    Each comparison returns 0 for false, 0xFFFF for true.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PCMPGTW </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [4 x i16].
/// \param __m2
///    A 64-bit integer vector of [4 x i16].
/// \returns A 64-bit integer vector of [4 x i16] containing the comparison
///    results.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi16(__m64 __m1,
                                                               __m64 __m2) {
  return (__m64)((__v4hi)__m1 > (__v4hi)__m2);
}

/// Compares the 32-bit integer elements of two 64-bit integer vectors of
///    [2 x i32] to determine if the element of the first vector is greater than
///    the corresponding element of the second vector.
///
````
- **L1225 EN**: Comment explains nearby logic, constraints, or intent: `[4 x i16] to determine if the element of the first vector is greater than`.
  **L1225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[4 x i16] to determine if the element of the first vector is greater than`。
- **L1226 EN**: Comment explains nearby logic, constraints, or intent: `the corresponding element of the second vector.`.
  **L1226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the corresponding element of the second vector.`。
- **L1227 EN**: Separator comment used for visual grouping.
  **L1227 CN**: 用于视觉分组的分隔注释。
- **L1228 EN**: Comment explains nearby logic, constraints, or intent: `Each comparison returns 0 for false, 0xFFFF for true.`.
  **L1228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each comparison returns 0 for false, 0xFFFF for true.`。
- **L1229 EN**: Separator comment used for visual grouping.
  **L1229 CN**: 用于视觉分组的分隔注释。
- **L1230 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1231 EN**: Separator comment used for visual grouping.
  **L1231 CN**: 用于视觉分组的分隔注释。
- **L1232 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PCMPGTW </c> instruction.`.
  **L1232 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PCMPGTW </c> instruction.`。
- **L1233 EN**: Separator comment used for visual grouping.
  **L1233 CN**: 用于视觉分组的分隔注释。
- **L1234 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1235 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L1235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L1236 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1237 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [4 x i16].`.
  **L1237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [4 x i16].`。
- **L1238 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [4 x i16] containing the comparison`.
  **L1238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [4 x i16] containing the comparison`。
- **L1239 EN**: Comment explains nearby logic, constraints, or intent: `results.`.
  **L1239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results.`。
- **L1240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi16(__m64 __m1,`.
  **L1240 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi16(__m64 __m1,`。
- **L1241 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1241 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L1242 EN**: Returns from the current function with `(__m64)((__v4hi)__m1 > (__v4hi)__m2)`.
  **L1242 CN**: 以 `(__m64)((__v4hi)__m1 > (__v4hi)__m2)` 从当前函数返回。
- **L1243 EN**: Closes the current lexical scope or compound statement.
  **L1243 CN**: 结束当前词法作用域或复合语句块。
- **L1244 EN**: Blank line separating nearby declarations or logic blocks.
  **L1244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1245 EN**: Comment explains nearby logic, constraints, or intent: `Compares the 32-bit integer elements of two 64-bit integer vectors of`.
  **L1245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compares the 32-bit integer elements of two 64-bit integer vectors of`。
- **L1246 EN**: Comment explains nearby logic, constraints, or intent: `[2 x i32] to determine if the element of the first vector is greater than`.
  **L1246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`[2 x i32] to determine if the element of the first vector is greater than`。
- **L1247 EN**: Comment explains nearby logic, constraints, or intent: `the corresponding element of the second vector.`.
  **L1247 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the corresponding element of the second vector.`。
- **L1248 EN**: Separator comment used for visual grouping.
  **L1248 CN**: 用于视觉分组的分隔注释。

### Lines 1249-1272

````c
///    Each comparison returns 0 for false, 0xFFFFFFFF for true.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PCMPGTD </c> instruction.
///
/// \param __m1
///    A 64-bit integer vector of [2 x i32].
/// \param __m2
///    A 64-bit integer vector of [2 x i32].
/// \returns A 64-bit integer vector of [2 x i32] containing the comparison
///    results.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi32(__m64 __m1,
                                                               __m64 __m2) {
  return (__m64)((__v2si)__m1 > (__v2si)__m2);
}

/// Constructs a 64-bit integer vector initialized to zero.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic corresponds to the <c> PXOR </c> instruction.
///
/// \returns An initialized 64-bit integer vector with all elements set to zero.
````
- **L1249 EN**: Comment explains nearby logic, constraints, or intent: `Each comparison returns 0 for false, 0xFFFFFFFF for true.`.
  **L1249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Each comparison returns 0 for false, 0xFFFFFFFF for true.`。
- **L1250 EN**: Separator comment used for visual grouping.
  **L1250 CN**: 用于视觉分组的分隔注释。
- **L1251 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1251 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1252 EN**: Separator comment used for visual grouping.
  **L1252 CN**: 用于视觉分组的分隔注释。
- **L1253 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PCMPGTD </c> instruction.`.
  **L1253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PCMPGTD </c> instruction.`。
- **L1254 EN**: Separator comment used for visual grouping.
  **L1254 CN**: 用于视觉分组的分隔注释。
- **L1255 EN**: Comment explains nearby logic, constraints, or intent: `param __m1`.
  **L1255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m1`。
- **L1256 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L1256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L1257 EN**: Comment explains nearby logic, constraints, or intent: `param __m2`.
  **L1257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __m2`。
- **L1258 EN**: Comment explains nearby logic, constraints, or intent: `A 64-bit integer vector of [2 x i32].`.
  **L1258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 64-bit integer vector of [2 x i32].`。
- **L1259 EN**: Comment explains nearby logic, constraints, or intent: `returns A 64-bit integer vector of [2 x i32] containing the comparison`.
  **L1259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 64-bit integer vector of [2 x i32] containing the comparison`。
- **L1260 EN**: Comment explains nearby logic, constraints, or intent: `results.`.
  **L1260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results.`。
- **L1261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi32(__m64 __m1,`.
  **L1261 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_cmpgt_pi32(__m64 __m1,`。
- **L1262 EN**: Continues the surrounding expression or declaration: `__m64 __m2) {`.
  **L1262 CN**: 继续构造周围的表达式或声明：`__m64 __m2) {`。
- **L1263 EN**: Returns from the current function with `(__m64)((__v2si)__m1 > (__v2si)__m2)`.
  **L1263 CN**: 以 `(__m64)((__v2si)__m1 > (__v2si)__m2)` 从当前函数返回。
- **L1264 EN**: Closes the current lexical scope or compound statement.
  **L1264 CN**: 结束当前词法作用域或复合语句块。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector initialized to zero.`.
  **L1266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector initialized to zero.`。
- **L1267 EN**: Separator comment used for visual grouping.
  **L1267 CN**: 用于视觉分组的分隔注释。
- **L1268 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1268 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1269 EN**: Separator comment used for visual grouping.
  **L1269 CN**: 用于视觉分组的分隔注释。
- **L1270 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the <c> PXOR </c> instruction.`.
  **L1270 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the <c> PXOR </c> instruction.`。
- **L1271 EN**: Separator comment used for visual grouping.
  **L1271 CN**: 用于视觉分组的分隔注释。
- **L1272 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector with all elements set to zero.`.
  **L1272 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector with all elements set to zero.`。

### Lines 1273-1296

````c
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setzero_si64(void) {
  return __extension__(__m64){0LL};
}

/// Constructs a 64-bit integer vector initialized with the specified
///    32-bit integer values.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
///    instruction.
///
/// \param __i1
///    A 32-bit integer value used to initialize the upper 32 bits of the
///    result.
/// \param __i0
///    A 32-bit integer value used to initialize the lower 32 bits of the
///    result.
/// \returns An initialized 64-bit integer vector.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set_pi32(int __i1,
                                                             int __i0) {
  return __extension__(__m64)(__v2si){__i0, __i1};
}

````
- **L1273 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setzero_si64(void) {`.
  **L1273 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setzero_si64(void) {`。
- **L1274 EN**: Returns from the current function with `__extension__(__m64){0LL}`.
  **L1274 CN**: 以 `__extension__(__m64){0LL}` 从当前函数返回。
- **L1275 EN**: Closes the current lexical scope or compound statement.
  **L1275 CN**: 结束当前词法作用域或复合语句块。
- **L1276 EN**: Blank line separating nearby declarations or logic blocks.
  **L1276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1277 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector initialized with the specified`.
  **L1277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector initialized with the specified`。
- **L1278 EN**: Comment explains nearby logic, constraints, or intent: `32-bit integer values.`.
  **L1278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit integer values.`。
- **L1279 EN**: Separator comment used for visual grouping.
  **L1279 CN**: 用于视觉分组的分隔注释。
- **L1280 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1281 EN**: Separator comment used for visual grouping.
  **L1281 CN**: 用于视觉分组的分隔注释。
- **L1282 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1282 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。
- **L1283 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1284 EN**: Separator comment used for visual grouping.
  **L1284 CN**: 用于视觉分组的分隔注释。
- **L1285 EN**: Comment explains nearby logic, constraints, or intent: `param __i1`.
  **L1285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __i1`。
- **L1286 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value used to initialize the upper 32 bits of the`.
  **L1286 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value used to initialize the upper 32 bits of the`。
- **L1287 EN**: Comment explains nearby logic, constraints, or intent: `result.`.
  **L1287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result.`。
- **L1288 EN**: Comment explains nearby logic, constraints, or intent: `param __i0`.
  **L1288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __i0`。
- **L1289 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value used to initialize the lower 32 bits of the`.
  **L1289 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value used to initialize the lower 32 bits of the`。
- **L1290 EN**: Comment explains nearby logic, constraints, or intent: `result.`.
  **L1290 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result.`。
- **L1291 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector.`.
  **L1291 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector.`。
- **L1292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set_pi32(int __i1,`.
  **L1292 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set_pi32(int __i1,`。
- **L1293 EN**: Continues the surrounding expression or declaration: `int __i0) {`.
  **L1293 CN**: 继续构造周围的表达式或声明：`int __i0) {`。
- **L1294 EN**: Returns from the current function with `__extension__(__m64)(__v2si){__i0, __i1}`.
  **L1294 CN**: 以 `__extension__(__m64)(__v2si){__i0, __i1}` 从当前函数返回。
- **L1295 EN**: Closes the current lexical scope or compound statement.
  **L1295 CN**: 结束当前词法作用域或复合语句块。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1297-1320

````c
/// Constructs a 64-bit integer vector initialized with the specified
///    16-bit integer values.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
///    instruction.
///
/// \param __s3
///    A 16-bit integer value used to initialize bits [63:48] of the result.
/// \param __s2
///    A 16-bit integer value used to initialize bits [47:32] of the result.
/// \param __s1
///    A 16-bit integer value used to initialize bits [31:16] of the result.
/// \param __s0
///    A 16-bit integer value used to initialize bits [15:0] of the result.
/// \returns An initialized 64-bit integer vector.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set_pi16(short __s3,
                                                             short __s2,
                                                             short __s1,
                                                             short __s0) {
  return __extension__(__m64)(__v4hi){__s0, __s1, __s2, __s3};
}

````
- **L1297 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector initialized with the specified`.
  **L1297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector initialized with the specified`。
- **L1298 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer values.`.
  **L1298 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer values.`。
- **L1299 EN**: Separator comment used for visual grouping.
  **L1299 CN**: 用于视觉分组的分隔注释。
- **L1300 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1300 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1301 EN**: Separator comment used for visual grouping.
  **L1301 CN**: 用于视觉分组的分隔注释。
- **L1302 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1302 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。
- **L1303 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1304 EN**: Separator comment used for visual grouping.
  **L1304 CN**: 用于视觉分组的分隔注释。
- **L1305 EN**: Comment explains nearby logic, constraints, or intent: `param __s3`.
  **L1305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __s3`。
- **L1306 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize bits [63:48] of the result.`.
  **L1306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize bits [63:48] of the result.`。
- **L1307 EN**: Comment explains nearby logic, constraints, or intent: `param __s2`.
  **L1307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __s2`。
- **L1308 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize bits [47:32] of the result.`.
  **L1308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize bits [47:32] of the result.`。
- **L1309 EN**: Comment explains nearby logic, constraints, or intent: `param __s1`.
  **L1309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __s1`。
- **L1310 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize bits [31:16] of the result.`.
  **L1310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize bits [31:16] of the result.`。
- **L1311 EN**: Comment explains nearby logic, constraints, or intent: `param __s0`.
  **L1311 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __s0`。
- **L1312 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize bits [15:0] of the result.`.
  **L1312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize bits [15:0] of the result.`。
- **L1313 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector.`.
  **L1313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector.`。
- **L1314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set_pi16(short __s3,`.
  **L1314 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set_pi16(short __s3,`。
- **L1315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `short __s2,`.
  **L1315 CN**: 继续一个多行参数列表、初始化器或聚合项：`short __s2,`。
- **L1316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `short __s1,`.
  **L1316 CN**: 继续一个多行参数列表、初始化器或聚合项：`short __s1,`。
- **L1317 EN**: Continues the surrounding expression or declaration: `short __s0) {`.
  **L1317 CN**: 继续构造周围的表达式或声明：`short __s0) {`。
- **L1318 EN**: Returns from the current function with `__extension__(__m64)(__v4hi){__s0, __s1, __s2, __s3}`.
  **L1318 CN**: 以 `__extension__(__m64)(__v4hi){__s0, __s1, __s2, __s3}` 从当前函数返回。
- **L1319 EN**: Closes the current lexical scope or compound statement.
  **L1319 CN**: 结束当前词法作用域或复合语句块。
- **L1320 EN**: Blank line separating nearby declarations or logic blocks.
  **L1320 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1321-1344

````c
/// Constructs a 64-bit integer vector initialized with the specified
///    8-bit integer values.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
///    instruction.
///
/// \param __b7
///    An 8-bit integer value used to initialize bits [63:56] of the result.
/// \param __b6
///    An 8-bit integer value used to initialize bits [55:48] of the result.
/// \param __b5
///    An 8-bit integer value used to initialize bits [47:40] of the result.
/// \param __b4
///    An 8-bit integer value used to initialize bits [39:32] of the result.
/// \param __b3
///    An 8-bit integer value used to initialize bits [31:24] of the result.
/// \param __b2
///    An 8-bit integer value used to initialize bits [23:16] of the result.
/// \param __b1
///    An 8-bit integer value used to initialize bits [15:8] of the result.
/// \param __b0
///    An 8-bit integer value used to initialize bits [7:0] of the result.
````
- **L1321 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector initialized with the specified`.
  **L1321 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector initialized with the specified`。
- **L1322 EN**: Comment explains nearby logic, constraints, or intent: `8-bit integer values.`.
  **L1322 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit integer values.`。
- **L1323 EN**: Separator comment used for visual grouping.
  **L1323 CN**: 用于视觉分组的分隔注释。
- **L1324 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1325 EN**: Separator comment used for visual grouping.
  **L1325 CN**: 用于视觉分组的分隔注释。
- **L1326 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1326 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。
- **L1327 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1328 EN**: Separator comment used for visual grouping.
  **L1328 CN**: 用于视觉分组的分隔注释。
- **L1329 EN**: Comment explains nearby logic, constraints, or intent: `param __b7`.
  **L1329 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b7`。
- **L1330 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [63:56] of the result.`.
  **L1330 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [63:56] of the result.`。
- **L1331 EN**: Comment explains nearby logic, constraints, or intent: `param __b6`.
  **L1331 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b6`。
- **L1332 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [55:48] of the result.`.
  **L1332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [55:48] of the result.`。
- **L1333 EN**: Comment explains nearby logic, constraints, or intent: `param __b5`.
  **L1333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b5`。
- **L1334 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [47:40] of the result.`.
  **L1334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [47:40] of the result.`。
- **L1335 EN**: Comment explains nearby logic, constraints, or intent: `param __b4`.
  **L1335 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b4`。
- **L1336 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [39:32] of the result.`.
  **L1336 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [39:32] of the result.`。
- **L1337 EN**: Comment explains nearby logic, constraints, or intent: `param __b3`.
  **L1337 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b3`。
- **L1338 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [31:24] of the result.`.
  **L1338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [31:24] of the result.`。
- **L1339 EN**: Comment explains nearby logic, constraints, or intent: `param __b2`.
  **L1339 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b2`。
- **L1340 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [23:16] of the result.`.
  **L1340 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [23:16] of the result.`。
- **L1341 EN**: Comment explains nearby logic, constraints, or intent: `param __b1`.
  **L1341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b1`。
- **L1342 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [15:8] of the result.`.
  **L1342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [15:8] of the result.`。
- **L1343 EN**: Comment explains nearby logic, constraints, or intent: `param __b0`.
  **L1343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b0`。
- **L1344 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [7:0] of the result.`.
  **L1344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [7:0] of the result.`。

### Lines 1345-1368

````c
/// \returns An initialized 64-bit integer vector.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_set_pi8(char __b7, char __b6, char __b5, char __b4, char __b3, char __b2,
            char __b1, char __b0) {
  return __extension__(__m64)(__v8qi){__b0, __b1, __b2, __b3,
                                      __b4, __b5, __b6, __b7};
}

/// Constructs a 64-bit integer vector of [2 x i32], with each of the
///    32-bit integer vector elements set to the specified 32-bit integer
///    value.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
///    instruction.
///
/// \param __i
///    A 32-bit integer value used to initialize each vector element of the
///    result.
/// \returns An initialized 64-bit integer vector of [2 x i32].
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi32(int __i) {
  return _mm_set_pi32(__i, __i);
}
````
- **L1345 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector.`.
  **L1345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector.`。
- **L1346 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L1346 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L1347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_set_pi8(char __b7, char __b6, char __b5, char __b4, char __b3, char __b2,`.
  **L1347 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_set_pi8(char __b7, char __b6, char __b5, char __b4, char __b3, char __b2,`。
- **L1348 EN**: Continues the surrounding expression or declaration: `char __b1, char __b0) {`.
  **L1348 CN**: 继续构造周围的表达式或声明：`char __b1, char __b0) {`。
- **L1349 EN**: Returns from the current function with `__extension__(__m64)(__v8qi){__b0, __b1, __b2, __b3,`.
  **L1349 CN**: 以 `__extension__(__m64)(__v8qi){__b0, __b1, __b2, __b3,` 从当前函数返回。
- **L1350 EN**: Adds a standalone statement or declaration: `__b4, __b5, __b6, __b7};`.
  **L1350 CN**: 添加一条独立语句或声明：`__b4, __b5, __b6, __b7};`。
- **L1351 EN**: Closes the current lexical scope or compound statement.
  **L1351 CN**: 结束当前词法作用域或复合语句块。
- **L1352 EN**: Blank line separating nearby declarations or logic blocks.
  **L1352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1353 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector of [2 x i32], with each of the`.
  **L1353 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector of [2 x i32], with each of the`。
- **L1354 EN**: Comment explains nearby logic, constraints, or intent: `32-bit integer vector elements set to the specified 32-bit integer`.
  **L1354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit integer vector elements set to the specified 32-bit integer`。
- **L1355 EN**: Comment explains nearby logic, constraints, or intent: `value.`.
  **L1355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value.`。
- **L1356 EN**: Separator comment used for visual grouping.
  **L1356 CN**: 用于视觉分组的分隔注释。
- **L1357 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1357 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1358 EN**: Separator comment used for visual grouping.
  **L1358 CN**: 用于视觉分组的分隔注释。
- **L1359 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1359 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。
- **L1360 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1360 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1361 EN**: Separator comment used for visual grouping.
  **L1361 CN**: 用于视觉分组的分隔注释。
- **L1362 EN**: Comment explains nearby logic, constraints, or intent: `param __i`.
  **L1362 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __i`。
- **L1363 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value used to initialize each vector element of the`.
  **L1363 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value used to initialize each vector element of the`。
- **L1364 EN**: Comment explains nearby logic, constraints, or intent: `result.`.
  **L1364 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result.`。
- **L1365 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector of [2 x i32].`.
  **L1365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector of [2 x i32].`。
- **L1366 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi32(int __i) {`.
  **L1366 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi32(int __i) {`。
- **L1367 EN**: Returns from the current function with `_mm_set_pi32(__i, __i)`.
  **L1367 CN**: 以 `_mm_set_pi32(__i, __i)` 从当前函数返回。
- **L1368 EN**: Closes the current lexical scope or compound statement.
  **L1368 CN**: 结束当前词法作用域或复合语句块。

### Lines 1369-1392

````c

/// Constructs a 64-bit integer vector of [4 x i16], with each of the
///    16-bit integer vector elements set to the specified 16-bit integer
///    value.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
///    instruction.
///
/// \param __w
///    A 16-bit integer value used to initialize each vector element of the
///    result.
/// \returns An initialized 64-bit integer vector of [4 x i16].
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi16(short __w) {
  return _mm_set_pi16(__w, __w, __w, __w);
}

/// Constructs a 64-bit integer vector of [8 x i8], with each of the
///    8-bit integer vector elements set to the specified 8-bit integer value.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
````
- **L1369 EN**: Blank line separating nearby declarations or logic blocks.
  **L1369 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1370 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector of [4 x i16], with each of the`.
  **L1370 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector of [4 x i16], with each of the`。
- **L1371 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer vector elements set to the specified 16-bit integer`.
  **L1371 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer vector elements set to the specified 16-bit integer`。
- **L1372 EN**: Comment explains nearby logic, constraints, or intent: `value.`.
  **L1372 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value.`。
- **L1373 EN**: Separator comment used for visual grouping.
  **L1373 CN**: 用于视觉分组的分隔注释。
- **L1374 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1375 EN**: Separator comment used for visual grouping.
  **L1375 CN**: 用于视觉分组的分隔注释。
- **L1376 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。
- **L1377 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1377 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1378 EN**: Separator comment used for visual grouping.
  **L1378 CN**: 用于视觉分组的分隔注释。
- **L1379 EN**: Comment explains nearby logic, constraints, or intent: `param __w`.
  **L1379 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __w`。
- **L1380 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize each vector element of the`.
  **L1380 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize each vector element of the`。
- **L1381 EN**: Comment explains nearby logic, constraints, or intent: `result.`.
  **L1381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result.`。
- **L1382 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector of [4 x i16].`.
  **L1382 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector of [4 x i16].`。
- **L1383 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi16(short __w) {`.
  **L1383 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi16(short __w) {`。
- **L1384 EN**: Returns from the current function with `_mm_set_pi16(__w, __w, __w, __w)`.
  **L1384 CN**: 以 `_mm_set_pi16(__w, __w, __w, __w)` 从当前函数返回。
- **L1385 EN**: Closes the current lexical scope or compound statement.
  **L1385 CN**: 结束当前词法作用域或复合语句块。
- **L1386 EN**: Blank line separating nearby declarations or logic blocks.
  **L1386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1387 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector of [8 x i8], with each of the`.
  **L1387 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector of [8 x i8], with each of the`。
- **L1388 EN**: Comment explains nearby logic, constraints, or intent: `8-bit integer vector elements set to the specified 8-bit integer value.`.
  **L1388 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit integer vector elements set to the specified 8-bit integer value.`。
- **L1389 EN**: Separator comment used for visual grouping.
  **L1389 CN**: 用于视觉分组的分隔注释。
- **L1390 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1390 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1391 EN**: Separator comment used for visual grouping.
  **L1391 CN**: 用于视觉分组的分隔注释。
- **L1392 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1392 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。

### Lines 1393-1416

````c
///    instruction.
///
/// \param __b
///    An 8-bit integer value used to initialize each vector element of the
///    result.
/// \returns An initialized 64-bit integer vector of [8 x i8].
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi8(char __b) {
  return _mm_set_pi8(__b, __b, __b, __b, __b, __b, __b, __b);
}

/// Constructs a 64-bit integer vector, initialized in reverse order with
///    the specified 32-bit integer values.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
///    instruction.
///
/// \param __i0
///    A 32-bit integer value used to initialize the lower 32 bits of the
///    result.
/// \param __i1
///    A 32-bit integer value used to initialize the upper 32 bits of the
///    result.
````
- **L1393 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1394 EN**: Separator comment used for visual grouping.
  **L1394 CN**: 用于视觉分组的分隔注释。
- **L1395 EN**: Comment explains nearby logic, constraints, or intent: `param __b`.
  **L1395 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b`。
- **L1396 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize each vector element of the`.
  **L1396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize each vector element of the`。
- **L1397 EN**: Comment explains nearby logic, constraints, or intent: `result.`.
  **L1397 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result.`。
- **L1398 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector of [8 x i8].`.
  **L1398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector of [8 x i8].`。
- **L1399 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi8(char __b) {`.
  **L1399 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_set1_pi8(char __b) {`。
- **L1400 EN**: Returns from the current function with `_mm_set_pi8(__b, __b, __b, __b, __b, __b, __b, __b)`.
  **L1400 CN**: 以 `_mm_set_pi8(__b, __b, __b, __b, __b, __b, __b, __b)` 从当前函数返回。
- **L1401 EN**: Closes the current lexical scope or compound statement.
  **L1401 CN**: 结束当前词法作用域或复合语句块。
- **L1402 EN**: Blank line separating nearby declarations or logic blocks.
  **L1402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1403 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector, initialized in reverse order with`.
  **L1403 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector, initialized in reverse order with`。
- **L1404 EN**: Comment explains nearby logic, constraints, or intent: `the specified 32-bit integer values.`.
  **L1404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the specified 32-bit integer values.`。
- **L1405 EN**: Separator comment used for visual grouping.
  **L1405 CN**: 用于视觉分组的分隔注释。
- **L1406 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1406 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1407 EN**: Separator comment used for visual grouping.
  **L1407 CN**: 用于视觉分组的分隔注释。
- **L1408 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1408 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。
- **L1409 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1409 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1410 EN**: Separator comment used for visual grouping.
  **L1410 CN**: 用于视觉分组的分隔注释。
- **L1411 EN**: Comment explains nearby logic, constraints, or intent: `param __i0`.
  **L1411 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __i0`。
- **L1412 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value used to initialize the lower 32 bits of the`.
  **L1412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value used to initialize the lower 32 bits of the`。
- **L1413 EN**: Comment explains nearby logic, constraints, or intent: `result.`.
  **L1413 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result.`。
- **L1414 EN**: Comment explains nearby logic, constraints, or intent: `param __i1`.
  **L1414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __i1`。
- **L1415 EN**: Comment explains nearby logic, constraints, or intent: `A 32-bit integer value used to initialize the upper 32 bits of the`.
  **L1415 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 32-bit integer value used to initialize the upper 32 bits of the`。
- **L1416 EN**: Comment explains nearby logic, constraints, or intent: `result.`.
  **L1416 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result.`。

### Lines 1417-1440

````c
/// \returns An initialized 64-bit integer vector.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setr_pi32(int __i0,
                                                              int __i1) {
  return _mm_set_pi32(__i1, __i0);
}

/// Constructs a 64-bit integer vector, initialized in reverse order with
///    the specified 16-bit integer values.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
///    instruction.
///
/// \param __w0
///    A 16-bit integer value used to initialize bits [15:0] of the result.
/// \param __w1
///    A 16-bit integer value used to initialize bits [31:16] of the result.
/// \param __w2
///    A 16-bit integer value used to initialize bits [47:32] of the result.
/// \param __w3
///    A 16-bit integer value used to initialize bits [63:48] of the result.
/// \returns An initialized 64-bit integer vector.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setr_pi16(short __w0,
````
- **L1417 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector.`.
  **L1417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector.`。
- **L1418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setr_pi32(int __i0,`.
  **L1418 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setr_pi32(int __i0,`。
- **L1419 EN**: Continues the surrounding expression or declaration: `int __i1) {`.
  **L1419 CN**: 继续构造周围的表达式或声明：`int __i1) {`。
- **L1420 EN**: Returns from the current function with `_mm_set_pi32(__i1, __i0)`.
  **L1420 CN**: 以 `_mm_set_pi32(__i1, __i0)` 从当前函数返回。
- **L1421 EN**: Closes the current lexical scope or compound statement.
  **L1421 CN**: 结束当前词法作用域或复合语句块。
- **L1422 EN**: Blank line separating nearby declarations or logic blocks.
  **L1422 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1423 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector, initialized in reverse order with`.
  **L1423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector, initialized in reverse order with`。
- **L1424 EN**: Comment explains nearby logic, constraints, or intent: `the specified 16-bit integer values.`.
  **L1424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the specified 16-bit integer values.`。
- **L1425 EN**: Separator comment used for visual grouping.
  **L1425 CN**: 用于视觉分组的分隔注释。
- **L1426 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1427 EN**: Separator comment used for visual grouping.
  **L1427 CN**: 用于视觉分组的分隔注释。
- **L1428 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1428 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。
- **L1429 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1429 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1430 EN**: Separator comment used for visual grouping.
  **L1430 CN**: 用于视觉分组的分隔注释。
- **L1431 EN**: Comment explains nearby logic, constraints, or intent: `param __w0`.
  **L1431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __w0`。
- **L1432 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize bits [15:0] of the result.`.
  **L1432 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize bits [15:0] of the result.`。
- **L1433 EN**: Comment explains nearby logic, constraints, or intent: `param __w1`.
  **L1433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __w1`。
- **L1434 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize bits [31:16] of the result.`.
  **L1434 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize bits [31:16] of the result.`。
- **L1435 EN**: Comment explains nearby logic, constraints, or intent: `param __w2`.
  **L1435 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __w2`。
- **L1436 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize bits [47:32] of the result.`.
  **L1436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize bits [47:32] of the result.`。
- **L1437 EN**: Comment explains nearby logic, constraints, or intent: `param __w3`.
  **L1437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __w3`。
- **L1438 EN**: Comment explains nearby logic, constraints, or intent: `A 16-bit integer value used to initialize bits [63:48] of the result.`.
  **L1438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 16-bit integer value used to initialize bits [63:48] of the result.`。
- **L1439 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector.`.
  **L1439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector.`。
- **L1440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setr_pi16(short __w0,`.
  **L1440 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2 _mm_setr_pi16(short __w0,`。

### Lines 1441-1464

````c
                                                              short __w1,
                                                              short __w2,
                                                              short __w3) {
  return _mm_set_pi16(__w3, __w2, __w1, __w0);
}

/// Constructs a 64-bit integer vector, initialized in reverse order with
///    the specified 8-bit integer values.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic is a utility function and does not correspond to a specific
///    instruction.
///
/// \param __b0
///    An 8-bit integer value used to initialize bits [7:0] of the result.
/// \param __b1
///    An 8-bit integer value used to initialize bits [15:8] of the result.
/// \param __b2
///    An 8-bit integer value used to initialize bits [23:16] of the result.
/// \param __b3
///    An 8-bit integer value used to initialize bits [31:24] of the result.
/// \param __b4
///    An 8-bit integer value used to initialize bits [39:32] of the result.
````
- **L1441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `short __w1,`.
  **L1441 CN**: 继续一个多行参数列表、初始化器或聚合项：`short __w1,`。
- **L1442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `short __w2,`.
  **L1442 CN**: 继续一个多行参数列表、初始化器或聚合项：`short __w2,`。
- **L1443 EN**: Continues the surrounding expression or declaration: `short __w3) {`.
  **L1443 CN**: 继续构造周围的表达式或声明：`short __w3) {`。
- **L1444 EN**: Returns from the current function with `_mm_set_pi16(__w3, __w2, __w1, __w0)`.
  **L1444 CN**: 以 `_mm_set_pi16(__w3, __w2, __w1, __w0)` 从当前函数返回。
- **L1445 EN**: Closes the current lexical scope or compound statement.
  **L1445 CN**: 结束当前词法作用域或复合语句块。
- **L1446 EN**: Blank line separating nearby declarations or logic blocks.
  **L1446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1447 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 64-bit integer vector, initialized in reverse order with`.
  **L1447 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 64-bit integer vector, initialized in reverse order with`。
- **L1448 EN**: Comment explains nearby logic, constraints, or intent: `the specified 8-bit integer values.`.
  **L1448 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the specified 8-bit integer values.`。
- **L1449 EN**: Separator comment used for visual grouping.
  **L1449 CN**: 用于视觉分组的分隔注释。
- **L1450 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L1450 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L1451 EN**: Separator comment used for visual grouping.
  **L1451 CN**: 用于视觉分组的分隔注释。
- **L1452 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic is a utility function and does not correspond to a specific`.
  **L1452 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic is a utility function and does not correspond to a specific`。
- **L1453 EN**: Comment explains nearby logic, constraints, or intent: `instruction.`.
  **L1453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction.`。
- **L1454 EN**: Separator comment used for visual grouping.
  **L1454 CN**: 用于视觉分组的分隔注释。
- **L1455 EN**: Comment explains nearby logic, constraints, or intent: `param __b0`.
  **L1455 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b0`。
- **L1456 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [7:0] of the result.`.
  **L1456 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [7:0] of the result.`。
- **L1457 EN**: Comment explains nearby logic, constraints, or intent: `param __b1`.
  **L1457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b1`。
- **L1458 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [15:8] of the result.`.
  **L1458 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [15:8] of the result.`。
- **L1459 EN**: Comment explains nearby logic, constraints, or intent: `param __b2`.
  **L1459 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b2`。
- **L1460 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [23:16] of the result.`.
  **L1460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [23:16] of the result.`。
- **L1461 EN**: Comment explains nearby logic, constraints, or intent: `param __b3`.
  **L1461 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b3`。
- **L1462 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [31:24] of the result.`.
  **L1462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [31:24] of the result.`。
- **L1463 EN**: Comment explains nearby logic, constraints, or intent: `param __b4`.
  **L1463 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b4`。
- **L1464 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [39:32] of the result.`.
  **L1464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [39:32] of the result.`。

### Lines 1465-1488

````c
/// \param __b5
///    An 8-bit integer value used to initialize bits [47:40] of the result.
/// \param __b6
///    An 8-bit integer value used to initialize bits [55:48] of the result.
/// \param __b7
///    An 8-bit integer value used to initialize bits [63:56] of the result.
/// \returns An initialized 64-bit integer vector.
static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2
_mm_setr_pi8(char __b0, char __b1, char __b2, char __b3, char __b4, char __b5,
             char __b6, char __b7) {
  return _mm_set_pi8(__b7, __b6, __b5, __b4, __b3, __b2, __b1, __b0);
}

#undef __trunc64
#undef __DEFAULT_FN_ATTRS_SSE2

/* Aliases for compatibility. */
#define _m_empty _mm_empty
#define _m_from_int _mm_cvtsi32_si64
#define _m_from_int64 _mm_cvtsi64_m64
#define _m_to_int _mm_cvtsi64_si32
#define _m_to_int64 _mm_cvtm64_si64
#define _m_packsswb _mm_packs_pi16
#define _m_packssdw _mm_packs_pi32
````
- **L1465 EN**: Comment explains nearby logic, constraints, or intent: `param __b5`.
  **L1465 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b5`。
- **L1466 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [47:40] of the result.`.
  **L1466 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [47:40] of the result.`。
- **L1467 EN**: Comment explains nearby logic, constraints, or intent: `param __b6`.
  **L1467 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b6`。
- **L1468 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [55:48] of the result.`.
  **L1468 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [55:48] of the result.`。
- **L1469 EN**: Comment explains nearby logic, constraints, or intent: `param __b7`.
  **L1469 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __b7`。
- **L1470 EN**: Comment explains nearby logic, constraints, or intent: `An 8-bit integer value used to initialize bits [63:56] of the result.`.
  **L1470 CN**: 注释解释附近代码的逻辑、约束或设计意图：`An 8-bit integer value used to initialize bits [63:56] of the result.`。
- **L1471 EN**: Comment explains nearby logic, constraints, or intent: `returns An initialized 64-bit integer vector.`.
  **L1471 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns An initialized 64-bit integer vector.`。
- **L1472 EN**: Continues the surrounding expression or declaration: `static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`.
  **L1472 CN**: 继续构造周围的表达式或声明：`static __inline__ __m64 __DEFAULT_FN_ATTRS_SSE2`。
- **L1473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_setr_pi8(char __b0, char __b1, char __b2, char __b3, char __b4, char __b5,`.
  **L1473 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_setr_pi8(char __b0, char __b1, char __b2, char __b3, char __b4, char __b5,`。
- **L1474 EN**: Continues the surrounding expression or declaration: `char __b6, char __b7) {`.
  **L1474 CN**: 继续构造周围的表达式或声明：`char __b6, char __b7) {`。
- **L1475 EN**: Returns from the current function with `_mm_set_pi8(__b7, __b6, __b5, __b4, __b3, __b2, __b1, __b0)`.
  **L1475 CN**: 以 `_mm_set_pi8(__b7, __b6, __b5, __b4, __b3, __b2, __b1, __b0)` 从当前函数返回。
- **L1476 EN**: Closes the current lexical scope or compound statement.
  **L1476 CN**: 结束当前词法作用域或复合语句块。
- **L1477 EN**: Blank line separating nearby declarations or logic blocks.
  **L1477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1478 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __trunc64`.
  **L1478 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __trunc64`。
- **L1479 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS_SSE2`.
  **L1479 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS_SSE2`。
- **L1480 EN**: Blank line separating nearby declarations or logic blocks.
  **L1480 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1481 EN**: Comment explains nearby logic, constraints, or intent: `Aliases for compatibility.`.
  **L1481 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Aliases for compatibility.`。
- **L1482 EN**: Defines macro `_m_empty` for conditional compilation, shorthand, or API generation.
  **L1482 CN**: 定义宏 `_m_empty`，用于条件编译、简写或 API 生成。
- **L1483 EN**: Defines macro `_m_from_int` for conditional compilation, shorthand, or API generation.
  **L1483 CN**: 定义宏 `_m_from_int`，用于条件编译、简写或 API 生成。
- **L1484 EN**: Defines macro `_m_from_int64` for conditional compilation, shorthand, or API generation.
  **L1484 CN**: 定义宏 `_m_from_int64`，用于条件编译、简写或 API 生成。
- **L1485 EN**: Defines macro `_m_to_int` for conditional compilation, shorthand, or API generation.
  **L1485 CN**: 定义宏 `_m_to_int`，用于条件编译、简写或 API 生成。
- **L1486 EN**: Defines macro `_m_to_int64` for conditional compilation, shorthand, or API generation.
  **L1486 CN**: 定义宏 `_m_to_int64`，用于条件编译、简写或 API 生成。
- **L1487 EN**: Defines macro `_m_packsswb` for conditional compilation, shorthand, or API generation.
  **L1487 CN**: 定义宏 `_m_packsswb`，用于条件编译、简写或 API 生成。
- **L1488 EN**: Defines macro `_m_packssdw` for conditional compilation, shorthand, or API generation.
  **L1488 CN**: 定义宏 `_m_packssdw`，用于条件编译、简写或 API 生成。

### Lines 1489-1512

````c
#define _m_packuswb _mm_packs_pu16
#define _m_punpckhbw _mm_unpackhi_pi8
#define _m_punpckhwd _mm_unpackhi_pi16
#define _m_punpckhdq _mm_unpackhi_pi32
#define _m_punpcklbw _mm_unpacklo_pi8
#define _m_punpcklwd _mm_unpacklo_pi16
#define _m_punpckldq _mm_unpacklo_pi32
#define _m_paddb _mm_add_pi8
#define _m_paddw _mm_add_pi16
#define _m_paddd _mm_add_pi32
#define _m_paddsb _mm_adds_pi8
#define _m_paddsw _mm_adds_pi16
#define _m_paddusb _mm_adds_pu8
#define _m_paddusw _mm_adds_pu16
#define _m_psubb _mm_sub_pi8
#define _m_psubw _mm_sub_pi16
#define _m_psubd _mm_sub_pi32
#define _m_psubsb _mm_subs_pi8
#define _m_psubsw _mm_subs_pi16
#define _m_psubusb _mm_subs_pu8
#define _m_psubusw _mm_subs_pu16
#define _m_pmaddwd _mm_madd_pi16
#define _m_pmulhw _mm_mulhi_pi16
#define _m_pmullw _mm_mullo_pi16
````
- **L1489 EN**: Defines macro `_m_packuswb` for conditional compilation, shorthand, or API generation.
  **L1489 CN**: 定义宏 `_m_packuswb`，用于条件编译、简写或 API 生成。
- **L1490 EN**: Defines macro `_m_punpckhbw` for conditional compilation, shorthand, or API generation.
  **L1490 CN**: 定义宏 `_m_punpckhbw`，用于条件编译、简写或 API 生成。
- **L1491 EN**: Defines macro `_m_punpckhwd` for conditional compilation, shorthand, or API generation.
  **L1491 CN**: 定义宏 `_m_punpckhwd`，用于条件编译、简写或 API 生成。
- **L1492 EN**: Defines macro `_m_punpckhdq` for conditional compilation, shorthand, or API generation.
  **L1492 CN**: 定义宏 `_m_punpckhdq`，用于条件编译、简写或 API 生成。
- **L1493 EN**: Defines macro `_m_punpcklbw` for conditional compilation, shorthand, or API generation.
  **L1493 CN**: 定义宏 `_m_punpcklbw`，用于条件编译、简写或 API 生成。
- **L1494 EN**: Defines macro `_m_punpcklwd` for conditional compilation, shorthand, or API generation.
  **L1494 CN**: 定义宏 `_m_punpcklwd`，用于条件编译、简写或 API 生成。
- **L1495 EN**: Defines macro `_m_punpckldq` for conditional compilation, shorthand, or API generation.
  **L1495 CN**: 定义宏 `_m_punpckldq`，用于条件编译、简写或 API 生成。
- **L1496 EN**: Defines macro `_m_paddb` for conditional compilation, shorthand, or API generation.
  **L1496 CN**: 定义宏 `_m_paddb`，用于条件编译、简写或 API 生成。
- **L1497 EN**: Defines macro `_m_paddw` for conditional compilation, shorthand, or API generation.
  **L1497 CN**: 定义宏 `_m_paddw`，用于条件编译、简写或 API 生成。
- **L1498 EN**: Defines macro `_m_paddd` for conditional compilation, shorthand, or API generation.
  **L1498 CN**: 定义宏 `_m_paddd`，用于条件编译、简写或 API 生成。
- **L1499 EN**: Defines macro `_m_paddsb` for conditional compilation, shorthand, or API generation.
  **L1499 CN**: 定义宏 `_m_paddsb`，用于条件编译、简写或 API 生成。
- **L1500 EN**: Defines macro `_m_paddsw` for conditional compilation, shorthand, or API generation.
  **L1500 CN**: 定义宏 `_m_paddsw`，用于条件编译、简写或 API 生成。
- **L1501 EN**: Defines macro `_m_paddusb` for conditional compilation, shorthand, or API generation.
  **L1501 CN**: 定义宏 `_m_paddusb`，用于条件编译、简写或 API 生成。
- **L1502 EN**: Defines macro `_m_paddusw` for conditional compilation, shorthand, or API generation.
  **L1502 CN**: 定义宏 `_m_paddusw`，用于条件编译、简写或 API 生成。
- **L1503 EN**: Defines macro `_m_psubb` for conditional compilation, shorthand, or API generation.
  **L1503 CN**: 定义宏 `_m_psubb`，用于条件编译、简写或 API 生成。
- **L1504 EN**: Defines macro `_m_psubw` for conditional compilation, shorthand, or API generation.
  **L1504 CN**: 定义宏 `_m_psubw`，用于条件编译、简写或 API 生成。
- **L1505 EN**: Defines macro `_m_psubd` for conditional compilation, shorthand, or API generation.
  **L1505 CN**: 定义宏 `_m_psubd`，用于条件编译、简写或 API 生成。
- **L1506 EN**: Defines macro `_m_psubsb` for conditional compilation, shorthand, or API generation.
  **L1506 CN**: 定义宏 `_m_psubsb`，用于条件编译、简写或 API 生成。
- **L1507 EN**: Defines macro `_m_psubsw` for conditional compilation, shorthand, or API generation.
  **L1507 CN**: 定义宏 `_m_psubsw`，用于条件编译、简写或 API 生成。
- **L1508 EN**: Defines macro `_m_psubusb` for conditional compilation, shorthand, or API generation.
  **L1508 CN**: 定义宏 `_m_psubusb`，用于条件编译、简写或 API 生成。
- **L1509 EN**: Defines macro `_m_psubusw` for conditional compilation, shorthand, or API generation.
  **L1509 CN**: 定义宏 `_m_psubusw`，用于条件编译、简写或 API 生成。
- **L1510 EN**: Defines macro `_m_pmaddwd` for conditional compilation, shorthand, or API generation.
  **L1510 CN**: 定义宏 `_m_pmaddwd`，用于条件编译、简写或 API 生成。
- **L1511 EN**: Defines macro `_m_pmulhw` for conditional compilation, shorthand, or API generation.
  **L1511 CN**: 定义宏 `_m_pmulhw`，用于条件编译、简写或 API 生成。
- **L1512 EN**: Defines macro `_m_pmullw` for conditional compilation, shorthand, or API generation.
  **L1512 CN**: 定义宏 `_m_pmullw`，用于条件编译、简写或 API 生成。

### Lines 1513-1536

````c
#define _m_psllw _mm_sll_pi16
#define _m_psllwi _mm_slli_pi16
#define _m_pslld _mm_sll_pi32
#define _m_pslldi _mm_slli_pi32
#define _m_psllq _mm_sll_si64
#define _m_psllqi _mm_slli_si64
#define _m_psraw _mm_sra_pi16
#define _m_psrawi _mm_srai_pi16
#define _m_psrad _mm_sra_pi32
#define _m_psradi _mm_srai_pi32
#define _m_psrlw _mm_srl_pi16
#define _m_psrlwi _mm_srli_pi16
#define _m_psrld _mm_srl_pi32
#define _m_psrldi _mm_srli_pi32
#define _m_psrlq _mm_srl_si64
#define _m_psrlqi _mm_srli_si64
#define _m_pand _mm_and_si64
#define _m_pandn _mm_andnot_si64
#define _m_por _mm_or_si64
#define _m_pxor _mm_xor_si64
#define _m_pcmpeqb _mm_cmpeq_pi8
#define _m_pcmpeqw _mm_cmpeq_pi16
#define _m_pcmpeqd _mm_cmpeq_pi32
#define _m_pcmpgtb _mm_cmpgt_pi8
````
- **L1513 EN**: Defines macro `_m_psllw` for conditional compilation, shorthand, or API generation.
  **L1513 CN**: 定义宏 `_m_psllw`，用于条件编译、简写或 API 生成。
- **L1514 EN**: Defines macro `_m_psllwi` for conditional compilation, shorthand, or API generation.
  **L1514 CN**: 定义宏 `_m_psllwi`，用于条件编译、简写或 API 生成。
- **L1515 EN**: Defines macro `_m_pslld` for conditional compilation, shorthand, or API generation.
  **L1515 CN**: 定义宏 `_m_pslld`，用于条件编译、简写或 API 生成。
- **L1516 EN**: Defines macro `_m_pslldi` for conditional compilation, shorthand, or API generation.
  **L1516 CN**: 定义宏 `_m_pslldi`，用于条件编译、简写或 API 生成。
- **L1517 EN**: Defines macro `_m_psllq` for conditional compilation, shorthand, or API generation.
  **L1517 CN**: 定义宏 `_m_psllq`，用于条件编译、简写或 API 生成。
- **L1518 EN**: Defines macro `_m_psllqi` for conditional compilation, shorthand, or API generation.
  **L1518 CN**: 定义宏 `_m_psllqi`，用于条件编译、简写或 API 生成。
- **L1519 EN**: Defines macro `_m_psraw` for conditional compilation, shorthand, or API generation.
  **L1519 CN**: 定义宏 `_m_psraw`，用于条件编译、简写或 API 生成。
- **L1520 EN**: Defines macro `_m_psrawi` for conditional compilation, shorthand, or API generation.
  **L1520 CN**: 定义宏 `_m_psrawi`，用于条件编译、简写或 API 生成。
- **L1521 EN**: Defines macro `_m_psrad` for conditional compilation, shorthand, or API generation.
  **L1521 CN**: 定义宏 `_m_psrad`，用于条件编译、简写或 API 生成。
- **L1522 EN**: Defines macro `_m_psradi` for conditional compilation, shorthand, or API generation.
  **L1522 CN**: 定义宏 `_m_psradi`，用于条件编译、简写或 API 生成。
- **L1523 EN**: Defines macro `_m_psrlw` for conditional compilation, shorthand, or API generation.
  **L1523 CN**: 定义宏 `_m_psrlw`，用于条件编译、简写或 API 生成。
- **L1524 EN**: Defines macro `_m_psrlwi` for conditional compilation, shorthand, or API generation.
  **L1524 CN**: 定义宏 `_m_psrlwi`，用于条件编译、简写或 API 生成。
- **L1525 EN**: Defines macro `_m_psrld` for conditional compilation, shorthand, or API generation.
  **L1525 CN**: 定义宏 `_m_psrld`，用于条件编译、简写或 API 生成。
- **L1526 EN**: Defines macro `_m_psrldi` for conditional compilation, shorthand, or API generation.
  **L1526 CN**: 定义宏 `_m_psrldi`，用于条件编译、简写或 API 生成。
- **L1527 EN**: Defines macro `_m_psrlq` for conditional compilation, shorthand, or API generation.
  **L1527 CN**: 定义宏 `_m_psrlq`，用于条件编译、简写或 API 生成。
- **L1528 EN**: Defines macro `_m_psrlqi` for conditional compilation, shorthand, or API generation.
  **L1528 CN**: 定义宏 `_m_psrlqi`，用于条件编译、简写或 API 生成。
- **L1529 EN**: Defines macro `_m_pand` for conditional compilation, shorthand, or API generation.
  **L1529 CN**: 定义宏 `_m_pand`，用于条件编译、简写或 API 生成。
- **L1530 EN**: Defines macro `_m_pandn` for conditional compilation, shorthand, or API generation.
  **L1530 CN**: 定义宏 `_m_pandn`，用于条件编译、简写或 API 生成。
- **L1531 EN**: Defines macro `_m_por` for conditional compilation, shorthand, or API generation.
  **L1531 CN**: 定义宏 `_m_por`，用于条件编译、简写或 API 生成。
- **L1532 EN**: Defines macro `_m_pxor` for conditional compilation, shorthand, or API generation.
  **L1532 CN**: 定义宏 `_m_pxor`，用于条件编译、简写或 API 生成。
- **L1533 EN**: Defines macro `_m_pcmpeqb` for conditional compilation, shorthand, or API generation.
  **L1533 CN**: 定义宏 `_m_pcmpeqb`，用于条件编译、简写或 API 生成。
- **L1534 EN**: Defines macro `_m_pcmpeqw` for conditional compilation, shorthand, or API generation.
  **L1534 CN**: 定义宏 `_m_pcmpeqw`，用于条件编译、简写或 API 生成。
- **L1535 EN**: Defines macro `_m_pcmpeqd` for conditional compilation, shorthand, or API generation.
  **L1535 CN**: 定义宏 `_m_pcmpeqd`，用于条件编译、简写或 API 生成。
- **L1536 EN**: Defines macro `_m_pcmpgtb` for conditional compilation, shorthand, or API generation.
  **L1536 CN**: 定义宏 `_m_pcmpgtb`，用于条件编译、简写或 API 生成。

### Lines 1537-1541

````c
#define _m_pcmpgtw _mm_cmpgt_pi16
#define _m_pcmpgtd _mm_cmpgt_pi32

#endif /* __MMINTRIN_H */

````
- **L1537 EN**: Defines macro `_m_pcmpgtw` for conditional compilation, shorthand, or API generation.
  **L1537 CN**: 定义宏 `_m_pcmpgtw`，用于条件编译、简写或 API 生成。
- **L1538 EN**: Defines macro `_m_pcmpgtd` for conditional compilation, shorthand, or API generation.
  **L1538 CN**: 定义宏 `_m_pcmpgtd`，用于条件编译、简写或 API 生成。
- **L1539 EN**: Blank line separating nearby declarations or logic blocks.
  **L1539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1540 EN**: Closes the current preprocessor conditional block.
  **L1540 CN**: 结束当前预处理条件块。
- **L1541 EN**: Blank line separating nearby declarations or logic blocks.
  **L1541 CN**: 空行，用于分隔附近的声明或逻辑块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Inline wrapper functions / 内联包装函数**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__MMINTRIN_H`, `__i386__`, `__x86_64__`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_shufflevector`, `__builtin_ia32_emms`, `__builtin_ia32_packsswb128`, `__builtin_ia32_packssdw128`, `__builtin_ia32_packuswb128`, `__builtin_elementwise_add_sat`, `__builtin_elementwise_sub_sat`, `__builtin_ia32_pmaddwd128`, `__builtin_ia32_pmulhw128`, `__builtin_ia32_psllw128`, `__builtin_ia32_psllwi128`, `__builtin_ia32_pslld128`
