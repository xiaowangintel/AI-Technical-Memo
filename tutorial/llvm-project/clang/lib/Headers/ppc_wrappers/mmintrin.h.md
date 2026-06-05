# mmintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/ppc_wrappers/mmintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implementation of MMX intrinsics on PowerPC.
- **Purpose (CN)**: 该头文件主要作用是：Implementation of MMX intrinsics on PowerPC。
- **Line Count / 行数**: 1453

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- mmintrin.h - Implementation of MMX intrinsics on PowerPC ---------===
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
/* This header file is to help porting code using Intel intrinsics
   explicitly from x86_64 to powerpc64/powerpc64le.

   Since PowerPC target doesn't support native 64-bit vector type, we
   typedef __m64 to 64-bit unsigned long long in MMX intrinsics, which
   works well for _si64 and some _pi32 operations.

   For _pi16 and _pi8 operations, it's better to transfer __m64 into
   128-bit PowerPC vector first. Power8 introduced direct register
   move instructions which helps for more efficient implementation.

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
- **L14 EN**: Comment explains nearby logic, constraints, or intent: `This header file is to help porting code using Intel intrinsics`.
  **L14 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This header file is to help porting code using Intel intrinsics`。
- **L15 EN**: Continues the surrounding expression or declaration: `explicitly from x86_64 to powerpc64/powerpc64le.`.
  **L15 CN**: 继续构造周围的表达式或声明：`explicitly from x86_64 to powerpc64/powerpc64le.`。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Continues the surrounding expression or declaration: `Since PowerPC target doesn't support native 64-bit vector type, we`.
  **L17 CN**: 继续构造周围的表达式或声明：`Since PowerPC target doesn't support native 64-bit vector type, we`。
- **L18 EN**: Introduces an alias or helper declaration: `typedef __m64 to 64-bit unsigned long long in MMX intrinsics, which`.
  **L18 CN**: 引入一条别名或辅助声明：`typedef __m64 to 64-bit unsigned long long in MMX intrinsics, which`。
- **L19 EN**: Continues the surrounding expression or declaration: `works well for _si64 and some _pi32 operations.`.
  **L19 CN**: 继续构造周围的表达式或声明：`works well for _si64 and some _pi32 operations.`。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L21 EN**: Continues the surrounding expression or declaration: `For _pi16 and _pi8 operations, it's better to transfer __m64 into`.
  **L21 CN**: 继续构造周围的表达式或声明：`For _pi16 and _pi8 operations, it's better to transfer __m64 into`。
- **L22 EN**: Continues the surrounding expression or declaration: `128-bit PowerPC vector first. Power8 introduced direct register`.
  **L22 CN**: 继续构造周围的表达式或声明：`128-bit PowerPC vector first. Power8 introduced direct register`。
- **L23 EN**: Continues the surrounding expression or declaration: `move instructions which helps for more efficient implementation.`.
  **L23 CN**: 继续构造周围的表达式或声明：`move instructions which helps for more efficient implementation.`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 25-48

````c
   It's user's responsibility to determine if the results of such port
   are acceptable or further changes are needed. Please note that much
   code using Intel intrinsics CAN BE REWRITTEN in more portable and
   efficient standard C or GNU C extensions with 64-bit scalar
   operations, or 128-bit SSE/Altivec operations, which are more
   recommended. */
#error                                                                         \
    "Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."
#endif

#ifndef _MMINTRIN_H_INCLUDED
#define _MMINTRIN_H_INCLUDED

#if defined(__powerpc64__) &&                                                  \
    (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))

#include <altivec.h>
/* The Intel API is flexible enough that we must allow aliasing with other
   vector types, and their scalar components.  */
typedef __attribute__((__aligned__(8))) unsigned long long __m64;

