# tmmintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/ppc_wrappers/tmmintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implementation of SSSE3 intrinsics on PowerPC.
- **Purpose (CN)**: 该头文件主要作用是：Implementation of SSSE3 intrinsics on PowerPC。
- **Line Count / 行数**: 453

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*===---- tmmintrin.h - Implementation of SSSE3 intrinsics on PowerPC ------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

/* Implemented from the specification included in the Intel C++ Compiler
   User Guide and Reference, version 9.0.  */

#ifndef NO_WARN_X86_INTRINSICS
/* This header is distributed to simplify porting x86_64 code that
   makes explicit use of Intel intrinsics to powerpc64le.

   It is the user's responsibility to determine if the results are
   acceptable and make additional changes as necessary.

   Note that much code that uses Intel intrinsics can be rewritten in
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
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `Implemented from the specification included in the Intel C++ Compiler`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Implemented from the specification included in the Intel C++ Compiler`。
- **L11 EN**: Continues the surrounding expression or declaration: `User Guide and Reference, version 9.0.  */`.
  **L11 CN**: 继续构造周围的表达式或声明：`User Guide and Reference, version 9.0.  */`。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#ifndef NO_WARN_X86_INTRINSICS`.
  **L13 CN**: 开始一个预处理条件块：`#ifndef NO_WARN_X86_INTRINSICS`。
- **L14 EN**: Comment explains nearby logic, constraints, or intent: `This header is distributed to simplify porting x86_64 code that`.
  **L14 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This header is distributed to simplify porting x86_64 code that`。
- **L15 EN**: Continues the surrounding expression or declaration: `makes explicit use of Intel intrinsics to powerpc64le.`.
  **L15 CN**: 继续构造周围的表达式或声明：`makes explicit use of Intel intrinsics to powerpc64le.`。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Continues the surrounding expression or declaration: `It is the user's responsibility to determine if the results are`.
  **L17 CN**: 继续构造周围的表达式或声明：`It is the user's responsibility to determine if the results are`。
- **L18 EN**: Continues the surrounding expression or declaration: `acceptable and make additional changes as necessary.`.
  **L18 CN**: 继续构造周围的表达式或声明：`acceptable and make additional changes as necessary.`。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L20 EN**: Continues the surrounding expression or declaration: `Note that much code that uses Intel intrinsics can be rewritten in`.
  **L20 CN**: 继续构造周围的表达式或声明：`Note that much code that uses Intel intrinsics can be rewritten in`。

### Lines 21-40

````c
   standard C or GNU C extensions, which are more portable and better
   optimized across multiple targets.  */
#endif

#ifndef TMMINTRIN_H_
#define TMMINTRIN_H_

#if defined(__powerpc64__) &&                                                  \
    (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))

#include <altivec.h>

/* We need definitions from the SSE header files.  */
#include <pmmintrin.h>

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_abs_epi16(__m128i __A) {
  return (__m128i)vec_abs((__v8hi)__A);
}
````
- **L21 EN**: Continues the surrounding expression or declaration: `standard C or GNU C extensions, which are more portable and better`.
  **L21 CN**: 继续构造周围的表达式或声明：`standard C or GNU C extensions, which are more portable and better`。
- **L22 EN**: Continues the surrounding expression or declaration: `optimized across multiple targets.  */`.
  **L22 CN**: 继续构造周围的表达式或声明：`optimized across multiple targets.  */`。
- **L23 EN**: Closes the current preprocessor conditional block.
  **L23 CN**: 结束当前预处理条件块。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L25 EN**: Starts a preprocessor conditional block: `#ifndef TMMINTRIN_H_`.
  **L25 CN**: 开始一个预处理条件块：`#ifndef TMMINTRIN_H_`。
- **L26 EN**: Defines macro `TMMINTRIN_H_` for conditional compilation, shorthand, or API generation.
  **L26 CN**: 定义宏 `TMMINTRIN_H_`，用于条件编译、简写或 API 生成。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Starts a preprocessor conditional block: `#if defined(__powerpc64__) &&                                                  \`.
  **L28 CN**: 开始一个预处理条件块：`#if defined(__powerpc64__) &&                                                  \`。
- **L29 EN**: Continues logic associated with callable symbol `defined`.
  **L29 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Includes <altivec.h> to access related header declarations.
  **L31 CN**: 引入 <altivec.h> 以使用相关头文件声明。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L33 EN**: Comment explains nearby logic, constraints, or intent: `We need definitions from the SSE header files.`.
  **L33 CN**: 注释解释附近代码的逻辑、约束或设计意图：`We need definitions from the SSE header files.`。