typedef __attribute__((__aligned__(8))) union {
  __m64 as_m64;
  char as_char[8];
````
- **L25 EN**: Continues the surrounding expression or declaration: `It's user's responsibility to determine if the results of such port`.
  **L25 CN**: 继续构造周围的表达式或声明：`It's user's responsibility to determine if the results of such port`。
- **L26 EN**: Continues the surrounding expression or declaration: `are acceptable or further changes are needed. Please note that much`.
  **L26 CN**: 继续构造周围的表达式或声明：`are acceptable or further changes are needed. Please note that much`。
- **L27 EN**: Continues the surrounding expression or declaration: `code using Intel intrinsics CAN BE REWRITTEN in more portable and`.
  **L27 CN**: 继续构造周围的表达式或声明：`code using Intel intrinsics CAN BE REWRITTEN in more portable and`。
- **L28 EN**: Continues the surrounding expression or declaration: `efficient standard C or GNU C extensions with 64-bit scalar`.
  **L28 CN**: 继续构造周围的表达式或声明：`efficient standard C or GNU C extensions with 64-bit scalar`。
- **L29 EN**: Continues the surrounding expression or declaration: `operations, or 128-bit SSE/Altivec operations, which are more`.
  **L29 CN**: 继续构造周围的表达式或声明：`operations, or 128-bit SSE/Altivec operations, which are more`。
- **L30 EN**: Continues the surrounding expression or declaration: `recommended. */`.
  **L30 CN**: 继续构造周围的表达式或声明：`recommended. */`。
- **L31 EN**: Emits a compilation error for an unsupported configuration: `#error                                                                         \`.
  **L31 CN**: 为不受支持的配置触发编译错误：`#error                                                                         \`。
- **L32 EN**: Continues the surrounding expression or declaration: `"Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."`.
  **L32 CN**: 继续构造周围的表达式或声明：`"Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."`。
- **L33 EN**: Closes the current preprocessor conditional block.
  **L33 CN**: 结束当前预处理条件块。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Starts a preprocessor conditional block: `#ifndef _MMINTRIN_H_INCLUDED`.
  **L35 CN**: 开始一个预处理条件块：`#ifndef _MMINTRIN_H_INCLUDED`。
- **L36 EN**: Defines macro `_MMINTRIN_H_INCLUDED` for conditional compilation, shorthand, or API generation.
  **L36 CN**: 定义宏 `_MMINTRIN_H_INCLUDED`，用于条件编译、简写或 API 生成。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Starts a preprocessor conditional block: `#if defined(__powerpc64__) &&                                                  \`.
  **L38 CN**: 开始一个预处理条件块：`#if defined(__powerpc64__) &&                                                  \`。
- **L39 EN**: Continues logic associated with callable symbol `defined`.
  **L39 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L41 EN**: Includes <altivec.h> to access related header declarations.
  **L41 CN**: 引入 <altivec.h> 以使用相关头文件声明。
- **L42 EN**: Comment explains nearby logic, constraints, or intent: `The Intel API is flexible enough that we must allow aliasing with other`.
  **L42 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The Intel API is flexible enough that we must allow aliasing with other`。
- **L43 EN**: Continues the surrounding expression or declaration: `vector types, and their scalar components.  */`.
  **L43 CN**: 继续构造周围的表达式或声明：`vector types, and their scalar components.  */`。
- **L44 EN**: Introduces an alias or helper declaration: `typedef __attribute__((__aligned__(8))) unsigned long long __m64;`.
  **L44 CN**: 引入一条别名或辅助声明：`typedef __attribute__((__aligned__(8))) unsigned long long __m64;`。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L46 EN**: Introduces an alias or helper declaration: `typedef __attribute__((__aligned__(8))) union {`.
  **L46 CN**: 引入一条别名或辅助声明：`typedef __attribute__((__aligned__(8))) union {`。
- **L47 EN**: Adds a standalone statement or declaration: `__m64 as_m64;`.
  **L47 CN**: 添加一条独立语句或声明：`__m64 as_m64;`。
- **L48 EN**: Adds a standalone statement or declaration: `char as_char[8];`.
  **L48 CN**: 添加一条独立语句或声明：`char as_char[8];`。

### Lines 49-72

````c
  signed char as_signed_char[8];
  short as_short[4];
  int as_int[2];
  long long as_long_long;
  float as_float[2];
  double as_double;
} __m64_union;

/* Empty the multimedia state.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_empty(void) {
  /* nothing to do on PowerPC.  */
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_empty(void) {
  /* nothing to do on PowerPC.  */
}

/* Convert I to a __m64 object.  The integer is zero-extended to 64-bits.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L49 EN**: Adds a standalone statement or declaration: `signed char as_signed_char[8];`.
  **L49 CN**: 添加一条独立语句或声明：`signed char as_signed_char[8];`。
- **L50 EN**: Adds a standalone statement or declaration: `short as_short[4];`.
  **L50 CN**: 添加一条独立语句或声明：`short as_short[4];`。
- **L51 EN**: Adds a standalone statement or declaration: `int as_int[2];`.
  **L51 CN**: 添加一条独立语句或声明：`int as_int[2];`。
- **L52 EN**: Adds a standalone statement or declaration: `long long as_long_long;`.
  **L52 CN**: 添加一条独立语句或声明：`long long as_long_long;`。
- **L53 EN**: Adds a standalone statement or declaration: `float as_float[2];`.
  **L53 CN**: 添加一条独立语句或声明：`float as_float[2];`。
- **L54 EN**: Adds a standalone statement or declaration: `double as_double;`.
  **L54 CN**: 添加一条独立语句或声明：`double as_double;`。
- **L55 EN**: Adds a standalone statement or declaration: `} __m64_union;`.
  **L55 CN**: 添加一条独立语句或声明：`} __m64_union;`。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `Empty the multimedia state.`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Empty the multimedia state.`。
- **L58 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L58 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L59 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L59 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L60 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_empty(void) {`.
  **L60 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_empty(void) {`。
- **L61 EN**: Comment explains nearby logic, constraints, or intent: `nothing to do on PowerPC.`.
  **L61 CN**: 注释解释附近代码的逻辑、约束或设计意图：`nothing to do on PowerPC.`。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L64 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L65 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L65 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L66 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_empty(void) {`.
  **L66 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_empty(void) {`。
- **L67 EN**: Comment explains nearby logic, constraints, or intent: `nothing to do on PowerPC.`.
  **L67 CN**: 注释解释附近代码的逻辑、约束或设计意图：`nothing to do on PowerPC.`。
- **L68 EN**: Closes the current lexical scope or compound statement.
  **L68 CN**: 结束当前词法作用域或复合语句块。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `Convert I to a __m64 object. The integer is zero-extended to 64-bits.`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert I to a __m64 object. The integer is zero-extended to 64-bits.`。
- **L71 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L71 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L72 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L72 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 73-96

````c
    _mm_cvtsi32_si64(int __i) {
  return (__m64)(unsigned int)__i;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_from_int(int __i) {
  return _mm_cvtsi32_si64(__i);
}

/* Convert the lower 32 bits of the __m64 object into an integer.  */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64_si32(__m64 __i) {
  return ((int)__i);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_to_int(__m64 __i) {
  return _mm_cvtsi64_si32(__i);
}

/* Convert I to a __m64 object.  */
````
- **L73 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi32_si64(int __i) {`.
  **L73 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi32_si64(int __i) {`。
- **L74 EN**: Returns from the current function with `(__m64)(unsigned int)__i`.
  **L74 CN**: 以 `(__m64)(unsigned int)__i` 从当前函数返回。
- **L75 EN**: Closes the current lexical scope or compound statement.
  **L75 CN**: 结束当前词法作用域或复合语句块。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L77 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L77 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L78 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L78 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L79 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_from_int(int __i) {`.
  **L79 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_from_int(int __i) {`。
- **L80 EN**: Returns from the current function with `_mm_cvtsi32_si64(__i)`.
  **L80 CN**: 以 `_mm_cvtsi32_si64(__i)` 从当前函数返回。
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L83 EN**: Comment explains nearby logic, constraints, or intent: `Convert the lower 32 bits of the __m64 object into an integer.`.
  **L83 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the lower 32 bits of the __m64 object into an integer.`。
- **L84 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L84 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L85 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L85 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L86 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64_si32(__m64 __i) {`.
  **L86 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64_si32(__m64 __i) {`。
- **L87 EN**: Returns from the current function with `((int)__i)`.
  **L87 CN**: 以 `((int)__i)` 从当前函数返回。
- **L88 EN**: Closes the current lexical scope or compound statement.
  **L88 CN**: 结束当前词法作用域或复合语句块。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L90 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L90 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L91 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L91 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L92 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_to_int(__m64 __i) {`.
  **L92 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_to_int(__m64 __i) {`。
- **L93 EN**: Returns from the current function with `_mm_cvtsi64_si32(__i)`.
  **L93 CN**: 以 `_mm_cvtsi64_si32(__i)` 从当前函数返回。
- **L94 EN**: Closes the current lexical scope or compound statement.
  **L94 CN**: 结束当前词法作用域或复合语句块。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `Convert I to a __m64 object.`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert I to a __m64 object.`。

### Lines 97-120

````c

/* Intel intrinsic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_from_int64(long long __i) {
  return (__m64)__i;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64_m64(long long __i) {
  return (__m64)__i;
}

/* Microsoft intrinsic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64x_si64(long long __i) {
  return (__m64)__i;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_pi64x(long long __i) {
````
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L98 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L98 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L99 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L99 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L100 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L100 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L101 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_from_int64(long long __i) {`.
  **L101 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_from_int64(long long __i) {`。
- **L102 EN**: Returns from the current function with `(__m64)__i`.
  **L102 CN**: 以 `(__m64)__i` 从当前函数返回。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L105 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L106 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L106 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L107 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64_m64(long long __i) {`.
  **L107 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64_m64(long long __i) {`。
- **L108 EN**: Returns from the current function with `(__m64)__i`.
  **L108 CN**: 以 `(__m64)__i` 从当前函数返回。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L111 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L112 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L112 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L113 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L113 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L114 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64x_si64(long long __i) {`.
  **L114 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64x_si64(long long __i) {`。
- **L115 EN**: Returns from the current function with `(__m64)__i`.
  **L115 CN**: 以 `(__m64)__i` 从当前函数返回。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L118 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L118 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L119 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L119 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L120 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_pi64x(long long __i) {`.
  **L120 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_pi64x(long long __i) {`。

### Lines 121-144

````c
  return (__m64)__i;
}

/* Convert the __m64 object to a 64bit integer.  */

/* Intel intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_to_int64(__m64 __i) {
  return (long long)__i;
}

extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtm64_si64(__m64 __i) {
  return (long long)__i;
}

/* Microsoft intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64_si64x(__m64 __i) {
  return (long long)__i;
}
````
- **L121 EN**: Returns from the current function with `(__m64)__i`.
  **L121 CN**: 以 `(__m64)__i` 从当前函数返回。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Blank line separating nearby declarations or logic blocks.
  **L123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L124 EN**: Comment explains nearby logic, constraints, or intent: `Convert the __m64 object to a 64bit integer.`.
  **L124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the __m64 object to a 64bit integer.`。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L126 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L127 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L127 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L128 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L128 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L129 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_to_int64(__m64 __i) {`.
  **L129 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_to_int64(__m64 __i) {`。
- **L130 EN**: Returns from the current function with `(long long)__i`.
  **L130 CN**: 以 `(long long)__i` 从当前函数返回。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L133 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L133 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L134 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L134 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L135 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtm64_si64(__m64 __i) {`.
  **L135 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtm64_si64(__m64 __i) {`。
- **L136 EN**: Returns from the current function with `(long long)__i`.
  **L136 CN**: 以 `(long long)__i` 从当前函数返回。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L139 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L140 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L140 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L141 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L141 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64_si64x(__m64 __i) {`.
  **L142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64_si64x(__m64 __i) {`。
- **L143 EN**: Returns from the current function with `(long long)__i`.
  **L143 CN**: 以 `(long long)__i` 从当前函数返回。
- **L144 EN**: Closes the current lexical scope or compound statement.
  **L144 CN**: 结束当前词法作用域或复合语句块。

### Lines 145-168

````c

#ifdef _ARCH_PWR8
/* Pack the four 16-bit values from M1 into the lower four 8-bit values of
   the result, and the four 16-bit values from M2 into the upper four 8-bit
   values of the result, all with signed saturation.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_packs_pi16(__m64 __m1, __m64 __m2) {
  __vector signed short __vm1;
  __vector signed char __vresult;

  __vm1 = (__vector signed short)(__vector unsigned long long)
#ifdef __LITTLE_ENDIAN__
      {__m1, __m2};
#else
      {__m2, __m1};
#endif
  __vresult = vec_packs(__vm1, __vm1);
  return (__m64)((__vector long long)__vresult)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_packsswb(__m64 __m1, __m64 __m2) {
````
- **L145 EN**: Blank line separating nearby declarations or logic blocks.
  **L145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L146 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L146 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L147 EN**: Comment explains nearby logic, constraints, or intent: `Pack the four 16-bit values from M1 into the lower four 8-bit values of`.
  **L147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pack the four 16-bit values from M1 into the lower four 8-bit values of`。
- **L148 EN**: Continues the surrounding expression or declaration: `the result, and the four 16-bit values from M2 into the upper four 8-bit`.
  **L148 CN**: 继续构造周围的表达式或声明：`the result, and the four 16-bit values from M2 into the upper four 8-bit`。
- **L149 EN**: Continues the surrounding expression or declaration: `values of the result, all with signed saturation.  */`.
  **L149 CN**: 继续构造周围的表达式或声明：`values of the result, all with signed saturation.  */`。
- **L150 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L150 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L151 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L151 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L152 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_packs_pi16(__m64 __m1, __m64 __m2) {`.
  **L152 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_packs_pi16(__m64 __m1, __m64 __m2) {`。
- **L153 EN**: Adds a standalone statement or declaration: `__vector signed short __vm1;`.
  **L153 CN**: 添加一条独立语句或声明：`__vector signed short __vm1;`。
- **L154 EN**: Adds a standalone statement or declaration: `__vector signed char __vresult;`.
  **L154 CN**: 添加一条独立语句或声明：`__vector signed char __vresult;`。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L156 EN**: Continues the surrounding expression or declaration: `__vm1 = (__vector signed short)(__vector unsigned long long)`.
  **L156 CN**: 继续构造周围的表达式或声明：`__vm1 = (__vector signed short)(__vector unsigned long long)`。
- **L157 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L157 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L158 EN**: Adds a standalone statement or declaration: `{__m1, __m2};`.
  **L158 CN**: 添加一条独立语句或声明：`{__m1, __m2};`。
- **L159 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L159 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L160 EN**: Adds a standalone statement or declaration: `{__m2, __m1};`.
  **L160 CN**: 添加一条独立语句或声明：`{__m2, __m1};`。
- **L161 EN**: Closes the current preprocessor conditional block.
  **L161 CN**: 结束当前预处理条件块。
- **L162 EN**: Executes a call or declaration centered on `vec_packs`.
  **L162 CN**: 执行以 `vec_packs` 为核心的调用或声明。
- **L163 EN**: Returns from the current function with `(__m64)((__vector long long)__vresult)[0]`.
  **L163 CN**: 以 `(__m64)((__vector long long)__vresult)[0]` 从当前函数返回。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L166 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L166 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L167 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L167 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L168 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_packsswb(__m64 __m1, __m64 __m2) {`.
  **L168 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_packsswb(__m64 __m1, __m64 __m2) {`。

### Lines 169-192

````c
  return _mm_packs_pi16(__m1, __m2);
}

/* Pack the two 32-bit values from M1 in to the lower two 16-bit values of
   the result, and the two 32-bit values from M2 into the upper two 16-bit
   values of the result, all with signed saturation.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_packs_pi32(__m64 __m1, __m64 __m2) {
  __vector signed int __vm1;
  __vector signed short __vresult;

  __vm1 = (__vector signed int)(__vector unsigned long long)
#ifdef __LITTLE_ENDIAN__
      {__m1, __m2};
#else
      {__m2, __m1};
#endif
  __vresult = vec_packs(__vm1, __vm1);
  return (__m64)((__vector long long)__vresult)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L169 EN**: Returns from the current function with `_mm_packs_pi16(__m1, __m2)`.
  **L169 CN**: 以 `_mm_packs_pi16(__m1, __m2)` 从当前函数返回。
- **L170 EN**: Closes the current lexical scope or compound statement.
  **L170 CN**: 结束当前词法作用域或复合语句块。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L172 EN**: Comment explains nearby logic, constraints, or intent: `Pack the two 32-bit values from M1 in to the lower two 16-bit values of`.
  **L172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pack the two 32-bit values from M1 in to the lower two 16-bit values of`。
- **L173 EN**: Continues the surrounding expression or declaration: `the result, and the two 32-bit values from M2 into the upper two 16-bit`.
  **L173 CN**: 继续构造周围的表达式或声明：`the result, and the two 32-bit values from M2 into the upper two 16-bit`。
- **L174 EN**: Continues the surrounding expression or declaration: `values of the result, all with signed saturation.  */`.
  **L174 CN**: 继续构造周围的表达式或声明：`values of the result, all with signed saturation.  */`。
- **L175 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L175 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L176 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L176 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L177 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_packs_pi32(__m64 __m1, __m64 __m2) {`.
  **L177 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_packs_pi32(__m64 __m1, __m64 __m2) {`。
- **L178 EN**: Adds a standalone statement or declaration: `__vector signed int __vm1;`.
  **L178 CN**: 添加一条独立语句或声明：`__vector signed int __vm1;`。
- **L179 EN**: Adds a standalone statement or declaration: `__vector signed short __vresult;`.
  **L179 CN**: 添加一条独立语句或声明：`__vector signed short __vresult;`。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L181 EN**: Continues the surrounding expression or declaration: `__vm1 = (__vector signed int)(__vector unsigned long long)`.
  **L181 CN**: 继续构造周围的表达式或声明：`__vm1 = (__vector signed int)(__vector unsigned long long)`。
- **L182 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L182 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L183 EN**: Adds a standalone statement or declaration: `{__m1, __m2};`.
  **L183 CN**: 添加一条独立语句或声明：`{__m1, __m2};`。
- **L184 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L184 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L185 EN**: Adds a standalone statement or declaration: `{__m2, __m1};`.
  **L185 CN**: 添加一条独立语句或声明：`{__m2, __m1};`。
- **L186 EN**: Closes the current preprocessor conditional block.
  **L186 CN**: 结束当前预处理条件块。
- **L187 EN**: Executes a call or declaration centered on `vec_packs`.
  **L187 CN**: 执行以 `vec_packs` 为核心的调用或声明。
- **L188 EN**: Returns from the current function with `(__m64)((__vector long long)__vresult)[0]`.
  **L188 CN**: 以 `(__m64)((__vector long long)__vresult)[0]` 从当前函数返回。
- **L189 EN**: Closes the current lexical scope or compound statement.
  **L189 CN**: 结束当前词法作用域或复合语句块。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L191 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L191 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L192 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L192 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 193-216

````c
    _m_packssdw(__m64 __m1, __m64 __m2) {
  return _mm_packs_pi32(__m1, __m2);
}

/* Pack the four 16-bit values from M1 into the lower four 8-bit values of
   the result, and the four 16-bit values from M2 into the upper four 8-bit
   values of the result, all with unsigned saturation.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_packs_pu16(__m64 __m1, __m64 __m2) {
  __vector unsigned char __r;
  __vector signed short __vm1 = (__vector signed short)(__vector long long)
#ifdef __LITTLE_ENDIAN__
      {__m1, __m2};
#else
      {__m2, __m1};
#endif
  const __vector signed short __zero = {0};
  __vector __bool short __select = vec_cmplt(__vm1, __zero);
  __r =
      vec_packs((__vector unsigned short)__vm1, (__vector unsigned short)__vm1);
  __vector __bool char __packsel = vec_pack(__select, __select);
  __r = vec_sel(__r, (const __vector unsigned char)__zero, __packsel);
  return (__m64)((__vector long long)__r)[0];
````
- **L193 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_packssdw(__m64 __m1, __m64 __m2) {`.
  **L193 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_packssdw(__m64 __m1, __m64 __m2) {`。
- **L194 EN**: Returns from the current function with `_mm_packs_pi32(__m1, __m2)`.
  **L194 CN**: 以 `_mm_packs_pi32(__m1, __m2)` 从当前函数返回。
- **L195 EN**: Closes the current lexical scope or compound statement.
  **L195 CN**: 结束当前词法作用域或复合语句块。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L197 EN**: Comment explains nearby logic, constraints, or intent: `Pack the four 16-bit values from M1 into the lower four 8-bit values of`.
  **L197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pack the four 16-bit values from M1 into the lower four 8-bit values of`。
- **L198 EN**: Continues the surrounding expression or declaration: `the result, and the four 16-bit values from M2 into the upper four 8-bit`.
  **L198 CN**: 继续构造周围的表达式或声明：`the result, and the four 16-bit values from M2 into the upper four 8-bit`。
- **L199 EN**: Continues the surrounding expression or declaration: `values of the result, all with unsigned saturation.  */`.
  **L199 CN**: 继续构造周围的表达式或声明：`values of the result, all with unsigned saturation.  */`。
- **L200 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L200 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L201 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L201 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L202 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_packs_pu16(__m64 __m1, __m64 __m2) {`.
  **L202 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_packs_pu16(__m64 __m1, __m64 __m2) {`。
- **L203 EN**: Adds a standalone statement or declaration: `__vector unsigned char __r;`.
  **L203 CN**: 添加一条独立语句或声明：`__vector unsigned char __r;`。
- **L204 EN**: Continues the surrounding expression or declaration: `__vector signed short __vm1 = (__vector signed short)(__vector long long)`.
  **L204 CN**: 继续构造周围的表达式或声明：`__vector signed short __vm1 = (__vector signed short)(__vector long long)`。
- **L205 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L205 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L206 EN**: Adds a standalone statement or declaration: `{__m1, __m2};`.
  **L206 CN**: 添加一条独立语句或声明：`{__m1, __m2};`。
- **L207 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L207 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L208 EN**: Adds a standalone statement or declaration: `{__m2, __m1};`.
  **L208 CN**: 添加一条独立语句或声明：`{__m2, __m1};`。
- **L209 EN**: Closes the current preprocessor conditional block.
  **L209 CN**: 结束当前预处理条件块。
- **L210 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L210 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L211 EN**: Initializes variable `__select` from the expression on the right-hand side.
  **L211 CN**: 使用右侧表达式初始化变量 `__select`。
- **L212 EN**: Continues the surrounding expression or declaration: `__r =`.
  **L212 CN**: 继续构造周围的表达式或声明：`__r =`。
- **L213 EN**: Executes a call or declaration centered on `vec_packs`.
  **L213 CN**: 执行以 `vec_packs` 为核心的调用或声明。
- **L214 EN**: Initializes variable `__packsel` from the expression on the right-hand side.
  **L214 CN**: 使用右侧表达式初始化变量 `__packsel`。
- **L215 EN**: Executes a call or declaration centered on `vec_sel`.
  **L215 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L216 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L216 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。

### Lines 217-240

````c
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_packuswb(__m64 __m1, __m64 __m2) {
  return _mm_packs_pu16(__m1, __m2);
}
#endif /* end ARCH_PWR8 */

/* Interleave the four 8-bit values from the high half of M1 with the four
   8-bit values from the high half of M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_pi8(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
  __vector unsigned char __a, __b, __c;

  __a = (__vector unsigned char)vec_splats(__m1);
  __b = (__vector unsigned char)vec_splats(__m2);
  __c = vec_mergel(__a, __b);
  return (__m64)((__vector long long)__c)[1];
#else
  __m64_union __mu1, __mu2, __res;

````
- **L217 EN**: Closes the current lexical scope or compound statement.
  **L217 CN**: 结束当前词法作用域或复合语句块。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L219 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L219 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L220 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L220 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L221 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_packuswb(__m64 __m1, __m64 __m2) {`.
  **L221 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_packuswb(__m64 __m1, __m64 __m2) {`。
- **L222 EN**: Returns from the current function with `_mm_packs_pu16(__m1, __m2)`.
  **L222 CN**: 以 `_mm_packs_pu16(__m1, __m2)` 从当前函数返回。
- **L223 EN**: Closes the current lexical scope or compound statement.
  **L223 CN**: 结束当前词法作用域或复合语句块。
- **L224 EN**: Closes the current preprocessor conditional block.
  **L224 CN**: 结束当前预处理条件块。
- **L225 EN**: Blank line separating nearby declarations or logic blocks.
  **L225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L226 EN**: Comment explains nearby logic, constraints, or intent: `Interleave the four 8-bit values from the high half of M1 with the four`.
  **L226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Interleave the four 8-bit values from the high half of M1 with the four`。
- **L227 EN**: Continues the surrounding expression or declaration: `8-bit values from the high half of M2.  */`.
  **L227 CN**: 继续构造周围的表达式或声明：`8-bit values from the high half of M2.  */`。
- **L228 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L228 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L229 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L229 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L230 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_pi8(__m64 __m1, __m64 __m2) {`.
  **L230 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_pi8(__m64 __m1, __m64 __m2) {`。
- **L231 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L231 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L232 EN**: Adds a standalone statement or declaration: `__vector unsigned char __a, __b, __c;`.
  **L232 CN**: 添加一条独立语句或声明：`__vector unsigned char __a, __b, __c;`。
- **L233 EN**: Blank line separating nearby declarations or logic blocks.
  **L233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L234 EN**: Executes a call or declaration centered on `=`.
  **L234 CN**: 执行以 `=` 为核心的调用或声明。
- **L235 EN**: Executes a call or declaration centered on `=`.
  **L235 CN**: 执行以 `=` 为核心的调用或声明。
- **L236 EN**: Executes a call or declaration centered on `vec_mergel`.
  **L236 CN**: 执行以 `vec_mergel` 为核心的调用或声明。
- **L237 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[1]`.
  **L237 CN**: 以 `(__m64)((__vector long long)__c)[1]` 从当前函数返回。
- **L238 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L238 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L239 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L239 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 241-264

````c
  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_char[0] = __mu1.as_char[4];
  __res.as_char[1] = __mu2.as_char[4];
  __res.as_char[2] = __mu1.as_char[5];
  __res.as_char[3] = __mu2.as_char[5];
  __res.as_char[4] = __mu1.as_char[6];
  __res.as_char[5] = __mu2.as_char[6];
  __res.as_char[6] = __mu1.as_char[7];
  __res.as_char[7] = __mu2.as_char[7];

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_punpckhbw(__m64 __m1, __m64 __m2) {
  return _mm_unpackhi_pi8(__m1, __m2);
}

/* Interleave the two 16-bit values from the high half of M1 with the two
   16-bit values from the high half of M2.  */
````
- **L241 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L241 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L242 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L242 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L243 EN**: Blank line separating nearby declarations or logic blocks.
  **L243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L244 EN**: Adds a standalone statement or declaration: `__res.as_char[0] = __mu1.as_char[4];`.
  **L244 CN**: 添加一条独立语句或声明：`__res.as_char[0] = __mu1.as_char[4];`。
- **L245 EN**: Adds a standalone statement or declaration: `__res.as_char[1] = __mu2.as_char[4];`.
  **L245 CN**: 添加一条独立语句或声明：`__res.as_char[1] = __mu2.as_char[4];`。
- **L246 EN**: Adds a standalone statement or declaration: `__res.as_char[2] = __mu1.as_char[5];`.
  **L246 CN**: 添加一条独立语句或声明：`__res.as_char[2] = __mu1.as_char[5];`。
- **L247 EN**: Adds a standalone statement or declaration: `__res.as_char[3] = __mu2.as_char[5];`.
  **L247 CN**: 添加一条独立语句或声明：`__res.as_char[3] = __mu2.as_char[5];`。
- **L248 EN**: Adds a standalone statement or declaration: `__res.as_char[4] = __mu1.as_char[6];`.
  **L248 CN**: 添加一条独立语句或声明：`__res.as_char[4] = __mu1.as_char[6];`。
- **L249 EN**: Adds a standalone statement or declaration: `__res.as_char[5] = __mu2.as_char[6];`.
  **L249 CN**: 添加一条独立语句或声明：`__res.as_char[5] = __mu2.as_char[6];`。
- **L250 EN**: Adds a standalone statement or declaration: `__res.as_char[6] = __mu1.as_char[7];`.
  **L250 CN**: 添加一条独立语句或声明：`__res.as_char[6] = __mu1.as_char[7];`。
- **L251 EN**: Adds a standalone statement or declaration: `__res.as_char[7] = __mu2.as_char[7];`.
  **L251 CN**: 添加一条独立语句或声明：`__res.as_char[7] = __mu2.as_char[7];`。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L253 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L253 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L254 EN**: Closes the current preprocessor conditional block.
  **L254 CN**: 结束当前预处理条件块。
- **L255 EN**: Closes the current lexical scope or compound statement.
  **L255 CN**: 结束当前词法作用域或复合语句块。
- **L256 EN**: Blank line separating nearby declarations or logic blocks.
  **L256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L257 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L257 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L258 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L258 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L259 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_punpckhbw(__m64 __m1, __m64 __m2) {`.
  **L259 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_punpckhbw(__m64 __m1, __m64 __m2) {`。
- **L260 EN**: Returns from the current function with `_mm_unpackhi_pi8(__m1, __m2)`.
  **L260 CN**: 以 `_mm_unpackhi_pi8(__m1, __m2)` 从当前函数返回。
- **L261 EN**: Closes the current lexical scope or compound statement.
  **L261 CN**: 结束当前词法作用域或复合语句块。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L263 EN**: Comment explains nearby logic, constraints, or intent: `Interleave the two 16-bit values from the high half of M1 with the two`.
  **L263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Interleave the two 16-bit values from the high half of M1 with the two`。
- **L264 EN**: Continues the surrounding expression or declaration: `16-bit values from the high half of M2.  */`.
  **L264 CN**: 继续构造周围的表达式或声明：`16-bit values from the high half of M2.  */`。

### Lines 265-288

````c
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_pi16(__m64 __m1, __m64 __m2) {
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_short[0] = __mu1.as_short[2];
  __res.as_short[1] = __mu2.as_short[2];
  __res.as_short[2] = __mu1.as_short[3];
  __res.as_short[3] = __mu2.as_short[3];

  return (__m64)__res.as_m64;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_punpckhwd(__m64 __m1, __m64 __m2) {
  return _mm_unpackhi_pi16(__m1, __m2);
}
/* Interleave the 32-bit value from the high half of M1 with the 32-bit
   value from the high half of M2.  */
extern __inline __m64
````
- **L265 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L265 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L266 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L266 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L267 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_pi16(__m64 __m1, __m64 __m2) {`.
  **L267 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_pi16(__m64 __m1, __m64 __m2) {`。
- **L268 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L268 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L270 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L270 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L271 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L271 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L273 EN**: Adds a standalone statement or declaration: `__res.as_short[0] = __mu1.as_short[2];`.
  **L273 CN**: 添加一条独立语句或声明：`__res.as_short[0] = __mu1.as_short[2];`。
- **L274 EN**: Adds a standalone statement or declaration: `__res.as_short[1] = __mu2.as_short[2];`.
  **L274 CN**: 添加一条独立语句或声明：`__res.as_short[1] = __mu2.as_short[2];`。
- **L275 EN**: Adds a standalone statement or declaration: `__res.as_short[2] = __mu1.as_short[3];`.
  **L275 CN**: 添加一条独立语句或声明：`__res.as_short[2] = __mu1.as_short[3];`。
- **L276 EN**: Adds a standalone statement or declaration: `__res.as_short[3] = __mu2.as_short[3];`.
  **L276 CN**: 添加一条独立语句或声明：`__res.as_short[3] = __mu2.as_short[3];`。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L278 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L278 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L281 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L281 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L282 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L282 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L283 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_punpckhwd(__m64 __m1, __m64 __m2) {`.
  **L283 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_punpckhwd(__m64 __m1, __m64 __m2) {`。
- **L284 EN**: Returns from the current function with `_mm_unpackhi_pi16(__m1, __m2)`.
  **L284 CN**: 以 `_mm_unpackhi_pi16(__m1, __m2)` 从当前函数返回。
- **L285 EN**: Closes the current lexical scope or compound statement.
  **L285 CN**: 结束当前词法作用域或复合语句块。
- **L286 EN**: Comment explains nearby logic, constraints, or intent: `Interleave the 32-bit value from the high half of M1 with the 32-bit`.
  **L286 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Interleave the 32-bit value from the high half of M1 with the 32-bit`。
- **L287 EN**: Continues the surrounding expression or declaration: `value from the high half of M2.  */`.
  **L287 CN**: 继续构造周围的表达式或声明：`value from the high half of M2.  */`。
- **L288 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L288 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。

### Lines 289-312

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_pi32(__m64 __m1, __m64 __m2) {
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_int[0] = __mu1.as_int[1];
  __res.as_int[1] = __mu2.as_int[1];

  return (__m64)__res.as_m64;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_punpckhdq(__m64 __m1, __m64 __m2) {
  return _mm_unpackhi_pi32(__m1, __m2);
}
/* Interleave the four 8-bit values from the low half of M1 with the four
   8-bit values from the low half of M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_pi8(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
````
- **L289 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L289 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L290 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_pi32(__m64 __m1, __m64 __m2) {`.
  **L290 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_pi32(__m64 __m1, __m64 __m2) {`。
- **L291 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L291 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L292 EN**: Blank line separating nearby declarations or logic blocks.
  **L292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L293 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L293 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L294 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L294 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L295 EN**: Blank line separating nearby declarations or logic blocks.
  **L295 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L296 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __mu1.as_int[1];`.
  **L296 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __mu1.as_int[1];`。
- **L297 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __mu2.as_int[1];`.
  **L297 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __mu2.as_int[1];`。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L299 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L299 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L302 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L302 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L303 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L303 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L304 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_punpckhdq(__m64 __m1, __m64 __m2) {`.
  **L304 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_punpckhdq(__m64 __m1, __m64 __m2) {`。
- **L305 EN**: Returns from the current function with `_mm_unpackhi_pi32(__m1, __m2)`.
  **L305 CN**: 以 `_mm_unpackhi_pi32(__m1, __m2)` 从当前函数返回。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Comment explains nearby logic, constraints, or intent: `Interleave the four 8-bit values from the low half of M1 with the four`.
  **L307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Interleave the four 8-bit values from the low half of M1 with the four`。
- **L308 EN**: Continues the surrounding expression or declaration: `8-bit values from the low half of M2.  */`.
  **L308 CN**: 继续构造周围的表达式或声明：`8-bit values from the low half of M2.  */`。
- **L309 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L309 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L310 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L310 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L311 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_pi8(__m64 __m1, __m64 __m2) {`.
  **L311 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_pi8(__m64 __m1, __m64 __m2) {`。
- **L312 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L312 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。

### Lines 313-336

````c
  __vector unsigned char __a, __b, __c;

  __a = (__vector unsigned char)vec_splats(__m1);
  __b = (__vector unsigned char)vec_splats(__m2);
  __c = vec_mergel(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_char[0] = __mu1.as_char[0];
  __res.as_char[1] = __mu2.as_char[0];
  __res.as_char[2] = __mu1.as_char[1];
  __res.as_char[3] = __mu2.as_char[1];
  __res.as_char[4] = __mu1.as_char[2];
  __res.as_char[5] = __mu2.as_char[2];
  __res.as_char[6] = __mu1.as_char[3];
  __res.as_char[7] = __mu2.as_char[3];

  return (__m64)__res.as_m64;
#endif
}
````
- **L313 EN**: Adds a standalone statement or declaration: `__vector unsigned char __a, __b, __c;`.
  **L313 CN**: 添加一条独立语句或声明：`__vector unsigned char __a, __b, __c;`。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Executes a call or declaration centered on `=`.
  **L315 CN**: 执行以 `=` 为核心的调用或声明。
- **L316 EN**: Executes a call or declaration centered on `=`.
  **L316 CN**: 执行以 `=` 为核心的调用或声明。
- **L317 EN**: Executes a call or declaration centered on `vec_mergel`.
  **L317 CN**: 执行以 `vec_mergel` 为核心的调用或声明。
- **L318 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L318 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L319 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L319 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L320 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L320 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L322 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L322 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L323 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L323 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L325 EN**: Adds a standalone statement or declaration: `__res.as_char[0] = __mu1.as_char[0];`.
  **L325 CN**: 添加一条独立语句或声明：`__res.as_char[0] = __mu1.as_char[0];`。
- **L326 EN**: Adds a standalone statement or declaration: `__res.as_char[1] = __mu2.as_char[0];`.
  **L326 CN**: 添加一条独立语句或声明：`__res.as_char[1] = __mu2.as_char[0];`。
- **L327 EN**: Adds a standalone statement or declaration: `__res.as_char[2] = __mu1.as_char[1];`.
  **L327 CN**: 添加一条独立语句或声明：`__res.as_char[2] = __mu1.as_char[1];`。
- **L328 EN**: Adds a standalone statement or declaration: `__res.as_char[3] = __mu2.as_char[1];`.
  **L328 CN**: 添加一条独立语句或声明：`__res.as_char[3] = __mu2.as_char[1];`。
- **L329 EN**: Adds a standalone statement or declaration: `__res.as_char[4] = __mu1.as_char[2];`.
  **L329 CN**: 添加一条独立语句或声明：`__res.as_char[4] = __mu1.as_char[2];`。
- **L330 EN**: Adds a standalone statement or declaration: `__res.as_char[5] = __mu2.as_char[2];`.
  **L330 CN**: 添加一条独立语句或声明：`__res.as_char[5] = __mu2.as_char[2];`。
- **L331 EN**: Adds a standalone statement or declaration: `__res.as_char[6] = __mu1.as_char[3];`.
  **L331 CN**: 添加一条独立语句或声明：`__res.as_char[6] = __mu1.as_char[3];`。
- **L332 EN**: Adds a standalone statement or declaration: `__res.as_char[7] = __mu2.as_char[3];`.
  **L332 CN**: 添加一条独立语句或声明：`__res.as_char[7] = __mu2.as_char[3];`。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L334 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L334 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L335 EN**: Closes the current preprocessor conditional block.
  **L335 CN**: 结束当前预处理条件块。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。

### Lines 337-360

````c

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_punpcklbw(__m64 __m1, __m64 __m2) {
  return _mm_unpacklo_pi8(__m1, __m2);
}
/* Interleave the two 16-bit values from the low half of M1 with the two
   16-bit values from the low half of M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_pi16(__m64 __m1, __m64 __m2) {
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_short[0] = __mu1.as_short[0];
  __res.as_short[1] = __mu2.as_short[0];
  __res.as_short[2] = __mu1.as_short[1];
  __res.as_short[3] = __mu2.as_short[1];

  return (__m64)__res.as_m64;
}

````
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L338 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L338 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L339 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L339 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L340 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_punpcklbw(__m64 __m1, __m64 __m2) {`.
  **L340 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_punpcklbw(__m64 __m1, __m64 __m2) {`。
- **L341 EN**: Returns from the current function with `_mm_unpacklo_pi8(__m1, __m2)`.
  **L341 CN**: 以 `_mm_unpacklo_pi8(__m1, __m2)` 从当前函数返回。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Comment explains nearby logic, constraints, or intent: `Interleave the two 16-bit values from the low half of M1 with the two`.
  **L343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Interleave the two 16-bit values from the low half of M1 with the two`。
- **L344 EN**: Continues the surrounding expression or declaration: `16-bit values from the low half of M2.  */`.
  **L344 CN**: 继续构造周围的表达式或声明：`16-bit values from the low half of M2.  */`。
- **L345 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L345 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L346 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L346 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L347 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_pi16(__m64 __m1, __m64 __m2) {`.
  **L347 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_pi16(__m64 __m1, __m64 __m2) {`。
- **L348 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L348 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L350 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L350 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L351 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L351 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L353 EN**: Adds a standalone statement or declaration: `__res.as_short[0] = __mu1.as_short[0];`.
  **L353 CN**: 添加一条独立语句或声明：`__res.as_short[0] = __mu1.as_short[0];`。
- **L354 EN**: Adds a standalone statement or declaration: `__res.as_short[1] = __mu2.as_short[0];`.
  **L354 CN**: 添加一条独立语句或声明：`__res.as_short[1] = __mu2.as_short[0];`。
- **L355 EN**: Adds a standalone statement or declaration: `__res.as_short[2] = __mu1.as_short[1];`.
  **L355 CN**: 添加一条独立语句或声明：`__res.as_short[2] = __mu1.as_short[1];`。
- **L356 EN**: Adds a standalone statement or declaration: `__res.as_short[3] = __mu2.as_short[1];`.
  **L356 CN**: 添加一条独立语句或声明：`__res.as_short[3] = __mu2.as_short[1];`。
- **L357 EN**: Blank line separating nearby declarations or logic blocks.
  **L357 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L358 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L358 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L359 EN**: Closes the current lexical scope or compound statement.
  **L359 CN**: 结束当前词法作用域或复合语句块。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 361-384

````c
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_punpcklwd(__m64 __m1, __m64 __m2) {
  return _mm_unpacklo_pi16(__m1, __m2);
}

/* Interleave the 32-bit value from the low half of M1 with the 32-bit
   value from the low half of M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_pi32(__m64 __m1, __m64 __m2) {
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_int[0] = __mu1.as_int[0];
  __res.as_int[1] = __mu2.as_int[0];

  return (__m64)__res.as_m64;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L361 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L361 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L362 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L362 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L363 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_punpcklwd(__m64 __m1, __m64 __m2) {`.
  **L363 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_punpcklwd(__m64 __m1, __m64 __m2) {`。
- **L364 EN**: Returns from the current function with `_mm_unpacklo_pi16(__m1, __m2)`.
  **L364 CN**: 以 `_mm_unpacklo_pi16(__m1, __m2)` 从当前函数返回。
- **L365 EN**: Closes the current lexical scope or compound statement.
  **L365 CN**: 结束当前词法作用域或复合语句块。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L367 EN**: Comment explains nearby logic, constraints, or intent: `Interleave the 32-bit value from the low half of M1 with the 32-bit`.
  **L367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Interleave the 32-bit value from the low half of M1 with the 32-bit`。
- **L368 EN**: Continues the surrounding expression or declaration: `value from the low half of M2.  */`.
  **L368 CN**: 继续构造周围的表达式或声明：`value from the low half of M2.  */`。
- **L369 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L369 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L370 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L370 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L371 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_pi32(__m64 __m1, __m64 __m2) {`.
  **L371 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_pi32(__m64 __m1, __m64 __m2) {`。
- **L372 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L372 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L373 EN**: Blank line separating nearby declarations or logic blocks.
  **L373 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L374 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L374 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L375 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L375 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L377 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __mu1.as_int[0];`.
  **L377 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __mu1.as_int[0];`。
- **L378 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __mu2.as_int[0];`.
  **L378 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __mu2.as_int[0];`。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L380 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L380 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L381 EN**: Closes the current lexical scope or compound statement.
  **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L383 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L383 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L384 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L384 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 385-408

````c
    _m_punpckldq(__m64 __m1, __m64 __m2) {
  return _mm_unpacklo_pi32(__m1, __m2);
}

/* Add the 8-bit values in M1 to the 8-bit values in M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_pi8(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
  __vector signed char __a, __b, __c;

  __a = (__vector signed char)vec_splats(__m1);
  __b = (__vector signed char)vec_splats(__m2);
  __c = vec_add(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_char[0] = __mu1.as_char[0] + __mu2.as_char[0];
  __res.as_char[1] = __mu1.as_char[1] + __mu2.as_char[1];
  __res.as_char[2] = __mu1.as_char[2] + __mu2.as_char[2];
````
- **L385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_punpckldq(__m64 __m1, __m64 __m2) {`.
  **L385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_punpckldq(__m64 __m1, __m64 __m2) {`。
- **L386 EN**: Returns from the current function with `_mm_unpacklo_pi32(__m1, __m2)`.
  **L386 CN**: 以 `_mm_unpacklo_pi32(__m1, __m2)` 从当前函数返回。
- **L387 EN**: Closes the current lexical scope or compound statement.
  **L387 CN**: 结束当前词法作用域或复合语句块。
- **L388 EN**: Blank line separating nearby declarations or logic blocks.
  **L388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L389 EN**: Comment explains nearby logic, constraints, or intent: `Add the 8-bit values in M1 to the 8-bit values in M2.`.
  **L389 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add the 8-bit values in M1 to the 8-bit values in M2.`。
- **L390 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L390 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L391 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L391 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L392 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_pi8(__m64 __m1, __m64 __m2) {`.
  **L392 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_pi8(__m64 __m1, __m64 __m2) {`。
- **L393 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L393 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L394 EN**: Adds a standalone statement or declaration: `__vector signed char __a, __b, __c;`.
  **L394 CN**: 添加一条独立语句或声明：`__vector signed char __a, __b, __c;`。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Executes a call or declaration centered on `=`.
  **L396 CN**: 执行以 `=` 为核心的调用或声明。
- **L397 EN**: Executes a call or declaration centered on `=`.
  **L397 CN**: 执行以 `=` 为核心的调用或声明。
- **L398 EN**: Executes a call or declaration centered on `vec_add`.
  **L398 CN**: 执行以 `vec_add` 为核心的调用或声明。
- **L399 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L399 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L400 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L400 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L401 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L401 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L403 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L403 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L404 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L404 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L406 EN**: Adds a standalone statement or declaration: `__res.as_char[0] = __mu1.as_char[0] + __mu2.as_char[0];`.
  **L406 CN**: 添加一条独立语句或声明：`__res.as_char[0] = __mu1.as_char[0] + __mu2.as_char[0];`。
- **L407 EN**: Adds a standalone statement or declaration: `__res.as_char[1] = __mu1.as_char[1] + __mu2.as_char[1];`.
  **L407 CN**: 添加一条独立语句或声明：`__res.as_char[1] = __mu1.as_char[1] + __mu2.as_char[1];`。
- **L408 EN**: Adds a standalone statement or declaration: `__res.as_char[2] = __mu1.as_char[2] + __mu2.as_char[2];`.
  **L408 CN**: 添加一条独立语句或声明：`__res.as_char[2] = __mu1.as_char[2] + __mu2.as_char[2];`。

### Lines 409-432

````c
  __res.as_char[3] = __mu1.as_char[3] + __mu2.as_char[3];
  __res.as_char[4] = __mu1.as_char[4] + __mu2.as_char[4];
  __res.as_char[5] = __mu1.as_char[5] + __mu2.as_char[5];
  __res.as_char[6] = __mu1.as_char[6] + __mu2.as_char[6];
  __res.as_char[7] = __mu1.as_char[7] + __mu2.as_char[7];

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_paddb(__m64 __m1, __m64 __m2) {
  return _mm_add_pi8(__m1, __m2);
}

/* Add the 16-bit values in M1 to the 16-bit values in M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_pi16(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
  __vector signed short __a, __b, __c;

  __a = (__vector signed short)vec_splats(__m1);
````
- **L409 EN**: Adds a standalone statement or declaration: `__res.as_char[3] = __mu1.as_char[3] + __mu2.as_char[3];`.
  **L409 CN**: 添加一条独立语句或声明：`__res.as_char[3] = __mu1.as_char[3] + __mu2.as_char[3];`。
- **L410 EN**: Adds a standalone statement or declaration: `__res.as_char[4] = __mu1.as_char[4] + __mu2.as_char[4];`.
  **L410 CN**: 添加一条独立语句或声明：`__res.as_char[4] = __mu1.as_char[4] + __mu2.as_char[4];`。
- **L411 EN**: Adds a standalone statement or declaration: `__res.as_char[5] = __mu1.as_char[5] + __mu2.as_char[5];`.
  **L411 CN**: 添加一条独立语句或声明：`__res.as_char[5] = __mu1.as_char[5] + __mu2.as_char[5];`。
- **L412 EN**: Adds a standalone statement or declaration: `__res.as_char[6] = __mu1.as_char[6] + __mu2.as_char[6];`.
  **L412 CN**: 添加一条独立语句或声明：`__res.as_char[6] = __mu1.as_char[6] + __mu2.as_char[6];`。
- **L413 EN**: Adds a standalone statement or declaration: `__res.as_char[7] = __mu1.as_char[7] + __mu2.as_char[7];`.
  **L413 CN**: 添加一条独立语句或声明：`__res.as_char[7] = __mu1.as_char[7] + __mu2.as_char[7];`。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L415 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L415 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L416 EN**: Closes the current preprocessor conditional block.
  **L416 CN**: 结束当前预处理条件块。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L419 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L419 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L420 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L420 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L421 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_paddb(__m64 __m1, __m64 __m2) {`.
  **L421 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_paddb(__m64 __m1, __m64 __m2) {`。
- **L422 EN**: Returns from the current function with `_mm_add_pi8(__m1, __m2)`.
  **L422 CN**: 以 `_mm_add_pi8(__m1, __m2)` 从当前函数返回。
- **L423 EN**: Closes the current lexical scope or compound statement.
  **L423 CN**: 结束当前词法作用域或复合语句块。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L425 EN**: Comment explains nearby logic, constraints, or intent: `Add the 16-bit values in M1 to the 16-bit values in M2.`.
  **L425 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add the 16-bit values in M1 to the 16-bit values in M2.`。
- **L426 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L426 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L427 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L427 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L428 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_pi16(__m64 __m1, __m64 __m2) {`.
  **L428 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_pi16(__m64 __m1, __m64 __m2) {`。
- **L429 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L429 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L430 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __c;`.
  **L430 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __c;`。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L432 EN**: Executes a call or declaration centered on `=`.
  **L432 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 433-456

````c
  __b = (__vector signed short)vec_splats(__m2);
  __c = vec_add(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_short[0] = __mu1.as_short[0] + __mu2.as_short[0];
  __res.as_short[1] = __mu1.as_short[1] + __mu2.as_short[1];
  __res.as_short[2] = __mu1.as_short[2] + __mu2.as_short[2];
  __res.as_short[3] = __mu1.as_short[3] + __mu2.as_short[3];

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_paddw(__m64 __m1, __m64 __m2) {
  return _mm_add_pi16(__m1, __m2);
}

````
- **L433 EN**: Executes a call or declaration centered on `=`.
  **L433 CN**: 执行以 `=` 为核心的调用或声明。
- **L434 EN**: Executes a call or declaration centered on `vec_add`.
  **L434 CN**: 执行以 `vec_add` 为核心的调用或声明。
- **L435 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L435 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L436 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L436 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L437 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L437 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L439 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L439 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L440 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L440 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L442 EN**: Adds a standalone statement or declaration: `__res.as_short[0] = __mu1.as_short[0] + __mu2.as_short[0];`.
  **L442 CN**: 添加一条独立语句或声明：`__res.as_short[0] = __mu1.as_short[0] + __mu2.as_short[0];`。
- **L443 EN**: Adds a standalone statement or declaration: `__res.as_short[1] = __mu1.as_short[1] + __mu2.as_short[1];`.
  **L443 CN**: 添加一条独立语句或声明：`__res.as_short[1] = __mu1.as_short[1] + __mu2.as_short[1];`。
- **L444 EN**: Adds a standalone statement or declaration: `__res.as_short[2] = __mu1.as_short[2] + __mu2.as_short[2];`.
  **L444 CN**: 添加一条独立语句或声明：`__res.as_short[2] = __mu1.as_short[2] + __mu2.as_short[2];`。
- **L445 EN**: Adds a standalone statement or declaration: `__res.as_short[3] = __mu1.as_short[3] + __mu2.as_short[3];`.
  **L445 CN**: 添加一条独立语句或声明：`__res.as_short[3] = __mu1.as_short[3] + __mu2.as_short[3];`。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L447 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L447 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L448 EN**: Closes the current preprocessor conditional block.
  **L448 CN**: 结束当前预处理条件块。
- **L449 EN**: Closes the current lexical scope or compound statement.
  **L449 CN**: 结束当前词法作用域或复合语句块。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L451 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L451 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L452 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L452 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L453 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_paddw(__m64 __m1, __m64 __m2) {`.
  **L453 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_paddw(__m64 __m1, __m64 __m2) {`。
- **L454 EN**: Returns from the current function with `_mm_add_pi16(__m1, __m2)`.
  **L454 CN**: 以 `_mm_add_pi16(__m1, __m2)` 从当前函数返回。
- **L455 EN**: Closes the current lexical scope or compound statement.
  **L455 CN**: 结束当前词法作用域或复合语句块。
- **L456 EN**: Blank line separating nearby declarations or logic blocks.
  **L456 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 457-480

````c
/* Add the 32-bit values in M1 to the 32-bit values in M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_pi32(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR9
  __vector signed int __a, __b, __c;

  __a = (__vector signed int)vec_splats(__m1);
  __b = (__vector signed int)vec_splats(__m2);
  __c = vec_add(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_int[0] = __mu1.as_int[0] + __mu2.as_int[0];
  __res.as_int[1] = __mu1.as_int[1] + __mu2.as_int[1];

  return (__m64)__res.as_m64;
#endif
}

````
- **L457 EN**: Comment explains nearby logic, constraints, or intent: `Add the 32-bit values in M1 to the 32-bit values in M2.`.
  **L457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add the 32-bit values in M1 to the 32-bit values in M2.`。
- **L458 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L458 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L459 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L459 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L460 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_pi32(__m64 __m1, __m64 __m2) {`.
  **L460 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_pi32(__m64 __m1, __m64 __m2) {`。
- **L461 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR9`.
  **L461 CN**: 开始一个预处理条件块：`#if _ARCH_PWR9`。
- **L462 EN**: Adds a standalone statement or declaration: `__vector signed int __a, __b, __c;`.
  **L462 CN**: 添加一条独立语句或声明：`__vector signed int __a, __b, __c;`。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L464 EN**: Executes a call or declaration centered on `=`.
  **L464 CN**: 执行以 `=` 为核心的调用或声明。
- **L465 EN**: Executes a call or declaration centered on `=`.
  **L465 CN**: 执行以 `=` 为核心的调用或声明。
- **L466 EN**: Executes a call or declaration centered on `vec_add`.
  **L466 CN**: 执行以 `vec_add` 为核心的调用或声明。
- **L467 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L467 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L468 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L468 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L469 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L469 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L471 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L472 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L472 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L473 EN**: Blank line separating nearby declarations or logic blocks.
  **L473 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L474 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __mu1.as_int[0] + __mu2.as_int[0];`.
  **L474 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __mu1.as_int[0] + __mu2.as_int[0];`。
- **L475 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __mu1.as_int[1] + __mu2.as_int[1];`.
  **L475 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __mu1.as_int[1] + __mu2.as_int[1];`。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L477 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L477 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L478 EN**: Closes the current preprocessor conditional block.
  **L478 CN**: 结束当前预处理条件块。
- **L479 EN**: Closes the current lexical scope or compound statement.
  **L479 CN**: 结束当前词法作用域或复合语句块。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 481-504

````c
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_paddd(__m64 __m1, __m64 __m2) {
  return _mm_add_pi32(__m1, __m2);
}

/* Subtract the 8-bit values in M2 from the 8-bit values in M1.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_pi8(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
  __vector signed char __a, __b, __c;

  __a = (__vector signed char)vec_splats(__m1);
  __b = (__vector signed char)vec_splats(__m2);
  __c = vec_sub(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_char[0] = __mu1.as_char[0] - __mu2.as_char[0];
````
- **L481 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L481 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L482 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L482 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L483 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_paddd(__m64 __m1, __m64 __m2) {`.
  **L483 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_paddd(__m64 __m1, __m64 __m2) {`。
- **L484 EN**: Returns from the current function with `_mm_add_pi32(__m1, __m2)`.
  **L484 CN**: 以 `_mm_add_pi32(__m1, __m2)` 从当前函数返回。
- **L485 EN**: Closes the current lexical scope or compound statement.
  **L485 CN**: 结束当前词法作用域或复合语句块。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L487 EN**: Comment explains nearby logic, constraints, or intent: `Subtract the 8-bit values in M2 from the 8-bit values in M1.`.
  **L487 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtract the 8-bit values in M2 from the 8-bit values in M1.`。
- **L488 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L488 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L489 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L489 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L490 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_pi8(__m64 __m1, __m64 __m2) {`.
  **L490 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_pi8(__m64 __m1, __m64 __m2) {`。
- **L491 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L491 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L492 EN**: Adds a standalone statement or declaration: `__vector signed char __a, __b, __c;`.
  **L492 CN**: 添加一条独立语句或声明：`__vector signed char __a, __b, __c;`。
- **L493 EN**: Blank line separating nearby declarations or logic blocks.
  **L493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L494 EN**: Executes a call or declaration centered on `=`.
  **L494 CN**: 执行以 `=` 为核心的调用或声明。
- **L495 EN**: Executes a call or declaration centered on `=`.
  **L495 CN**: 执行以 `=` 为核心的调用或声明。
- **L496 EN**: Executes a call or declaration centered on `vec_sub`.
  **L496 CN**: 执行以 `vec_sub` 为核心的调用或声明。
- **L497 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L497 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L498 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L498 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L499 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L499 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L500 EN**: Blank line separating nearby declarations or logic blocks.
  **L500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L501 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L501 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L502 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L502 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L504 EN**: Adds a standalone statement or declaration: `__res.as_char[0] = __mu1.as_char[0] - __mu2.as_char[0];`.
  **L504 CN**: 添加一条独立语句或声明：`__res.as_char[0] = __mu1.as_char[0] - __mu2.as_char[0];`。

### Lines 505-528

````c
  __res.as_char[1] = __mu1.as_char[1] - __mu2.as_char[1];
  __res.as_char[2] = __mu1.as_char[2] - __mu2.as_char[2];
  __res.as_char[3] = __mu1.as_char[3] - __mu2.as_char[3];
  __res.as_char[4] = __mu1.as_char[4] - __mu2.as_char[4];
  __res.as_char[5] = __mu1.as_char[5] - __mu2.as_char[5];
  __res.as_char[6] = __mu1.as_char[6] - __mu2.as_char[6];
  __res.as_char[7] = __mu1.as_char[7] - __mu2.as_char[7];

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psubb(__m64 __m1, __m64 __m2) {
  return _mm_sub_pi8(__m1, __m2);
}

/* Subtract the 16-bit values in M2 from the 16-bit values in M1.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_pi16(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
  __vector signed short __a, __b, __c;
````
- **L505 EN**: Adds a standalone statement or declaration: `__res.as_char[1] = __mu1.as_char[1] - __mu2.as_char[1];`.
  **L505 CN**: 添加一条独立语句或声明：`__res.as_char[1] = __mu1.as_char[1] - __mu2.as_char[1];`。
- **L506 EN**: Adds a standalone statement or declaration: `__res.as_char[2] = __mu1.as_char[2] - __mu2.as_char[2];`.
  **L506 CN**: 添加一条独立语句或声明：`__res.as_char[2] = __mu1.as_char[2] - __mu2.as_char[2];`。
- **L507 EN**: Adds a standalone statement or declaration: `__res.as_char[3] = __mu1.as_char[3] - __mu2.as_char[3];`.
  **L507 CN**: 添加一条独立语句或声明：`__res.as_char[3] = __mu1.as_char[3] - __mu2.as_char[3];`。
- **L508 EN**: Adds a standalone statement or declaration: `__res.as_char[4] = __mu1.as_char[4] - __mu2.as_char[4];`.
  **L508 CN**: 添加一条独立语句或声明：`__res.as_char[4] = __mu1.as_char[4] - __mu2.as_char[4];`。
- **L509 EN**: Adds a standalone statement or declaration: `__res.as_char[5] = __mu1.as_char[5] - __mu2.as_char[5];`.
  **L509 CN**: 添加一条独立语句或声明：`__res.as_char[5] = __mu1.as_char[5] - __mu2.as_char[5];`。
- **L510 EN**: Adds a standalone statement or declaration: `__res.as_char[6] = __mu1.as_char[6] - __mu2.as_char[6];`.
  **L510 CN**: 添加一条独立语句或声明：`__res.as_char[6] = __mu1.as_char[6] - __mu2.as_char[6];`。
- **L511 EN**: Adds a standalone statement or declaration: `__res.as_char[7] = __mu1.as_char[7] - __mu2.as_char[7];`.
  **L511 CN**: 添加一条独立语句或声明：`__res.as_char[7] = __mu1.as_char[7] - __mu2.as_char[7];`。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L513 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L513 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L514 EN**: Closes the current preprocessor conditional block.
  **L514 CN**: 结束当前预处理条件块。
- **L515 EN**: Closes the current lexical scope or compound statement.
  **L515 CN**: 结束当前词法作用域或复合语句块。
- **L516 EN**: Blank line separating nearby declarations or logic blocks.
  **L516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L517 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L517 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L518 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L518 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L519 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psubb(__m64 __m1, __m64 __m2) {`.
  **L519 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psubb(__m64 __m1, __m64 __m2) {`。
- **L520 EN**: Returns from the current function with `_mm_sub_pi8(__m1, __m2)`.
  **L520 CN**: 以 `_mm_sub_pi8(__m1, __m2)` 从当前函数返回。
- **L521 EN**: Closes the current lexical scope or compound statement.
  **L521 CN**: 结束当前词法作用域或复合语句块。
- **L522 EN**: Blank line separating nearby declarations or logic blocks.
  **L522 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L523 EN**: Comment explains nearby logic, constraints, or intent: `Subtract the 16-bit values in M2 from the 16-bit values in M1.`.
  **L523 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtract the 16-bit values in M2 from the 16-bit values in M1.`。
- **L524 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L524 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L525 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L525 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L526 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_pi16(__m64 __m1, __m64 __m2) {`.
  **L526 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_pi16(__m64 __m1, __m64 __m2) {`。
- **L527 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L527 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L528 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __c;`.
  **L528 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __c;`。

### Lines 529-552

````c

  __a = (__vector signed short)vec_splats(__m1);
  __b = (__vector signed short)vec_splats(__m2);
  __c = vec_sub(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_short[0] = __mu1.as_short[0] - __mu2.as_short[0];
  __res.as_short[1] = __mu1.as_short[1] - __mu2.as_short[1];
  __res.as_short[2] = __mu1.as_short[2] - __mu2.as_short[2];
  __res.as_short[3] = __mu1.as_short[3] - __mu2.as_short[3];

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psubw(__m64 __m1, __m64 __m2) {
  return _mm_sub_pi16(__m1, __m2);
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Executes a call or declaration centered on `=`.
  **L530 CN**: 执行以 `=` 为核心的调用或声明。
- **L531 EN**: Executes a call or declaration centered on `=`.
  **L531 CN**: 执行以 `=` 为核心的调用或声明。
- **L532 EN**: Executes a call or declaration centered on `vec_sub`.
  **L532 CN**: 执行以 `vec_sub` 为核心的调用或声明。
- **L533 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L533 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L534 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L534 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L535 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L535 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L537 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L538 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L538 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L540 EN**: Adds a standalone statement or declaration: `__res.as_short[0] = __mu1.as_short[0] - __mu2.as_short[0];`.
  **L540 CN**: 添加一条独立语句或声明：`__res.as_short[0] = __mu1.as_short[0] - __mu2.as_short[0];`。
- **L541 EN**: Adds a standalone statement or declaration: `__res.as_short[1] = __mu1.as_short[1] - __mu2.as_short[1];`.
  **L541 CN**: 添加一条独立语句或声明：`__res.as_short[1] = __mu1.as_short[1] - __mu2.as_short[1];`。
- **L542 EN**: Adds a standalone statement or declaration: `__res.as_short[2] = __mu1.as_short[2] - __mu2.as_short[2];`.
  **L542 CN**: 添加一条独立语句或声明：`__res.as_short[2] = __mu1.as_short[2] - __mu2.as_short[2];`。
- **L543 EN**: Adds a standalone statement or declaration: `__res.as_short[3] = __mu1.as_short[3] - __mu2.as_short[3];`.
  **L543 CN**: 添加一条独立语句或声明：`__res.as_short[3] = __mu1.as_short[3] - __mu2.as_short[3];`。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L545 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L545 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L546 EN**: Closes the current preprocessor conditional block.
  **L546 CN**: 结束当前预处理条件块。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Blank line separating nearby declarations or logic blocks.
  **L548 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L549 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L549 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L550 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L550 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L551 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psubw(__m64 __m1, __m64 __m2) {`.
  **L551 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psubw(__m64 __m1, __m64 __m2) {`。
- **L552 EN**: Returns from the current function with `_mm_sub_pi16(__m1, __m2)`.
  **L552 CN**: 以 `_mm_sub_pi16(__m1, __m2)` 从当前函数返回。

### Lines 553-576

````c
}

/* Subtract the 32-bit values in M2 from the 32-bit values in M1.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_pi32(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR9
  __vector signed int __a, __b, __c;

  __a = (__vector signed int)vec_splats(__m1);
  __b = (__vector signed int)vec_splats(__m2);
  __c = vec_sub(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_int[0] = __mu1.as_int[0] - __mu2.as_int[0];
  __res.as_int[1] = __mu1.as_int[1] - __mu2.as_int[1];

  return (__m64)__res.as_m64;
#endif
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Comment explains nearby logic, constraints, or intent: `Subtract the 32-bit values in M2 from the 32-bit values in M1.`.
  **L555 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtract the 32-bit values in M2 from the 32-bit values in M1.`。
- **L556 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L556 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L557 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L557 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L558 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_pi32(__m64 __m1, __m64 __m2) {`.
  **L558 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_pi32(__m64 __m1, __m64 __m2) {`。
- **L559 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR9`.
  **L559 CN**: 开始一个预处理条件块：`#if _ARCH_PWR9`。
- **L560 EN**: Adds a standalone statement or declaration: `__vector signed int __a, __b, __c;`.
  **L560 CN**: 添加一条独立语句或声明：`__vector signed int __a, __b, __c;`。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L562 EN**: Executes a call or declaration centered on `=`.
  **L562 CN**: 执行以 `=` 为核心的调用或声明。
- **L563 EN**: Executes a call or declaration centered on `=`.
  **L563 CN**: 执行以 `=` 为核心的调用或声明。
- **L564 EN**: Executes a call or declaration centered on `vec_sub`.
  **L564 CN**: 执行以 `vec_sub` 为核心的调用或声明。
- **L565 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L565 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L566 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L566 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L567 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L567 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L569 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L569 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L570 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L570 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __mu1.as_int[0] - __mu2.as_int[0];`.
  **L572 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __mu1.as_int[0] - __mu2.as_int[0];`。
- **L573 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __mu1.as_int[1] - __mu2.as_int[1];`.
  **L573 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __mu1.as_int[1] - __mu2.as_int[1];`。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L575 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L576 EN**: Closes the current preprocessor conditional block.
  **L576 CN**: 结束当前预处理条件块。

### Lines 577-600

````c
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psubd(__m64 __m1, __m64 __m2) {
  return _mm_sub_pi32(__m1, __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_si64(__m64 __m1, __m64 __m2) {
  return (__m1 + __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_si64(__m64 __m1, __m64 __m2) {
  return (__m1 - __m2);
}

/* Shift the 64-bit value in M left by COUNT.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sll_si64(__m64 __m, __m64 __count) {
````
- **L577 EN**: Closes the current lexical scope or compound statement.
  **L577 CN**: 结束当前词法作用域或复合语句块。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L579 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L579 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L580 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L580 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L581 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psubd(__m64 __m1, __m64 __m2) {`.
  **L581 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psubd(__m64 __m1, __m64 __m2) {`。
- **L582 EN**: Returns from the current function with `_mm_sub_pi32(__m1, __m2)`.
  **L582 CN**: 以 `_mm_sub_pi32(__m1, __m2)` 从当前函数返回。
- **L583 EN**: Closes the current lexical scope or compound statement.
  **L583 CN**: 结束当前词法作用域或复合语句块。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L585 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L585 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L586 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L586 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L587 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_si64(__m64 __m1, __m64 __m2) {`.
  **L587 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_si64(__m64 __m1, __m64 __m2) {`。
- **L588 EN**: Returns from the current function with `(__m1 + __m2)`.
  **L588 CN**: 以 `(__m1 + __m2)` 从当前函数返回。
- **L589 EN**: Closes the current lexical scope or compound statement.
  **L589 CN**: 结束当前词法作用域或复合语句块。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L591 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L591 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L592 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L592 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L593 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_si64(__m64 __m1, __m64 __m2) {`.
  **L593 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_si64(__m64 __m1, __m64 __m2) {`。
- **L594 EN**: Returns from the current function with `(__m1 - __m2)`.
  **L594 CN**: 以 `(__m1 - __m2)` 从当前函数返回。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L597 EN**: Comment explains nearby logic, constraints, or intent: `Shift the 64-bit value in M left by COUNT.`.
  **L597 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift the 64-bit value in M left by COUNT.`。
- **L598 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L598 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L599 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L599 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L600 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sll_si64(__m64 __m, __m64 __count) {`.
  **L600 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sll_si64(__m64 __m, __m64 __count) {`。

### Lines 601-624

````c
  return (__m << __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psllq(__m64 __m, __m64 __count) {
  return _mm_sll_si64(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_slli_si64(__m64 __m, const int __count) {
  return (__m << __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psllqi(__m64 __m, const int __count) {
  return _mm_slli_si64(__m, __count);
}

/* Shift the 64-bit value in M left by COUNT; shift in zeros.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L601 EN**: Returns from the current function with `(__m << __count)`.
  **L601 CN**: 以 `(__m << __count)` 从当前函数返回。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L604 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L604 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L605 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L605 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L606 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psllq(__m64 __m, __m64 __count) {`.
  **L606 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psllq(__m64 __m, __m64 __count) {`。
- **L607 EN**: Returns from the current function with `_mm_sll_si64(__m, __count)`.
  **L607 CN**: 以 `_mm_sll_si64(__m, __count)` 从当前函数返回。
- **L608 EN**: Closes the current lexical scope or compound statement.
  **L608 CN**: 结束当前词法作用域或复合语句块。
- **L609 EN**: Blank line separating nearby declarations or logic blocks.
  **L609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L610 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L610 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L611 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L611 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L612 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_slli_si64(__m64 __m, const int __count) {`.
  **L612 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_slli_si64(__m64 __m, const int __count) {`。
- **L613 EN**: Returns from the current function with `(__m << __count)`.
  **L613 CN**: 以 `(__m << __count)` 从当前函数返回。
- **L614 EN**: Closes the current lexical scope or compound statement.
  **L614 CN**: 结束当前词法作用域或复合语句块。
- **L615 EN**: Blank line separating nearby declarations or logic blocks.
  **L615 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L616 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L616 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L617 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L617 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L618 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psllqi(__m64 __m, const int __count) {`.
  **L618 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psllqi(__m64 __m, const int __count) {`。
- **L619 EN**: Returns from the current function with `_mm_slli_si64(__m, __count)`.
  **L619 CN**: 以 `_mm_slli_si64(__m, __count)` 从当前函数返回。
- **L620 EN**: Closes the current lexical scope or compound statement.
  **L620 CN**: 结束当前词法作用域或复合语句块。
- **L621 EN**: Blank line separating nearby declarations or logic blocks.
  **L621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L622 EN**: Comment explains nearby logic, constraints, or intent: `Shift the 64-bit value in M left by COUNT; shift in zeros.`.
  **L622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift the 64-bit value in M left by COUNT; shift in zeros.`。
- **L623 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L623 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L624 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L624 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 625-648

````c
    _mm_srl_si64(__m64 __m, __m64 __count) {
  return (__m >> __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psrlq(__m64 __m, __m64 __count) {
  return _mm_srl_si64(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srli_si64(__m64 __m, const int __count) {
  return (__m >> __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psrlqi(__m64 __m, const int __count) {
  return _mm_srli_si64(__m, __count);
}

/* Bit-wise AND the 64-bit values in M1 and M2.  */
extern __inline __m64
````
- **L625 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srl_si64(__m64 __m, __m64 __count) {`.
  **L625 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srl_si64(__m64 __m, __m64 __count) {`。
- **L626 EN**: Returns from the current function with `(__m >> __count)`.
  **L626 CN**: 以 `(__m >> __count)` 从当前函数返回。
- **L627 EN**: Closes the current lexical scope or compound statement.
  **L627 CN**: 结束当前词法作用域或复合语句块。
- **L628 EN**: Blank line separating nearby declarations or logic blocks.
  **L628 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L629 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L629 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L630 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L630 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L631 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psrlq(__m64 __m, __m64 __count) {`.
  **L631 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psrlq(__m64 __m, __m64 __count) {`。
- **L632 EN**: Returns from the current function with `_mm_srl_si64(__m, __count)`.
  **L632 CN**: 以 `_mm_srl_si64(__m, __count)` 从当前函数返回。
- **L633 EN**: Closes the current lexical scope or compound statement.
  **L633 CN**: 结束当前词法作用域或复合语句块。
- **L634 EN**: Blank line separating nearby declarations or logic blocks.
  **L634 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L635 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L635 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L636 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L636 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L637 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srli_si64(__m64 __m, const int __count) {`.
  **L637 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srli_si64(__m64 __m, const int __count) {`。
- **L638 EN**: Returns from the current function with `(__m >> __count)`.
  **L638 CN**: 以 `(__m >> __count)` 从当前函数返回。
- **L639 EN**: Closes the current lexical scope or compound statement.
  **L639 CN**: 结束当前词法作用域或复合语句块。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L641 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L641 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L642 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L642 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L643 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psrlqi(__m64 __m, const int __count) {`.
  **L643 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psrlqi(__m64 __m, const int __count) {`。
- **L644 EN**: Returns from the current function with `_mm_srli_si64(__m, __count)`.
  **L644 CN**: 以 `_mm_srli_si64(__m, __count)` 从当前函数返回。
- **L645 EN**: Closes the current lexical scope or compound statement.
  **L645 CN**: 结束当前词法作用域或复合语句块。
- **L646 EN**: Blank line separating nearby declarations or logic blocks.
  **L646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L647 EN**: Comment explains nearby logic, constraints, or intent: `Bit-wise AND the 64-bit values in M1 and M2.`.
  **L647 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit-wise AND the 64-bit values in M1 and M2.`。
- **L648 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L648 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。

### Lines 649-672

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_and_si64(__m64 __m1, __m64 __m2) {
  return (__m1 & __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pand(__m64 __m1, __m64 __m2) {
  return _mm_and_si64(__m1, __m2);
}

/* Bit-wise complement the 64-bit value in M1 and bit-wise AND it with the
   64-bit value in M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_andnot_si64(__m64 __m1, __m64 __m2) {
  return (~__m1 & __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pandn(__m64 __m1, __m64 __m2) {
  return _mm_andnot_si64(__m1, __m2);
}
````
- **L649 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L649 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L650 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_and_si64(__m64 __m1, __m64 __m2) {`.
  **L650 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_and_si64(__m64 __m1, __m64 __m2) {`。
- **L651 EN**: Returns from the current function with `(__m1 & __m2)`.
  **L651 CN**: 以 `(__m1 & __m2)` 从当前函数返回。
- **L652 EN**: Closes the current lexical scope or compound statement.
  **L652 CN**: 结束当前词法作用域或复合语句块。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L654 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L655 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L655 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L656 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pand(__m64 __m1, __m64 __m2) {`.
  **L656 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pand(__m64 __m1, __m64 __m2) {`。
- **L657 EN**: Returns from the current function with `_mm_and_si64(__m1, __m2)`.
  **L657 CN**: 以 `_mm_and_si64(__m1, __m2)` 从当前函数返回。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L660 EN**: Comment explains nearby logic, constraints, or intent: `Bit-wise complement the 64-bit value in M1 and bit-wise AND it with the`.
  **L660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit-wise complement the 64-bit value in M1 and bit-wise AND it with the`。
- **L661 EN**: Continues the surrounding expression or declaration: `64-bit value in M2.  */`.
  **L661 CN**: 继续构造周围的表达式或声明：`64-bit value in M2.  */`。
- **L662 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L662 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L663 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L663 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L664 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_andnot_si64(__m64 __m1, __m64 __m2) {`.
  **L664 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_andnot_si64(__m64 __m1, __m64 __m2) {`。
- **L665 EN**: Returns from the current function with `(~__m1 & __m2)`.
  **L665 CN**: 以 `(~__m1 & __m2)` 从当前函数返回。
- **L666 EN**: Closes the current lexical scope or compound statement.
  **L666 CN**: 结束当前词法作用域或复合语句块。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L668 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L668 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L669 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L669 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L670 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pandn(__m64 __m1, __m64 __m2) {`.
  **L670 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pandn(__m64 __m1, __m64 __m2) {`。
- **L671 EN**: Returns from the current function with `_mm_andnot_si64(__m1, __m2)`.
  **L671 CN**: 以 `_mm_andnot_si64(__m1, __m2)` 从当前函数返回。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````c

/* Bit-wise inclusive OR the 64-bit values in M1 and M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_or_si64(__m64 __m1, __m64 __m2) {
  return (__m1 | __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_por(__m64 __m1, __m64 __m2) {
  return _mm_or_si64(__m1, __m2);
}

/* Bit-wise exclusive OR the 64-bit values in M1 and M2.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_xor_si64(__m64 __m1, __m64 __m2) {
  return (__m1 ^ __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pxor(__m64 __m1, __m64 __m2) {
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Comment explains nearby logic, constraints, or intent: `Bit-wise inclusive OR the 64-bit values in M1 and M2.`.
  **L674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit-wise inclusive OR the 64-bit values in M1 and M2.`。
- **L675 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L675 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L676 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L676 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L677 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_or_si64(__m64 __m1, __m64 __m2) {`.
  **L677 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_or_si64(__m64 __m1, __m64 __m2) {`。
- **L678 EN**: Returns from the current function with `(__m1 | __m2)`.
  **L678 CN**: 以 `(__m1 | __m2)` 从当前函数返回。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L681 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L681 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L682 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L682 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L683 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_por(__m64 __m1, __m64 __m2) {`.
  **L683 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_por(__m64 __m1, __m64 __m2) {`。
- **L684 EN**: Returns from the current function with `_mm_or_si64(__m1, __m2)`.
  **L684 CN**: 以 `_mm_or_si64(__m1, __m2)` 从当前函数返回。
- **L685 EN**: Closes the current lexical scope or compound statement.
  **L685 CN**: 结束当前词法作用域或复合语句块。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L687 EN**: Comment explains nearby logic, constraints, or intent: `Bit-wise exclusive OR the 64-bit values in M1 and M2.`.
  **L687 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit-wise exclusive OR the 64-bit values in M1 and M2.`。
- **L688 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L688 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L689 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L689 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L690 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_xor_si64(__m64 __m1, __m64 __m2) {`.
  **L690 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_xor_si64(__m64 __m1, __m64 __m2) {`。
- **L691 EN**: Returns from the current function with `(__m1 ^ __m2)`.
  **L691 CN**: 以 `(__m1 ^ __m2)` 从当前函数返回。
- **L692 EN**: Closes the current lexical scope or compound statement.
  **L692 CN**: 结束当前词法作用域或复合语句块。
- **L693 EN**: Blank line separating nearby declarations or logic blocks.
  **L693 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L694 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L694 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L695 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L695 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L696 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pxor(__m64 __m1, __m64 __m2) {`.
  **L696 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pxor(__m64 __m1, __m64 __m2) {`。

### Lines 697-720

````c
  return _mm_xor_si64(__m1, __m2);
}

/* Creates a 64-bit zero.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setzero_si64(void) {
  return (__m64)0;
}

/* Compare eight 8-bit values.  The result of the comparison is 0xFF if the
   test is true and zero if false.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_pi8(__m64 __m1, __m64 __m2) {
#if defined(_ARCH_PWR6) && defined(__powerpc64__)
  __m64 __res;
  __asm__("cmpb %0,%1,%2;\n" : "=r"(__res) : "r"(__m1), "r"(__m2) :);
  return (__res);
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;
````
- **L697 EN**: Returns from the current function with `_mm_xor_si64(__m1, __m2)`.
  **L697 CN**: 以 `_mm_xor_si64(__m1, __m2)` 从当前函数返回。
- **L698 EN**: Closes the current lexical scope or compound statement.
  **L698 CN**: 结束当前词法作用域或复合语句块。
- **L699 EN**: Blank line separating nearby declarations or logic blocks.
  **L699 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L700 EN**: Comment explains nearby logic, constraints, or intent: `Creates a 64-bit zero.`.
  **L700 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a 64-bit zero.`。
- **L701 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L701 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L702 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L702 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L703 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setzero_si64(void) {`.
  **L703 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setzero_si64(void) {`。
- **L704 EN**: Returns from the current function with `(__m64)0`.
  **L704 CN**: 以 `(__m64)0` 从当前函数返回。
- **L705 EN**: Closes the current lexical scope or compound statement.
  **L705 CN**: 结束当前词法作用域或复合语句块。
- **L706 EN**: Blank line separating nearby declarations or logic blocks.
  **L706 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L707 EN**: Comment explains nearby logic, constraints, or intent: `Compare eight 8-bit values. The result of the comparison is 0xFF if the`.
  **L707 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare eight 8-bit values. The result of the comparison is 0xFF if the`。
- **L708 EN**: Continues the surrounding expression or declaration: `test is true and zero if false.  */`.
  **L708 CN**: 继续构造周围的表达式或声明：`test is true and zero if false.  */`。
- **L709 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L709 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L710 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L710 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L711 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_pi8(__m64 __m1, __m64 __m2) {`.
  **L711 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_pi8(__m64 __m1, __m64 __m2) {`。
- **L712 EN**: Starts a preprocessor conditional block: `#if defined(_ARCH_PWR6) && defined(__powerpc64__)`.
  **L712 CN**: 开始一个预处理条件块：`#if defined(_ARCH_PWR6) && defined(__powerpc64__)`。
- **L713 EN**: Adds a standalone statement or declaration: `__m64 __res;`.
  **L713 CN**: 添加一条独立语句或声明：`__m64 __res;`。
- **L714 EN**: Executes a call or declaration centered on `__asm__`.
  **L714 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L715 EN**: Returns from the current function with `(__res)`.
  **L715 CN**: 以 `(__res)` 从当前函数返回。
- **L716 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L716 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L717 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L717 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L718 EN**: Blank line separating nearby declarations or logic blocks.
  **L718 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L719 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L719 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L720 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L720 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。

### Lines 721-744

````c

  __res.as_char[0] = (__mu1.as_char[0] == __mu2.as_char[0]) ? -1 : 0;
  __res.as_char[1] = (__mu1.as_char[1] == __mu2.as_char[1]) ? -1 : 0;
  __res.as_char[2] = (__mu1.as_char[2] == __mu2.as_char[2]) ? -1 : 0;
  __res.as_char[3] = (__mu1.as_char[3] == __mu2.as_char[3]) ? -1 : 0;
  __res.as_char[4] = (__mu1.as_char[4] == __mu2.as_char[4]) ? -1 : 0;
  __res.as_char[5] = (__mu1.as_char[5] == __mu2.as_char[5]) ? -1 : 0;
  __res.as_char[6] = (__mu1.as_char[6] == __mu2.as_char[6]) ? -1 : 0;
  __res.as_char[7] = (__mu1.as_char[7] == __mu2.as_char[7]) ? -1 : 0;

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pcmpeqb(__m64 __m1, __m64 __m2) {
  return _mm_cmpeq_pi8(__m1, __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_pi8(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
````
- **L721 EN**: Blank line separating nearby declarations or logic blocks.
  **L721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L722 EN**: Executes a call or declaration centered on `=`.
  **L722 CN**: 执行以 `=` 为核心的调用或声明。
- **L723 EN**: Executes a call or declaration centered on `=`.
  **L723 CN**: 执行以 `=` 为核心的调用或声明。
- **L724 EN**: Executes a call or declaration centered on `=`.
  **L724 CN**: 执行以 `=` 为核心的调用或声明。
- **L725 EN**: Executes a call or declaration centered on `=`.
  **L725 CN**: 执行以 `=` 为核心的调用或声明。
- **L726 EN**: Executes a call or declaration centered on `=`.
  **L726 CN**: 执行以 `=` 为核心的调用或声明。
- **L727 EN**: Executes a call or declaration centered on `=`.
  **L727 CN**: 执行以 `=` 为核心的调用或声明。
- **L728 EN**: Executes a call or declaration centered on `=`.
  **L728 CN**: 执行以 `=` 为核心的调用或声明。
- **L729 EN**: Executes a call or declaration centered on `=`.
  **L729 CN**: 执行以 `=` 为核心的调用或声明。
- **L730 EN**: Blank line separating nearby declarations or logic blocks.
  **L730 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L731 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L731 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L732 EN**: Closes the current preprocessor conditional block.
  **L732 CN**: 结束当前预处理条件块。
- **L733 EN**: Closes the current lexical scope or compound statement.
  **L733 CN**: 结束当前词法作用域或复合语句块。
- **L734 EN**: Blank line separating nearby declarations or logic blocks.
  **L734 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L735 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L735 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L736 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L736 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L737 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pcmpeqb(__m64 __m1, __m64 __m2) {`.
  **L737 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pcmpeqb(__m64 __m1, __m64 __m2) {`。
- **L738 EN**: Returns from the current function with `_mm_cmpeq_pi8(__m1, __m2)`.
  **L738 CN**: 以 `_mm_cmpeq_pi8(__m1, __m2)` 从当前函数返回。
- **L739 EN**: Closes the current lexical scope or compound statement.
  **L739 CN**: 结束当前词法作用域或复合语句块。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L741 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L742 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L742 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L743 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_pi8(__m64 __m1, __m64 __m2) {`.
  **L743 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_pi8(__m64 __m1, __m64 __m2) {`。
- **L744 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L744 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。

### Lines 745-768

````c
  __vector signed char __a, __b, __c;

  __a = (__vector signed char)vec_splats(__m1);
  __b = (__vector signed char)vec_splats(__m2);
  __c = (__vector signed char)vec_cmpgt(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_char[0] = (__mu1.as_char[0] > __mu2.as_char[0]) ? -1 : 0;
  __res.as_char[1] = (__mu1.as_char[1] > __mu2.as_char[1]) ? -1 : 0;
  __res.as_char[2] = (__mu1.as_char[2] > __mu2.as_char[2]) ? -1 : 0;
  __res.as_char[3] = (__mu1.as_char[3] > __mu2.as_char[3]) ? -1 : 0;
  __res.as_char[4] = (__mu1.as_char[4] > __mu2.as_char[4]) ? -1 : 0;
  __res.as_char[5] = (__mu1.as_char[5] > __mu2.as_char[5]) ? -1 : 0;
  __res.as_char[6] = (__mu1.as_char[6] > __mu2.as_char[6]) ? -1 : 0;
  __res.as_char[7] = (__mu1.as_char[7] > __mu2.as_char[7]) ? -1 : 0;

  return (__m64)__res.as_m64;
#endif
}
````
- **L745 EN**: Adds a standalone statement or declaration: `__vector signed char __a, __b, __c;`.
  **L745 CN**: 添加一条独立语句或声明：`__vector signed char __a, __b, __c;`。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L747 EN**: Executes a call or declaration centered on `=`.
  **L747 CN**: 执行以 `=` 为核心的调用或声明。
- **L748 EN**: Executes a call or declaration centered on `=`.
  **L748 CN**: 执行以 `=` 为核心的调用或声明。
- **L749 EN**: Executes a call or declaration centered on `=`.
  **L749 CN**: 执行以 `=` 为核心的调用或声明。
- **L750 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L750 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L751 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L751 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L752 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L752 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L753 EN**: Blank line separating nearby declarations or logic blocks.
  **L753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L754 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L754 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L755 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L755 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L756 EN**: Blank line separating nearby declarations or logic blocks.
  **L756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L757 EN**: Executes a call or declaration centered on `=`.
  **L757 CN**: 执行以 `=` 为核心的调用或声明。
- **L758 EN**: Executes a call or declaration centered on `=`.
  **L758 CN**: 执行以 `=` 为核心的调用或声明。
- **L759 EN**: Executes a call or declaration centered on `=`.
  **L759 CN**: 执行以 `=` 为核心的调用或声明。
- **L760 EN**: Executes a call or declaration centered on `=`.
  **L760 CN**: 执行以 `=` 为核心的调用或声明。
- **L761 EN**: Executes a call or declaration centered on `=`.
  **L761 CN**: 执行以 `=` 为核心的调用或声明。
- **L762 EN**: Executes a call or declaration centered on `=`.
  **L762 CN**: 执行以 `=` 为核心的调用或声明。
- **L763 EN**: Executes a call or declaration centered on `=`.
  **L763 CN**: 执行以 `=` 为核心的调用或声明。
- **L764 EN**: Executes a call or declaration centered on `=`.
  **L764 CN**: 执行以 `=` 为核心的调用或声明。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L766 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L766 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L767 EN**: Closes the current preprocessor conditional block.
  **L767 CN**: 结束当前预处理条件块。
- **L768 EN**: Closes the current lexical scope or compound statement.
  **L768 CN**: 结束当前词法作用域或复合语句块。

### Lines 769-792

````c

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pcmpgtb(__m64 __m1, __m64 __m2) {
  return _mm_cmpgt_pi8(__m1, __m2);
}

/* Compare four 16-bit values.  The result of the comparison is 0xFFFF if
   the test is true and zero if false.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_pi16(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
  __vector signed short __a, __b, __c;

  __a = (__vector signed short)vec_splats(__m1);
  __b = (__vector signed short)vec_splats(__m2);
  __c = (__vector signed short)vec_cmpeq(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;
````
- **L769 EN**: Blank line separating nearby declarations or logic blocks.
  **L769 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L770 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L770 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L771 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L771 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L772 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pcmpgtb(__m64 __m1, __m64 __m2) {`.
  **L772 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pcmpgtb(__m64 __m1, __m64 __m2) {`。
- **L773 EN**: Returns from the current function with `_mm_cmpgt_pi8(__m1, __m2)`.
  **L773 CN**: 以 `_mm_cmpgt_pi8(__m1, __m2)` 从当前函数返回。
- **L774 EN**: Closes the current lexical scope or compound statement.
  **L774 CN**: 结束当前词法作用域或复合语句块。
- **L775 EN**: Blank line separating nearby declarations or logic blocks.
  **L775 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L776 EN**: Comment explains nearby logic, constraints, or intent: `Compare four 16-bit values. The result of the comparison is 0xFFFF if`.
  **L776 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare four 16-bit values. The result of the comparison is 0xFFFF if`。
- **L777 EN**: Continues the surrounding expression or declaration: `the test is true and zero if false.  */`.
  **L777 CN**: 继续构造周围的表达式或声明：`the test is true and zero if false.  */`。
- **L778 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L778 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L779 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L779 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L780 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_pi16(__m64 __m1, __m64 __m2) {`.
  **L780 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_pi16(__m64 __m1, __m64 __m2) {`。
- **L781 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L781 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L782 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __c;`.
  **L782 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __c;`。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L784 EN**: Executes a call or declaration centered on `=`.
  **L784 CN**: 执行以 `=` 为核心的调用或声明。
- **L785 EN**: Executes a call or declaration centered on `=`.
  **L785 CN**: 执行以 `=` 为核心的调用或声明。
- **L786 EN**: Executes a call or declaration centered on `=`.
  **L786 CN**: 执行以 `=` 为核心的调用或声明。
- **L787 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L787 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L788 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L788 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L789 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L789 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L790 EN**: Blank line separating nearby declarations or logic blocks.
  **L790 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L791 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L791 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L792 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L792 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。

### Lines 793-816

````c

  __res.as_short[0] = (__mu1.as_short[0] == __mu2.as_short[0]) ? -1 : 0;
  __res.as_short[1] = (__mu1.as_short[1] == __mu2.as_short[1]) ? -1 : 0;
  __res.as_short[2] = (__mu1.as_short[2] == __mu2.as_short[2]) ? -1 : 0;
  __res.as_short[3] = (__mu1.as_short[3] == __mu2.as_short[3]) ? -1 : 0;

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pcmpeqw(__m64 __m1, __m64 __m2) {
  return _mm_cmpeq_pi16(__m1, __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_pi16(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR8
  __vector signed short __a, __b, __c;

  __a = (__vector signed short)vec_splats(__m1);
  __b = (__vector signed short)vec_splats(__m2);
````
- **L793 EN**: Blank line separating nearby declarations or logic blocks.
  **L793 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L794 EN**: Executes a call or declaration centered on `=`.
  **L794 CN**: 执行以 `=` 为核心的调用或声明。
- **L795 EN**: Executes a call or declaration centered on `=`.
  **L795 CN**: 执行以 `=` 为核心的调用或声明。
- **L796 EN**: Executes a call or declaration centered on `=`.
  **L796 CN**: 执行以 `=` 为核心的调用或声明。
- **L797 EN**: Executes a call or declaration centered on `=`.
  **L797 CN**: 执行以 `=` 为核心的调用或声明。
- **L798 EN**: Blank line separating nearby declarations or logic blocks.
  **L798 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L799 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L799 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L800 EN**: Closes the current preprocessor conditional block.
  **L800 CN**: 结束当前预处理条件块。
- **L801 EN**: Closes the current lexical scope or compound statement.
  **L801 CN**: 结束当前词法作用域或复合语句块。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L803 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L803 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L804 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L804 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L805 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pcmpeqw(__m64 __m1, __m64 __m2) {`.
  **L805 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pcmpeqw(__m64 __m1, __m64 __m2) {`。
- **L806 EN**: Returns from the current function with `_mm_cmpeq_pi16(__m1, __m2)`.
  **L806 CN**: 以 `_mm_cmpeq_pi16(__m1, __m2)` 从当前函数返回。
- **L807 EN**: Closes the current lexical scope or compound statement.
  **L807 CN**: 结束当前词法作用域或复合语句块。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L809 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L809 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L810 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L810 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L811 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_pi16(__m64 __m1, __m64 __m2) {`.
  **L811 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_pi16(__m64 __m1, __m64 __m2) {`。
- **L812 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L812 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L813 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __c;`.
  **L813 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __c;`。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Executes a call or declaration centered on `=`.
  **L815 CN**: 执行以 `=` 为核心的调用或声明。
- **L816 EN**: Executes a call or declaration centered on `=`.
  **L816 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 817-840

````c
  __c = (__vector signed short)vec_cmpgt(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_short[0] = (__mu1.as_short[0] > __mu2.as_short[0]) ? -1 : 0;
  __res.as_short[1] = (__mu1.as_short[1] > __mu2.as_short[1]) ? -1 : 0;
  __res.as_short[2] = (__mu1.as_short[2] > __mu2.as_short[2]) ? -1 : 0;
  __res.as_short[3] = (__mu1.as_short[3] > __mu2.as_short[3]) ? -1 : 0;

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pcmpgtw(__m64 __m1, __m64 __m2) {
  return _mm_cmpgt_pi16(__m1, __m2);
}

/* Compare two 32-bit values.  The result of the comparison is 0xFFFFFFFF if
````
- **L817 EN**: Executes a call or declaration centered on `=`.
  **L817 CN**: 执行以 `=` 为核心的调用或声明。
- **L818 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L818 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L819 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L819 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L820 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L820 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L822 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L822 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L823 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L823 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L824 EN**: Blank line separating nearby declarations or logic blocks.
  **L824 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L825 EN**: Executes a call or declaration centered on `=`.
  **L825 CN**: 执行以 `=` 为核心的调用或声明。
- **L826 EN**: Executes a call or declaration centered on `=`.
  **L826 CN**: 执行以 `=` 为核心的调用或声明。
- **L827 EN**: Executes a call or declaration centered on `=`.
  **L827 CN**: 执行以 `=` 为核心的调用或声明。
- **L828 EN**: Executes a call or declaration centered on `=`.
  **L828 CN**: 执行以 `=` 为核心的调用或声明。
- **L829 EN**: Blank line separating nearby declarations or logic blocks.
  **L829 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L830 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L830 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L831 EN**: Closes the current preprocessor conditional block.
  **L831 CN**: 结束当前预处理条件块。
- **L832 EN**: Closes the current lexical scope or compound statement.
  **L832 CN**: 结束当前词法作用域或复合语句块。
- **L833 EN**: Blank line separating nearby declarations or logic blocks.
  **L833 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L834 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L834 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L835 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L835 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L836 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pcmpgtw(__m64 __m1, __m64 __m2) {`.
  **L836 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pcmpgtw(__m64 __m1, __m64 __m2) {`。
- **L837 EN**: Returns from the current function with `_mm_cmpgt_pi16(__m1, __m2)`.
  **L837 CN**: 以 `_mm_cmpgt_pi16(__m1, __m2)` 从当前函数返回。
- **L838 EN**: Closes the current lexical scope or compound statement.
  **L838 CN**: 结束当前词法作用域或复合语句块。
- **L839 EN**: Blank line separating nearby declarations or logic blocks.
  **L839 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L840 EN**: Comment explains nearby logic, constraints, or intent: `Compare two 32-bit values. The result of the comparison is 0xFFFFFFFF if`.
  **L840 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare two 32-bit values. The result of the comparison is 0xFFFFFFFF if`。

### Lines 841-864

````c
   the test is true and zero if false.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_pi32(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR9
  __vector signed int __a, __b, __c;

  __a = (__vector signed int)vec_splats(__m1);
  __b = (__vector signed int)vec_splats(__m2);
  __c = (__vector signed int)vec_cmpeq(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_int[0] = (__mu1.as_int[0] == __mu2.as_int[0]) ? -1 : 0;
  __res.as_int[1] = (__mu1.as_int[1] == __mu2.as_int[1]) ? -1 : 0;

  return (__m64)__res.as_m64;
#endif
}

````
- **L841 EN**: Continues the surrounding expression or declaration: `the test is true and zero if false.  */`.
  **L841 CN**: 继续构造周围的表达式或声明：`the test is true and zero if false.  */`。
- **L842 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L842 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L843 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L843 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L844 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_pi32(__m64 __m1, __m64 __m2) {`.
  **L844 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_pi32(__m64 __m1, __m64 __m2) {`。
- **L845 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR9`.
  **L845 CN**: 开始一个预处理条件块：`#if _ARCH_PWR9`。
- **L846 EN**: Adds a standalone statement or declaration: `__vector signed int __a, __b, __c;`.
  **L846 CN**: 添加一条独立语句或声明：`__vector signed int __a, __b, __c;`。
- **L847 EN**: Blank line separating nearby declarations or logic blocks.
  **L847 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L848 EN**: Executes a call or declaration centered on `=`.
  **L848 CN**: 执行以 `=` 为核心的调用或声明。
- **L849 EN**: Executes a call or declaration centered on `=`.
  **L849 CN**: 执行以 `=` 为核心的调用或声明。
- **L850 EN**: Executes a call or declaration centered on `=`.
  **L850 CN**: 执行以 `=` 为核心的调用或声明。
- **L851 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L851 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L852 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L852 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L853 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L853 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L855 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L855 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L856 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L856 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L857 EN**: Blank line separating nearby declarations or logic blocks.
  **L857 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L858 EN**: Executes a call or declaration centered on `=`.
  **L858 CN**: 执行以 `=` 为核心的调用或声明。
- **L859 EN**: Executes a call or declaration centered on `=`.
  **L859 CN**: 执行以 `=` 为核心的调用或声明。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L861 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L861 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L862 EN**: Closes the current preprocessor conditional block.
  **L862 CN**: 结束当前预处理条件块。
- **L863 EN**: Closes the current lexical scope or compound statement.
  **L863 CN**: 结束当前词法作用域或复合语句块。
- **L864 EN**: Blank line separating nearby declarations or logic blocks.
  **L864 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 865-888

````c
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pcmpeqd(__m64 __m1, __m64 __m2) {
  return _mm_cmpeq_pi32(__m1, __m2);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_pi32(__m64 __m1, __m64 __m2) {
#if _ARCH_PWR9
  __vector signed int __a, __b, __c;

  __a = (__vector signed int)vec_splats(__m1);
  __b = (__vector signed int)vec_splats(__m2);
  __c = (__vector signed int)vec_cmpgt(__a, __b);
  return (__m64)((__vector long long)__c)[0];
#else
  __m64_union __mu1, __mu2, __res;

  __mu1.as_m64 = __m1;
  __mu2.as_m64 = __m2;

  __res.as_int[0] = (__mu1.as_int[0] > __mu2.as_int[0]) ? -1 : 0;
  __res.as_int[1] = (__mu1.as_int[1] > __mu2.as_int[1]) ? -1 : 0;
````
- **L865 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L865 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L866 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L866 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L867 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pcmpeqd(__m64 __m1, __m64 __m2) {`.
  **L867 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pcmpeqd(__m64 __m1, __m64 __m2) {`。
- **L868 EN**: Returns from the current function with `_mm_cmpeq_pi32(__m1, __m2)`.
  **L868 CN**: 以 `_mm_cmpeq_pi32(__m1, __m2)` 从当前函数返回。
- **L869 EN**: Closes the current lexical scope or compound statement.
  **L869 CN**: 结束当前词法作用域或复合语句块。
- **L870 EN**: Blank line separating nearby declarations or logic blocks.
  **L870 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L871 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L871 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L872 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L872 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L873 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_pi32(__m64 __m1, __m64 __m2) {`.
  **L873 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_pi32(__m64 __m1, __m64 __m2) {`。
- **L874 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR9`.
  **L874 CN**: 开始一个预处理条件块：`#if _ARCH_PWR9`。
- **L875 EN**: Adds a standalone statement or declaration: `__vector signed int __a, __b, __c;`.
  **L875 CN**: 添加一条独立语句或声明：`__vector signed int __a, __b, __c;`。
- **L876 EN**: Blank line separating nearby declarations or logic blocks.
  **L876 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L877 EN**: Executes a call or declaration centered on `=`.
  **L877 CN**: 执行以 `=` 为核心的调用或声明。
- **L878 EN**: Executes a call or declaration centered on `=`.
  **L878 CN**: 执行以 `=` 为核心的调用或声明。
- **L879 EN**: Executes a call or declaration centered on `=`.
  **L879 CN**: 执行以 `=` 为核心的调用或声明。
- **L880 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L880 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L881 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L881 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L882 EN**: Adds a standalone statement or declaration: `__m64_union __mu1, __mu2, __res;`.
  **L882 CN**: 添加一条独立语句或声明：`__m64_union __mu1, __mu2, __res;`。
- **L883 EN**: Blank line separating nearby declarations or logic blocks.
  **L883 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L884 EN**: Adds a standalone statement or declaration: `__mu1.as_m64 = __m1;`.
  **L884 CN**: 添加一条独立语句或声明：`__mu1.as_m64 = __m1;`。
- **L885 EN**: Adds a standalone statement or declaration: `__mu2.as_m64 = __m2;`.
  **L885 CN**: 添加一条独立语句或声明：`__mu2.as_m64 = __m2;`。
- **L886 EN**: Blank line separating nearby declarations or logic blocks.
  **L886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L887 EN**: Executes a call or declaration centered on `=`.
  **L887 CN**: 执行以 `=` 为核心的调用或声明。
- **L888 EN**: Executes a call or declaration centered on `=`.
  **L888 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 889-912

````c

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pcmpgtd(__m64 __m1, __m64 __m2) {
  return _mm_cmpgt_pi32(__m1, __m2);
}

#if _ARCH_PWR8
/* Add the 8-bit values in M1 to the 8-bit values in M2 using signed
   saturated arithmetic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_adds_pi8(__m64 __m1, __m64 __m2) {
  __vector signed char __a, __b, __c;

  __a = (__vector signed char)vec_splats(__m1);
  __b = (__vector signed char)vec_splats(__m2);
  __c = vec_adds(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L890 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L890 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L891 EN**: Closes the current preprocessor conditional block.
  **L891 CN**: 结束当前预处理条件块。
- **L892 EN**: Closes the current lexical scope or compound statement.
  **L892 CN**: 结束当前词法作用域或复合语句块。
- **L893 EN**: Blank line separating nearby declarations or logic blocks.
  **L893 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L894 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L894 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L895 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L895 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L896 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pcmpgtd(__m64 __m1, __m64 __m2) {`.
  **L896 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pcmpgtd(__m64 __m1, __m64 __m2) {`。
- **L897 EN**: Returns from the current function with `_mm_cmpgt_pi32(__m1, __m2)`.
  **L897 CN**: 以 `_mm_cmpgt_pi32(__m1, __m2)` 从当前函数返回。
- **L898 EN**: Closes the current lexical scope or compound statement.
  **L898 CN**: 结束当前词法作用域或复合语句块。
- **L899 EN**: Blank line separating nearby declarations or logic blocks.
  **L899 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L900 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L900 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L901 EN**: Comment explains nearby logic, constraints, or intent: `Add the 8-bit values in M1 to the 8-bit values in M2 using signed`.
  **L901 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add the 8-bit values in M1 to the 8-bit values in M2 using signed`。
- **L902 EN**: Continues the surrounding expression or declaration: `saturated arithmetic.  */`.
  **L902 CN**: 继续构造周围的表达式或声明：`saturated arithmetic.  */`。
- **L903 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L903 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L904 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L904 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L905 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_adds_pi8(__m64 __m1, __m64 __m2) {`.
  **L905 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_adds_pi8(__m64 __m1, __m64 __m2) {`。
- **L906 EN**: Adds a standalone statement or declaration: `__vector signed char __a, __b, __c;`.
  **L906 CN**: 添加一条独立语句或声明：`__vector signed char __a, __b, __c;`。
- **L907 EN**: Blank line separating nearby declarations or logic blocks.
  **L907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L908 EN**: Executes a call or declaration centered on `=`.
  **L908 CN**: 执行以 `=` 为核心的调用或声明。
- **L909 EN**: Executes a call or declaration centered on `=`.
  **L909 CN**: 执行以 `=` 为核心的调用或声明。
- **L910 EN**: Executes a call or declaration centered on `vec_adds`.
  **L910 CN**: 执行以 `vec_adds` 为核心的调用或声明。
- **L911 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L911 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L912 EN**: Closes the current lexical scope or compound statement.
  **L912 CN**: 结束当前词法作用域或复合语句块。

### Lines 913-936

````c

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_paddsb(__m64 __m1, __m64 __m2) {
  return _mm_adds_pi8(__m1, __m2);
}
/* Add the 16-bit values in M1 to the 16-bit values in M2 using signed
   saturated arithmetic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_adds_pi16(__m64 __m1, __m64 __m2) {
  __vector signed short __a, __b, __c;

  __a = (__vector signed short)vec_splats(__m1);
  __b = (__vector signed short)vec_splats(__m2);
  __c = vec_adds(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_paddsw(__m64 __m1, __m64 __m2) {
  return _mm_adds_pi16(__m1, __m2);
}
````
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L914 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L914 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L915 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L915 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L916 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_paddsb(__m64 __m1, __m64 __m2) {`.
  **L916 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_paddsb(__m64 __m1, __m64 __m2) {`。
- **L917 EN**: Returns from the current function with `_mm_adds_pi8(__m1, __m2)`.
  **L917 CN**: 以 `_mm_adds_pi8(__m1, __m2)` 从当前函数返回。
- **L918 EN**: Closes the current lexical scope or compound statement.
  **L918 CN**: 结束当前词法作用域或复合语句块。
- **L919 EN**: Comment explains nearby logic, constraints, or intent: `Add the 16-bit values in M1 to the 16-bit values in M2 using signed`.
  **L919 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add the 16-bit values in M1 to the 16-bit values in M2 using signed`。
- **L920 EN**: Continues the surrounding expression or declaration: `saturated arithmetic.  */`.
  **L920 CN**: 继续构造周围的表达式或声明：`saturated arithmetic.  */`。
- **L921 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L921 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L922 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L922 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L923 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_adds_pi16(__m64 __m1, __m64 __m2) {`.
  **L923 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_adds_pi16(__m64 __m1, __m64 __m2) {`。
- **L924 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __c;`.
  **L924 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __c;`。
- **L925 EN**: Blank line separating nearby declarations or logic blocks.
  **L925 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L926 EN**: Executes a call or declaration centered on `=`.
  **L926 CN**: 执行以 `=` 为核心的调用或声明。
- **L927 EN**: Executes a call or declaration centered on `=`.
  **L927 CN**: 执行以 `=` 为核心的调用或声明。
- **L928 EN**: Executes a call or declaration centered on `vec_adds`.
  **L928 CN**: 执行以 `vec_adds` 为核心的调用或声明。
- **L929 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L929 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L930 EN**: Closes the current lexical scope or compound statement.
  **L930 CN**: 结束当前词法作用域或复合语句块。
- **L931 EN**: Blank line separating nearby declarations or logic blocks.
  **L931 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L932 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L932 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L933 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L933 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L934 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_paddsw(__m64 __m1, __m64 __m2) {`.
  **L934 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_paddsw(__m64 __m1, __m64 __m2) {`。
- **L935 EN**: Returns from the current function with `_mm_adds_pi16(__m1, __m2)`.
  **L935 CN**: 以 `_mm_adds_pi16(__m1, __m2)` 从当前函数返回。
- **L936 EN**: Closes the current lexical scope or compound statement.
  **L936 CN**: 结束当前词法作用域或复合语句块。

### Lines 937-960

````c
/* Add the 8-bit values in M1 to the 8-bit values in M2 using unsigned
   saturated arithmetic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_adds_pu8(__m64 __m1, __m64 __m2) {
  __vector unsigned char __a, __b, __c;

  __a = (__vector unsigned char)vec_splats(__m1);
  __b = (__vector unsigned char)vec_splats(__m2);
  __c = vec_adds(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_paddusb(__m64 __m1, __m64 __m2) {
  return _mm_adds_pu8(__m1, __m2);
}

/* Add the 16-bit values in M1 to the 16-bit values in M2 using unsigned
   saturated arithmetic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_adds_pu16(__m64 __m1, __m64 __m2) {
````
- **L937 EN**: Comment explains nearby logic, constraints, or intent: `Add the 8-bit values in M1 to the 8-bit values in M2 using unsigned`.
  **L937 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add the 8-bit values in M1 to the 8-bit values in M2 using unsigned`。
- **L938 EN**: Continues the surrounding expression or declaration: `saturated arithmetic.  */`.
  **L938 CN**: 继续构造周围的表达式或声明：`saturated arithmetic.  */`。
- **L939 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L939 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L940 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L940 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L941 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_adds_pu8(__m64 __m1, __m64 __m2) {`.
  **L941 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_adds_pu8(__m64 __m1, __m64 __m2) {`。
- **L942 EN**: Adds a standalone statement or declaration: `__vector unsigned char __a, __b, __c;`.
  **L942 CN**: 添加一条独立语句或声明：`__vector unsigned char __a, __b, __c;`。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Executes a call or declaration centered on `=`.
  **L944 CN**: 执行以 `=` 为核心的调用或声明。
- **L945 EN**: Executes a call or declaration centered on `=`.
  **L945 CN**: 执行以 `=` 为核心的调用或声明。
- **L946 EN**: Executes a call or declaration centered on `vec_adds`.
  **L946 CN**: 执行以 `vec_adds` 为核心的调用或声明。
- **L947 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L947 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L948 EN**: Closes the current lexical scope or compound statement.
  **L948 CN**: 结束当前词法作用域或复合语句块。
- **L949 EN**: Blank line separating nearby declarations or logic blocks.
  **L949 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L950 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L950 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L951 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L951 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_paddusb(__m64 __m1, __m64 __m2) {`.
  **L952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_paddusb(__m64 __m1, __m64 __m2) {`。
- **L953 EN**: Returns from the current function with `_mm_adds_pu8(__m1, __m2)`.
  **L953 CN**: 以 `_mm_adds_pu8(__m1, __m2)` 从当前函数返回。
- **L954 EN**: Closes the current lexical scope or compound statement.
  **L954 CN**: 结束当前词法作用域或复合语句块。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L956 EN**: Comment explains nearby logic, constraints, or intent: `Add the 16-bit values in M1 to the 16-bit values in M2 using unsigned`.
  **L956 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add the 16-bit values in M1 to the 16-bit values in M2 using unsigned`。
- **L957 EN**: Continues the surrounding expression or declaration: `saturated arithmetic.  */`.
  **L957 CN**: 继续构造周围的表达式或声明：`saturated arithmetic.  */`。
- **L958 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L958 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L959 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L959 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L960 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_adds_pu16(__m64 __m1, __m64 __m2) {`.
  **L960 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_adds_pu16(__m64 __m1, __m64 __m2) {`。

### Lines 961-984

````c
  __vector unsigned short __a, __b, __c;

  __a = (__vector unsigned short)vec_splats(__m1);
  __b = (__vector unsigned short)vec_splats(__m2);
  __c = vec_adds(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_paddusw(__m64 __m1, __m64 __m2) {
  return _mm_adds_pu16(__m1, __m2);
}

/* Subtract the 8-bit values in M2 from the 8-bit values in M1 using signed
   saturating arithmetic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_subs_pi8(__m64 __m1, __m64 __m2) {
  __vector signed char __a, __b, __c;

  __a = (__vector signed char)vec_splats(__m1);
  __b = (__vector signed char)vec_splats(__m2);
  __c = vec_subs(__a, __b);
````
- **L961 EN**: Adds a standalone statement or declaration: `__vector unsigned short __a, __b, __c;`.
  **L961 CN**: 添加一条独立语句或声明：`__vector unsigned short __a, __b, __c;`。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L963 EN**: Executes a call or declaration centered on `=`.
  **L963 CN**: 执行以 `=` 为核心的调用或声明。
- **L964 EN**: Executes a call or declaration centered on `=`.
  **L964 CN**: 执行以 `=` 为核心的调用或声明。
- **L965 EN**: Executes a call or declaration centered on `vec_adds`.
  **L965 CN**: 执行以 `vec_adds` 为核心的调用或声明。
- **L966 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L966 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L967 EN**: Closes the current lexical scope or compound statement.
  **L967 CN**: 结束当前词法作用域或复合语句块。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L969 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L970 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L970 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L971 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_paddusw(__m64 __m1, __m64 __m2) {`.
  **L971 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_paddusw(__m64 __m1, __m64 __m2) {`。
- **L972 EN**: Returns from the current function with `_mm_adds_pu16(__m1, __m2)`.
  **L972 CN**: 以 `_mm_adds_pu16(__m1, __m2)` 从当前函数返回。
- **L973 EN**: Closes the current lexical scope or compound statement.
  **L973 CN**: 结束当前词法作用域或复合语句块。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Comment explains nearby logic, constraints, or intent: `Subtract the 8-bit values in M2 from the 8-bit values in M1 using signed`.
  **L975 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtract the 8-bit values in M2 from the 8-bit values in M1 using signed`。
- **L976 EN**: Continues the surrounding expression or declaration: `saturating arithmetic.  */`.
  **L976 CN**: 继续构造周围的表达式或声明：`saturating arithmetic.  */`。
- **L977 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L977 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L978 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L978 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L979 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_subs_pi8(__m64 __m1, __m64 __m2) {`.
  **L979 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_subs_pi8(__m64 __m1, __m64 __m2) {`。
- **L980 EN**: Adds a standalone statement or declaration: `__vector signed char __a, __b, __c;`.
  **L980 CN**: 添加一条独立语句或声明：`__vector signed char __a, __b, __c;`。
- **L981 EN**: Blank line separating nearby declarations or logic blocks.
  **L981 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L982 EN**: Executes a call or declaration centered on `=`.
  **L982 CN**: 执行以 `=` 为核心的调用或声明。
- **L983 EN**: Executes a call or declaration centered on `=`.
  **L983 CN**: 执行以 `=` 为核心的调用或声明。
- **L984 EN**: Executes a call or declaration centered on `vec_subs`.
  **L984 CN**: 执行以 `vec_subs` 为核心的调用或声明。

### Lines 985-1008

````c
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psubsb(__m64 __m1, __m64 __m2) {
  return _mm_subs_pi8(__m1, __m2);
}

/* Subtract the 16-bit values in M2 from the 16-bit values in M1 using
   signed saturating arithmetic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_subs_pi16(__m64 __m1, __m64 __m2) {
  __vector signed short __a, __b, __c;

  __a = (__vector signed short)vec_splats(__m1);
  __b = (__vector signed short)vec_splats(__m2);
  __c = vec_subs(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L985 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L985 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L986 EN**: Closes the current lexical scope or compound statement.
  **L986 CN**: 结束当前词法作用域或复合语句块。
- **L987 EN**: Blank line separating nearby declarations or logic blocks.
  **L987 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L988 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L988 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L989 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L989 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L990 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psubsb(__m64 __m1, __m64 __m2) {`.
  **L990 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psubsb(__m64 __m1, __m64 __m2) {`。
- **L991 EN**: Returns from the current function with `_mm_subs_pi8(__m1, __m2)`.
  **L991 CN**: 以 `_mm_subs_pi8(__m1, __m2)` 从当前函数返回。
- **L992 EN**: Closes the current lexical scope or compound statement.
  **L992 CN**: 结束当前词法作用域或复合语句块。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L994 EN**: Comment explains nearby logic, constraints, or intent: `Subtract the 16-bit values in M2 from the 16-bit values in M1 using`.
  **L994 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtract the 16-bit values in M2 from the 16-bit values in M1 using`。
- **L995 EN**: Continues the surrounding expression or declaration: `signed saturating arithmetic.  */`.
  **L995 CN**: 继续构造周围的表达式或声明：`signed saturating arithmetic.  */`。
- **L996 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L996 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L997 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L997 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L998 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_subs_pi16(__m64 __m1, __m64 __m2) {`.
  **L998 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_subs_pi16(__m64 __m1, __m64 __m2) {`。
- **L999 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __c;`.
  **L999 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __c;`。
- **L1000 EN**: Blank line separating nearby declarations or logic blocks.
  **L1000 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1001 EN**: Executes a call or declaration centered on `=`.
  **L1001 CN**: 执行以 `=` 为核心的调用或声明。
- **L1002 EN**: Executes a call or declaration centered on `=`.
  **L1002 CN**: 执行以 `=` 为核心的调用或声明。
- **L1003 EN**: Executes a call or declaration centered on `vec_subs`.
  **L1003 CN**: 执行以 `vec_subs` 为核心的调用或声明。
- **L1004 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1004 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1005 EN**: Closes the current lexical scope or compound statement.
  **L1005 CN**: 结束当前词法作用域或复合语句块。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1007 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1007 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1008 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1008 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 1009-1032

````c
    _m_psubsw(__m64 __m1, __m64 __m2) {
  return _mm_subs_pi16(__m1, __m2);
}

/* Subtract the 8-bit values in M2 from the 8-bit values in M1 using
   unsigned saturating arithmetic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_subs_pu8(__m64 __m1, __m64 __m2) {
  __vector unsigned char __a, __b, __c;

  __a = (__vector unsigned char)vec_splats(__m1);
  __b = (__vector unsigned char)vec_splats(__m2);
  __c = vec_subs(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psubusb(__m64 __m1, __m64 __m2) {
  return _mm_subs_pu8(__m1, __m2);
}

/* Subtract the 16-bit values in M2 from the 16-bit values in M1 using
````
- **L1009 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psubsw(__m64 __m1, __m64 __m2) {`.
  **L1009 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psubsw(__m64 __m1, __m64 __m2) {`。
- **L1010 EN**: Returns from the current function with `_mm_subs_pi16(__m1, __m2)`.
  **L1010 CN**: 以 `_mm_subs_pi16(__m1, __m2)` 从当前函数返回。
- **L1011 EN**: Closes the current lexical scope or compound statement.
  **L1011 CN**: 结束当前词法作用域或复合语句块。
- **L1012 EN**: Blank line separating nearby declarations or logic blocks.
  **L1012 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1013 EN**: Comment explains nearby logic, constraints, or intent: `Subtract the 8-bit values in M2 from the 8-bit values in M1 using`.
  **L1013 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtract the 8-bit values in M2 from the 8-bit values in M1 using`。
- **L1014 EN**: Continues the surrounding expression or declaration: `unsigned saturating arithmetic.  */`.
  **L1014 CN**: 继续构造周围的表达式或声明：`unsigned saturating arithmetic.  */`。
- **L1015 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1015 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1016 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1016 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1017 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_subs_pu8(__m64 __m1, __m64 __m2) {`.
  **L1017 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_subs_pu8(__m64 __m1, __m64 __m2) {`。
- **L1018 EN**: Adds a standalone statement or declaration: `__vector unsigned char __a, __b, __c;`.
  **L1018 CN**: 添加一条独立语句或声明：`__vector unsigned char __a, __b, __c;`。
- **L1019 EN**: Blank line separating nearby declarations or logic blocks.
  **L1019 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1020 EN**: Executes a call or declaration centered on `=`.
  **L1020 CN**: 执行以 `=` 为核心的调用或声明。
- **L1021 EN**: Executes a call or declaration centered on `=`.
  **L1021 CN**: 执行以 `=` 为核心的调用或声明。
- **L1022 EN**: Executes a call or declaration centered on `vec_subs`.
  **L1022 CN**: 执行以 `vec_subs` 为核心的调用或声明。
- **L1023 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1023 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1024 EN**: Closes the current lexical scope or compound statement.
  **L1024 CN**: 结束当前词法作用域或复合语句块。
- **L1025 EN**: Blank line separating nearby declarations or logic blocks.
  **L1025 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1026 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1026 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1027 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1027 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1028 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psubusb(__m64 __m1, __m64 __m2) {`.
  **L1028 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psubusb(__m64 __m1, __m64 __m2) {`。
- **L1029 EN**: Returns from the current function with `_mm_subs_pu8(__m1, __m2)`.
  **L1029 CN**: 以 `_mm_subs_pu8(__m1, __m2)` 从当前函数返回。
- **L1030 EN**: Closes the current lexical scope or compound statement.
  **L1030 CN**: 结束当前词法作用域或复合语句块。
- **L1031 EN**: Blank line separating nearby declarations or logic blocks.
  **L1031 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1032 EN**: Comment explains nearby logic, constraints, or intent: `Subtract the 16-bit values in M2 from the 16-bit values in M1 using`.
  **L1032 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtract the 16-bit values in M2 from the 16-bit values in M1 using`。

### Lines 1033-1056

````c
   unsigned saturating arithmetic.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_subs_pu16(__m64 __m1, __m64 __m2) {
  __vector unsigned short __a, __b, __c;

  __a = (__vector unsigned short)vec_splats(__m1);
  __b = (__vector unsigned short)vec_splats(__m2);
  __c = vec_subs(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psubusw(__m64 __m1, __m64 __m2) {
  return _mm_subs_pu16(__m1, __m2);
}

/* Multiply four 16-bit values in M1 by four 16-bit values in M2 producing
   four 32-bit intermediate results, which are then summed by pairs to
   produce two 32-bit results.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_madd_pi16(__m64 __m1, __m64 __m2) {
````
- **L1033 EN**: Continues the surrounding expression or declaration: `unsigned saturating arithmetic.  */`.
  **L1033 CN**: 继续构造周围的表达式或声明：`unsigned saturating arithmetic.  */`。
- **L1034 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1034 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1035 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1035 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1036 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_subs_pu16(__m64 __m1, __m64 __m2) {`.
  **L1036 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_subs_pu16(__m64 __m1, __m64 __m2) {`。
- **L1037 EN**: Adds a standalone statement or declaration: `__vector unsigned short __a, __b, __c;`.
  **L1037 CN**: 添加一条独立语句或声明：`__vector unsigned short __a, __b, __c;`。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1039 EN**: Executes a call or declaration centered on `=`.
  **L1039 CN**: 执行以 `=` 为核心的调用或声明。
- **L1040 EN**: Executes a call or declaration centered on `=`.
  **L1040 CN**: 执行以 `=` 为核心的调用或声明。
- **L1041 EN**: Executes a call or declaration centered on `vec_subs`.
  **L1041 CN**: 执行以 `vec_subs` 为核心的调用或声明。
- **L1042 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1042 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1043 EN**: Closes the current lexical scope or compound statement.
  **L1043 CN**: 结束当前词法作用域或复合语句块。
- **L1044 EN**: Blank line separating nearby declarations or logic blocks.
  **L1044 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1045 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1045 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1046 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1046 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1047 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psubusw(__m64 __m1, __m64 __m2) {`.
  **L1047 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psubusw(__m64 __m1, __m64 __m2) {`。
- **L1048 EN**: Returns from the current function with `_mm_subs_pu16(__m1, __m2)`.
  **L1048 CN**: 以 `_mm_subs_pu16(__m1, __m2)` 从当前函数返回。
- **L1049 EN**: Closes the current lexical scope or compound statement.
  **L1049 CN**: 结束当前词法作用域或复合语句块。
- **L1050 EN**: Blank line separating nearby declarations or logic blocks.
  **L1050 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1051 EN**: Comment explains nearby logic, constraints, or intent: `Multiply four 16-bit values in M1 by four 16-bit values in M2 producing`.
  **L1051 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiply four 16-bit values in M1 by four 16-bit values in M2 producing`。
- **L1052 EN**: Continues the surrounding expression or declaration: `four 32-bit intermediate results, which are then summed by pairs to`.
  **L1052 CN**: 继续构造周围的表达式或声明：`four 32-bit intermediate results, which are then summed by pairs to`。
- **L1053 EN**: Continues the surrounding expression or declaration: `produce two 32-bit results.  */`.
  **L1053 CN**: 继续构造周围的表达式或声明：`produce two 32-bit results.  */`。
- **L1054 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1054 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1055 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1055 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1056 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_madd_pi16(__m64 __m1, __m64 __m2) {`.
  **L1056 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_madd_pi16(__m64 __m1, __m64 __m2) {`。

### Lines 1057-1080

````c
  __vector signed short __a, __b;
  __vector signed int __c;
  __vector signed int __zero = {0, 0, 0, 0};

  __a = (__vector signed short)vec_splats(__m1);
  __b = (__vector signed short)vec_splats(__m2);
  __c = vec_vmsumshm(__a, __b, __zero);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pmaddwd(__m64 __m1, __m64 __m2) {
  return _mm_madd_pi16(__m1, __m2);
}
/* Multiply four signed 16-bit values in M1 by four signed 16-bit values in
   M2 and produce the high 16 bits of the 32-bit results.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mulhi_pi16(__m64 __m1, __m64 __m2) {
  __vector signed short __a, __b;
  __vector signed short __c;
  __vector signed int __w0, __w1;
  __vector unsigned char __xform1 = {
````
- **L1057 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b;`.
  **L1057 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b;`。
- **L1058 EN**: Adds a standalone statement or declaration: `__vector signed int __c;`.
  **L1058 CN**: 添加一条独立语句或声明：`__vector signed int __c;`。
- **L1059 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L1059 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1061 EN**: Executes a call or declaration centered on `=`.
  **L1061 CN**: 执行以 `=` 为核心的调用或声明。
- **L1062 EN**: Executes a call or declaration centered on `=`.
  **L1062 CN**: 执行以 `=` 为核心的调用或声明。
- **L1063 EN**: Executes a call or declaration centered on `vec_vmsumshm`.
  **L1063 CN**: 执行以 `vec_vmsumshm` 为核心的调用或声明。
- **L1064 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1064 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1065 EN**: Closes the current lexical scope or compound statement.
  **L1065 CN**: 结束当前词法作用域或复合语句块。
- **L1066 EN**: Blank line separating nearby declarations or logic blocks.
  **L1066 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1067 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1067 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1068 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1068 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1069 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pmaddwd(__m64 __m1, __m64 __m2) {`.
  **L1069 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pmaddwd(__m64 __m1, __m64 __m2) {`。
- **L1070 EN**: Returns from the current function with `_mm_madd_pi16(__m1, __m2)`.
  **L1070 CN**: 以 `_mm_madd_pi16(__m1, __m2)` 从当前函数返回。
- **L1071 EN**: Closes the current lexical scope or compound statement.
  **L1071 CN**: 结束当前词法作用域或复合语句块。
- **L1072 EN**: Comment explains nearby logic, constraints, or intent: `Multiply four signed 16-bit values in M1 by four signed 16-bit values in`.
  **L1072 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiply four signed 16-bit values in M1 by four signed 16-bit values in`。
- **L1073 EN**: Continues the surrounding expression or declaration: `M2 and produce the high 16 bits of the 32-bit results.  */`.
  **L1073 CN**: 继续构造周围的表达式或声明：`M2 and produce the high 16 bits of the 32-bit results.  */`。
- **L1074 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1074 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1075 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1075 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1076 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mulhi_pi16(__m64 __m1, __m64 __m2) {`.
  **L1076 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mulhi_pi16(__m64 __m1, __m64 __m2) {`。
- **L1077 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b;`.
  **L1077 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b;`。
- **L1078 EN**: Adds a standalone statement or declaration: `__vector signed short __c;`.
  **L1078 CN**: 添加一条独立语句或声明：`__vector signed short __c;`。
- **L1079 EN**: Adds a standalone statement or declaration: `__vector signed int __w0, __w1;`.
  **L1079 CN**: 添加一条独立语句或声明：`__vector signed int __w0, __w1;`。
- **L1080 EN**: Continues the surrounding expression or declaration: `__vector unsigned char __xform1 = {`.
  **L1080 CN**: 继续构造周围的表达式或声明：`__vector unsigned char __xform1 = {`。

### Lines 1081-1104

````c
#ifdef __LITTLE_ENDIAN__
      0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,
      0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F
#else
      0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x00,
      0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15
#endif
  };

  __a = (__vector signed short)vec_splats(__m1);
  __b = (__vector signed short)vec_splats(__m2);

  __w0 = vec_vmulesh(__a, __b);
  __w1 = vec_vmulosh(__a, __b);
  __c = (__vector signed short)vec_perm(__w0, __w1, __xform1);

  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pmulhw(__m64 __m1, __m64 __m2) {
  return _mm_mulhi_pi16(__m1, __m2);
}
````
- **L1081 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1081 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,`.
  **L1082 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,`。
- **L1083 EN**: Continues the surrounding expression or declaration: `0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F`.
  **L1083 CN**: 继续构造周围的表达式或声明：`0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F`。
- **L1084 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1084 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x00,`.
  **L1085 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x00,`。
- **L1086 EN**: Continues the surrounding expression or declaration: `0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15`.
  **L1086 CN**: 继续构造周围的表达式或声明：`0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15`。
- **L1087 EN**: Closes the current preprocessor conditional block.
  **L1087 CN**: 结束当前预处理条件块。
- **L1088 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1088 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1089 EN**: Blank line separating nearby declarations or logic blocks.
  **L1089 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1090 EN**: Executes a call or declaration centered on `=`.
  **L1090 CN**: 执行以 `=` 为核心的调用或声明。
- **L1091 EN**: Executes a call or declaration centered on `=`.
  **L1091 CN**: 执行以 `=` 为核心的调用或声明。
- **L1092 EN**: Blank line separating nearby declarations or logic blocks.
  **L1092 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1093 EN**: Executes a call or declaration centered on `vec_vmulesh`.
  **L1093 CN**: 执行以 `vec_vmulesh` 为核心的调用或声明。
- **L1094 EN**: Executes a call or declaration centered on `vec_vmulosh`.
  **L1094 CN**: 执行以 `vec_vmulosh` 为核心的调用或声明。
- **L1095 EN**: Executes a call or declaration centered on `=`.
  **L1095 CN**: 执行以 `=` 为核心的调用或声明。
- **L1096 EN**: Blank line separating nearby declarations or logic blocks.
  **L1096 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1097 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1097 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1098 EN**: Closes the current lexical scope or compound statement.
  **L1098 CN**: 结束当前词法作用域或复合语句块。
- **L1099 EN**: Blank line separating nearby declarations or logic blocks.
  **L1099 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1100 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1100 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1101 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1101 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1102 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pmulhw(__m64 __m1, __m64 __m2) {`.
  **L1102 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pmulhw(__m64 __m1, __m64 __m2) {`。
- **L1103 EN**: Returns from the current function with `_mm_mulhi_pi16(__m1, __m2)`.
  **L1103 CN**: 以 `_mm_mulhi_pi16(__m1, __m2)` 从当前函数返回。
- **L1104 EN**: Closes the current lexical scope or compound statement.
  **L1104 CN**: 结束当前词法作用域或复合语句块。

### Lines 1105-1128

````c

/* Multiply four 16-bit values in M1 by four 16-bit values in M2 and produce
   the low 16 bits of the results.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mullo_pi16(__m64 __m1, __m64 __m2) {
  __vector signed short __a, __b, __c;

  __a = (__vector signed short)vec_splats(__m1);
  __b = (__vector signed short)vec_splats(__m2);
  __c = __a * __b;
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pmullw(__m64 __m1, __m64 __m2) {
  return _mm_mullo_pi16(__m1, __m2);
}

/* Shift four 16-bit values in M left by COUNT.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sll_pi16(__m64 __m, __m64 __count) {
````
- **L1105 EN**: Blank line separating nearby declarations or logic blocks.
  **L1105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1106 EN**: Comment explains nearby logic, constraints, or intent: `Multiply four 16-bit values in M1 by four 16-bit values in M2 and produce`.
  **L1106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiply four 16-bit values in M1 by four 16-bit values in M2 and produce`。
- **L1107 EN**: Continues the surrounding expression or declaration: `the low 16 bits of the results.  */`.
  **L1107 CN**: 继续构造周围的表达式或声明：`the low 16 bits of the results.  */`。
- **L1108 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1108 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1109 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1109 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1110 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mullo_pi16(__m64 __m1, __m64 __m2) {`.
  **L1110 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mullo_pi16(__m64 __m1, __m64 __m2) {`。
- **L1111 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __c;`.
  **L1111 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __c;`。
- **L1112 EN**: Blank line separating nearby declarations or logic blocks.
  **L1112 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1113 EN**: Executes a call or declaration centered on `=`.
  **L1113 CN**: 执行以 `=` 为核心的调用或声明。
- **L1114 EN**: Executes a call or declaration centered on `=`.
  **L1114 CN**: 执行以 `=` 为核心的调用或声明。
- **L1115 EN**: Adds a standalone statement or declaration: `__c = __a * __b;`.
  **L1115 CN**: 添加一条独立语句或声明：`__c = __a * __b;`。
- **L1116 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1116 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1117 EN**: Closes the current lexical scope or compound statement.
  **L1117 CN**: 结束当前词法作用域或复合语句块。
- **L1118 EN**: Blank line separating nearby declarations or logic blocks.
  **L1118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1119 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1119 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1120 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1120 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1121 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pmullw(__m64 __m1, __m64 __m2) {`.
  **L1121 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pmullw(__m64 __m1, __m64 __m2) {`。
- **L1122 EN**: Returns from the current function with `_mm_mullo_pi16(__m1, __m2)`.
  **L1122 CN**: 以 `_mm_mullo_pi16(__m1, __m2)` 从当前函数返回。
- **L1123 EN**: Closes the current lexical scope or compound statement.
  **L1123 CN**: 结束当前词法作用域或复合语句块。
- **L1124 EN**: Blank line separating nearby declarations or logic blocks.
  **L1124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1125 EN**: Comment explains nearby logic, constraints, or intent: `Shift four 16-bit values in M left by COUNT.`.
  **L1125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift four 16-bit values in M left by COUNT.`。
- **L1126 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1126 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1127 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1127 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1128 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sll_pi16(__m64 __m, __m64 __count) {`.
  **L1128 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sll_pi16(__m64 __m, __m64 __count) {`。

### Lines 1129-1152

````c
  __vector signed short __r;
  __vector unsigned short __c;

  if (__count <= 15) {
    __r = (__vector signed short)vec_splats(__m);
    __c = (__vector unsigned short)vec_splats((unsigned short)__count);
    __r = vec_sl(__r, (__vector unsigned short)__c);
    return (__m64)((__vector long long)__r)[0];
  } else
    return (0);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psllw(__m64 __m, __m64 __count) {
  return _mm_sll_pi16(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_slli_pi16(__m64 __m, int __count) {
  /* Promote int to long then invoke mm_sll_pi16.  */
  return _mm_sll_pi16(__m, __count);
}
````
- **L1129 EN**: Adds a standalone statement or declaration: `__vector signed short __r;`.
  **L1129 CN**: 添加一条独立语句或声明：`__vector signed short __r;`。
- **L1130 EN**: Adds a standalone statement or declaration: `__vector unsigned short __c;`.
  **L1130 CN**: 添加一条独立语句或声明：`__vector unsigned short __c;`。
- **L1131 EN**: Blank line separating nearby declarations or logic blocks.
  **L1131 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1132 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1132 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1133 EN**: Executes a call or declaration centered on `=`.
  **L1133 CN**: 执行以 `=` 为核心的调用或声明。
- **L1134 EN**: Executes a call or declaration centered on `=`.
  **L1134 CN**: 执行以 `=` 为核心的调用或声明。
- **L1135 EN**: Executes a call or declaration centered on `vec_sl`.
  **L1135 CN**: 执行以 `vec_sl` 为核心的调用或声明。
- **L1136 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L1136 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。
- **L1137 EN**: Continues the surrounding expression or declaration: `} else`.
  **L1137 CN**: 继续构造周围的表达式或声明：`} else`。
- **L1138 EN**: Returns from the current function with `(0)`.
  **L1138 CN**: 以 `(0)` 从当前函数返回。
- **L1139 EN**: Closes the current lexical scope or compound statement.
  **L1139 CN**: 结束当前词法作用域或复合语句块。
- **L1140 EN**: Blank line separating nearby declarations or logic blocks.
  **L1140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1141 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1141 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1142 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1142 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1143 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psllw(__m64 __m, __m64 __count) {`.
  **L1143 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psllw(__m64 __m, __m64 __count) {`。
- **L1144 EN**: Returns from the current function with `_mm_sll_pi16(__m, __count)`.
  **L1144 CN**: 以 `_mm_sll_pi16(__m, __count)` 从当前函数返回。
- **L1145 EN**: Closes the current lexical scope or compound statement.
  **L1145 CN**: 结束当前词法作用域或复合语句块。
- **L1146 EN**: Blank line separating nearby declarations or logic blocks.
  **L1146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1147 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1147 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1148 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1148 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_slli_pi16(__m64 __m, int __count) {`.
  **L1149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_slli_pi16(__m64 __m, int __count) {`。
- **L1150 EN**: Comment explains nearby logic, constraints, or intent: `Promote int to long then invoke mm_sll_pi16.`.
  **L1150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Promote int to long then invoke mm_sll_pi16.`。
- **L1151 EN**: Returns from the current function with `_mm_sll_pi16(__m, __count)`.
  **L1151 CN**: 以 `_mm_sll_pi16(__m, __count)` 从当前函数返回。
- **L1152 EN**: Closes the current lexical scope or compound statement.
  **L1152 CN**: 结束当前词法作用域或复合语句块。

### Lines 1153-1176

````c

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psllwi(__m64 __m, int __count) {
  return _mm_slli_pi16(__m, __count);
}

/* Shift two 32-bit values in M left by COUNT.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sll_pi32(__m64 __m, __m64 __count) {
  __m64_union __res;

  __res.as_m64 = __m;

  __res.as_int[0] = __res.as_int[0] << __count;
  __res.as_int[1] = __res.as_int[1] << __count;
  return (__res.as_m64);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pslld(__m64 __m, __m64 __count) {
  return _mm_sll_pi32(__m, __count);
````
- **L1153 EN**: Blank line separating nearby declarations or logic blocks.
  **L1153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1154 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1154 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1155 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1155 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psllwi(__m64 __m, int __count) {`.
  **L1156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psllwi(__m64 __m, int __count) {`。
- **L1157 EN**: Returns from the current function with `_mm_slli_pi16(__m, __count)`.
  **L1157 CN**: 以 `_mm_slli_pi16(__m, __count)` 从当前函数返回。
- **L1158 EN**: Closes the current lexical scope or compound statement.
  **L1158 CN**: 结束当前词法作用域或复合语句块。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1160 EN**: Comment explains nearby logic, constraints, or intent: `Shift two 32-bit values in M left by COUNT.`.
  **L1160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift two 32-bit values in M left by COUNT.`。
- **L1161 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1161 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1162 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1162 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sll_pi32(__m64 __m, __m64 __count) {`.
  **L1163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sll_pi32(__m64 __m, __m64 __count) {`。
- **L1164 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1164 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1166 EN**: Adds a standalone statement or declaration: `__res.as_m64 = __m;`.
  **L1166 CN**: 添加一条独立语句或声明：`__res.as_m64 = __m;`。
- **L1167 EN**: Blank line separating nearby declarations or logic blocks.
  **L1167 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1168 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __res.as_int[0] << __count;`.
  **L1168 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __res.as_int[0] << __count;`。
- **L1169 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __res.as_int[1] << __count;`.
  **L1169 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __res.as_int[1] << __count;`。
- **L1170 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1170 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1171 EN**: Closes the current lexical scope or compound statement.
  **L1171 CN**: 结束当前词法作用域或复合语句块。
- **L1172 EN**: Blank line separating nearby declarations or logic blocks.
  **L1172 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1173 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1173 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1174 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1174 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1175 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pslld(__m64 __m, __m64 __count) {`.
  **L1175 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pslld(__m64 __m, __m64 __count) {`。
- **L1176 EN**: Returns from the current function with `_mm_sll_pi32(__m, __count)`.
  **L1176 CN**: 以 `_mm_sll_pi32(__m, __count)` 从当前函数返回。

### Lines 1177-1200

````c
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_slli_pi32(__m64 __m, int __count) {
  /* Promote int to long then invoke mm_sll_pi32.  */
  return _mm_sll_pi32(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pslldi(__m64 __m, int __count) {
  return _mm_slli_pi32(__m, __count);
}

/* Shift four 16-bit values in M right by COUNT; shift in the sign bit.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sra_pi16(__m64 __m, __m64 __count) {
  __vector signed short __r;
  __vector unsigned short __c;

  if (__count <= 15) {
    __r = (__vector signed short)vec_splats(__m);
````
- **L1177 EN**: Closes the current lexical scope or compound statement.
  **L1177 CN**: 结束当前词法作用域或复合语句块。
- **L1178 EN**: Blank line separating nearby declarations or logic blocks.
  **L1178 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1179 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1179 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1180 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1180 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1181 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_slli_pi32(__m64 __m, int __count) {`.
  **L1181 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_slli_pi32(__m64 __m, int __count) {`。
- **L1182 EN**: Comment explains nearby logic, constraints, or intent: `Promote int to long then invoke mm_sll_pi32.`.
  **L1182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Promote int to long then invoke mm_sll_pi32.`。
- **L1183 EN**: Returns from the current function with `_mm_sll_pi32(__m, __count)`.
  **L1183 CN**: 以 `_mm_sll_pi32(__m, __count)` 从当前函数返回。
- **L1184 EN**: Closes the current lexical scope or compound statement.
  **L1184 CN**: 结束当前词法作用域或复合语句块。
- **L1185 EN**: Blank line separating nearby declarations or logic blocks.
  **L1185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1186 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1186 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1187 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1187 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1188 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pslldi(__m64 __m, int __count) {`.
  **L1188 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pslldi(__m64 __m, int __count) {`。
- **L1189 EN**: Returns from the current function with `_mm_slli_pi32(__m, __count)`.
  **L1189 CN**: 以 `_mm_slli_pi32(__m, __count)` 从当前函数返回。
- **L1190 EN**: Closes the current lexical scope or compound statement.
  **L1190 CN**: 结束当前词法作用域或复合语句块。
- **L1191 EN**: Blank line separating nearby declarations or logic blocks.
  **L1191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1192 EN**: Comment explains nearby logic, constraints, or intent: `Shift four 16-bit values in M right by COUNT; shift in the sign bit.`.
  **L1192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift four 16-bit values in M right by COUNT; shift in the sign bit.`。
- **L1193 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1193 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1194 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1194 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1195 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sra_pi16(__m64 __m, __m64 __count) {`.
  **L1195 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sra_pi16(__m64 __m, __m64 __count) {`。
- **L1196 EN**: Adds a standalone statement or declaration: `__vector signed short __r;`.
  **L1196 CN**: 添加一条独立语句或声明：`__vector signed short __r;`。
- **L1197 EN**: Adds a standalone statement or declaration: `__vector unsigned short __c;`.
  **L1197 CN**: 添加一条独立语句或声明：`__vector unsigned short __c;`。
- **L1198 EN**: Blank line separating nearby declarations or logic blocks.
  **L1198 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1199 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1199 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1200 EN**: Executes a call or declaration centered on `=`.
  **L1200 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 1201-1224

````c
    __c = (__vector unsigned short)vec_splats((unsigned short)__count);
    __r = vec_sra(__r, (__vector unsigned short)__c);
    return (__m64)((__vector long long)__r)[0];
  } else
    return (0);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psraw(__m64 __m, __m64 __count) {
  return _mm_sra_pi16(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srai_pi16(__m64 __m, int __count) {
  /* Promote int to long then invoke mm_sra_pi32.  */
  return _mm_sra_pi16(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psrawi(__m64 __m, int __count) {
  return _mm_srai_pi16(__m, __count);
````
- **L1201 EN**: Executes a call or declaration centered on `=`.
  **L1201 CN**: 执行以 `=` 为核心的调用或声明。
- **L1202 EN**: Executes a call or declaration centered on `vec_sra`.
  **L1202 CN**: 执行以 `vec_sra` 为核心的调用或声明。
- **L1203 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L1203 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。
- **L1204 EN**: Continues the surrounding expression or declaration: `} else`.
  **L1204 CN**: 继续构造周围的表达式或声明：`} else`。
- **L1205 EN**: Returns from the current function with `(0)`.
  **L1205 CN**: 以 `(0)` 从当前函数返回。
- **L1206 EN**: Closes the current lexical scope or compound statement.
  **L1206 CN**: 结束当前词法作用域或复合语句块。
- **L1207 EN**: Blank line separating nearby declarations or logic blocks.
  **L1207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1208 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1208 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1209 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1209 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1210 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psraw(__m64 __m, __m64 __count) {`.
  **L1210 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psraw(__m64 __m, __m64 __count) {`。
- **L1211 EN**: Returns from the current function with `_mm_sra_pi16(__m, __count)`.
  **L1211 CN**: 以 `_mm_sra_pi16(__m, __count)` 从当前函数返回。
- **L1212 EN**: Closes the current lexical scope or compound statement.
  **L1212 CN**: 结束当前词法作用域或复合语句块。
- **L1213 EN**: Blank line separating nearby declarations or logic blocks.
  **L1213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1214 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1214 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1215 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1215 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1216 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srai_pi16(__m64 __m, int __count) {`.
  **L1216 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srai_pi16(__m64 __m, int __count) {`。
- **L1217 EN**: Comment explains nearby logic, constraints, or intent: `Promote int to long then invoke mm_sra_pi32.`.
  **L1217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Promote int to long then invoke mm_sra_pi32.`。
- **L1218 EN**: Returns from the current function with `_mm_sra_pi16(__m, __count)`.
  **L1218 CN**: 以 `_mm_sra_pi16(__m, __count)` 从当前函数返回。
- **L1219 EN**: Closes the current lexical scope or compound statement.
  **L1219 CN**: 结束当前词法作用域或复合语句块。
- **L1220 EN**: Blank line separating nearby declarations or logic blocks.
  **L1220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1221 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1221 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1222 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1222 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1223 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psrawi(__m64 __m, int __count) {`.
  **L1223 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psrawi(__m64 __m, int __count) {`。
- **L1224 EN**: Returns from the current function with `_mm_srai_pi16(__m, __count)`.
  **L1224 CN**: 以 `_mm_srai_pi16(__m, __count)` 从当前函数返回。

### Lines 1225-1248

````c
}

/* Shift two 32-bit values in M right by COUNT; shift in the sign bit.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sra_pi32(__m64 __m, __m64 __count) {
  __m64_union __res;

  __res.as_m64 = __m;

  __res.as_int[0] = __res.as_int[0] >> __count;
  __res.as_int[1] = __res.as_int[1] >> __count;
  return (__res.as_m64);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psrad(__m64 __m, __m64 __count) {
  return _mm_sra_pi32(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srai_pi32(__m64 __m, int __count) {
````
- **L1225 EN**: Closes the current lexical scope or compound statement.
  **L1225 CN**: 结束当前词法作用域或复合语句块。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1227 EN**: Comment explains nearby logic, constraints, or intent: `Shift two 32-bit values in M right by COUNT; shift in the sign bit.`.
  **L1227 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift two 32-bit values in M right by COUNT; shift in the sign bit.`。
- **L1228 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1228 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1229 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1229 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1230 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sra_pi32(__m64 __m, __m64 __count) {`.
  **L1230 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sra_pi32(__m64 __m, __m64 __count) {`。
- **L1231 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1231 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1232 EN**: Blank line separating nearby declarations or logic blocks.
  **L1232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1233 EN**: Adds a standalone statement or declaration: `__res.as_m64 = __m;`.
  **L1233 CN**: 添加一条独立语句或声明：`__res.as_m64 = __m;`。
- **L1234 EN**: Blank line separating nearby declarations or logic blocks.
  **L1234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1235 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __res.as_int[0] >> __count;`.
  **L1235 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __res.as_int[0] >> __count;`。
- **L1236 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __res.as_int[1] >> __count;`.
  **L1236 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __res.as_int[1] >> __count;`。
- **L1237 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1237 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1238 EN**: Closes the current lexical scope or compound statement.
  **L1238 CN**: 结束当前词法作用域或复合语句块。
- **L1239 EN**: Blank line separating nearby declarations or logic blocks.
  **L1239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1240 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1240 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1241 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1241 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1242 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psrad(__m64 __m, __m64 __count) {`.
  **L1242 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psrad(__m64 __m, __m64 __count) {`。
- **L1243 EN**: Returns from the current function with `_mm_sra_pi32(__m, __count)`.
  **L1243 CN**: 以 `_mm_sra_pi32(__m, __count)` 从当前函数返回。
- **L1244 EN**: Closes the current lexical scope or compound statement.
  **L1244 CN**: 结束当前词法作用域或复合语句块。
- **L1245 EN**: Blank line separating nearby declarations or logic blocks.
  **L1245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1246 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1246 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1247 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1247 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1248 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srai_pi32(__m64 __m, int __count) {`.
  **L1248 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srai_pi32(__m64 __m, int __count) {`。

### Lines 1249-1272

````c
  /* Promote int to long then invoke mm_sra_pi32.  */
  return _mm_sra_pi32(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psradi(__m64 __m, int __count) {
  return _mm_srai_pi32(__m, __count);
}

/* Shift four 16-bit values in M right by COUNT; shift in zeros.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srl_pi16(__m64 __m, __m64 __count) {
  __vector unsigned short __r;
  __vector unsigned short __c;

  if (__count <= 15) {
    __r = (__vector unsigned short)vec_splats(__m);
    __c = (__vector unsigned short)vec_splats((unsigned short)__count);
    __r = vec_sr(__r, (__vector unsigned short)__c);
    return (__m64)((__vector long long)__r)[0];
  } else
    return (0);
````
- **L1249 EN**: Comment explains nearby logic, constraints, or intent: `Promote int to long then invoke mm_sra_pi32.`.
  **L1249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Promote int to long then invoke mm_sra_pi32.`。
- **L1250 EN**: Returns from the current function with `_mm_sra_pi32(__m, __count)`.
  **L1250 CN**: 以 `_mm_sra_pi32(__m, __count)` 从当前函数返回。
- **L1251 EN**: Closes the current lexical scope or compound statement.
  **L1251 CN**: 结束当前词法作用域或复合语句块。
- **L1252 EN**: Blank line separating nearby declarations or logic blocks.
  **L1252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1253 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1253 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1254 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1254 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1255 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psradi(__m64 __m, int __count) {`.
  **L1255 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psradi(__m64 __m, int __count) {`。
- **L1256 EN**: Returns from the current function with `_mm_srai_pi32(__m, __count)`.
  **L1256 CN**: 以 `_mm_srai_pi32(__m, __count)` 从当前函数返回。
- **L1257 EN**: Closes the current lexical scope or compound statement.
  **L1257 CN**: 结束当前词法作用域或复合语句块。
- **L1258 EN**: Blank line separating nearby declarations or logic blocks.
  **L1258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1259 EN**: Comment explains nearby logic, constraints, or intent: `Shift four 16-bit values in M right by COUNT; shift in zeros.`.
  **L1259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift four 16-bit values in M right by COUNT; shift in zeros.`。
- **L1260 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1260 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1261 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1261 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1262 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srl_pi16(__m64 __m, __m64 __count) {`.
  **L1262 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srl_pi16(__m64 __m, __m64 __count) {`。
- **L1263 EN**: Adds a standalone statement or declaration: `__vector unsigned short __r;`.
  **L1263 CN**: 添加一条独立语句或声明：`__vector unsigned short __r;`。
- **L1264 EN**: Adds a standalone statement or declaration: `__vector unsigned short __c;`.
  **L1264 CN**: 添加一条独立语句或声明：`__vector unsigned short __c;`。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1266 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1267 EN**: Executes a call or declaration centered on `=`.
  **L1267 CN**: 执行以 `=` 为核心的调用或声明。
- **L1268 EN**: Executes a call or declaration centered on `=`.
  **L1268 CN**: 执行以 `=` 为核心的调用或声明。
- **L1269 EN**: Executes a call or declaration centered on `vec_sr`.
  **L1269 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L1270 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L1270 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。
- **L1271 EN**: Continues the surrounding expression or declaration: `} else`.
  **L1271 CN**: 继续构造周围的表达式或声明：`} else`。
- **L1272 EN**: Returns from the current function with `(0)`.
  **L1272 CN**: 以 `(0)` 从当前函数返回。

### Lines 1273-1296

````c
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psrlw(__m64 __m, __m64 __count) {
  return _mm_srl_pi16(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srli_pi16(__m64 __m, int __count) {
  /* Promote int to long then invoke mm_sra_pi32.  */
  return _mm_srl_pi16(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psrlwi(__m64 __m, int __count) {
  return _mm_srli_pi16(__m, __count);
}

/* Shift two 32-bit values in M right by COUNT; shift in zeros.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L1273 EN**: Closes the current lexical scope or compound statement.
  **L1273 CN**: 结束当前词法作用域或复合语句块。
- **L1274 EN**: Blank line separating nearby declarations or logic blocks.
  **L1274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1275 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1275 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1276 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1276 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1277 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psrlw(__m64 __m, __m64 __count) {`.
  **L1277 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psrlw(__m64 __m, __m64 __count) {`。
- **L1278 EN**: Returns from the current function with `_mm_srl_pi16(__m, __count)`.
  **L1278 CN**: 以 `_mm_srl_pi16(__m, __count)` 从当前函数返回。
- **L1279 EN**: Closes the current lexical scope or compound statement.
  **L1279 CN**: 结束当前词法作用域或复合语句块。
- **L1280 EN**: Blank line separating nearby declarations or logic blocks.
  **L1280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1281 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1281 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1282 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1282 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1283 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srli_pi16(__m64 __m, int __count) {`.
  **L1283 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srli_pi16(__m64 __m, int __count) {`。
- **L1284 EN**: Comment explains nearby logic, constraints, or intent: `Promote int to long then invoke mm_sra_pi32.`.
  **L1284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Promote int to long then invoke mm_sra_pi32.`。
- **L1285 EN**: Returns from the current function with `_mm_srl_pi16(__m, __count)`.
  **L1285 CN**: 以 `_mm_srl_pi16(__m, __count)` 从当前函数返回。
- **L1286 EN**: Closes the current lexical scope or compound statement.
  **L1286 CN**: 结束当前词法作用域或复合语句块。
- **L1287 EN**: Blank line separating nearby declarations or logic blocks.
  **L1287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1288 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1288 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1289 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1289 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1290 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psrlwi(__m64 __m, int __count) {`.
  **L1290 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psrlwi(__m64 __m, int __count) {`。
- **L1291 EN**: Returns from the current function with `_mm_srli_pi16(__m, __count)`.
  **L1291 CN**: 以 `_mm_srli_pi16(__m, __count)` 从当前函数返回。
- **L1292 EN**: Closes the current lexical scope or compound statement.
  **L1292 CN**: 结束当前词法作用域或复合语句块。
- **L1293 EN**: Blank line separating nearby declarations or logic blocks.
  **L1293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1294 EN**: Comment explains nearby logic, constraints, or intent: `Shift two 32-bit values in M right by COUNT; shift in zeros.`.
  **L1294 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift two 32-bit values in M right by COUNT; shift in zeros.`。
- **L1295 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1295 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1296 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1296 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 1297-1320

````c
    _mm_srl_pi32(__m64 __m, __m64 __count) {
  __m64_union __res;

  __res.as_m64 = __m;

  __res.as_int[0] = (unsigned int)__res.as_int[0] >> __count;
  __res.as_int[1] = (unsigned int)__res.as_int[1] >> __count;
  return (__res.as_m64);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psrld(__m64 __m, __m64 __count) {
  return _mm_srl_pi32(__m, __count);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srli_pi32(__m64 __m, int __count) {
  /* Promote int to long then invoke mm_srl_pi32.  */
  return _mm_srl_pi32(__m, __count);
}

extern __inline __m64
````
- **L1297 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srl_pi32(__m64 __m, __m64 __count) {`.
  **L1297 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srl_pi32(__m64 __m, __m64 __count) {`。
- **L1298 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1298 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1299 EN**: Blank line separating nearby declarations or logic blocks.
  **L1299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1300 EN**: Adds a standalone statement or declaration: `__res.as_m64 = __m;`.
  **L1300 CN**: 添加一条独立语句或声明：`__res.as_m64 = __m;`。
- **L1301 EN**: Blank line separating nearby declarations or logic blocks.
  **L1301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1302 EN**: Executes a call or declaration centered on `=`.
  **L1302 CN**: 执行以 `=` 为核心的调用或声明。
- **L1303 EN**: Executes a call or declaration centered on `=`.
  **L1303 CN**: 执行以 `=` 为核心的调用或声明。
- **L1304 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1304 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1305 EN**: Closes the current lexical scope or compound statement.
  **L1305 CN**: 结束当前词法作用域或复合语句块。
- **L1306 EN**: Blank line separating nearby declarations or logic blocks.
  **L1306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1307 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1307 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1308 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1308 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1309 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psrld(__m64 __m, __m64 __count) {`.
  **L1309 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psrld(__m64 __m, __m64 __count) {`。
- **L1310 EN**: Returns from the current function with `_mm_srl_pi32(__m, __count)`.
  **L1310 CN**: 以 `_mm_srl_pi32(__m, __count)` 从当前函数返回。
- **L1311 EN**: Closes the current lexical scope or compound statement.
  **L1311 CN**: 结束当前词法作用域或复合语句块。
- **L1312 EN**: Blank line separating nearby declarations or logic blocks.
  **L1312 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1313 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1313 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1314 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1314 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1315 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srli_pi32(__m64 __m, int __count) {`.
  **L1315 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srli_pi32(__m64 __m, int __count) {`。
- **L1316 EN**: Comment explains nearby logic, constraints, or intent: `Promote int to long then invoke mm_srl_pi32.`.
  **L1316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Promote int to long then invoke mm_srl_pi32.`。
- **L1317 EN**: Returns from the current function with `_mm_srl_pi32(__m, __count)`.
  **L1317 CN**: 以 `_mm_srl_pi32(__m, __count)` 从当前函数返回。
- **L1318 EN**: Closes the current lexical scope or compound statement.
  **L1318 CN**: 结束当前词法作用域或复合语句块。
- **L1319 EN**: Blank line separating nearby declarations or logic blocks.
  **L1319 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1320 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1320 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。

### Lines 1321-1344

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psrldi(__m64 __m, int __count) {
  return _mm_srli_pi32(__m, __count);
}
#endif /* _ARCH_PWR8 */

/* Creates a vector of two 32-bit values; I0 is least significant.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_pi32(int __i1, int __i0) {
  __m64_union __res;

  __res.as_int[0] = __i0;
  __res.as_int[1] = __i1;
  return (__res.as_m64);
}

/* Creates a vector of four 16-bit values; W0 is least significant.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_pi16(short __w3, short __w2, short __w1, short __w0) {
  __m64_union __res;

  __res.as_short[0] = __w0;
````
- **L1321 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1321 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1322 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psrldi(__m64 __m, int __count) {`.
  **L1322 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psrldi(__m64 __m, int __count) {`。
- **L1323 EN**: Returns from the current function with `_mm_srli_pi32(__m, __count)`.
  **L1323 CN**: 以 `_mm_srli_pi32(__m, __count)` 从当前函数返回。
- **L1324 EN**: Closes the current lexical scope or compound statement.
  **L1324 CN**: 结束当前词法作用域或复合语句块。
- **L1325 EN**: Closes the current preprocessor conditional block.
  **L1325 CN**: 结束当前预处理条件块。
- **L1326 EN**: Blank line separating nearby declarations or logic blocks.
  **L1326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1327 EN**: Comment explains nearby logic, constraints, or intent: `Creates a vector of two 32-bit values; I0 is least significant.`.
  **L1327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a vector of two 32-bit values; I0 is least significant.`。
- **L1328 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1328 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1329 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1329 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1330 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_pi32(int __i1, int __i0) {`.
  **L1330 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_pi32(int __i1, int __i0) {`。
- **L1331 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1331 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1332 EN**: Blank line separating nearby declarations or logic blocks.
  **L1332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1333 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __i0;`.
  **L1333 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __i0;`。
- **L1334 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __i1;`.
  **L1334 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __i1;`。
- **L1335 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1335 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1336 EN**: Closes the current lexical scope or compound statement.
  **L1336 CN**: 结束当前词法作用域或复合语句块。
- **L1337 EN**: Blank line separating nearby declarations or logic blocks.
  **L1337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1338 EN**: Comment explains nearby logic, constraints, or intent: `Creates a vector of four 16-bit values; W0 is least significant.`.
  **L1338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a vector of four 16-bit values; W0 is least significant.`。
- **L1339 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1339 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1340 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1340 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1341 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_pi16(short __w3, short __w2, short __w1, short __w0) {`.
  **L1341 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_pi16(short __w3, short __w2, short __w1, short __w0) {`。
- **L1342 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1342 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1343 EN**: Blank line separating nearby declarations or logic blocks.
  **L1343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1344 EN**: Adds a standalone statement or declaration: `__res.as_short[0] = __w0;`.
  **L1344 CN**: 添加一条独立语句或声明：`__res.as_short[0] = __w0;`。

### Lines 1345-1368

````c
  __res.as_short[1] = __w1;
  __res.as_short[2] = __w2;
  __res.as_short[3] = __w3;
  return (__res.as_m64);
}

/* Creates a vector of eight 8-bit values; B0 is least significant.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_pi8(char __b7, char __b6, char __b5, char __b4, char __b3,
                char __b2, char __b1, char __b0) {
  __m64_union __res;

  __res.as_char[0] = __b0;
  __res.as_char[1] = __b1;
  __res.as_char[2] = __b2;
  __res.as_char[3] = __b3;
  __res.as_char[4] = __b4;
  __res.as_char[5] = __b5;
  __res.as_char[6] = __b6;
  __res.as_char[7] = __b7;
  return (__res.as_m64);
}

````
- **L1345 EN**: Adds a standalone statement or declaration: `__res.as_short[1] = __w1;`.
  **L1345 CN**: 添加一条独立语句或声明：`__res.as_short[1] = __w1;`。
- **L1346 EN**: Adds a standalone statement or declaration: `__res.as_short[2] = __w2;`.
  **L1346 CN**: 添加一条独立语句或声明：`__res.as_short[2] = __w2;`。
- **L1347 EN**: Adds a standalone statement or declaration: `__res.as_short[3] = __w3;`.
  **L1347 CN**: 添加一条独立语句或声明：`__res.as_short[3] = __w3;`。
- **L1348 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1348 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1349 EN**: Closes the current lexical scope or compound statement.
  **L1349 CN**: 结束当前词法作用域或复合语句块。
- **L1350 EN**: Blank line separating nearby declarations or logic blocks.
  **L1350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1351 EN**: Comment explains nearby logic, constraints, or intent: `Creates a vector of eight 8-bit values; B0 is least significant.`.
  **L1351 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a vector of eight 8-bit values; B0 is least significant.`。
- **L1352 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1352 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1353 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1353 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_set_pi8(char __b7, char __b6, char __b5, char __b4, char __b3,`.
  **L1354 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_set_pi8(char __b7, char __b6, char __b5, char __b4, char __b3,`。
- **L1355 EN**: Continues the surrounding expression or declaration: `char __b2, char __b1, char __b0) {`.
  **L1355 CN**: 继续构造周围的表达式或声明：`char __b2, char __b1, char __b0) {`。
- **L1356 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1356 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1357 EN**: Blank line separating nearby declarations or logic blocks.
  **L1357 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1358 EN**: Adds a standalone statement or declaration: `__res.as_char[0] = __b0;`.
  **L1358 CN**: 添加一条独立语句或声明：`__res.as_char[0] = __b0;`。
- **L1359 EN**: Adds a standalone statement or declaration: `__res.as_char[1] = __b1;`.
  **L1359 CN**: 添加一条独立语句或声明：`__res.as_char[1] = __b1;`。
- **L1360 EN**: Adds a standalone statement or declaration: `__res.as_char[2] = __b2;`.
  **L1360 CN**: 添加一条独立语句或声明：`__res.as_char[2] = __b2;`。
- **L1361 EN**: Adds a standalone statement or declaration: `__res.as_char[3] = __b3;`.
  **L1361 CN**: 添加一条独立语句或声明：`__res.as_char[3] = __b3;`。
- **L1362 EN**: Adds a standalone statement or declaration: `__res.as_char[4] = __b4;`.
  **L1362 CN**: 添加一条独立语句或声明：`__res.as_char[4] = __b4;`。
- **L1363 EN**: Adds a standalone statement or declaration: `__res.as_char[5] = __b5;`.
  **L1363 CN**: 添加一条独立语句或声明：`__res.as_char[5] = __b5;`。
- **L1364 EN**: Adds a standalone statement or declaration: `__res.as_char[6] = __b6;`.
  **L1364 CN**: 添加一条独立语句或声明：`__res.as_char[6] = __b6;`。
- **L1365 EN**: Adds a standalone statement or declaration: `__res.as_char[7] = __b7;`.
  **L1365 CN**: 添加一条独立语句或声明：`__res.as_char[7] = __b7;`。
- **L1366 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1366 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1367 EN**: Closes the current lexical scope or compound statement.
  **L1367 CN**: 结束当前词法作用域或复合语句块。
- **L1368 EN**: Blank line separating nearby declarations or logic blocks.
  **L1368 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1369-1392

````c
/* Similar, but with the arguments in reverse order.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_pi32(int __i0, int __i1) {
  __m64_union __res;

  __res.as_int[0] = __i0;
  __res.as_int[1] = __i1;
  return (__res.as_m64);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_pi16(short __w0, short __w1, short __w2, short __w3) {
  return _mm_set_pi16(__w3, __w2, __w1, __w0);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_pi8(char __b0, char __b1, char __b2, char __b3, char __b4,
                 char __b5, char __b6, char __b7) {
  return _mm_set_pi8(__b7, __b6, __b5, __b4, __b3, __b2, __b1, __b0);
}

````
- **L1369 EN**: Comment explains nearby logic, constraints, or intent: `Similar, but with the arguments in reverse order.`.
  **L1369 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Similar, but with the arguments in reverse order.`。
- **L1370 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1370 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1371 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1371 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1372 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setr_pi32(int __i0, int __i1) {`.
  **L1372 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setr_pi32(int __i0, int __i1) {`。
- **L1373 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1373 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1374 EN**: Blank line separating nearby declarations or logic blocks.
  **L1374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1375 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __i0;`.
  **L1375 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __i0;`。
- **L1376 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __i1;`.
  **L1376 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __i1;`。
- **L1377 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1377 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1378 EN**: Closes the current lexical scope or compound statement.
  **L1378 CN**: 结束当前词法作用域或复合语句块。
- **L1379 EN**: Blank line separating nearby declarations or logic blocks.
  **L1379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1380 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1380 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1381 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1381 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1382 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setr_pi16(short __w0, short __w1, short __w2, short __w3) {`.
  **L1382 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setr_pi16(short __w0, short __w1, short __w2, short __w3) {`。
- **L1383 EN**: Returns from the current function with `_mm_set_pi16(__w3, __w2, __w1, __w0)`.
  **L1383 CN**: 以 `_mm_set_pi16(__w3, __w2, __w1, __w0)` 从当前函数返回。
- **L1384 EN**: Closes the current lexical scope or compound statement.
  **L1384 CN**: 结束当前词法作用域或复合语句块。
- **L1385 EN**: Blank line separating nearby declarations or logic blocks.
  **L1385 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1386 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1386 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1387 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1387 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_setr_pi8(char __b0, char __b1, char __b2, char __b3, char __b4,`.
  **L1388 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_setr_pi8(char __b0, char __b1, char __b2, char __b3, char __b4,`。
- **L1389 EN**: Continues the surrounding expression or declaration: `char __b5, char __b6, char __b7) {`.
  **L1389 CN**: 继续构造周围的表达式或声明：`char __b5, char __b6, char __b7) {`。
- **L1390 EN**: Returns from the current function with `_mm_set_pi8(__b7, __b6, __b5, __b4, __b3, __b2, __b1, __b0)`.
  **L1390 CN**: 以 `_mm_set_pi8(__b7, __b6, __b5, __b4, __b3, __b2, __b1, __b0)` 从当前函数返回。
- **L1391 EN**: Closes the current lexical scope or compound statement.
  **L1391 CN**: 结束当前词法作用域或复合语句块。
- **L1392 EN**: Blank line separating nearby declarations or logic blocks.
  **L1392 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1393-1416

````c
/* Creates a vector of two 32-bit values, both elements containing I.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_pi32(int __i) {
  __m64_union __res;

  __res.as_int[0] = __i;
  __res.as_int[1] = __i;
  return (__res.as_m64);
}

/* Creates a vector of four 16-bit values, all elements containing W.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_pi16(short __w) {
#if _ARCH_PWR9
  __vector signed short w;

  w = (__vector signed short)vec_splats(__w);
  return (__m64)((__vector long long)w)[0];
#else
  __m64_union __res;

  __res.as_short[0] = __w;
````
- **L1393 EN**: Comment explains nearby logic, constraints, or intent: `Creates a vector of two 32-bit values, both elements containing I.`.
  **L1393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a vector of two 32-bit values, both elements containing I.`。
- **L1394 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1394 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1395 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1395 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1396 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_pi32(int __i) {`.
  **L1396 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_pi32(int __i) {`。
- **L1397 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1397 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1398 EN**: Blank line separating nearby declarations or logic blocks.
  **L1398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1399 EN**: Adds a standalone statement or declaration: `__res.as_int[0] = __i;`.
  **L1399 CN**: 添加一条独立语句或声明：`__res.as_int[0] = __i;`。
- **L1400 EN**: Adds a standalone statement or declaration: `__res.as_int[1] = __i;`.
  **L1400 CN**: 添加一条独立语句或声明：`__res.as_int[1] = __i;`。
- **L1401 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1401 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1402 EN**: Closes the current lexical scope or compound statement.
  **L1402 CN**: 结束当前词法作用域或复合语句块。
- **L1403 EN**: Blank line separating nearby declarations or logic blocks.
  **L1403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1404 EN**: Comment explains nearby logic, constraints, or intent: `Creates a vector of four 16-bit values, all elements containing W.`.
  **L1404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a vector of four 16-bit values, all elements containing W.`。
- **L1405 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1405 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1406 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1406 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1407 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_pi16(short __w) {`.
  **L1407 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_pi16(short __w) {`。
- **L1408 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR9`.
  **L1408 CN**: 开始一个预处理条件块：`#if _ARCH_PWR9`。
- **L1409 EN**: Adds a standalone statement or declaration: `__vector signed short w;`.
  **L1409 CN**: 添加一条独立语句或声明：`__vector signed short w;`。
- **L1410 EN**: Blank line separating nearby declarations or logic blocks.
  **L1410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1411 EN**: Executes a call or declaration centered on `=`.
  **L1411 CN**: 执行以 `=` 为核心的调用或声明。
- **L1412 EN**: Returns from the current function with `(__m64)((__vector long long)w)[0]`.
  **L1412 CN**: 以 `(__m64)((__vector long long)w)[0]` 从当前函数返回。
- **L1413 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1413 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1414 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1414 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1415 EN**: Blank line separating nearby declarations or logic blocks.
  **L1415 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1416 EN**: Adds a standalone statement or declaration: `__res.as_short[0] = __w;`.
  **L1416 CN**: 添加一条独立语句或声明：`__res.as_short[0] = __w;`。

### Lines 1417-1440

````c
  __res.as_short[1] = __w;
  __res.as_short[2] = __w;
  __res.as_short[3] = __w;
  return (__res.as_m64);
#endif
}

/* Creates a vector of eight 8-bit values, all elements containing B.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_pi8(signed char __b) {
#if _ARCH_PWR8
  __vector signed char __res;

  __res = (__vector signed char)vec_splats(__b);
  return (__m64)((__vector long long)__res)[0];
#else
  __m64_union __res;

  __res.as_char[0] = __b;
  __res.as_char[1] = __b;
  __res.as_char[2] = __b;
  __res.as_char[3] = __b;
  __res.as_char[4] = __b;
````
- **L1417 EN**: Adds a standalone statement or declaration: `__res.as_short[1] = __w;`.
  **L1417 CN**: 添加一条独立语句或声明：`__res.as_short[1] = __w;`。
- **L1418 EN**: Adds a standalone statement or declaration: `__res.as_short[2] = __w;`.
  **L1418 CN**: 添加一条独立语句或声明：`__res.as_short[2] = __w;`。
- **L1419 EN**: Adds a standalone statement or declaration: `__res.as_short[3] = __w;`.
  **L1419 CN**: 添加一条独立语句或声明：`__res.as_short[3] = __w;`。
- **L1420 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1420 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1421 EN**: Closes the current preprocessor conditional block.
  **L1421 CN**: 结束当前预处理条件块。
- **L1422 EN**: Closes the current lexical scope or compound statement.
  **L1422 CN**: 结束当前词法作用域或复合语句块。
- **L1423 EN**: Blank line separating nearby declarations or logic blocks.
  **L1423 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1424 EN**: Comment explains nearby logic, constraints, or intent: `Creates a vector of eight 8-bit values, all elements containing B.`.
  **L1424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a vector of eight 8-bit values, all elements containing B.`。
- **L1425 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1425 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1426 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1426 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1427 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_pi8(signed char __b) {`.
  **L1427 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_pi8(signed char __b) {`。
- **L1428 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L1428 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L1429 EN**: Adds a standalone statement or declaration: `__vector signed char __res;`.
  **L1429 CN**: 添加一条独立语句或声明：`__vector signed char __res;`。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1431 EN**: Executes a call or declaration centered on `=`.
  **L1431 CN**: 执行以 `=` 为核心的调用或声明。
- **L1432 EN**: Returns from the current function with `(__m64)((__vector long long)__res)[0]`.
  **L1432 CN**: 以 `(__m64)((__vector long long)__res)[0]` 从当前函数返回。
- **L1433 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1433 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1434 EN**: Adds a standalone statement or declaration: `__m64_union __res;`.
  **L1434 CN**: 添加一条独立语句或声明：`__m64_union __res;`。
- **L1435 EN**: Blank line separating nearby declarations or logic blocks.
  **L1435 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1436 EN**: Adds a standalone statement or declaration: `__res.as_char[0] = __b;`.
  **L1436 CN**: 添加一条独立语句或声明：`__res.as_char[0] = __b;`。
- **L1437 EN**: Adds a standalone statement or declaration: `__res.as_char[1] = __b;`.
  **L1437 CN**: 添加一条独立语句或声明：`__res.as_char[1] = __b;`。
- **L1438 EN**: Adds a standalone statement or declaration: `__res.as_char[2] = __b;`.
  **L1438 CN**: 添加一条独立语句或声明：`__res.as_char[2] = __b;`。
- **L1439 EN**: Adds a standalone statement or declaration: `__res.as_char[3] = __b;`.
  **L1439 CN**: 添加一条独立语句或声明：`__res.as_char[3] = __b;`。
- **L1440 EN**: Adds a standalone statement or declaration: `__res.as_char[4] = __b;`.
  **L1440 CN**: 添加一条独立语句或声明：`__res.as_char[4] = __b;`。

### Lines 1441-1453

````c
  __res.as_char[5] = __b;
  __res.as_char[6] = __b;
  __res.as_char[7] = __b;
  return (__res.as_m64);
#endif
}

#else
#include_next <mmintrin.h>
#endif /* defined(__powerpc64__) &&                                            \
        *   (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX)) */

#endif /* _MMINTRIN_H_INCLUDED */
````
- **L1441 EN**: Adds a standalone statement or declaration: `__res.as_char[5] = __b;`.
  **L1441 CN**: 添加一条独立语句或声明：`__res.as_char[5] = __b;`。
- **L1442 EN**: Adds a standalone statement or declaration: `__res.as_char[6] = __b;`.
  **L1442 CN**: 添加一条独立语句或声明：`__res.as_char[6] = __b;`。
- **L1443 EN**: Adds a standalone statement or declaration: `__res.as_char[7] = __b;`.
  **L1443 CN**: 添加一条独立语句或声明：`__res.as_char[7] = __b;`。
- **L1444 EN**: Returns from the current function with `(__res.as_m64)`.
  **L1444 CN**: 以 `(__res.as_m64)` 从当前函数返回。
- **L1445 EN**: Closes the current preprocessor conditional block.
  **L1445 CN**: 结束当前预处理条件块。
- **L1446 EN**: Closes the current lexical scope or compound statement.
  **L1446 CN**: 结束当前词法作用域或复合语句块。
- **L1447 EN**: Blank line separating nearby declarations or logic blocks.
  **L1447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1448 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1448 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1449 EN**: Includes <mmintrin.h> to access related header declarations.
  **L1449 CN**: 引入 <mmintrin.h> 以使用相关头文件声明。
- **L1450 EN**: Closes the current preprocessor conditional block.
  **L1450 CN**: 结束当前预处理条件块。
- **L1451 EN**: Comment explains nearby logic, constraints, or intent: `(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`.
  **L1451 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`。
- **L1452 EN**: Blank line separating nearby declarations or logic blocks.
  **L1452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1453 EN**: Closes the current preprocessor conditional block.
  **L1453 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **PowerPC vector or system interfaces / PowerPC 向量或系统接口**
- **wrapper forwarding layers / 包装转发层**
- **Header composition through includes / 通过 include 组合头文件**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `altivec.h`: Provides related header declarations. / 提供相关头文件声明。
  - `mmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `NO_WARN_X86_INTRINSICS`, `_MMINTRIN_H_INCLUDED`, `__powerpc64__`, `__linux__`, `__FreeBSD__`, `_AIX`, `_ARCH_PWR8`, `__LITTLE_ENDIAN__`, `_ARCH_PWR6`
- **External builtins / 外部 builtin**: No direct builtin forwarding detected. / 未检测到直接的 builtin 转发。