- **L34 EN**: Includes <pmmintrin.h> to access related header declarations.
  **L34 CN**: 引入 <pmmintrin.h> 以使用相关头文件声明。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L36 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L36 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L37 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L37 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L38 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_abs_epi16(__m128i __A) {`.
  **L38 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_abs_epi16(__m128i __A) {`。
- **L39 EN**: Returns from the current function with `(__m128i)vec_abs((__v8hi)__A)`.
  **L39 CN**: 以 `(__m128i)vec_abs((__v8hi)__A)` 从当前函数返回。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。

### Lines 41-60

````c

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_abs_epi32(__m128i __A) {
  return (__m128i)vec_abs((__v4si)__A);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_abs_epi8(__m128i __A) {
  return (__m128i)vec_abs((__v16qi)__A);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_abs_pi16(__m64 __A) {
  __v8hi __B = (__v8hi)(__v2du){__A, __A};
  return (__m64)((__v2du)vec_abs(__B))[0];
}

````
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L42 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L43 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L43 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L44 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_abs_epi32(__m128i __A) {`.
  **L44 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_abs_epi32(__m128i __A) {`。
- **L45 EN**: Returns from the current function with `(__m128i)vec_abs((__v4si)__A)`.
  **L45 CN**: 以 `(__m128i)vec_abs((__v4si)__A)` 从当前函数返回。
- **L46 EN**: Closes the current lexical scope or compound statement.
  **L46 CN**: 结束当前词法作用域或复合语句块。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L48 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L48 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L49 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L49 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L50 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_abs_epi8(__m128i __A) {`.
  **L50 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_abs_epi8(__m128i __A) {`。
- **L51 EN**: Returns from the current function with `(__m128i)vec_abs((__v16qi)__A)`.
  **L51 CN**: 以 `(__m128i)vec_abs((__v16qi)__A)` 从当前函数返回。
- **L52 EN**: Closes the current lexical scope or compound statement.
  **L52 CN**: 结束当前词法作用域或复合语句块。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L54 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L54 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L55 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L55 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L56 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_abs_pi16(__m64 __A) {`.
  **L56 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_abs_pi16(__m64 __A) {`。
- **L57 EN**: Initializes variable `__B` from the expression on the right-hand side.
  **L57 CN**: 使用右侧表达式初始化变量 `__B`。
- **L58 EN**: Returns from the current function with `(__m64)((__v2du)vec_abs(__B))[0]`.
  **L58 CN**: 以 `(__m64)((__v2du)vec_abs(__B))[0]` 从当前函数返回。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 61-80

````c
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_abs_pi32(__m64 __A) {
  __v4si __B = (__v4si)(__v2du){__A, __A};
  return (__m64)((__v2du)vec_abs(__B))[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_abs_pi8(__m64 __A) {
  __v16qi __B = (__v16qi)(__v2du){__A, __A};
  return (__m64)((__v2du)vec_abs(__B))[0];
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_alignr_epi8(__m128i __A, __m128i __B, const unsigned int __count) {
  if (__builtin_constant_p(__count) && __count < 16) {
#ifdef __LITTLE_ENDIAN__
    __A = (__m128i)vec_reve((__v16qu)__A);
````
- **L61 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L61 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L62 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L62 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L63 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_abs_pi32(__m64 __A) {`.
  **L63 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_abs_pi32(__m64 __A) {`。
- **L64 EN**: Initializes variable `__B` from the expression on the right-hand side.
  **L64 CN**: 使用右侧表达式初始化变量 `__B`。
- **L65 EN**: Returns from the current function with `(__m64)((__v2du)vec_abs(__B))[0]`.
  **L65 CN**: 以 `(__m64)((__v2du)vec_abs(__B))[0]` 从当前函数返回。
- **L66 EN**: Closes the current lexical scope or compound statement.
  **L66 CN**: 结束当前词法作用域或复合语句块。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L68 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L68 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L69 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L69 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L70 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_abs_pi8(__m64 __A) {`.
  **L70 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_abs_pi8(__m64 __A) {`。
- **L71 EN**: Initializes variable `__B` from the expression on the right-hand side.
  **L71 CN**: 使用右侧表达式初始化变量 `__B`。
- **L72 EN**: Returns from the current function with `(__m64)((__v2du)vec_abs(__B))[0]`.
  **L72 CN**: 以 `(__m64)((__v2du)vec_abs(__B))[0]` 从当前函数返回。
- **L73 EN**: Closes the current lexical scope or compound statement.
  **L73 CN**: 结束当前词法作用域或复合语句块。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L75 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L75 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L76 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L76 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L77 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_alignr_epi8(__m128i __A, __m128i __B, const unsigned int __count) {`.
  **L77 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_alignr_epi8(__m128i __A, __m128i __B, const unsigned int __count) {`。
- **L78 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L78 CN**: 开始 `if` 控制流语句并计算其条件。
- **L79 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L79 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L80 EN**: Executes a call or declaration centered on `=`.
  **L80 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 81-100

````c
    __B = (__m128i)vec_reve((__v16qu)__B);
#endif
    __A = (__m128i)vec_sld((__v16qu)__B, (__v16qu)__A, __count);
#ifdef __LITTLE_ENDIAN__
    __A = (__m128i)vec_reve((__v16qu)__A);
#endif
    return __A;
  }

  if (__count == 0)
    return __B;

  if (__count >= 16) {
    if (__count >= 32) {
      const __v16qu __zero = {0};
      return (__m128i)__zero;
    } else {
      const __v16qu __shift = vec_splats((unsigned char)((__count - 16) * 8));
#ifdef __LITTLE_ENDIAN__
      return (__m128i)vec_sro((__v16qu)__A, __shift);
````
- **L81 EN**: Executes a call or declaration centered on `=`.
  **L81 CN**: 执行以 `=` 为核心的调用或声明。
- **L82 EN**: Closes the current preprocessor conditional block.
  **L82 CN**: 结束当前预处理条件块。
- **L83 EN**: Executes a call or declaration centered on `=`.
  **L83 CN**: 执行以 `=` 为核心的调用或声明。
- **L84 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L84 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L85 EN**: Executes a call or declaration centered on `=`.
  **L85 CN**: 执行以 `=` 为核心的调用或声明。
- **L86 EN**: Closes the current preprocessor conditional block.
  **L86 CN**: 结束当前预处理条件块。
- **L87 EN**: Returns from the current function with `__A`.
  **L87 CN**: 以 `__A` 从当前函数返回。
- **L88 EN**: Closes the current lexical scope or compound statement.
  **L88 CN**: 结束当前词法作用域或复合语句块。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L90 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L90 CN**: 开始 `if` 控制流语句并计算其条件。
- **L91 EN**: Returns from the current function with `__B`.
  **L91 CN**: 以 `__B` 从当前函数返回。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L93 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L93 CN**: 开始 `if` 控制流语句并计算其条件。
- **L94 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L94 CN**: 开始 `if` 控制流语句并计算其条件。
- **L95 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L95 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L96 EN**: Returns from the current function with `(__m128i)__zero`.
  **L96 CN**: 以 `(__m128i)__zero` 从当前函数返回。
- **L97 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L97 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L98 EN**: Initializes variable `__shift` from the expression on the right-hand side.
  **L98 CN**: 使用右侧表达式初始化变量 `__shift`。
- **L99 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L99 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L100 EN**: Returns from the current function with `(__m128i)vec_sro((__v16qu)__A, __shift)`.
  **L100 CN**: 以 `(__m128i)vec_sro((__v16qu)__A, __shift)` 从当前函数返回。

### Lines 101-120

````c
#else
      return (__m128i)vec_slo((__v16qu)__A, __shift);
#endif
    }
  } else {
    const __v16qu __shiftA = vec_splats((unsigned char)((16 - __count) * 8));
    const __v16qu __shiftB = vec_splats((unsigned char)(__count * 8));
#ifdef __LITTLE_ENDIAN__
    __A = (__m128i)vec_slo((__v16qu)__A, __shiftA);
    __B = (__m128i)vec_sro((__v16qu)__B, __shiftB);
#else
    __A = (__m128i)vec_sro((__v16qu)__A, __shiftA);
    __B = (__m128i)vec_slo((__v16qu)__B, __shiftB);
#endif
    return (__m128i)vec_or((__v16qu)__A, (__v16qu)__B);
  }
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L101 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L101 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L102 EN**: Returns from the current function with `(__m128i)vec_slo((__v16qu)__A, __shift)`.
  **L102 CN**: 以 `(__m128i)vec_slo((__v16qu)__A, __shift)` 从当前函数返回。
- **L103 EN**: Closes the current preprocessor conditional block.
  **L103 CN**: 结束当前预处理条件块。
- **L104 EN**: Closes the current lexical scope or compound statement.
  **L104 CN**: 结束当前词法作用域或复合语句块。
- **L105 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L105 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L106 EN**: Initializes variable `__shiftA` from the expression on the right-hand side.
  **L106 CN**: 使用右侧表达式初始化变量 `__shiftA`。
- **L107 EN**: Initializes variable `__shiftB` from the expression on the right-hand side.
  **L107 CN**: 使用右侧表达式初始化变量 `__shiftB`。
- **L108 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L108 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L109 EN**: Executes a call or declaration centered on `=`.
  **L109 CN**: 执行以 `=` 为核心的调用或声明。
- **L110 EN**: Executes a call or declaration centered on `=`.
  **L110 CN**: 执行以 `=` 为核心的调用或声明。
- **L111 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L111 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L112 EN**: Executes a call or declaration centered on `=`.
  **L112 CN**: 执行以 `=` 为核心的调用或声明。
- **L113 EN**: Executes a call or declaration centered on `=`.
  **L113 CN**: 执行以 `=` 为核心的调用或声明。
- **L114 EN**: Closes the current preprocessor conditional block.
  **L114 CN**: 结束当前预处理条件块。
- **L115 EN**: Returns from the current function with `(__m128i)vec_or((__v16qu)__A, (__v16qu)__B)`.
  **L115 CN**: 以 `(__m128i)vec_or((__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Closes the current lexical scope or compound statement.
  **L117 CN**: 结束当前词法作用域或复合语句块。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L119 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L119 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L120 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L120 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 121-140

````c
    _mm_alignr_pi8(__m64 __A, __m64 __B, unsigned int __count) {
  if (__count < 16) {
    __v2du __C = {__B, __A};
#ifdef __LITTLE_ENDIAN__
    const __v4su __shift = {__count << 3, 0, 0, 0};
    __C = (__v2du)vec_sro((__v16qu)__C, (__v16qu)__shift);
#else
    const __v4su __shift = {0, 0, 0, __count << 3};
    __C = (__v2du)vec_slo((__v16qu)__C, (__v16qu)__shift);
#endif
    return (__m64)__C[0];
  } else {
    const __m64 __zero = {0};
    return __zero;
  }
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hadd_epi16(__m128i __A, __m128i __B) {
````
- **L121 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_alignr_pi8(__m64 __A, __m64 __B, unsigned int __count) {`.
  **L121 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_alignr_pi8(__m64 __A, __m64 __B, unsigned int __count) {`。
- **L122 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L122 CN**: 开始 `if` 控制流语句并计算其条件。
- **L123 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L123 CN**: 使用右侧表达式初始化变量 `__C`。
- **L124 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L124 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L125 EN**: Initializes variable `__shift` from the expression on the right-hand side.
  **L125 CN**: 使用右侧表达式初始化变量 `__shift`。
- **L126 EN**: Executes a call or declaration centered on `=`.
  **L126 CN**: 执行以 `=` 为核心的调用或声明。
- **L127 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L127 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L128 EN**: Initializes variable `__shift` from the expression on the right-hand side.
  **L128 CN**: 使用右侧表达式初始化变量 `__shift`。
- **L129 EN**: Executes a call or declaration centered on `=`.
  **L129 CN**: 执行以 `=` 为核心的调用或声明。
- **L130 EN**: Closes the current preprocessor conditional block.
  **L130 CN**: 结束当前预处理条件块。
- **L131 EN**: Returns from the current function with `(__m64)__C[0]`.
  **L131 CN**: 以 `(__m64)__C[0]` 从当前函数返回。
- **L132 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L132 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L133 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L133 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L134 EN**: Returns from the current function with `__zero`.
  **L134 CN**: 以 `__zero` 从当前函数返回。
- **L135 EN**: Closes the current lexical scope or compound statement.
  **L135 CN**: 结束当前词法作用域或复合语句块。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L138 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L139 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L139 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L140 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hadd_epi16(__m128i __A, __m128i __B) {`.
  **L140 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hadd_epi16(__m128i __A, __m128i __B) {`。

### Lines 141-160

````c
  const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,
                       16, 17, 20, 21, 24, 25, 28, 29};
  const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,
                       18, 19, 22, 23, 26, 27, 30, 31};
  __v8hi __C = vec_perm((__v8hi)__A, (__v8hi)__B, __P);
  __v8hi __D = vec_perm((__v8hi)__A, (__v8hi)__B, __Q);
  return (__m128i)vec_add(__C, __D);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hadd_epi32(__m128i __A, __m128i __B) {
  const __v16qu __P = {0,  1,  2,  3,  8,  9,  10, 11,
                       16, 17, 18, 19, 24, 25, 26, 27};
  const __v16qu __Q = {4,  5,  6,  7,  12, 13, 14, 15,
                       20, 21, 22, 23, 28, 29, 30, 31};
  __v4si __C = vec_perm((__v4si)__A, (__v4si)__B, __P);
  __v4si __D = vec_perm((__v4si)__A, (__v4si)__B, __Q);
  return (__m128i)vec_add(__C, __D);
}
````
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,`。
- **L142 EN**: Adds a standalone statement or declaration: `16, 17, 20, 21, 24, 25, 28, 29};`.
  **L142 CN**: 添加一条独立语句或声明：`16, 17, 20, 21, 24, 25, 28, 29};`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,`。
- **L144 EN**: Adds a standalone statement or declaration: `18, 19, 22, 23, 26, 27, 30, 31};`.
  **L144 CN**: 添加一条独立语句或声明：`18, 19, 22, 23, 26, 27, 30, 31};`。
- **L145 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L145 CN**: 使用右侧表达式初始化变量 `__C`。
- **L146 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L146 CN**: 使用右侧表达式初始化变量 `__D`。
- **L147 EN**: Returns from the current function with `(__m128i)vec_add(__C, __D)`.
  **L147 CN**: 以 `(__m128i)vec_add(__C, __D)` 从当前函数返回。
- **L148 EN**: Closes the current lexical scope or compound statement.
  **L148 CN**: 结束当前词法作用域或复合语句块。
- **L149 EN**: Blank line separating nearby declarations or logic blocks.
  **L149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L150 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L150 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L151 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L151 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L152 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hadd_epi32(__m128i __A, __m128i __B) {`.
  **L152 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hadd_epi32(__m128i __A, __m128i __B) {`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __P = {0,  1,  2,  3,  8,  9,  10, 11,`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __P = {0,  1,  2,  3,  8,  9,  10, 11,`。
- **L154 EN**: Adds a standalone statement or declaration: `16, 17, 18, 19, 24, 25, 26, 27};`.
  **L154 CN**: 添加一条独立语句或声明：`16, 17, 18, 19, 24, 25, 26, 27};`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __Q = {4,  5,  6,  7,  12, 13, 14, 15,`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __Q = {4,  5,  6,  7,  12, 13, 14, 15,`。
- **L156 EN**: Adds a standalone statement or declaration: `20, 21, 22, 23, 28, 29, 30, 31};`.
  **L156 CN**: 添加一条独立语句或声明：`20, 21, 22, 23, 28, 29, 30, 31};`。
- **L157 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L157 CN**: 使用右侧表达式初始化变量 `__C`。
- **L158 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L158 CN**: 使用右侧表达式初始化变量 `__D`。
- **L159 EN**: Returns from the current function with `(__m128i)vec_add(__C, __D)`.
  **L159 CN**: 以 `(__m128i)vec_add(__C, __D)` 从当前函数返回。
- **L160 EN**: Closes the current lexical scope or compound statement.
  **L160 CN**: 结束当前词法作用域或复合语句块。

### Lines 161-180

````c

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hadd_pi16(__m64 __A, __m64 __B) {
  __v8hi __C = (__v8hi)(__v2du){__A, __B};
  const __v16qu __P = {0, 1, 4, 5, 8, 9, 12, 13, 0, 1, 4, 5, 8, 9, 12, 13};
  const __v16qu __Q = {2, 3, 6, 7, 10, 11, 14, 15, 2, 3, 6, 7, 10, 11, 14, 15};
  __v8hi __D = vec_perm(__C, __C, __Q);
  __C = vec_perm(__C, __C, __P);
  __C = vec_add(__C, __D);
  return (__m64)((__v2du)__C)[1];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hadd_pi32(__m64 __A, __m64 __B) {
  __v4si __C = (__v4si)(__v2du){__A, __B};
  const __v16qu __P = {0, 1, 2, 3, 8, 9, 10, 11, 0, 1, 2, 3, 8, 9, 10, 11};
  const __v16qu __Q = {4, 5, 6, 7, 12, 13, 14, 15, 4, 5, 6, 7, 12, 13, 14, 15};
  __v4si __D = vec_perm(__C, __C, __Q);
````
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L162 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L163 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L163 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L164 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hadd_pi16(__m64 __A, __m64 __B) {`.
  **L164 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hadd_pi16(__m64 __A, __m64 __B) {`。
- **L165 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L165 CN**: 使用右侧表达式初始化变量 `__C`。
- **L166 EN**: Initializes variable `__P` from the expression on the right-hand side.
  **L166 CN**: 使用右侧表达式初始化变量 `__P`。
- **L167 EN**: Initializes variable `__Q` from the expression on the right-hand side.
  **L167 CN**: 使用右侧表达式初始化变量 `__Q`。
- **L168 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L168 CN**: 使用右侧表达式初始化变量 `__D`。
- **L169 EN**: Executes a call or declaration centered on `vec_perm`.
  **L169 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L170 EN**: Executes a call or declaration centered on `vec_add`.
  **L170 CN**: 执行以 `vec_add` 为核心的调用或声明。
- **L171 EN**: Returns from the current function with `(__m64)((__v2du)__C)[1]`.
  **L171 CN**: 以 `(__m64)((__v2du)__C)[1]` 从当前函数返回。
- **L172 EN**: Closes the current lexical scope or compound statement.
  **L172 CN**: 结束当前词法作用域或复合语句块。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L174 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L174 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L175 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L175 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L176 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hadd_pi32(__m64 __A, __m64 __B) {`.
  **L176 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hadd_pi32(__m64 __A, __m64 __B) {`。
- **L177 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L177 CN**: 使用右侧表达式初始化变量 `__C`。
- **L178 EN**: Initializes variable `__P` from the expression on the right-hand side.
  **L178 CN**: 使用右侧表达式初始化变量 `__P`。
- **L179 EN**: Initializes variable `__Q` from the expression on the right-hand side.
  **L179 CN**: 使用右侧表达式初始化变量 `__Q`。
- **L180 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L180 CN**: 使用右侧表达式初始化变量 `__D`。

### Lines 181-200

````c
  __C = vec_perm(__C, __C, __P);
  __C = vec_add(__C, __D);
  return (__m64)((__v2du)__C)[1];
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hadds_epi16(__m128i __A, __m128i __B) {
  __v4si __C = {0}, __D = {0};
  __C = vec_sum4s((__v8hi)__A, __C);
  __D = vec_sum4s((__v8hi)__B, __D);
  __C = (__v4si)vec_packs(__C, __D);
  return (__m128i)__C;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hadds_pi16(__m64 __A, __m64 __B) {
  const __v4si __zero = {0};
  __v8hi __C = (__v8hi)(__v2du){__A, __B};
````
- **L181 EN**: Executes a call or declaration centered on `vec_perm`.
  **L181 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L182 EN**: Executes a call or declaration centered on `vec_add`.
  **L182 CN**: 执行以 `vec_add` 为核心的调用或声明。
- **L183 EN**: Returns from the current function with `(__m64)((__v2du)__C)[1]`.
  **L183 CN**: 以 `(__m64)((__v2du)__C)[1]` 从当前函数返回。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L186 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L187 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L187 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L188 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hadds_epi16(__m128i __A, __m128i __B) {`.
  **L188 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hadds_epi16(__m128i __A, __m128i __B) {`。
- **L189 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L189 CN**: 使用右侧表达式初始化变量 `__C`。
- **L190 EN**: Executes a call or declaration centered on `vec_sum4s`.
  **L190 CN**: 执行以 `vec_sum4s` 为核心的调用或声明。
- **L191 EN**: Executes a call or declaration centered on `vec_sum4s`.
  **L191 CN**: 执行以 `vec_sum4s` 为核心的调用或声明。
- **L192 EN**: Executes a call or declaration centered on `=`.
  **L192 CN**: 执行以 `=` 为核心的调用或声明。
- **L193 EN**: Returns from the current function with `(__m128i)__C`.
  **L193 CN**: 以 `(__m128i)__C` 从当前函数返回。
- **L194 EN**: Closes the current lexical scope or compound statement.
  **L194 CN**: 结束当前词法作用域或复合语句块。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L196 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L196 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L197 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L197 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L198 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hadds_pi16(__m64 __A, __m64 __B) {`.
  **L198 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hadds_pi16(__m64 __A, __m64 __B) {`。
- **L199 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L199 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L200 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L200 CN**: 使用右侧表达式初始化变量 `__C`。

### Lines 201-220

````c
  __v4si __D = vec_sum4s(__C, __zero);
  __C = vec_packs(__D, __D);
  return (__m64)((__v2du)__C)[1];
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hsub_epi16(__m128i __A, __m128i __B) {
  const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,
                       16, 17, 20, 21, 24, 25, 28, 29};
  const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,
                       18, 19, 22, 23, 26, 27, 30, 31};
  __v8hi __C = vec_perm((__v8hi)__A, (__v8hi)__B, __P);
  __v8hi __D = vec_perm((__v8hi)__A, (__v8hi)__B, __Q);
  return (__m128i)vec_sub(__C, __D);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hsub_epi32(__m128i __A, __m128i __B) {
````
- **L201 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L201 CN**: 使用右侧表达式初始化变量 `__D`。
- **L202 EN**: Executes a call or declaration centered on `vec_packs`.
  **L202 CN**: 执行以 `vec_packs` 为核心的调用或声明。
- **L203 EN**: Returns from the current function with `(__m64)((__v2du)__C)[1]`.
  **L203 CN**: 以 `(__m64)((__v2du)__C)[1]` 从当前函数返回。
- **L204 EN**: Closes the current lexical scope or compound statement.
  **L204 CN**: 结束当前词法作用域或复合语句块。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L206 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L207 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L207 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L208 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hsub_epi16(__m128i __A, __m128i __B) {`.
  **L208 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hsub_epi16(__m128i __A, __m128i __B) {`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,`。
- **L210 EN**: Adds a standalone statement or declaration: `16, 17, 20, 21, 24, 25, 28, 29};`.
  **L210 CN**: 添加一条独立语句或声明：`16, 17, 20, 21, 24, 25, 28, 29};`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,`。
- **L212 EN**: Adds a standalone statement or declaration: `18, 19, 22, 23, 26, 27, 30, 31};`.
  **L212 CN**: 添加一条独立语句或声明：`18, 19, 22, 23, 26, 27, 30, 31};`。
- **L213 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L213 CN**: 使用右侧表达式初始化变量 `__C`。
- **L214 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L214 CN**: 使用右侧表达式初始化变量 `__D`。
- **L215 EN**: Returns from the current function with `(__m128i)vec_sub(__C, __D)`.
  **L215 CN**: 以 `(__m128i)vec_sub(__C, __D)` 从当前函数返回。
- **L216 EN**: Closes the current lexical scope or compound statement.
  **L216 CN**: 结束当前词法作用域或复合语句块。
- **L217 EN**: Blank line separating nearby declarations or logic blocks.
  **L217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L218 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L218 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L219 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L219 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L220 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hsub_epi32(__m128i __A, __m128i __B) {`.
  **L220 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hsub_epi32(__m128i __A, __m128i __B) {`。

### Lines 221-240

````c
  const __v16qu __P = {0,  1,  2,  3,  8,  9,  10, 11,
                       16, 17, 18, 19, 24, 25, 26, 27};
  const __v16qu __Q = {4,  5,  6,  7,  12, 13, 14, 15,
                       20, 21, 22, 23, 28, 29, 30, 31};
  __v4si __C = vec_perm((__v4si)__A, (__v4si)__B, __P);
  __v4si __D = vec_perm((__v4si)__A, (__v4si)__B, __Q);
  return (__m128i)vec_sub(__C, __D);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hsub_pi16(__m64 __A, __m64 __B) {
  const __v16qu __P = {0, 1, 4, 5, 8, 9, 12, 13, 0, 1, 4, 5, 8, 9, 12, 13};
  const __v16qu __Q = {2, 3, 6, 7, 10, 11, 14, 15, 2, 3, 6, 7, 10, 11, 14, 15};
  __v8hi __C = (__v8hi)(__v2du){__A, __B};
  __v8hi __D = vec_perm(__C, __C, __Q);
  __C = vec_perm(__C, __C, __P);
  __C = vec_sub(__C, __D);
  return (__m64)((__v2du)__C)[1];
}
````
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __P = {0,  1,  2,  3,  8,  9,  10, 11,`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __P = {0,  1,  2,  3,  8,  9,  10, 11,`。
- **L222 EN**: Adds a standalone statement or declaration: `16, 17, 18, 19, 24, 25, 26, 27};`.
  **L222 CN**: 添加一条独立语句或声明：`16, 17, 18, 19, 24, 25, 26, 27};`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __Q = {4,  5,  6,  7,  12, 13, 14, 15,`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __Q = {4,  5,  6,  7,  12, 13, 14, 15,`。
- **L224 EN**: Adds a standalone statement or declaration: `20, 21, 22, 23, 28, 29, 30, 31};`.
  **L224 CN**: 添加一条独立语句或声明：`20, 21, 22, 23, 28, 29, 30, 31};`。
- **L225 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L225 CN**: 使用右侧表达式初始化变量 `__C`。
- **L226 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L226 CN**: 使用右侧表达式初始化变量 `__D`。
- **L227 EN**: Returns from the current function with `(__m128i)vec_sub(__C, __D)`.
  **L227 CN**: 以 `(__m128i)vec_sub(__C, __D)` 从当前函数返回。
- **L228 EN**: Closes the current lexical scope or compound statement.
  **L228 CN**: 结束当前词法作用域或复合语句块。
- **L229 EN**: Blank line separating nearby declarations or logic blocks.
  **L229 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L230 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L230 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L231 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L231 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L232 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hsub_pi16(__m64 __A, __m64 __B) {`.
  **L232 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hsub_pi16(__m64 __A, __m64 __B) {`。
- **L233 EN**: Initializes variable `__P` from the expression on the right-hand side.
  **L233 CN**: 使用右侧表达式初始化变量 `__P`。
- **L234 EN**: Initializes variable `__Q` from the expression on the right-hand side.
  **L234 CN**: 使用右侧表达式初始化变量 `__Q`。
- **L235 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L235 CN**: 使用右侧表达式初始化变量 `__C`。
- **L236 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L236 CN**: 使用右侧表达式初始化变量 `__D`。
- **L237 EN**: Executes a call or declaration centered on `vec_perm`.
  **L237 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L238 EN**: Executes a call or declaration centered on `vec_sub`.
  **L238 CN**: 执行以 `vec_sub` 为核心的调用或声明。
- **L239 EN**: Returns from the current function with `(__m64)((__v2du)__C)[1]`.
  **L239 CN**: 以 `(__m64)((__v2du)__C)[1]` 从当前函数返回。
- **L240 EN**: Closes the current lexical scope or compound statement.
  **L240 CN**: 结束当前词法作用域或复合语句块。

### Lines 241-260

````c

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hsub_pi32(__m64 __A, __m64 __B) {
  const __v16qu __P = {0, 1, 2, 3, 8, 9, 10, 11, 0, 1, 2, 3, 8, 9, 10, 11};
  const __v16qu __Q = {4, 5, 6, 7, 12, 13, 14, 15, 4, 5, 6, 7, 12, 13, 14, 15};
  __v4si __C = (__v4si)(__v2du){__A, __B};
  __v4si __D = vec_perm(__C, __C, __Q);
  __C = vec_perm(__C, __C, __P);
  __C = vec_sub(__C, __D);
  return (__m64)((__v2du)__C)[1];
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hsubs_epi16(__m128i __A, __m128i __B) {
  const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,
                       16, 17, 20, 21, 24, 25, 28, 29};
  const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,
                       18, 19, 22, 23, 26, 27, 30, 31};
````
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L242 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L242 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L243 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L243 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L244 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hsub_pi32(__m64 __A, __m64 __B) {`.
  **L244 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hsub_pi32(__m64 __A, __m64 __B) {`。
- **L245 EN**: Initializes variable `__P` from the expression on the right-hand side.
  **L245 CN**: 使用右侧表达式初始化变量 `__P`。
- **L246 EN**: Initializes variable `__Q` from the expression on the right-hand side.
  **L246 CN**: 使用右侧表达式初始化变量 `__Q`。
- **L247 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L247 CN**: 使用右侧表达式初始化变量 `__C`。
- **L248 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L248 CN**: 使用右侧表达式初始化变量 `__D`。
- **L249 EN**: Executes a call or declaration centered on `vec_perm`.
  **L249 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L250 EN**: Executes a call or declaration centered on `vec_sub`.
  **L250 CN**: 执行以 `vec_sub` 为核心的调用或声明。
- **L251 EN**: Returns from the current function with `(__m64)((__v2du)__C)[1]`.
  **L251 CN**: 以 `(__m64)((__v2du)__C)[1]` 从当前函数返回。
- **L252 EN**: Closes the current lexical scope or compound statement.
  **L252 CN**: 结束当前词法作用域或复合语句块。
- **L253 EN**: Blank line separating nearby declarations or logic blocks.
  **L253 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L254 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L254 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L255 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L255 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L256 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hsubs_epi16(__m128i __A, __m128i __B) {`.
  **L256 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hsubs_epi16(__m128i __A, __m128i __B) {`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __P = {0,  1,  4,  5,  8,  9,  12, 13,`。
- **L258 EN**: Adds a standalone statement or declaration: `16, 17, 20, 21, 24, 25, 28, 29};`.
  **L258 CN**: 添加一条独立语句或声明：`16, 17, 20, 21, 24, 25, 28, 29};`。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,`.
  **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __Q = {2,  3,  6,  7,  10, 11, 14, 15,`。
- **L260 EN**: Adds a standalone statement or declaration: `18, 19, 22, 23, 26, 27, 30, 31};`.
  **L260 CN**: 添加一条独立语句或声明：`18, 19, 22, 23, 26, 27, 30, 31};`。

### Lines 261-280

````c
  __v8hi __C = vec_perm((__v8hi)__A, (__v8hi)__B, __P);
  __v8hi __D = vec_perm((__v8hi)__A, (__v8hi)__B, __Q);
  return (__m128i)vec_subs(__C, __D);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_hsubs_pi16(__m64 __A, __m64 __B) {
  const __v16qu __P = {0, 1, 4, 5, 8, 9, 12, 13, 0, 1, 4, 5, 8, 9, 12, 13};
  const __v16qu __Q = {2, 3, 6, 7, 10, 11, 14, 15, 2, 3, 6, 7, 10, 11, 14, 15};
  __v8hi __C = (__v8hi)(__v2du){__A, __B};
  __v8hi __D = vec_perm(__C, __C, __P);
  __v8hi __E = vec_perm(__C, __C, __Q);
  __C = vec_subs(__D, __E);
  return (__m64)((__v2du)__C)[1];
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_shuffle_epi8(__m128i __A, __m128i __B) {
````
- **L261 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L261 CN**: 使用右侧表达式初始化变量 `__C`。
- **L262 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L262 CN**: 使用右侧表达式初始化变量 `__D`。
- **L263 EN**: Returns from the current function with `(__m128i)vec_subs(__C, __D)`.
  **L263 CN**: 以 `(__m128i)vec_subs(__C, __D)` 从当前函数返回。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L266 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L266 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L267 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L267 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L268 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_hsubs_pi16(__m64 __A, __m64 __B) {`.
  **L268 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_hsubs_pi16(__m64 __A, __m64 __B) {`。
- **L269 EN**: Initializes variable `__P` from the expression on the right-hand side.
  **L269 CN**: 使用右侧表达式初始化变量 `__P`。
- **L270 EN**: Initializes variable `__Q` from the expression on the right-hand side.
  **L270 CN**: 使用右侧表达式初始化变量 `__Q`。
- **L271 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L271 CN**: 使用右侧表达式初始化变量 `__C`。
- **L272 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L272 CN**: 使用右侧表达式初始化变量 `__D`。
- **L273 EN**: Initializes variable `__E` from the expression on the right-hand side.
  **L273 CN**: 使用右侧表达式初始化变量 `__E`。
- **L274 EN**: Executes a call or declaration centered on `vec_subs`.
  **L274 CN**: 执行以 `vec_subs` 为核心的调用或声明。
- **L275 EN**: Returns from the current function with `(__m64)((__v2du)__C)[1]`.
  **L275 CN**: 以 `(__m64)((__v2du)__C)[1]` 从当前函数返回。
- **L276 EN**: Closes the current lexical scope or compound statement.
  **L276 CN**: 结束当前词法作用域或复合语句块。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L278 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L278 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L279 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L279 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L280 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_shuffle_epi8(__m128i __A, __m128i __B) {`.
  **L280 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_shuffle_epi8(__m128i __A, __m128i __B) {`。

### Lines 281-300

````c
  const __v16qi __zero = {0};
  __vector __bool char __select = vec_cmplt((__v16qi)__B, __zero);
  __v16qi __C = vec_perm((__v16qi)__A, (__v16qi)__A, (__v16qu)__B);
  return (__m128i)vec_sel(__C, __zero, __select);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_shuffle_pi8(__m64 __A, __m64 __B) {
  const __v16qi __zero = {0};
  __v16qi __C = (__v16qi)(__v2du){__A, __A};
  __v16qi __D = (__v16qi)(__v2du){__B, __B};
  __vector __bool char __select = vec_cmplt((__v16qi)__D, __zero);
  __C = vec_perm((__v16qi)__C, (__v16qi)__C, (__v16qu)__D);
  __C = vec_sel(__C, __zero, __select);
  return (__m64)((__v2du)(__C))[0];
}

#ifdef _ARCH_PWR8
extern __inline __m128i
````
- **L281 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L281 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L282 EN**: Initializes variable `__select` from the expression on the right-hand side.
  **L282 CN**: 使用右侧表达式初始化变量 `__select`。
- **L283 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L283 CN**: 使用右侧表达式初始化变量 `__C`。
- **L284 EN**: Returns from the current function with `(__m128i)vec_sel(__C, __zero, __select)`.
  **L284 CN**: 以 `(__m128i)vec_sel(__C, __zero, __select)` 从当前函数返回。
- **L285 EN**: Closes the current lexical scope or compound statement.
  **L285 CN**: 结束当前词法作用域或复合语句块。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L287 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L287 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L288 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L288 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L289 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_shuffle_pi8(__m64 __A, __m64 __B) {`.
  **L289 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_shuffle_pi8(__m64 __A, __m64 __B) {`。
- **L290 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L290 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L291 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L291 CN**: 使用右侧表达式初始化变量 `__C`。
- **L292 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L292 CN**: 使用右侧表达式初始化变量 `__D`。
- **L293 EN**: Initializes variable `__select` from the expression on the right-hand side.
  **L293 CN**: 使用右侧表达式初始化变量 `__select`。
- **L294 EN**: Executes a call or declaration centered on `vec_perm`.
  **L294 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L295 EN**: Executes a call or declaration centered on `vec_sel`.
  **L295 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L296 EN**: Returns from the current function with `(__m64)((__v2du)(__C))[0]`.
  **L296 CN**: 以 `(__m64)((__v2du)(__C))[0]` 从当前函数返回。
- **L297 EN**: Closes the current lexical scope or compound statement.
  **L297 CN**: 结束当前词法作用域或复合语句块。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L299 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L299 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L300 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L300 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。

### Lines 301-320

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sign_epi8(__m128i __A, __m128i __B) {
  const __v16qi __zero = {0};
  __v16qi __selectneg = (__v16qi)vec_cmplt((__v16qi)__B, __zero);
  __v16qi __selectpos =
      (__v16qi)vec_neg((__v16qi)vec_cmpgt((__v16qi)__B, __zero));
  __v16qi __conv = vec_add(__selectneg, __selectpos);
  return (__m128i)vec_mul((__v16qi)__A, (__v16qi)__conv);
}
#endif

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sign_epi16(__m128i __A, __m128i __B) {
  const __v8hi __zero = {0};
  __v8hi __selectneg = (__v8hi)vec_cmplt((__v8hi)__B, __zero);
  __v8hi __selectpos = (__v8hi)vec_neg((__v8hi)vec_cmpgt((__v8hi)__B, __zero));
  __v8hi __conv = vec_add(__selectneg, __selectpos);
  return (__m128i)vec_mul((__v8hi)__A, (__v8hi)__conv);
````
- **L301 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L301 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L302 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sign_epi8(__m128i __A, __m128i __B) {`.
  **L302 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sign_epi8(__m128i __A, __m128i __B) {`。
- **L303 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L303 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L304 EN**: Initializes variable `__selectneg` from the expression on the right-hand side.
  **L304 CN**: 使用右侧表达式初始化变量 `__selectneg`。
- **L305 EN**: Continues the surrounding expression or declaration: `__v16qi __selectpos =`.
  **L305 CN**: 继续构造周围的表达式或声明：`__v16qi __selectpos =`。
- **L306 EN**: Executes a call or declaration centered on `statement`.
  **L306 CN**: 执行以 `statement` 为核心的调用或声明。
- **L307 EN**: Initializes variable `__conv` from the expression on the right-hand side.
  **L307 CN**: 使用右侧表达式初始化变量 `__conv`。
- **L308 EN**: Returns from the current function with `(__m128i)vec_mul((__v16qi)__A, (__v16qi)__conv)`.
  **L308 CN**: 以 `(__m128i)vec_mul((__v16qi)__A, (__v16qi)__conv)` 从当前函数返回。
- **L309 EN**: Closes the current lexical scope or compound statement.
  **L309 CN**: 结束当前词法作用域或复合语句块。
- **L310 EN**: Closes the current preprocessor conditional block.
  **L310 CN**: 结束当前预处理条件块。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L312 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L312 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L313 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L313 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L314 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L314 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L315 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sign_epi16(__m128i __A, __m128i __B) {`.
  **L315 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sign_epi16(__m128i __A, __m128i __B) {`。
- **L316 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L316 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L317 EN**: Initializes variable `__selectneg` from the expression on the right-hand side.
  **L317 CN**: 使用右侧表达式初始化变量 `__selectneg`。
- **L318 EN**: Initializes variable `__selectpos` from the expression on the right-hand side.
  **L318 CN**: 使用右侧表达式初始化变量 `__selectpos`。
- **L319 EN**: Initializes variable `__conv` from the expression on the right-hand side.
  **L319 CN**: 使用右侧表达式初始化变量 `__conv`。
- **L320 EN**: Returns from the current function with `(__m128i)vec_mul((__v8hi)__A, (__v8hi)__conv)`.
  **L320 CN**: 以 `(__m128i)vec_mul((__v8hi)__A, (__v8hi)__conv)` 从当前函数返回。

### Lines 321-340

````c
}
#endif

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sign_epi32(__m128i __A, __m128i __B) {
  const __v4si __zero = {0};
  __v4si __selectneg = (__v4si)vec_cmplt((__v4si)__B, __zero);
  __v4si __selectpos = (__v4si)vec_neg((__v4si)vec_cmpgt((__v4si)__B, __zero));
  __v4si __conv = vec_add(__selectneg, __selectpos);
  return (__m128i)vec_mul((__v4si)__A, (__v4si)__conv);
}
#endif

#ifdef _ARCH_PWR8
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sign_pi8(__m64 __A, __m64 __B) {
  const __v16qi __zero = {0};
````
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。
- **L322 EN**: Closes the current preprocessor conditional block.
  **L322 CN**: 结束当前预处理条件块。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L324 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L324 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L325 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L325 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L326 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L326 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L327 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sign_epi32(__m128i __A, __m128i __B) {`.
  **L327 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sign_epi32(__m128i __A, __m128i __B) {`。
- **L328 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L328 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L329 EN**: Initializes variable `__selectneg` from the expression on the right-hand side.
  **L329 CN**: 使用右侧表达式初始化变量 `__selectneg`。
- **L330 EN**: Initializes variable `__selectpos` from the expression on the right-hand side.
  **L330 CN**: 使用右侧表达式初始化变量 `__selectpos`。
- **L331 EN**: Initializes variable `__conv` from the expression on the right-hand side.
  **L331 CN**: 使用右侧表达式初始化变量 `__conv`。
- **L332 EN**: Returns from the current function with `(__m128i)vec_mul((__v4si)__A, (__v4si)__conv)`.
  **L332 CN**: 以 `(__m128i)vec_mul((__v4si)__A, (__v4si)__conv)` 从当前函数返回。
- **L333 EN**: Closes the current lexical scope or compound statement.
  **L333 CN**: 结束当前词法作用域或复合语句块。
- **L334 EN**: Closes the current preprocessor conditional block.
  **L334 CN**: 结束当前预处理条件块。
- **L335 EN**: Blank line separating nearby declarations or logic blocks.
  **L335 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L336 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L336 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L337 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L337 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L338 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L338 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L339 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sign_pi8(__m64 __A, __m64 __B) {`.
  **L339 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sign_pi8(__m64 __A, __m64 __B) {`。
- **L340 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L340 CN**: 使用右侧表达式初始化变量 `__zero`。

### Lines 341-360

````c
  __v16qi __C = (__v16qi)(__v2du){__A, __A};
  __v16qi __D = (__v16qi)(__v2du){__B, __B};
  __C = (__v16qi)_mm_sign_epi8((__m128i)__C, (__m128i)__D);
  return (__m64)((__v2du)(__C))[0];
}
#endif

#ifdef _ARCH_PWR8
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sign_pi16(__m64 __A, __m64 __B) {
  const __v8hi __zero = {0};
  __v8hi __C = (__v8hi)(__v2du){__A, __A};
  __v8hi __D = (__v8hi)(__v2du){__B, __B};
  __C = (__v8hi)_mm_sign_epi16((__m128i)__C, (__m128i)__D);
  return (__m64)((__v2du)(__C))[0];
}
#endif

#ifdef _ARCH_PWR8
````
- **L341 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L341 CN**: 使用右侧表达式初始化变量 `__C`。
- **L342 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L342 CN**: 使用右侧表达式初始化变量 `__D`。
- **L343 EN**: Executes a call or declaration centered on `=`.
  **L343 CN**: 执行以 `=` 为核心的调用或声明。
- **L344 EN**: Returns from the current function with `(__m64)((__v2du)(__C))[0]`.
  **L344 CN**: 以 `(__m64)((__v2du)(__C))[0]` 从当前函数返回。
- **L345 EN**: Closes the current lexical scope or compound statement.
  **L345 CN**: 结束当前词法作用域或复合语句块。
- **L346 EN**: Closes the current preprocessor conditional block.
  **L346 CN**: 结束当前预处理条件块。
- **L347 EN**: Blank line separating nearby declarations or logic blocks.
  **L347 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L348 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L348 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L349 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L349 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L350 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L350 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L351 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sign_pi16(__m64 __A, __m64 __B) {`.
  **L351 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sign_pi16(__m64 __A, __m64 __B) {`。
- **L352 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L352 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L353 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L353 CN**: 使用右侧表达式初始化变量 `__C`。
- **L354 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L354 CN**: 使用右侧表达式初始化变量 `__D`。
- **L355 EN**: Executes a call or declaration centered on `=`.
  **L355 CN**: 执行以 `=` 为核心的调用或声明。
- **L356 EN**: Returns from the current function with `(__m64)((__v2du)(__C))[0]`.
  **L356 CN**: 以 `(__m64)((__v2du)(__C))[0]` 从当前函数返回。
- **L357 EN**: Closes the current lexical scope or compound statement.
  **L357 CN**: 结束当前词法作用域或复合语句块。
- **L358 EN**: Closes the current preprocessor conditional block.
  **L358 CN**: 结束当前预处理条件块。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L360 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L360 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。

### Lines 361-380

````c
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sign_pi32(__m64 __A, __m64 __B) {
  const __v4si __zero = {0};
  __v4si __C = (__v4si)(__v2du){__A, __A};
  __v4si __D = (__v4si)(__v2du){__B, __B};
  __C = (__v4si)_mm_sign_epi32((__m128i)__C, (__m128i)__D);
  return (__m64)((__v2du)(__C))[0];
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_maddubs_epi16(__m128i __A, __m128i __B) {
  __v8hi __unsigned = vec_splats((signed short)0x00ff);
  __v8hi __C = vec_and(vec_unpackh((__v16qi)__A), __unsigned);
  __v8hi __D = vec_and(vec_unpackl((__v16qi)__A), __unsigned);
  __v8hi __E = vec_unpackh((__v16qi)__B);
  __v8hi __F = vec_unpackl((__v16qi)__B);
  __C = vec_mul(__C, __E);
````
- **L361 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L361 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L362 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L362 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L363 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sign_pi32(__m64 __A, __m64 __B) {`.
  **L363 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sign_pi32(__m64 __A, __m64 __B) {`。
- **L364 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L364 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L365 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L365 CN**: 使用右侧表达式初始化变量 `__C`。
- **L366 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L366 CN**: 使用右侧表达式初始化变量 `__D`。
- **L367 EN**: Executes a call or declaration centered on `=`.
  **L367 CN**: 执行以 `=` 为核心的调用或声明。
- **L368 EN**: Returns from the current function with `(__m64)((__v2du)(__C))[0]`.
  **L368 CN**: 以 `(__m64)((__v2du)(__C))[0]` 从当前函数返回。
- **L369 EN**: Closes the current lexical scope or compound statement.
  **L369 CN**: 结束当前词法作用域或复合语句块。
- **L370 EN**: Closes the current preprocessor conditional block.
  **L370 CN**: 结束当前预处理条件块。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L372 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L372 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L373 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L373 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L374 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maddubs_epi16(__m128i __A, __m128i __B) {`.
  **L374 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maddubs_epi16(__m128i __A, __m128i __B) {`。
- **L375 EN**: Initializes variable `__unsigned` from the expression on the right-hand side.
  **L375 CN**: 使用右侧表达式初始化变量 `__unsigned`。
- **L376 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L376 CN**: 使用右侧表达式初始化变量 `__C`。
- **L377 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L377 CN**: 使用右侧表达式初始化变量 `__D`。
- **L378 EN**: Initializes variable `__E` from the expression on the right-hand side.
  **L378 CN**: 使用右侧表达式初始化变量 `__E`。
- **L379 EN**: Initializes variable `__F` from the expression on the right-hand side.
  **L379 CN**: 使用右侧表达式初始化变量 `__F`。
- **L380 EN**: Executes a call or declaration centered on `vec_mul`.
  **L380 CN**: 执行以 `vec_mul` 为核心的调用或声明。

### Lines 381-400

````c
  __D = vec_mul(__D, __F);
  const __v16qu __odds = {0,  1,  4,  5,  8,  9,  12, 13,
                          16, 17, 20, 21, 24, 25, 28, 29};
  const __v16qu __evens = {2,  3,  6,  7,  10, 11, 14, 15,
                           18, 19, 22, 23, 26, 27, 30, 31};
  __E = vec_perm(__C, __D, __odds);
  __F = vec_perm(__C, __D, __evens);
  return (__m128i)vec_adds(__E, __F);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_maddubs_pi16(__m64 __A, __m64 __B) {
  __v8hi __C = (__v8hi)(__v2du){__A, __A};
  __C = vec_unpackl((__v16qi)__C);
  const __v8hi __unsigned = vec_splats((signed short)0x00ff);
  __C = vec_and(__C, __unsigned);
  __v8hi __D = (__v8hi)(__v2du){__B, __B};
  __D = vec_unpackl((__v16qi)__D);
  __D = vec_mul(__C, __D);
````
- **L381 EN**: Executes a call or declaration centered on `vec_mul`.
  **L381 CN**: 执行以 `vec_mul` 为核心的调用或声明。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __odds = {0,  1,  4,  5,  8,  9,  12, 13,`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __odds = {0,  1,  4,  5,  8,  9,  12, 13,`。
- **L383 EN**: Adds a standalone statement or declaration: `16, 17, 20, 21, 24, 25, 28, 29};`.
  **L383 CN**: 添加一条独立语句或声明：`16, 17, 20, 21, 24, 25, 28, 29};`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __evens = {2,  3,  6,  7,  10, 11, 14, 15,`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __evens = {2,  3,  6,  7,  10, 11, 14, 15,`。
- **L385 EN**: Adds a standalone statement or declaration: `18, 19, 22, 23, 26, 27, 30, 31};`.
  **L385 CN**: 添加一条独立语句或声明：`18, 19, 22, 23, 26, 27, 30, 31};`。
- **L386 EN**: Executes a call or declaration centered on `vec_perm`.
  **L386 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L387 EN**: Executes a call or declaration centered on `vec_perm`.
  **L387 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L388 EN**: Returns from the current function with `(__m128i)vec_adds(__E, __F)`.
  **L388 CN**: 以 `(__m128i)vec_adds(__E, __F)` 从当前函数返回。
- **L389 EN**: Closes the current lexical scope or compound statement.
  **L389 CN**: 结束当前词法作用域或复合语句块。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L391 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L391 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L392 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L392 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L393 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maddubs_pi16(__m64 __A, __m64 __B) {`.
  **L393 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maddubs_pi16(__m64 __A, __m64 __B) {`。
- **L394 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L394 CN**: 使用右侧表达式初始化变量 `__C`。
- **L395 EN**: Executes a call or declaration centered on `vec_unpackl`.
  **L395 CN**: 执行以 `vec_unpackl` 为核心的调用或声明。
- **L396 EN**: Initializes variable `__unsigned` from the expression on the right-hand side.
  **L396 CN**: 使用右侧表达式初始化变量 `__unsigned`。
- **L397 EN**: Executes a call or declaration centered on `vec_and`.
  **L397 CN**: 执行以 `vec_and` 为核心的调用或声明。
- **L398 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L398 CN**: 使用右侧表达式初始化变量 `__D`。
- **L399 EN**: Executes a call or declaration centered on `vec_unpackl`.
  **L399 CN**: 执行以 `vec_unpackl` 为核心的调用或声明。
- **L400 EN**: Executes a call or declaration centered on `vec_mul`.
  **L400 CN**: 执行以 `vec_mul` 为核心的调用或声明。

### Lines 401-420

````c
  const __v16qu __odds = {0,  1,  4,  5,  8,  9,  12, 13,
                          16, 17, 20, 21, 24, 25, 28, 29};
  const __v16qu __evens = {2,  3,  6,  7,  10, 11, 14, 15,
                           18, 19, 22, 23, 26, 27, 30, 31};
  __C = vec_perm(__D, __D, __odds);
  __D = vec_perm(__D, __D, __evens);
  __C = vec_adds(__C, __D);
  return (__m64)((__v2du)(__C))[0];
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mulhrs_epi16(__m128i __A, __m128i __B) {
  __v4si __C = vec_unpackh((__v8hi)__A);
  __v4si __D = vec_unpackh((__v8hi)__B);
  __C = vec_mul(__C, __D);
  __D = vec_unpackl((__v8hi)__A);
  __v4si __E = vec_unpackl((__v8hi)__B);
  __D = vec_mul(__D, __E);
  const __v4su __shift = vec_splats((unsigned int)14);
````
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __odds = {0,  1,  4,  5,  8,  9,  12, 13,`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __odds = {0,  1,  4,  5,  8,  9,  12, 13,`。
- **L402 EN**: Adds a standalone statement or declaration: `16, 17, 20, 21, 24, 25, 28, 29};`.
  **L402 CN**: 添加一条独立语句或声明：`16, 17, 20, 21, 24, 25, 28, 29};`。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __evens = {2,  3,  6,  7,  10, 11, 14, 15,`.
  **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __evens = {2,  3,  6,  7,  10, 11, 14, 15,`。
- **L404 EN**: Adds a standalone statement or declaration: `18, 19, 22, 23, 26, 27, 30, 31};`.
  **L404 CN**: 添加一条独立语句或声明：`18, 19, 22, 23, 26, 27, 30, 31};`。
- **L405 EN**: Executes a call or declaration centered on `vec_perm`.
  **L405 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L406 EN**: Executes a call or declaration centered on `vec_perm`.
  **L406 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L407 EN**: Executes a call or declaration centered on `vec_adds`.
  **L407 CN**: 执行以 `vec_adds` 为核心的调用或声明。
- **L408 EN**: Returns from the current function with `(__m64)((__v2du)(__C))[0]`.
  **L408 CN**: 以 `(__m64)((__v2du)(__C))[0]` 从当前函数返回。
- **L409 EN**: Closes the current lexical scope or compound statement.
  **L409 CN**: 结束当前词法作用域或复合语句块。
- **L410 EN**: Blank line separating nearby declarations or logic blocks.
  **L410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L411 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L411 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L412 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L412 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L413 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mulhrs_epi16(__m128i __A, __m128i __B) {`.
  **L413 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mulhrs_epi16(__m128i __A, __m128i __B) {`。
- **L414 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L414 CN**: 使用右侧表达式初始化变量 `__C`。
- **L415 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L415 CN**: 使用右侧表达式初始化变量 `__D`。
- **L416 EN**: Executes a call or declaration centered on `vec_mul`.
  **L416 CN**: 执行以 `vec_mul` 为核心的调用或声明。
- **L417 EN**: Executes a call or declaration centered on `vec_unpackl`.
  **L417 CN**: 执行以 `vec_unpackl` 为核心的调用或声明。
- **L418 EN**: Initializes variable `__E` from the expression on the right-hand side.
  **L418 CN**: 使用右侧表达式初始化变量 `__E`。
- **L419 EN**: Executes a call or declaration centered on `vec_mul`.
  **L419 CN**: 执行以 `vec_mul` 为核心的调用或声明。
- **L420 EN**: Initializes variable `__shift` from the expression on the right-hand side.
  **L420 CN**: 使用右侧表达式初始化变量 `__shift`。

### Lines 421-440

````c
  __C = vec_sr(__C, __shift);
  __D = vec_sr(__D, __shift);
  const __v4si __ones = vec_splats((signed int)1);
  __C = vec_add(__C, __ones);
  __C = vec_sr(__C, (__v4su)__ones);
  __D = vec_add(__D, __ones);
  __D = vec_sr(__D, (__v4su)__ones);
  return (__m128i)vec_pack(__C, __D);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mulhrs_pi16(__m64 __A, __m64 __B) {
  __v4si __C = (__v4si)(__v2du){__A, __A};
  __C = vec_unpackh((__v8hi)__C);
  __v4si __D = (__v4si)(__v2du){__B, __B};
  __D = vec_unpackh((__v8hi)__D);
  __C = vec_mul(__C, __D);
  const __v4su __shift = vec_splats((unsigned int)14);
  __C = vec_sr(__C, __shift);
````
- **L421 EN**: Executes a call or declaration centered on `vec_sr`.
  **L421 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L422 EN**: Executes a call or declaration centered on `vec_sr`.
  **L422 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L423 EN**: Initializes variable `__ones` from the expression on the right-hand side.
  **L423 CN**: 使用右侧表达式初始化变量 `__ones`。
- **L424 EN**: Executes a call or declaration centered on `vec_add`.
  **L424 CN**: 执行以 `vec_add` 为核心的调用或声明。
- **L425 EN**: Executes a call or declaration centered on `vec_sr`.
  **L425 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L426 EN**: Executes a call or declaration centered on `vec_add`.
  **L426 CN**: 执行以 `vec_add` 为核心的调用或声明。
- **L427 EN**: Executes a call or declaration centered on `vec_sr`.
  **L427 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L428 EN**: Returns from the current function with `(__m128i)vec_pack(__C, __D)`.
  **L428 CN**: 以 `(__m128i)vec_pack(__C, __D)` 从当前函数返回。
- **L429 EN**: Closes the current lexical scope or compound statement.
  **L429 CN**: 结束当前词法作用域或复合语句块。
- **L430 EN**: Blank line separating nearby declarations or logic blocks.
  **L430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L431 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L431 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L432 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L432 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L433 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mulhrs_pi16(__m64 __A, __m64 __B) {`.
  **L433 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mulhrs_pi16(__m64 __A, __m64 __B) {`。
- **L434 EN**: Initializes variable `__C` from the expression on the right-hand side.
  **L434 CN**: 使用右侧表达式初始化变量 `__C`。
- **L435 EN**: Executes a call or declaration centered on `vec_unpackh`.
  **L435 CN**: 执行以 `vec_unpackh` 为核心的调用或声明。
- **L436 EN**: Initializes variable `__D` from the expression on the right-hand side.
  **L436 CN**: 使用右侧表达式初始化变量 `__D`。
- **L437 EN**: Executes a call or declaration centered on `vec_unpackh`.
  **L437 CN**: 执行以 `vec_unpackh` 为核心的调用或声明。
- **L438 EN**: Executes a call or declaration centered on `vec_mul`.
  **L438 CN**: 执行以 `vec_mul` 为核心的调用或声明。
- **L439 EN**: Initializes variable `__shift` from the expression on the right-hand side.
  **L439 CN**: 使用右侧表达式初始化变量 `__shift`。
- **L440 EN**: Executes a call or declaration centered on `vec_sr`.
  **L440 CN**: 执行以 `vec_sr` 为核心的调用或声明。

### Lines 441-453

````c
  const __v4si __ones = vec_splats((signed int)1);
  __C = vec_add(__C, __ones);
  __C = vec_sr(__C, (__v4su)__ones);
  __v8hi __E = vec_pack(__C, __D);
  return (__m64)((__v2du)(__E))[0];
}

#else
#include_next <tmmintrin.h>
#endif /* defined(__powerpc64__) &&                                            \
        *   (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX)) */

#endif /* TMMINTRIN_H_ */
````
- **L441 EN**: Initializes variable `__ones` from the expression on the right-hand side.
  **L441 CN**: 使用右侧表达式初始化变量 `__ones`。
- **L442 EN**: Executes a call or declaration centered on `vec_add`.
  **L442 CN**: 执行以 `vec_add` 为核心的调用或声明。
- **L443 EN**: Executes a call or declaration centered on `vec_sr`.
  **L443 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L444 EN**: Initializes variable `__E` from the expression on the right-hand side.
  **L444 CN**: 使用右侧表达式初始化变量 `__E`。
- **L445 EN**: Returns from the current function with `(__m64)((__v2du)(__E))[0]`.
  **L445 CN**: 以 `(__m64)((__v2du)(__E))[0]` 从当前函数返回。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L448 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L449 EN**: Includes <tmmintrin.h> to access related header declarations.
  **L449 CN**: 引入 <tmmintrin.h> 以使用相关头文件声明。
- **L450 EN**: Closes the current preprocessor conditional block.
  **L450 CN**: 结束当前预处理条件块。
- **L451 EN**: Comment explains nearby logic, constraints, or intent: `(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`.
  **L451 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`。
- **L452 EN**: Blank line separating nearby declarations or logic blocks.
  **L452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L453 EN**: Closes the current preprocessor conditional block.
  **L453 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **PowerPC vector or system interfaces / PowerPC 向量或系统接口**
- **wrapper forwarding layers / 包装转发层**
- **Header composition through includes / 通过 include 组合头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `altivec.h`: Provides related header declarations. / 提供相关头文件声明。
  - `pmmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
  - `tmmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `NO_WARN_X86_INTRINSICS`, `TMMINTRIN_H_`, `__powerpc64__`, `__linux__`, `__FreeBSD__`, `_AIX`, `__LITTLE_ENDIAN__`, `_ARCH_PWR8`
- **External builtins / 外部 builtin**: `__builtin_constant_p`
