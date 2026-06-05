# emmintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/ppc_wrappers/emmintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implementation of SSE2 intrinsics on PowerPC.
- **Purpose (CN)**: 该头文件主要作用是：Implementation of SSE2 intrinsics on PowerPC。
- **Line Count / 行数**: 2269

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- emmintrin.h - Implementation of SSE2 intrinsics on PowerPC -------===
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

   Since X86 SSE2 intrinsics mainly handles __m128i and __m128d type,
   PowerPC VMX/VSX ISA is a good match for vector float SIMD operations.
   However scalar float operations in vector (XMM) registers require
   the POWER8 VSX ISA (2.07) level. There are differences for data
   format and placement of float scalars in the vector register, which
   require extra steps to match SSE2 scalar float semantics on POWER.

   It should be noted that there's much difference between X86_64's
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
- **L17 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Since X86 SSE2 intrinsics mainly handles __m128i and __m128d type,`.
  **L17 CN**: 继续一个多行参数列表、初始化器或聚合项：`Since X86 SSE2 intrinsics mainly handles __m128i and __m128d type,`。
- **L18 EN**: Continues the surrounding expression or declaration: `PowerPC VMX/VSX ISA is a good match for vector float SIMD operations.`.
  **L18 CN**: 继续构造周围的表达式或声明：`PowerPC VMX/VSX ISA is a good match for vector float SIMD operations.`。
- **L19 EN**: Continues logic associated with callable symbol `vector`.
  **L19 CN**: 继续与可调用符号 `vector` 相关的逻辑。
- **L20 EN**: Continues logic associated with callable symbol `ISA`.
  **L20 CN**: 继续与可调用符号 `ISA` 相关的逻辑。
- **L21 EN**: Continues the surrounding expression or declaration: `format and placement of float scalars in the vector register, which`.
  **L21 CN**: 继续构造周围的表达式或声明：`format and placement of float scalars in the vector register, which`。
- **L22 EN**: Continues the surrounding expression or declaration: `require extra steps to match SSE2 scalar float semantics on POWER.`.
  **L22 CN**: 继续构造周围的表达式或声明：`require extra steps to match SSE2 scalar float semantics on POWER.`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Continues the surrounding expression or declaration: `It should be noted that there's much difference between X86_64's`.
  **L24 CN**: 继续构造周围的表达式或声明：`It should be noted that there's much difference between X86_64's`。

### Lines 25-48

````c
   MXSCR and PowerISA's FPSCR/VSCR registers. It's recommended to use
   portable <fenv.h> instead of access MXSCR directly.

   Most SSE2 scalar float intrinsic operations can be performed more
   efficiently as C language float scalar operations or optimized to
   use vector SIMD operations. We recommend this for new applications.
*/
#error                                                                         \
    "Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."
#endif

#ifndef EMMINTRIN_H_
#define EMMINTRIN_H_

#if defined(__powerpc64__) &&                                                  \
    (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))

#include <altivec.h>

/* We need definitions from the SSE header files.  */
#include <xmmintrin.h>

/* SSE2 */
typedef __vector double __v2df;
````
- **L25 EN**: Continues the surrounding expression or declaration: `MXSCR and PowerISA's FPSCR/VSCR registers. It's recommended to use`.
  **L25 CN**: 继续构造周围的表达式或声明：`MXSCR and PowerISA's FPSCR/VSCR registers. It's recommended to use`。
- **L26 EN**: Continues the surrounding expression or declaration: `portable <fenv.h> instead of access MXSCR directly.`.
  **L26 CN**: 继续构造周围的表达式或声明：`portable <fenv.h> instead of access MXSCR directly.`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Continues the surrounding expression or declaration: `Most SSE2 scalar float intrinsic operations can be performed more`.
  **L28 CN**: 继续构造周围的表达式或声明：`Most SSE2 scalar float intrinsic operations can be performed more`。
- **L29 EN**: Continues the surrounding expression or declaration: `efficiently as C language float scalar operations or optimized to`.
  **L29 CN**: 继续构造周围的表达式或声明：`efficiently as C language float scalar operations or optimized to`。
- **L30 EN**: Continues the surrounding expression or declaration: `use vector SIMD operations. We recommend this for new applications.`.
  **L30 CN**: 继续构造周围的表达式或声明：`use vector SIMD operations. We recommend this for new applications.`。
- **L31 EN**: Separator comment used for visual grouping.
  **L31 CN**: 用于视觉分组的分隔注释。
- **L32 EN**: Emits a compilation error for an unsupported configuration: `#error                                                                         \`.
  **L32 CN**: 为不受支持的配置触发编译错误：`#error                                                                         \`。
- **L33 EN**: Continues the surrounding expression or declaration: `"Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."`.
  **L33 CN**: 继续构造周围的表达式或声明：`"Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."`。
- **L34 EN**: Closes the current preprocessor conditional block.
  **L34 CN**: 结束当前预处理条件块。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L36 EN**: Starts a preprocessor conditional block: `#ifndef EMMINTRIN_H_`.
  **L36 CN**: 开始一个预处理条件块：`#ifndef EMMINTRIN_H_`。
- **L37 EN**: Defines macro `EMMINTRIN_H_` for conditional compilation, shorthand, or API generation.
  **L37 CN**: 定义宏 `EMMINTRIN_H_`，用于条件编译、简写或 API 生成。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L39 EN**: Starts a preprocessor conditional block: `#if defined(__powerpc64__) &&                                                  \`.
  **L39 CN**: 开始一个预处理条件块：`#if defined(__powerpc64__) &&                                                  \`。
- **L40 EN**: Continues logic associated with callable symbol `defined`.
  **L40 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Includes <altivec.h> to access related header declarations.
  **L42 CN**: 引入 <altivec.h> 以使用相关头文件声明。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L44 EN**: Comment explains nearby logic, constraints, or intent: `We need definitions from the SSE header files.`.
  **L44 CN**: 注释解释附近代码的逻辑、约束或设计意图：`We need definitions from the SSE header files.`。
- **L45 EN**: Includes <xmmintrin.h> to access related header declarations.
  **L45 CN**: 引入 <xmmintrin.h> 以使用相关头文件声明。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Comment explains nearby logic, constraints, or intent: `SSE2`.
  **L47 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SSE2`。
- **L48 EN**: Introduces an alias or helper declaration: `typedef __vector double __v2df;`.
  **L48 CN**: 引入一条别名或辅助声明：`typedef __vector double __v2df;`。

### Lines 49-72

````c
typedef __vector float __v4f;
typedef __vector long long __v2di;
typedef __vector unsigned long long __v2du;
typedef __vector int __v4si;
typedef __vector unsigned int __v4su;
typedef __vector short __v8hi;
typedef __vector unsigned short __v8hu;
typedef __vector signed char __v16qi;
typedef __vector unsigned char __v16qu;

/* The Intel API is flexible enough that we must allow aliasing with other
   vector types, and their scalar components.  */
typedef long long __m128i __attribute__((__vector_size__(16), __may_alias__));
typedef double __m128d __attribute__((__vector_size__(16), __may_alias__));

/* Unaligned version of the same types.  */
typedef long long __m128i_u
    __attribute__((__vector_size__(16), __may_alias__, __aligned__(1)));
typedef double __m128d_u
    __attribute__((__vector_size__(16), __may_alias__, __aligned__(1)));

/* Define two value permute mask.  */
#define _MM_SHUFFLE2(x, y) (((x) << 1) | (y))

````
- **L49 EN**: Introduces an alias or helper declaration: `typedef __vector float __v4f;`.
  **L49 CN**: 引入一条别名或辅助声明：`typedef __vector float __v4f;`。
- **L50 EN**: Introduces an alias or helper declaration: `typedef __vector long long __v2di;`.
  **L50 CN**: 引入一条别名或辅助声明：`typedef __vector long long __v2di;`。
- **L51 EN**: Introduces an alias or helper declaration: `typedef __vector unsigned long long __v2du;`.
  **L51 CN**: 引入一条别名或辅助声明：`typedef __vector unsigned long long __v2du;`。
- **L52 EN**: Introduces an alias or helper declaration: `typedef __vector int __v4si;`.
  **L52 CN**: 引入一条别名或辅助声明：`typedef __vector int __v4si;`。
- **L53 EN**: Introduces an alias or helper declaration: `typedef __vector unsigned int __v4su;`.
  **L53 CN**: 引入一条别名或辅助声明：`typedef __vector unsigned int __v4su;`。
- **L54 EN**: Introduces an alias or helper declaration: `typedef __vector short __v8hi;`.
  **L54 CN**: 引入一条别名或辅助声明：`typedef __vector short __v8hi;`。
- **L55 EN**: Introduces an alias or helper declaration: `typedef __vector unsigned short __v8hu;`.
  **L55 CN**: 引入一条别名或辅助声明：`typedef __vector unsigned short __v8hu;`。
- **L56 EN**: Introduces an alias or helper declaration: `typedef __vector signed char __v16qi;`.
  **L56 CN**: 引入一条别名或辅助声明：`typedef __vector signed char __v16qi;`。
- **L57 EN**: Introduces an alias or helper declaration: `typedef __vector unsigned char __v16qu;`.
  **L57 CN**: 引入一条别名或辅助声明：`typedef __vector unsigned char __v16qu;`。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `The Intel API is flexible enough that we must allow aliasing with other`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The Intel API is flexible enough that we must allow aliasing with other`。
- **L60 EN**: Continues the surrounding expression or declaration: `vector types, and their scalar components.  */`.
  **L60 CN**: 继续构造周围的表达式或声明：`vector types, and their scalar components.  */`。
- **L61 EN**: Introduces an alias or helper declaration: `typedef long long __m128i __attribute__((__vector_size__(16), __may_alias__));`.
  **L61 CN**: 引入一条别名或辅助声明：`typedef long long __m128i __attribute__((__vector_size__(16), __may_alias__));`。
- **L62 EN**: Introduces an alias or helper declaration: `typedef double __m128d __attribute__((__vector_size__(16), __may_alias__));`.
  **L62 CN**: 引入一条别名或辅助声明：`typedef double __m128d __attribute__((__vector_size__(16), __may_alias__));`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `Unaligned version of the same types.`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unaligned version of the same types.`。
- **L65 EN**: Introduces an alias or helper declaration: `typedef long long __m128i_u`.
  **L65 CN**: 引入一条别名或辅助声明：`typedef long long __m128i_u`。
- **L66 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(16), __may_alias__, __aligned__(1)));`.
  **L66 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(16), __may_alias__, __aligned__(1)));`。
- **L67 EN**: Introduces an alias or helper declaration: `typedef double __m128d_u`.
  **L67 CN**: 引入一条别名或辅助声明：`typedef double __m128d_u`。
- **L68 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(16), __may_alias__, __aligned__(1)));`.
  **L68 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(16), __may_alias__, __aligned__(1)));`。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `Define two value permute mask.`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define two value permute mask.`。
- **L71 EN**: Defines macro `_MM_SHUFFLE2(x, y)` for conditional compilation, shorthand, or API generation.
  **L71 CN**: 定义宏 `_MM_SHUFFLE2(x, y)`，用于条件编译、简写或 API 生成。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 73-96

````c
/* Create a vector with element 0 as F and the rest zero.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_sd(double __F) {
  return __extension__(__m128d){__F, 0.0};
}

/* Create a vector with both elements equal to F.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_pd(double __F) {
  return __extension__(__m128d){__F, __F};
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_pd1(double __F) {
  return _mm_set1_pd(__F);
}

/* Create a vector with the lower value X and upper value W.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_pd(double __W, double __X) {
````
- **L73 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with element 0 as F and the rest zero.`.
  **L73 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with element 0 as F and the rest zero.`。
- **L74 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L74 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L75 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L75 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L76 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_sd(double __F) {`.
  **L76 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_sd(double __F) {`。
- **L77 EN**: Returns from the current function with `__extension__(__m128d){__F, 0.0}`.
  **L77 CN**: 以 `__extension__(__m128d){__F, 0.0}` 从当前函数返回。
- **L78 EN**: Closes the current lexical scope or compound statement.
  **L78 CN**: 结束当前词法作用域或复合语句块。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with both elements equal to F.`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with both elements equal to F.`。
- **L81 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L81 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L82 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L82 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L83 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_pd(double __F) {`.
  **L83 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_pd(double __F) {`。
- **L84 EN**: Returns from the current function with `__extension__(__m128d){__F, __F}`.
  **L84 CN**: 以 `__extension__(__m128d){__F, __F}` 从当前函数返回。
- **L85 EN**: Closes the current lexical scope or compound statement.
  **L85 CN**: 结束当前词法作用域或复合语句块。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L87 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L87 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L88 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L88 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L89 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_pd1(double __F) {`.
  **L89 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_pd1(double __F) {`。
- **L90 EN**: Returns from the current function with `_mm_set1_pd(__F)`.
  **L90 CN**: 以 `_mm_set1_pd(__F)` 从当前函数返回。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L93 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with the lower value X and upper value W.`.
  **L93 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with the lower value X and upper value W.`。
- **L94 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L94 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L95 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L95 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L96 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_pd(double __W, double __X) {`.
  **L96 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_pd(double __W, double __X) {`。

### Lines 97-120

````c
  return __extension__(__m128d){__X, __W};
}

/* Create a vector with the lower value W and upper value X.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_pd(double __W, double __X) {
  return __extension__(__m128d){__W, __X};
}

/* Create an undefined vector.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_undefined_pd(void) {
  __m128d __Y = __Y;
  return __Y;
}

/* Create a vector of zeros.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setzero_pd(void) {
  return (__m128d)vec_splats(0);
}
````
- **L97 EN**: Returns from the current function with `__extension__(__m128d){__X, __W}`.
  **L97 CN**: 以 `__extension__(__m128d){__X, __W}` 从当前函数返回。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L100 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with the lower value W and upper value X.`.
  **L100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with the lower value W and upper value X.`。
- **L101 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L101 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L102 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L102 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L103 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setr_pd(double __W, double __X) {`.
  **L103 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setr_pd(double __W, double __X) {`。
- **L104 EN**: Returns from the current function with `__extension__(__m128d){__W, __X}`.
  **L104 CN**: 以 `__extension__(__m128d){__W, __X}` 从当前函数返回。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L107 EN**: Comment explains nearby logic, constraints, or intent: `Create an undefined vector.`.
  **L107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create an undefined vector.`。
- **L108 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L108 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L109 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L109 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L110 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_undefined_pd(void) {`.
  **L110 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_undefined_pd(void) {`。
- **L111 EN**: Initializes variable `__Y` from the expression on the right-hand side.
  **L111 CN**: 使用右侧表达式初始化变量 `__Y`。
- **L112 EN**: Returns from the current function with `__Y`.
  **L112 CN**: 以 `__Y` 从当前函数返回。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector of zeros.`.
  **L115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector of zeros.`。
- **L116 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L116 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L117 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L117 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L118 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setzero_pd(void) {`.
  **L118 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setzero_pd(void) {`。
- **L119 EN**: Returns from the current function with `(__m128d)vec_splats(0)`.
  **L119 CN**: 以 `(__m128d)vec_splats(0)` 从当前函数返回。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。

### Lines 121-144

````c

/* Sets the low DPFP value of A from the low value of B.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_move_sd(__m128d __A, __m128d __B) {
  __v2df __result = (__v2df)__A;
  __result[0] = ((__v2df)__B)[0];
  return (__m128d)__result;
}

/* Load two DPFP values from P.  The address must be 16-byte aligned.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load_pd(double const *__P) {
  return ((__m128d)vec_ld(0, (__v16qu *)__P));
}

/* Load two DPFP values from P.  The address need not be 16-byte aligned.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadu_pd(double const *__P) {
  return (vec_vsx_ld(0, __P));
}

````
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L122 EN**: Comment explains nearby logic, constraints, or intent: `Sets the low DPFP value of A from the low value of B.`.
  **L122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sets the low DPFP value of A from the low value of B.`。
- **L123 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L123 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L124 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L124 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_move_sd(__m128d __A, __m128d __B) {`.
  **L125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_move_sd(__m128d __A, __m128d __B) {`。
- **L126 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L126 CN**: 使用右侧表达式初始化变量 `__result`。
- **L127 EN**: Executes a call or declaration centered on `=`.
  **L127 CN**: 执行以 `=` 为核心的调用或声明。
- **L128 EN**: Returns from the current function with `(__m128d)__result`.
  **L128 CN**: 以 `(__m128d)__result` 从当前函数返回。
- **L129 EN**: Closes the current lexical scope or compound statement.
  **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L131 EN**: Comment explains nearby logic, constraints, or intent: `Load two DPFP values from P. The address must be 16-byte aligned.`.
  **L131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load two DPFP values from P. The address must be 16-byte aligned.`。
- **L132 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L132 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L133 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L133 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L134 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load_pd(double const *__P) {`.
  **L134 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load_pd(double const *__P) {`。
- **L135 EN**: Returns from the current function with `((__m128d)vec_ld(0, (__v16qu *)__P))`.
  **L135 CN**: 以 `((__m128d)vec_ld(0, (__v16qu *)__P))` 从当前函数返回。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Comment explains nearby logic, constraints, or intent: `Load two DPFP values from P. The address need not be 16-byte aligned.`.
  **L138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load two DPFP values from P. The address need not be 16-byte aligned.`。
- **L139 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L139 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L140 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L140 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L141 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadu_pd(double const *__P) {`.
  **L141 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadu_pd(double const *__P) {`。
- **L142 EN**: Returns from the current function with `(vec_vsx_ld(0, __P))`.
  **L142 CN**: 以 `(vec_vsx_ld(0, __P))` 从当前函数返回。
- **L143 EN**: Closes the current lexical scope or compound statement.
  **L143 CN**: 结束当前词法作用域或复合语句块。
- **L144 EN**: Blank line separating nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 145-168

````c
/* Create a vector with all two elements equal to *P.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load1_pd(double const *__P) {
  return (vec_splats(*__P));
}

/* Create a vector with element 0 as *P and the rest zero.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load_sd(double const *__P) {
  return _mm_set_sd(*__P);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load_pd1(double const *__P) {
  return _mm_load1_pd(__P);
}

/* Load two DPFP values in reverse order.  The address must be aligned.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadr_pd(double const *__P) {
````
- **L145 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with all two elements equal to *P.`.
  **L145 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with all two elements equal to *P.`。
- **L146 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L146 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L147 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L147 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L148 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load1_pd(double const *__P) {`.
  **L148 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load1_pd(double const *__P) {`。
- **L149 EN**: Returns from the current function with `(vec_splats(*__P))`.
  **L149 CN**: 以 `(vec_splats(*__P))` 从当前函数返回。
- **L150 EN**: Closes the current lexical scope or compound statement.
  **L150 CN**: 结束当前词法作用域或复合语句块。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L152 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with element 0 as *P and the rest zero.`.
  **L152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with element 0 as *P and the rest zero.`。
- **L153 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L153 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L154 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L154 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L155 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load_sd(double const *__P) {`.
  **L155 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load_sd(double const *__P) {`。
- **L156 EN**: Returns from the current function with `_mm_set_sd(*__P)`.
  **L156 CN**: 以 `_mm_set_sd(*__P)` 从当前函数返回。
- **L157 EN**: Closes the current lexical scope or compound statement.
  **L157 CN**: 结束当前词法作用域或复合语句块。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L159 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L159 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L160 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L160 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L161 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load_pd1(double const *__P) {`.
  **L161 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load_pd1(double const *__P) {`。
- **L162 EN**: Returns from the current function with `_mm_load1_pd(__P)`.
  **L162 CN**: 以 `_mm_load1_pd(__P)` 从当前函数返回。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `Load two DPFP values in reverse order. The address must be aligned.`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load two DPFP values in reverse order. The address must be aligned.`。
- **L166 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L166 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L167 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L167 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L168 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadr_pd(double const *__P) {`.
  **L168 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadr_pd(double const *__P) {`。

### Lines 169-192

````c
  __v2df __tmp = _mm_load_pd(__P);
  return (__m128d)vec_xxpermdi(__tmp, __tmp, 2);
}

/* Store two DPFP values.  The address must be 16-byte aligned.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store_pd(double *__P, __m128d __A) {
  vec_st((__v16qu)__A, 0, (__v16qu *)__P);
}

/* Store two DPFP values.  The address need not be 16-byte aligned.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storeu_pd(double *__P, __m128d __A) {
  *(__m128d_u *)__P = __A;
}

/* Stores the lower DPFP value.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store_sd(double *__P, __m128d __A) {
  *__P = ((__v2df)__A)[0];
}
````
- **L169 EN**: Initializes variable `__tmp` from the expression on the right-hand side.
  **L169 CN**: 使用右侧表达式初始化变量 `__tmp`。
- **L170 EN**: Returns from the current function with `(__m128d)vec_xxpermdi(__tmp, __tmp, 2)`.
  **L170 CN**: 以 `(__m128d)vec_xxpermdi(__tmp, __tmp, 2)` 从当前函数返回。
- **L171 EN**: Closes the current lexical scope or compound statement.
  **L171 CN**: 结束当前词法作用域或复合语句块。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L173 EN**: Comment explains nearby logic, constraints, or intent: `Store two DPFP values. The address must be 16-byte aligned.`.
  **L173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store two DPFP values. The address must be 16-byte aligned.`。
- **L174 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L174 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L175 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L175 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L176 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store_pd(double *__P, __m128d __A) {`.
  **L176 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store_pd(double *__P, __m128d __A) {`。
- **L177 EN**: Executes a call or declaration centered on `vec_st`.
  **L177 CN**: 执行以 `vec_st` 为核心的调用或声明。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Blank line separating nearby declarations or logic blocks.
  **L179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L180 EN**: Comment explains nearby logic, constraints, or intent: `Store two DPFP values. The address need not be 16-byte aligned.`.
  **L180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store two DPFP values. The address need not be 16-byte aligned.`。
- **L181 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L181 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L182 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L182 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L183 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storeu_pd(double *__P, __m128d __A) {`.
  **L183 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storeu_pd(double *__P, __m128d __A) {`。
- **L184 EN**: Comment explains nearby logic, constraints, or intent: `(__m128d_u *)__P __A;`.
  **L184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__m128d_u *)__P __A;`。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Blank line separating nearby declarations or logic blocks.
  **L186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L187 EN**: Comment explains nearby logic, constraints, or intent: `Stores the lower DPFP value.`.
  **L187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores the lower DPFP value.`。
- **L188 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L188 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L189 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L189 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L190 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store_sd(double *__P, __m128d __A) {`.
  **L190 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store_sd(double *__P, __m128d __A) {`。
- **L191 EN**: Comment explains nearby logic, constraints, or intent: `__P ((__v2df)__A)[0];`.
  **L191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__P ((__v2df)__A)[0];`。
- **L192 EN**: Closes the current lexical scope or compound statement.
  **L192 CN**: 结束当前词法作用域或复合语句块。

### Lines 193-216

````c

extern __inline double
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsd_f64(__m128d __A) {
  return ((__v2df)__A)[0];
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storel_pd(double *__P, __m128d __A) {
  _mm_store_sd(__P, __A);
}

/* Stores the upper DPFP value.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storeh_pd(double *__P, __m128d __A) {
  *__P = ((__v2df)__A)[1];
}
/* Store the lower DPFP value across two words.
   The address must be 16-byte aligned.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store1_pd(double *__P, __m128d __A) {
````
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L194 EN**: Continues the surrounding expression or declaration: `extern __inline double`.
  **L194 CN**: 继续构造周围的表达式或声明：`extern __inline double`。
- **L195 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L195 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L196 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsd_f64(__m128d __A) {`.
  **L196 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsd_f64(__m128d __A) {`。
- **L197 EN**: Returns from the current function with `((__v2df)__A)[0]`.
  **L197 CN**: 以 `((__v2df)__A)[0]` 从当前函数返回。
- **L198 EN**: Closes the current lexical scope or compound statement.
  **L198 CN**: 结束当前词法作用域或复合语句块。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L200 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L200 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L201 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L201 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L202 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storel_pd(double *__P, __m128d __A) {`.
  **L202 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storel_pd(double *__P, __m128d __A) {`。
- **L203 EN**: Executes a call or declaration centered on `_mm_store_sd`.
  **L203 CN**: 执行以 `_mm_store_sd` 为核心的调用或声明。
- **L204 EN**: Closes the current lexical scope or compound statement.
  **L204 CN**: 结束当前词法作用域或复合语句块。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Comment explains nearby logic, constraints, or intent: `Stores the upper DPFP value.`.
  **L206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores the upper DPFP value.`。
- **L207 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L207 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L208 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L208 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L209 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storeh_pd(double *__P, __m128d __A) {`.
  **L209 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storeh_pd(double *__P, __m128d __A) {`。
- **L210 EN**: Comment explains nearby logic, constraints, or intent: `__P ((__v2df)__A)[1];`.
  **L210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__P ((__v2df)__A)[1];`。
- **L211 EN**: Closes the current lexical scope or compound statement.
  **L211 CN**: 结束当前词法作用域或复合语句块。
- **L212 EN**: Comment explains nearby logic, constraints, or intent: `Store the lower DPFP value across two words.`.
  **L212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store the lower DPFP value across two words.`。
- **L213 EN**: Continues the surrounding expression or declaration: `The address must be 16-byte aligned.  */`.
  **L213 CN**: 继续构造周围的表达式或声明：`The address must be 16-byte aligned.  */`。
- **L214 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L214 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L215 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L215 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L216 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store1_pd(double *__P, __m128d __A) {`.
  **L216 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store1_pd(double *__P, __m128d __A) {`。

### Lines 217-240

````c
  _mm_store_pd(__P, vec_splat(__A, 0));
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store_pd1(double *__P, __m128d __A) {
  _mm_store1_pd(__P, __A);
}

/* Store two DPFP values in reverse order.  The address must be aligned.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storer_pd(double *__P, __m128d __A) {
  _mm_store_pd(__P, vec_xxpermdi(__A, __A, 2));
}

/* Intel intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi128_si64(__m128i __A) {
  return ((__v2di)__A)[0];
}

/* Microsoft intrinsic.  */
````
- **L217 EN**: Executes a call or declaration centered on `_mm_store_pd`.
  **L217 CN**: 执行以 `_mm_store_pd` 为核心的调用或声明。
- **L218 EN**: Closes the current lexical scope or compound statement.
  **L218 CN**: 结束当前词法作用域或复合语句块。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L220 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L220 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L221 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L221 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L222 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store_pd1(double *__P, __m128d __A) {`.
  **L222 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store_pd1(double *__P, __m128d __A) {`。
- **L223 EN**: Executes a call or declaration centered on `_mm_store1_pd`.
  **L223 CN**: 执行以 `_mm_store1_pd` 为核心的调用或声明。
- **L224 EN**: Closes the current lexical scope or compound statement.
  **L224 CN**: 结束当前词法作用域或复合语句块。
- **L225 EN**: Blank line separating nearby declarations or logic blocks.
  **L225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L226 EN**: Comment explains nearby logic, constraints, or intent: `Store two DPFP values in reverse order. The address must be aligned.`.
  **L226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store two DPFP values in reverse order. The address must be aligned.`。
- **L227 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L227 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L228 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L228 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L229 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storer_pd(double *__P, __m128d __A) {`.
  **L229 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storer_pd(double *__P, __m128d __A) {`。
- **L230 EN**: Executes a call or declaration centered on `_mm_store_pd`.
  **L230 CN**: 执行以 `_mm_store_pd` 为核心的调用或声明。
- **L231 EN**: Closes the current lexical scope or compound statement.
  **L231 CN**: 结束当前词法作用域或复合语句块。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L234 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L234 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L235 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L235 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L236 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi128_si64(__m128i __A) {`.
  **L236 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi128_si64(__m128i __A) {`。
- **L237 EN**: Returns from the current function with `((__v2di)__A)[0]`.
  **L237 CN**: 以 `((__v2di)__A)[0]` 从当前函数返回。
- **L238 EN**: Closes the current lexical scope or compound statement.
  **L238 CN**: 结束当前词法作用域或复合语句块。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L240 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。

### Lines 241-264

````c
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi128_si64x(__m128i __A) {
  return ((__v2di)__A)[0];
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_pd(__m128d __A, __m128d __B) {
  return (__m128d)((__v2df)__A + (__v2df)__B);
}

/* Add the lower double-precision (64-bit) floating-point element in
   a and b, store the result in the lower element of dst, and copy
   the upper element from a to the upper element of dst. */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_sd(__m128d __A, __m128d __B) {
  __A[0] = __A[0] + __B[0];
  return (__A);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L241 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L241 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L242 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L242 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L243 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi128_si64x(__m128i __A) {`.
  **L243 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi128_si64x(__m128i __A) {`。
- **L244 EN**: Returns from the current function with `((__v2di)__A)[0]`.
  **L244 CN**: 以 `((__v2di)__A)[0]` 从当前函数返回。
- **L245 EN**: Closes the current lexical scope or compound statement.
  **L245 CN**: 结束当前词法作用域或复合语句块。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L247 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L247 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L248 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L248 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L249 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_pd(__m128d __A, __m128d __B) {`.
  **L249 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_pd(__m128d __A, __m128d __B) {`。
- **L250 EN**: Returns from the current function with `(__m128d)((__v2df)__A + (__v2df)__B)`.
  **L250 CN**: 以 `(__m128d)((__v2df)__A + (__v2df)__B)` 从当前函数返回。
- **L251 EN**: Closes the current lexical scope or compound statement.
  **L251 CN**: 结束当前词法作用域或复合语句块。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L253 EN**: Comment explains nearby logic, constraints, or intent: `Add the lower double-precision (64-bit) floating-point element in`.
  **L253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add the lower double-precision (64-bit) floating-point element in`。
- **L254 EN**: Continues the surrounding expression or declaration: `a and b, store the result in the lower element of dst, and copy`.
  **L254 CN**: 继续构造周围的表达式或声明：`a and b, store the result in the lower element of dst, and copy`。
- **L255 EN**: Continues the surrounding expression or declaration: `the upper element from a to the upper element of dst. */`.
  **L255 CN**: 继续构造周围的表达式或声明：`the upper element from a to the upper element of dst. */`。
- **L256 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L256 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L257 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L257 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L258 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_sd(__m128d __A, __m128d __B) {`.
  **L258 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_sd(__m128d __A, __m128d __B) {`。
- **L259 EN**: Adds a standalone statement or declaration: `__A[0] = __A[0] + __B[0];`.
  **L259 CN**: 添加一条独立语句或声明：`__A[0] = __A[0] + __B[0];`。
- **L260 EN**: Returns from the current function with `(__A)`.
  **L260 CN**: 以 `(__A)` 从当前函数返回。
- **L261 EN**: Closes the current lexical scope or compound statement.
  **L261 CN**: 结束当前词法作用域或复合语句块。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L263 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L263 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L264 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L264 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 265-288

````c
    _mm_sub_pd(__m128d __A, __m128d __B) {
  return (__m128d)((__v2df)__A - (__v2df)__B);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_sd(__m128d __A, __m128d __B) {
  __A[0] = __A[0] - __B[0];
  return (__A);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mul_pd(__m128d __A, __m128d __B) {
  return (__m128d)((__v2df)__A * (__v2df)__B);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mul_sd(__m128d __A, __m128d __B) {
  __A[0] = __A[0] * __B[0];
  return (__A);
}

````
- **L265 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_pd(__m128d __A, __m128d __B) {`.
  **L265 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_pd(__m128d __A, __m128d __B) {`。
- **L266 EN**: Returns from the current function with `(__m128d)((__v2df)__A - (__v2df)__B)`.
  **L266 CN**: 以 `(__m128d)((__v2df)__A - (__v2df)__B)` 从当前函数返回。
- **L267 EN**: Closes the current lexical scope or compound statement.
  **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L269 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L269 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L270 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L270 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L271 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_sd(__m128d __A, __m128d __B) {`.
  **L271 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_sd(__m128d __A, __m128d __B) {`。
- **L272 EN**: Adds a standalone statement or declaration: `__A[0] = __A[0] - __B[0];`.
  **L272 CN**: 添加一条独立语句或声明：`__A[0] = __A[0] - __B[0];`。
- **L273 EN**: Returns from the current function with `(__A)`.
  **L273 CN**: 以 `(__A)` 从当前函数返回。
- **L274 EN**: Closes the current lexical scope or compound statement.
  **L274 CN**: 结束当前词法作用域或复合语句块。
- **L275 EN**: Blank line separating nearby declarations or logic blocks.
  **L275 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L276 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L276 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L277 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L277 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L278 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mul_pd(__m128d __A, __m128d __B) {`.
  **L278 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mul_pd(__m128d __A, __m128d __B) {`。
- **L279 EN**: Returns from the current function with `(__m128d)((__v2df)__A * (__v2df)__B)`.
  **L279 CN**: 以 `(__m128d)((__v2df)__A * (__v2df)__B)` 从当前函数返回。
- **L280 EN**: Closes the current lexical scope or compound statement.
  **L280 CN**: 结束当前词法作用域或复合语句块。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L282 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L282 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L283 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L283 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L284 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mul_sd(__m128d __A, __m128d __B) {`.
  **L284 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mul_sd(__m128d __A, __m128d __B) {`。
- **L285 EN**: Adds a standalone statement or declaration: `__A[0] = __A[0] * __B[0];`.
  **L285 CN**: 添加一条独立语句或声明：`__A[0] = __A[0] * __B[0];`。
- **L286 EN**: Returns from the current function with `(__A)`.
  **L286 CN**: 以 `(__A)` 从当前函数返回。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 289-312

````c
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_div_pd(__m128d __A, __m128d __B) {
  return (__m128d)((__v2df)__A / (__v2df)__B);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_div_sd(__m128d __A, __m128d __B) {
  __A[0] = __A[0] / __B[0];
  return (__A);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sqrt_pd(__m128d __A) {
  return (vec_sqrt(__A));
}

/* Return pair {sqrt (B[0]), A[1]}.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sqrt_sd(__m128d __A, __m128d __B) {
  __v2df __c;
````
- **L289 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L289 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L290 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L290 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L291 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_div_pd(__m128d __A, __m128d __B) {`.
  **L291 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_div_pd(__m128d __A, __m128d __B) {`。
- **L292 EN**: Returns from the current function with `(__m128d)((__v2df)__A / (__v2df)__B)`.
  **L292 CN**: 以 `(__m128d)((__v2df)__A / (__v2df)__B)` 从当前函数返回。
- **L293 EN**: Closes the current lexical scope or compound statement.
  **L293 CN**: 结束当前词法作用域或复合语句块。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L295 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L295 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L296 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L296 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L297 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_div_sd(__m128d __A, __m128d __B) {`.
  **L297 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_div_sd(__m128d __A, __m128d __B) {`。
- **L298 EN**: Adds a standalone statement or declaration: `__A[0] = __A[0] / __B[0];`.
  **L298 CN**: 添加一条独立语句或声明：`__A[0] = __A[0] / __B[0];`。
- **L299 EN**: Returns from the current function with `(__A)`.
  **L299 CN**: 以 `(__A)` 从当前函数返回。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L302 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L302 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L303 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L303 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L304 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sqrt_pd(__m128d __A) {`.
  **L304 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sqrt_pd(__m128d __A) {`。
- **L305 EN**: Returns from the current function with `(vec_sqrt(__A))`.
  **L305 CN**: 以 `(vec_sqrt(__A))` 从当前函数返回。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L308 EN**: Comment explains nearby logic, constraints, or intent: `Return pair {sqrt (B[0]), A[1]}.`.
  **L308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Return pair {sqrt (B[0]), A[1]}.`。
- **L309 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L309 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L310 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L310 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L311 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sqrt_sd(__m128d __A, __m128d __B) {`.
  **L311 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sqrt_sd(__m128d __A, __m128d __B) {`。
- **L312 EN**: Adds a standalone statement or declaration: `__v2df __c;`.
  **L312 CN**: 添加一条独立语句或声明：`__v2df __c;`。

### Lines 313-336

````c
  __c = vec_sqrt((__v2df)_mm_set1_pd(__B[0]));
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_pd(__m128d __A, __m128d __B) {
  return (vec_min(__A, __B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  __c = vec_min(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_pd(__m128d __A, __m128d __B) {
  return (vec_max(__A, __B));
````
- **L313 EN**: Executes a call or declaration centered on `vec_sqrt`.
  **L313 CN**: 执行以 `vec_sqrt` 为核心的调用或声明。
- **L314 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L314 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L315 EN**: Closes the current lexical scope or compound statement.
  **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L317 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L317 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L318 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L318 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L319 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_pd(__m128d __A, __m128d __B) {`.
  **L319 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_pd(__m128d __A, __m128d __B) {`。
- **L320 EN**: Returns from the current function with `(vec_min(__A, __B))`.
  **L320 CN**: 以 `(vec_min(__A, __B))` 从当前函数返回。
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L323 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L324 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L324 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L325 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_sd(__m128d __A, __m128d __B) {`.
  **L325 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_sd(__m128d __A, __m128d __B) {`。
- **L326 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L326 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L327 EN**: Executes a call or declaration centered on `vec_splats`.
  **L327 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L328 EN**: Executes a call or declaration centered on `vec_splats`.
  **L328 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L329 EN**: Executes a call or declaration centered on `vec_min`.
  **L329 CN**: 执行以 `vec_min` 为核心的调用或声明。
- **L330 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L330 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L333 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L334 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L334 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L335 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_pd(__m128d __A, __m128d __B) {`.
  **L335 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_pd(__m128d __A, __m128d __B) {`。
- **L336 EN**: Returns from the current function with `(vec_max(__A, __B))`.
  **L336 CN**: 以 `(vec_max(__A, __B))` 从当前函数返回。

### Lines 337-360

````c
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  __c = vec_max(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmpeq((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmplt_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmplt((__v2df)__A, (__v2df)__B));
}

````
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L339 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L340 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L340 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L341 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_sd(__m128d __A, __m128d __B) {`.
  **L341 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_sd(__m128d __A, __m128d __B) {`。
- **L342 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L342 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L343 EN**: Executes a call or declaration centered on `vec_splats`.
  **L343 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L344 EN**: Executes a call or declaration centered on `vec_splats`.
  **L344 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L345 EN**: Executes a call or declaration centered on `vec_max`.
  **L345 CN**: 执行以 `vec_max` 为核心的调用或声明。
- **L346 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L346 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L349 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L350 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L350 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L351 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_pd(__m128d __A, __m128d __B) {`.
  **L351 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_pd(__m128d __A, __m128d __B) {`。
- **L352 EN**: Returns from the current function with `((__m128d)vec_cmpeq((__v2df)__A, (__v2df)__B))`.
  **L352 CN**: 以 `((__m128d)vec_cmpeq((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L353 EN**: Closes the current lexical scope or compound statement.
  **L353 CN**: 结束当前词法作用域或复合语句块。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L355 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L355 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L356 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L356 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L357 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmplt_pd(__m128d __A, __m128d __B) {`.
  **L357 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmplt_pd(__m128d __A, __m128d __B) {`。
- **L358 EN**: Returns from the current function with `((__m128d)vec_cmplt((__v2df)__A, (__v2df)__B))`.
  **L358 CN**: 以 `((__m128d)vec_cmplt((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L359 EN**: Closes the current lexical scope or compound statement.
  **L359 CN**: 结束当前词法作用域或复合语句块。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 361-384

````c
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmple_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmple((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmpgt((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpge_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmpge((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpneq_pd(__m128d __A, __m128d __B) {
  __v2df __temp = (__v2df)vec_cmpeq((__v2df)__A, (__v2df)__B);
  return ((__m128d)vec_nor(__temp, __temp));
}
````
- **L361 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L361 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L362 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L362 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L363 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmple_pd(__m128d __A, __m128d __B) {`.
  **L363 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmple_pd(__m128d __A, __m128d __B) {`。
- **L364 EN**: Returns from the current function with `((__m128d)vec_cmple((__v2df)__A, (__v2df)__B))`.
  **L364 CN**: 以 `((__m128d)vec_cmple((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L365 EN**: Closes the current lexical scope or compound statement.
  **L365 CN**: 结束当前词法作用域或复合语句块。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L367 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L367 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L368 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L368 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L369 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_pd(__m128d __A, __m128d __B) {`.
  **L369 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_pd(__m128d __A, __m128d __B) {`。
- **L370 EN**: Returns from the current function with `((__m128d)vec_cmpgt((__v2df)__A, (__v2df)__B))`.
  **L370 CN**: 以 `((__m128d)vec_cmpgt((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L371 EN**: Closes the current lexical scope or compound statement.
  **L371 CN**: 结束当前词法作用域或复合语句块。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L373 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L373 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L374 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L374 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L375 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpge_pd(__m128d __A, __m128d __B) {`.
  **L375 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpge_pd(__m128d __A, __m128d __B) {`。
- **L376 EN**: Returns from the current function with `((__m128d)vec_cmpge((__v2df)__A, (__v2df)__B))`.
  **L376 CN**: 以 `((__m128d)vec_cmpge((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L377 EN**: Closes the current lexical scope or compound statement.
  **L377 CN**: 结束当前词法作用域或复合语句块。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L379 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L379 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L380 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L380 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L381 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpneq_pd(__m128d __A, __m128d __B) {`.
  **L381 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpneq_pd(__m128d __A, __m128d __B) {`。
- **L382 EN**: Initializes variable `__temp` from the expression on the right-hand side.
  **L382 CN**: 使用右侧表达式初始化变量 `__temp`。
- **L383 EN**: Returns from the current function with `((__m128d)vec_nor(__temp, __temp))`.
  **L383 CN**: 以 `((__m128d)vec_nor(__temp, __temp))` 从当前函数返回。
- **L384 EN**: Closes the current lexical scope or compound statement.
  **L384 CN**: 结束当前词法作用域或复合语句块。

### Lines 385-408

````c

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnlt_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmpge((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnle_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmpgt((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpngt_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmple((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnge_pd(__m128d __A, __m128d __B) {
  return ((__m128d)vec_cmplt((__v2df)__A, (__v2df)__B));
}
````
- **L385 EN**: Blank line separating nearby declarations or logic blocks.
  **L385 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L386 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L386 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L387 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L387 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L388 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnlt_pd(__m128d __A, __m128d __B) {`.
  **L388 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnlt_pd(__m128d __A, __m128d __B) {`。
- **L389 EN**: Returns from the current function with `((__m128d)vec_cmpge((__v2df)__A, (__v2df)__B))`.
  **L389 CN**: 以 `((__m128d)vec_cmpge((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L390 EN**: Closes the current lexical scope or compound statement.
  **L390 CN**: 结束当前词法作用域或复合语句块。
- **L391 EN**: Blank line separating nearby declarations or logic blocks.
  **L391 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L392 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L392 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L393 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L393 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L394 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnle_pd(__m128d __A, __m128d __B) {`.
  **L394 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnle_pd(__m128d __A, __m128d __B) {`。
- **L395 EN**: Returns from the current function with `((__m128d)vec_cmpgt((__v2df)__A, (__v2df)__B))`.
  **L395 CN**: 以 `((__m128d)vec_cmpgt((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L396 EN**: Closes the current lexical scope or compound statement.
  **L396 CN**: 结束当前词法作用域或复合语句块。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L398 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L398 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L399 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L399 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L400 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpngt_pd(__m128d __A, __m128d __B) {`.
  **L400 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpngt_pd(__m128d __A, __m128d __B) {`。
- **L401 EN**: Returns from the current function with `((__m128d)vec_cmple((__v2df)__A, (__v2df)__B))`.
  **L401 CN**: 以 `((__m128d)vec_cmple((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L402 EN**: Closes the current lexical scope or compound statement.
  **L402 CN**: 结束当前词法作用域或复合语句块。
- **L403 EN**: Blank line separating nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L404 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L404 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L405 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L405 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L406 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnge_pd(__m128d __A, __m128d __B) {`.
  **L406 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnge_pd(__m128d __A, __m128d __B) {`。
- **L407 EN**: Returns from the current function with `((__m128d)vec_cmplt((__v2df)__A, (__v2df)__B))`.
  **L407 CN**: 以 `((__m128d)vec_cmplt((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L408 EN**: Closes the current lexical scope or compound statement.
  **L408 CN**: 结束当前词法作用域或复合语句块。

### Lines 409-432

````c

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpord_pd(__m128d __A, __m128d __B) {
  __v2du __c, __d;
  /* Compare against self will return false (0's) if NAN.  */
  __c = (__v2du)vec_cmpeq(__A, __A);
  __d = (__v2du)vec_cmpeq(__B, __B);
  /* A != NAN and B != NAN.  */
  return ((__m128d)vec_and(__c, __d));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpunord_pd(__m128d __A, __m128d __B) {
#if _ARCH_PWR8
  __v2du __c, __d;
  /* Compare against self will return false (0's) if NAN.  */
  __c = (__v2du)vec_cmpeq((__v2df)__A, (__v2df)__A);
  __d = (__v2du)vec_cmpeq((__v2df)__B, (__v2df)__B);
  /* A == NAN OR B == NAN converts too:
     NOT(A != NAN) OR NOT(B != NAN).  */
  __c = vec_nor(__c, __c);
  return ((__m128d)vec_orc(__c, __d));
````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L410 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L410 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L411 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L411 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L412 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpord_pd(__m128d __A, __m128d __B) {`.
  **L412 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpord_pd(__m128d __A, __m128d __B) {`。
- **L413 EN**: Adds a standalone statement or declaration: `__v2du __c, __d;`.
  **L413 CN**: 添加一条独立语句或声明：`__v2du __c, __d;`。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `Compare against self will return false (0's) if NAN.`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare against self will return false (0's) if NAN.`。
- **L415 EN**: Executes a call or declaration centered on `=`.
  **L415 CN**: 执行以 `=` 为核心的调用或声明。
- **L416 EN**: Executes a call or declaration centered on `=`.
  **L416 CN**: 执行以 `=` 为核心的调用或声明。
- **L417 EN**: Comment explains nearby logic, constraints, or intent: `A ! NAN and B ! NAN.`.
  **L417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A ! NAN and B ! NAN.`。
- **L418 EN**: Returns from the current function with `((__m128d)vec_and(__c, __d))`.
  **L418 CN**: 以 `((__m128d)vec_and(__c, __d))` 从当前函数返回。
- **L419 EN**: Closes the current lexical scope or compound statement.
  **L419 CN**: 结束当前词法作用域或复合语句块。
- **L420 EN**: Blank line separating nearby declarations or logic blocks.
  **L420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L421 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L421 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L422 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L422 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L423 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpunord_pd(__m128d __A, __m128d __B) {`.
  **L423 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpunord_pd(__m128d __A, __m128d __B) {`。
- **L424 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L424 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L425 EN**: Adds a standalone statement or declaration: `__v2du __c, __d;`.
  **L425 CN**: 添加一条独立语句或声明：`__v2du __c, __d;`。
- **L426 EN**: Comment explains nearby logic, constraints, or intent: `Compare against self will return false (0's) if NAN.`.
  **L426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare against self will return false (0's) if NAN.`。
- **L427 EN**: Executes a call or declaration centered on `=`.
  **L427 CN**: 执行以 `=` 为核心的调用或声明。
- **L428 EN**: Executes a call or declaration centered on `=`.
  **L428 CN**: 执行以 `=` 为核心的调用或声明。
- **L429 EN**: Comment explains nearby logic, constraints, or intent: `A NAN OR B NAN converts too:`.
  **L429 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A NAN OR B NAN converts too:`。
- **L430 EN**: Continues logic associated with callable symbol `NOT`.
  **L430 CN**: 继续与可调用符号 `NOT` 相关的逻辑。
- **L431 EN**: Executes a call or declaration centered on `vec_nor`.
  **L431 CN**: 执行以 `vec_nor` 为核心的调用或声明。
- **L432 EN**: Returns from the current function with `((__m128d)vec_orc(__c, __d))`.
  **L432 CN**: 以 `((__m128d)vec_orc(__c, __d))` 从当前函数返回。

### Lines 433-456

````c
#else
  __v2du __c, __d;
  /* Compare against self will return false (0's) if NAN.  */
  __c = (__v2du)vec_cmpeq((__v2df)__A, (__v2df)__A);
  __d = (__v2du)vec_cmpeq((__v2df)__B, (__v2df)__B);
  /* Convert the true ('1's) is NAN.  */
  __c = vec_nor(__c, __c);
  __d = vec_nor(__d, __d);
  return ((__m128d)vec_or(__c, __d));
#endif
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  /* PowerISA VSX does not allow partial (for just lower double)
     results. So to insure we don't generate spurious exceptions
     (from the upper double values) we splat the lower double
     before we do the operation. */
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  __c = (__v2df)vec_cmpeq(__a, __b);
  /* Then we merge the lower double result with the original upper
````
- **L433 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L433 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L434 EN**: Adds a standalone statement or declaration: `__v2du __c, __d;`.
  **L434 CN**: 添加一条独立语句或声明：`__v2du __c, __d;`。
- **L435 EN**: Comment explains nearby logic, constraints, or intent: `Compare against self will return false (0's) if NAN.`.
  **L435 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare against self will return false (0's) if NAN.`。
- **L436 EN**: Executes a call or declaration centered on `=`.
  **L436 CN**: 执行以 `=` 为核心的调用或声明。
- **L437 EN**: Executes a call or declaration centered on `=`.
  **L437 CN**: 执行以 `=` 为核心的调用或声明。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `Convert the true ('1's) is NAN.`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the true ('1's) is NAN.`。
- **L439 EN**: Executes a call or declaration centered on `vec_nor`.
  **L439 CN**: 执行以 `vec_nor` 为核心的调用或声明。
- **L440 EN**: Executes a call or declaration centered on `vec_nor`.
  **L440 CN**: 执行以 `vec_nor` 为核心的调用或声明。
- **L441 EN**: Returns from the current function with `((__m128d)vec_or(__c, __d))`.
  **L441 CN**: 以 `((__m128d)vec_or(__c, __d))` 从当前函数返回。
- **L442 EN**: Closes the current preprocessor conditional block.
  **L442 CN**: 结束当前预处理条件块。
- **L443 EN**: Closes the current lexical scope or compound statement.
  **L443 CN**: 结束当前词法作用域或复合语句块。
- **L444 EN**: Blank line separating nearby declarations or logic blocks.
  **L444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L445 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L445 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L446 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L446 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L447 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_sd(__m128d __A, __m128d __B) {`.
  **L447 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_sd(__m128d __A, __m128d __B) {`。
- **L448 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L448 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L449 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower double)`.
  **L449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower double)`。
- **L450 EN**: Continues the surrounding expression or declaration: `results. So to insure we don't generate spurious exceptions`.
  **L450 CN**: 继续构造周围的表达式或声明：`results. So to insure we don't generate spurious exceptions`。
- **L451 EN**: Continues the surrounding expression or declaration: `(from the upper double values) we splat the lower double`.
  **L451 CN**: 继续构造周围的表达式或声明：`(from the upper double values) we splat the lower double`。
- **L452 EN**: Continues the surrounding expression or declaration: `before we do the operation. */`.
  **L452 CN**: 继续构造周围的表达式或声明：`before we do the operation. */`。
- **L453 EN**: Executes a call or declaration centered on `vec_splats`.
  **L453 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L454 EN**: Executes a call or declaration centered on `vec_splats`.
  **L454 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L455 EN**: Executes a call or declaration centered on `=`.
  **L455 CN**: 执行以 `=` 为核心的调用或声明。
- **L456 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower double result with the original upper`.
  **L456 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower double result with the original upper`。

### Lines 457-480

````c
     double from __A.  */
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmplt_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  __c = (__v2df)vec_cmplt(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmple_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  __c = (__v2df)vec_cmple(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

````
- **L457 EN**: Continues the surrounding expression or declaration: `double from __A.  */`.
  **L457 CN**: 继续构造周围的表达式或声明：`double from __A.  */`。
- **L458 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L458 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L459 EN**: Closes the current lexical scope or compound statement.
  **L459 CN**: 结束当前词法作用域或复合语句块。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L461 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L461 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L462 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L462 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L463 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmplt_sd(__m128d __A, __m128d __B) {`.
  **L463 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmplt_sd(__m128d __A, __m128d __B) {`。
- **L464 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L464 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L465 EN**: Executes a call or declaration centered on `vec_splats`.
  **L465 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L466 EN**: Executes a call or declaration centered on `vec_splats`.
  **L466 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L467 EN**: Executes a call or declaration centered on `=`.
  **L467 CN**: 执行以 `=` 为核心的调用或声明。
- **L468 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L468 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L471 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L472 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L472 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L473 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmple_sd(__m128d __A, __m128d __B) {`.
  **L473 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmple_sd(__m128d __A, __m128d __B) {`。
- **L474 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L474 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L475 EN**: Executes a call or declaration centered on `vec_splats`.
  **L475 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L476 EN**: Executes a call or declaration centered on `vec_splats`.
  **L476 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L477 EN**: Executes a call or declaration centered on `=`.
  **L477 CN**: 执行以 `=` 为核心的调用或声明。
- **L478 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L478 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L479 EN**: Closes the current lexical scope or compound statement.
  **L479 CN**: 结束当前词法作用域或复合语句块。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 481-504

````c
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  __c = (__v2df)vec_cmpgt(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpge_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  __c = (__v2df)vec_cmpge(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpneq_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
````
- **L481 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L481 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L482 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L482 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L483 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_sd(__m128d __A, __m128d __B) {`.
  **L483 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_sd(__m128d __A, __m128d __B) {`。
- **L484 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L484 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L485 EN**: Executes a call or declaration centered on `vec_splats`.
  **L485 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L486 EN**: Executes a call or declaration centered on `vec_splats`.
  **L486 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L487 EN**: Executes a call or declaration centered on `=`.
  **L487 CN**: 执行以 `=` 为核心的调用或声明。
- **L488 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L488 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L489 EN**: Closes the current lexical scope or compound statement.
  **L489 CN**: 结束当前词法作用域或复合语句块。
- **L490 EN**: Blank line separating nearby declarations or logic blocks.
  **L490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L491 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L491 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L492 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L492 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L493 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpge_sd(__m128d __A, __m128d __B) {`.
  **L493 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpge_sd(__m128d __A, __m128d __B) {`。
- **L494 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L494 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L495 EN**: Executes a call or declaration centered on `vec_splats`.
  **L495 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L496 EN**: Executes a call or declaration centered on `vec_splats`.
  **L496 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L497 EN**: Executes a call or declaration centered on `=`.
  **L497 CN**: 执行以 `=` 为核心的调用或声明。
- **L498 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L498 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Blank line separating nearby declarations or logic blocks.
  **L500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L501 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L501 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L502 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L502 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L503 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpneq_sd(__m128d __A, __m128d __B) {`.
  **L503 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpneq_sd(__m128d __A, __m128d __B) {`。
- **L504 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L504 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。

### Lines 505-528

````c
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  __c = (__v2df)vec_cmpeq(__a, __b);
  __c = vec_nor(__c, __c);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnlt_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  /* Not less than is just greater than or equal.  */
  __c = (__v2df)vec_cmpge(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnle_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
````
- **L505 EN**: Executes a call or declaration centered on `vec_splats`.
  **L505 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L506 EN**: Executes a call or declaration centered on `vec_splats`.
  **L506 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L507 EN**: Executes a call or declaration centered on `=`.
  **L507 CN**: 执行以 `=` 为核心的调用或声明。
- **L508 EN**: Executes a call or declaration centered on `vec_nor`.
  **L508 CN**: 执行以 `vec_nor` 为核心的调用或声明。
- **L509 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L509 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L510 EN**: Closes the current lexical scope or compound statement.
  **L510 CN**: 结束当前词法作用域或复合语句块。
- **L511 EN**: Blank line separating nearby declarations or logic blocks.
  **L511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L512 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L512 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L513 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L513 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L514 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnlt_sd(__m128d __A, __m128d __B) {`.
  **L514 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnlt_sd(__m128d __A, __m128d __B) {`。
- **L515 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L515 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L516 EN**: Executes a call or declaration centered on `vec_splats`.
  **L516 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L517 EN**: Executes a call or declaration centered on `vec_splats`.
  **L517 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L518 EN**: Comment explains nearby logic, constraints, or intent: `Not less than is just greater than or equal.`.
  **L518 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Not less than is just greater than or equal.`。
- **L519 EN**: Executes a call or declaration centered on `=`.
  **L519 CN**: 执行以 `=` 为核心的调用或声明。
- **L520 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L520 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L521 EN**: Closes the current lexical scope or compound statement.
  **L521 CN**: 结束当前词法作用域或复合语句块。
- **L522 EN**: Blank line separating nearby declarations or logic blocks.
  **L522 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L523 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L523 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L524 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L524 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L525 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnle_sd(__m128d __A, __m128d __B) {`.
  **L525 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnle_sd(__m128d __A, __m128d __B) {`。
- **L526 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L526 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L527 EN**: Executes a call or declaration centered on `vec_splats`.
  **L527 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L528 EN**: Executes a call or declaration centered on `vec_splats`.
  **L528 CN**: 执行以 `vec_splats` 为核心的调用或声明。

### Lines 529-552

````c
  /* Not less than or equal is just greater than.  */
  __c = (__v2df)vec_cmpge(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpngt_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  /* Not greater than is just less than or equal.  */
  __c = (__v2df)vec_cmple(__a, __b);
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnge_sd(__m128d __A, __m128d __B) {
  __v2df __a, __b, __c;
  __a = vec_splats(__A[0]);
  __b = vec_splats(__B[0]);
  /* Not greater than or equal is just less than.  */
  __c = (__v2df)vec_cmplt(__a, __b);
````
- **L529 EN**: Comment explains nearby logic, constraints, or intent: `Not less than or equal is just greater than.`.
  **L529 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Not less than or equal is just greater than.`。
- **L530 EN**: Executes a call or declaration centered on `=`.
  **L530 CN**: 执行以 `=` 为核心的调用或声明。
- **L531 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L531 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L532 EN**: Closes the current lexical scope or compound statement.
  **L532 CN**: 结束当前词法作用域或复合语句块。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L534 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L534 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L535 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L535 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L536 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpngt_sd(__m128d __A, __m128d __B) {`.
  **L536 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpngt_sd(__m128d __A, __m128d __B) {`。
- **L537 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L537 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L538 EN**: Executes a call or declaration centered on `vec_splats`.
  **L538 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L539 EN**: Executes a call or declaration centered on `vec_splats`.
  **L539 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L540 EN**: Comment explains nearby logic, constraints, or intent: `Not greater than is just less than or equal.`.
  **L540 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Not greater than is just less than or equal.`。
- **L541 EN**: Executes a call or declaration centered on `=`.
  **L541 CN**: 执行以 `=` 为核心的调用或声明。
- **L542 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L542 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L545 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L545 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L546 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L546 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L547 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnge_sd(__m128d __A, __m128d __B) {`.
  **L547 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnge_sd(__m128d __A, __m128d __B) {`。
- **L548 EN**: Adds a standalone statement or declaration: `__v2df __a, __b, __c;`.
  **L548 CN**: 添加一条独立语句或声明：`__v2df __a, __b, __c;`。
- **L549 EN**: Executes a call or declaration centered on `vec_splats`.
  **L549 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L550 EN**: Executes a call or declaration centered on `vec_splats`.
  **L550 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L551 EN**: Comment explains nearby logic, constraints, or intent: `Not greater than or equal is just less than.`.
  **L551 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Not greater than or equal is just less than.`。
- **L552 EN**: Executes a call or declaration centered on `=`.
  **L552 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 553-576

````c
  return (__m128d)_mm_setr_pd(__c[0], __A[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpord_sd(__m128d __A, __m128d __B) {
  __v2df __r;
  __r = (__v2df)_mm_cmpord_pd(vec_splats(__A[0]), vec_splats(__B[0]));
  return (__m128d)_mm_setr_pd(__r[0], ((__v2df)__A)[1]);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpunord_sd(__m128d __A, __m128d __B) {
  __v2df __r;
  __r = _mm_cmpunord_pd(vec_splats(__A[0]), vec_splats(__B[0]));
  return (__m128d)_mm_setr_pd(__r[0], __A[1]);
}

/* FIXME
   The __mm_comi??_sd and __mm_ucomi??_sd implementations below are
   exactly the same because GCC for PowerPC only generates unordered
   compares (scalar and vector).
   Technically __mm_comieq_sp et all should be using the ordered
````
- **L553 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__c[0], __A[1])`.
  **L553 CN**: 以 `(__m128d)_mm_setr_pd(__c[0], __A[1])` 从当前函数返回。
- **L554 EN**: Closes the current lexical scope or compound statement.
  **L554 CN**: 结束当前词法作用域或复合语句块。
- **L555 EN**: Blank line separating nearby declarations or logic blocks.
  **L555 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L556 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L556 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L557 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L557 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L558 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpord_sd(__m128d __A, __m128d __B) {`.
  **L558 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpord_sd(__m128d __A, __m128d __B) {`。
- **L559 EN**: Adds a standalone statement or declaration: `__v2df __r;`.
  **L559 CN**: 添加一条独立语句或声明：`__v2df __r;`。
- **L560 EN**: Executes a call or declaration centered on `=`.
  **L560 CN**: 执行以 `=` 为核心的调用或声明。
- **L561 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__r[0], ((__v2df)__A)[1])`.
  **L561 CN**: 以 `(__m128d)_mm_setr_pd(__r[0], ((__v2df)__A)[1])` 从当前函数返回。
- **L562 EN**: Closes the current lexical scope or compound statement.
  **L562 CN**: 结束当前词法作用域或复合语句块。
- **L563 EN**: Blank line separating nearby declarations or logic blocks.
  **L563 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L564 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L564 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L565 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L565 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L566 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpunord_sd(__m128d __A, __m128d __B) {`.
  **L566 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpunord_sd(__m128d __A, __m128d __B) {`。
- **L567 EN**: Adds a standalone statement or declaration: `__v2df __r;`.
  **L567 CN**: 添加一条独立语句或声明：`__v2df __r;`。
- **L568 EN**: Executes a call or declaration centered on `_mm_cmpunord_pd`.
  **L568 CN**: 执行以 `_mm_cmpunord_pd` 为核心的调用或声明。
- **L569 EN**: Returns from the current function with `(__m128d)_mm_setr_pd(__r[0], __A[1])`.
  **L569 CN**: 以 `(__m128d)_mm_setr_pd(__r[0], __A[1])` 从当前函数返回。
- **L570 EN**: Closes the current lexical scope or compound statement.
  **L570 CN**: 结束当前词法作用域或复合语句块。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Comment records a pending task or caution: `FIXME`.
  **L572 CN**: 注释记录待办事项或注意点：`FIXME`。
- **L573 EN**: Continues the surrounding expression or declaration: `The __mm_comi??_sd and __mm_ucomi??_sd implementations below are`.
  **L573 CN**: 继续构造周围的表达式或声明：`The __mm_comi??_sd and __mm_ucomi??_sd implementations below are`。
- **L574 EN**: Continues the surrounding expression or declaration: `exactly the same because GCC for PowerPC only generates unordered`.
  **L574 CN**: 继续构造周围的表达式或声明：`exactly the same because GCC for PowerPC only generates unordered`。
- **L575 EN**: Continues logic associated with callable symbol `compares`.
  **L575 CN**: 继续与可调用符号 `compares` 相关的逻辑。
- **L576 EN**: Continues the surrounding expression or declaration: `Technically __mm_comieq_sp et all should be using the ordered`.
  **L576 CN**: 继续构造周围的表达式或声明：`Technically __mm_comieq_sp et all should be using the ordered`。

### Lines 577-600

````c
   compare and signal for QNaNs.  The __mm_ucomieq_sd et all should
   be OK.   */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comieq_sd(__m128d __A, __m128d __B) {
  return (__A[0] == __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comilt_sd(__m128d __A, __m128d __B) {
  return (__A[0] < __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comile_sd(__m128d __A, __m128d __B) {
  return (__A[0] <= __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comigt_sd(__m128d __A, __m128d __B) {
  return (__A[0] > __B[0]);
````
- **L577 EN**: Continues the surrounding expression or declaration: `compare and signal for QNaNs.  The __mm_ucomieq_sd et all should`.
  **L577 CN**: 继续构造周围的表达式或声明：`compare and signal for QNaNs.  The __mm_ucomieq_sd et all should`。
- **L578 EN**: Continues the surrounding expression or declaration: `be OK.   */`.
  **L578 CN**: 继续构造周围的表达式或声明：`be OK.   */`。
- **L579 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L579 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L580 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L580 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L581 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comieq_sd(__m128d __A, __m128d __B) {`.
  **L581 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comieq_sd(__m128d __A, __m128d __B) {`。
- **L582 EN**: Returns from the current function with `(__A[0] == __B[0])`.
  **L582 CN**: 以 `(__A[0] == __B[0])` 从当前函数返回。
- **L583 EN**: Closes the current lexical scope or compound statement.
  **L583 CN**: 结束当前词法作用域或复合语句块。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L585 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L585 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L586 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L586 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L587 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comilt_sd(__m128d __A, __m128d __B) {`.
  **L587 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comilt_sd(__m128d __A, __m128d __B) {`。
- **L588 EN**: Returns from the current function with `(__A[0] < __B[0])`.
  **L588 CN**: 以 `(__A[0] < __B[0])` 从当前函数返回。
- **L589 EN**: Closes the current lexical scope or compound statement.
  **L589 CN**: 结束当前词法作用域或复合语句块。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L591 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L591 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L592 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L592 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L593 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comile_sd(__m128d __A, __m128d __B) {`.
  **L593 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comile_sd(__m128d __A, __m128d __B) {`。
- **L594 EN**: Returns from the current function with `(__A[0] <= __B[0])`.
  **L594 CN**: 以 `(__A[0] <= __B[0])` 从当前函数返回。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L597 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L597 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L598 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L598 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L599 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comigt_sd(__m128d __A, __m128d __B) {`.
  **L599 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comigt_sd(__m128d __A, __m128d __B) {`。
- **L600 EN**: Returns from the current function with `(__A[0] > __B[0])`.
  **L600 CN**: 以 `(__A[0] > __B[0])` 从当前函数返回。

### Lines 601-624

````c
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comige_sd(__m128d __A, __m128d __B) {
  return (__A[0] >= __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comineq_sd(__m128d __A, __m128d __B) {
  return (__A[0] != __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomieq_sd(__m128d __A, __m128d __B) {
  return (__A[0] == __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomilt_sd(__m128d __A, __m128d __B) {
  return (__A[0] < __B[0]);
````
- **L601 EN**: Closes the current lexical scope or compound statement.
  **L601 CN**: 结束当前词法作用域或复合语句块。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L603 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L603 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L604 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L604 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L605 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comige_sd(__m128d __A, __m128d __B) {`.
  **L605 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comige_sd(__m128d __A, __m128d __B) {`。
- **L606 EN**: Returns from the current function with `(__A[0] >= __B[0])`.
  **L606 CN**: 以 `(__A[0] >= __B[0])` 从当前函数返回。
- **L607 EN**: Closes the current lexical scope or compound statement.
  **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L609 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L610 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L610 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L611 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comineq_sd(__m128d __A, __m128d __B) {`.
  **L611 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comineq_sd(__m128d __A, __m128d __B) {`。
- **L612 EN**: Returns from the current function with `(__A[0] != __B[0])`.
  **L612 CN**: 以 `(__A[0] != __B[0])` 从当前函数返回。
- **L613 EN**: Closes the current lexical scope or compound statement.
  **L613 CN**: 结束当前词法作用域或复合语句块。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L615 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L615 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L616 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L616 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L617 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomieq_sd(__m128d __A, __m128d __B) {`.
  **L617 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomieq_sd(__m128d __A, __m128d __B) {`。
- **L618 EN**: Returns from the current function with `(__A[0] == __B[0])`.
  **L618 CN**: 以 `(__A[0] == __B[0])` 从当前函数返回。
- **L619 EN**: Closes the current lexical scope or compound statement.
  **L619 CN**: 结束当前词法作用域或复合语句块。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L621 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L622 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L622 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L623 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomilt_sd(__m128d __A, __m128d __B) {`.
  **L623 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomilt_sd(__m128d __A, __m128d __B) {`。
- **L624 EN**: Returns from the current function with `(__A[0] < __B[0])`.
  **L624 CN**: 以 `(__A[0] < __B[0])` 从当前函数返回。

### Lines 625-648

````c
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomile_sd(__m128d __A, __m128d __B) {
  return (__A[0] <= __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomigt_sd(__m128d __A, __m128d __B) {
  return (__A[0] > __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomige_sd(__m128d __A, __m128d __B) {
  return (__A[0] >= __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomineq_sd(__m128d __A, __m128d __B) {
  return (__A[0] != __B[0]);
````
- **L625 EN**: Closes the current lexical scope or compound statement.
  **L625 CN**: 结束当前词法作用域或复合语句块。
- **L626 EN**: Blank line separating nearby declarations or logic blocks.
  **L626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L627 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L627 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L628 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L628 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L629 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomile_sd(__m128d __A, __m128d __B) {`.
  **L629 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomile_sd(__m128d __A, __m128d __B) {`。
- **L630 EN**: Returns from the current function with `(__A[0] <= __B[0])`.
  **L630 CN**: 以 `(__A[0] <= __B[0])` 从当前函数返回。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L633 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L634 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L634 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L635 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomigt_sd(__m128d __A, __m128d __B) {`.
  **L635 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomigt_sd(__m128d __A, __m128d __B) {`。
- **L636 EN**: Returns from the current function with `(__A[0] > __B[0])`.
  **L636 CN**: 以 `(__A[0] > __B[0])` 从当前函数返回。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L639 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L639 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L640 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L640 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L641 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomige_sd(__m128d __A, __m128d __B) {`.
  **L641 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomige_sd(__m128d __A, __m128d __B) {`。
- **L642 EN**: Returns from the current function with `(__A[0] >= __B[0])`.
  **L642 CN**: 以 `(__A[0] >= __B[0])` 从当前函数返回。
- **L643 EN**: Closes the current lexical scope or compound statement.
  **L643 CN**: 结束当前词法作用域或复合语句块。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L645 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L645 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L646 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L646 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L647 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomineq_sd(__m128d __A, __m128d __B) {`.
  **L647 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomineq_sd(__m128d __A, __m128d __B) {`。
- **L648 EN**: Returns from the current function with `(__A[0] != __B[0])`.
  **L648 CN**: 以 `(__A[0] != __B[0])` 从当前函数返回。

### Lines 649-672

````c
}

/* Create a vector of Qi, where i is the element number.  */
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_epi64x(long long __q1, long long __q0) {
  return __extension__(__m128i)(__v2di){__q0, __q1};
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_epi64(__m64 __q1, __m64 __q0) {
  return _mm_set_epi64x((long long)__q1, (long long)__q0);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_epi32(int __q3, int __q2, int __q1, int __q0) {
  return __extension__(__m128i)(__v4si){__q0, __q1, __q2, __q3};
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_epi16(short __q7, short __q6, short __q5, short __q4, short __q3,
````
- **L649 EN**: Closes the current lexical scope or compound statement.
  **L649 CN**: 结束当前词法作用域或复合语句块。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L651 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector of Qi, where i is the element number.`.
  **L651 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector of Qi, where i is the element number.`。
- **L652 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L652 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L653 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L653 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L654 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_epi64x(long long __q1, long long __q0) {`.
  **L654 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_epi64x(long long __q1, long long __q0) {`。
- **L655 EN**: Returns from the current function with `__extension__(__m128i)(__v2di){__q0, __q1}`.
  **L655 CN**: 以 `__extension__(__m128i)(__v2di){__q0, __q1}` 从当前函数返回。
- **L656 EN**: Closes the current lexical scope or compound statement.
  **L656 CN**: 结束当前词法作用域或复合语句块。
- **L657 EN**: Blank line separating nearby declarations or logic blocks.
  **L657 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L658 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L658 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L659 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L659 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L660 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_epi64(__m64 __q1, __m64 __q0) {`.
  **L660 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_epi64(__m64 __q1, __m64 __q0) {`。
- **L661 EN**: Returns from the current function with `_mm_set_epi64x((long long)__q1, (long long)__q0)`.
  **L661 CN**: 以 `_mm_set_epi64x((long long)__q1, (long long)__q0)` 从当前函数返回。
- **L662 EN**: Closes the current lexical scope or compound statement.
  **L662 CN**: 结束当前词法作用域或复合语句块。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L664 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L664 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L665 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L665 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L666 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_epi32(int __q3, int __q2, int __q1, int __q0) {`.
  **L666 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_epi32(int __q3, int __q2, int __q1, int __q0) {`。
- **L667 EN**: Returns from the current function with `__extension__(__m128i)(__v4si){__q0, __q1, __q2, __q3}`.
  **L667 CN**: 以 `__extension__(__m128i)(__v4si){__q0, __q1, __q2, __q3}` 从当前函数返回。
- **L668 EN**: Closes the current lexical scope or compound statement.
  **L668 CN**: 结束当前词法作用域或复合语句块。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L670 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L670 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L671 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L671 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_set_epi16(short __q7, short __q6, short __q5, short __q4, short __q3,`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_set_epi16(short __q7, short __q6, short __q5, short __q4, short __q3,`。

### Lines 673-696

````c
                  short __q2, short __q1, short __q0) {
  return __extension__(__m128i)(__v8hi){__q0, __q1, __q2, __q3,
                                        __q4, __q5, __q6, __q7};
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_epi8(char __q15, char __q14, char __q13, char __q12, char __q11,
                 char __q10, char __q09, char __q08, char __q07, char __q06,
                 char __q05, char __q04, char __q03, char __q02, char __q01,
                 char __q00) {
  return __extension__(__m128i)(__v16qi){
      __q00, __q01, __q02, __q03, __q04, __q05, __q06, __q07,
      __q08, __q09, __q10, __q11, __q12, __q13, __q14, __q15};
}

/* Set all of the elements of the vector to A.  */
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_epi64x(long long __A) {
  return _mm_set_epi64x(__A, __A);
}

extern __inline __m128i
````
- **L673 EN**: Continues the surrounding expression or declaration: `short __q2, short __q1, short __q0) {`.
  **L673 CN**: 继续构造周围的表达式或声明：`short __q2, short __q1, short __q0) {`。
- **L674 EN**: Returns from the current function with `__extension__(__m128i)(__v8hi){__q0, __q1, __q2, __q3,`.
  **L674 CN**: 以 `__extension__(__m128i)(__v8hi){__q0, __q1, __q2, __q3,` 从当前函数返回。
- **L675 EN**: Adds a standalone statement or declaration: `__q4, __q5, __q6, __q7};`.
  **L675 CN**: 添加一条独立语句或声明：`__q4, __q5, __q6, __q7};`。
- **L676 EN**: Closes the current lexical scope or compound statement.
  **L676 CN**: 结束当前词法作用域或复合语句块。
- **L677 EN**: Blank line separating nearby declarations or logic blocks.
  **L677 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L678 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L678 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L679 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L679 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_set_epi8(char __q15, char __q14, char __q13, char __q12, char __q11,`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_set_epi8(char __q15, char __q14, char __q13, char __q12, char __q11,`。
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `char __q10, char __q09, char __q08, char __q07, char __q06,`.
  **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`char __q10, char __q09, char __q08, char __q07, char __q06,`。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `char __q05, char __q04, char __q03, char __q02, char __q01,`.
  **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`char __q05, char __q04, char __q03, char __q02, char __q01,`。
- **L683 EN**: Continues the surrounding expression or declaration: `char __q00) {`.
  **L683 CN**: 继续构造周围的表达式或声明：`char __q00) {`。
- **L684 EN**: Returns from the current function with `__extension__(__m128i)(__v16qi){`.
  **L684 CN**: 以 `__extension__(__m128i)(__v16qi){` 从当前函数返回。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__q00, __q01, __q02, __q03, __q04, __q05, __q06, __q07,`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`__q00, __q01, __q02, __q03, __q04, __q05, __q06, __q07,`。
- **L686 EN**: Adds a standalone statement or declaration: `__q08, __q09, __q10, __q11, __q12, __q13, __q14, __q15};`.
  **L686 CN**: 添加一条独立语句或声明：`__q08, __q09, __q10, __q11, __q12, __q13, __q14, __q15};`。
- **L687 EN**: Closes the current lexical scope or compound statement.
  **L687 CN**: 结束当前词法作用域或复合语句块。
- **L688 EN**: Blank line separating nearby declarations or logic blocks.
  **L688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L689 EN**: Comment explains nearby logic, constraints, or intent: `Set all of the elements of the vector to A.`.
  **L689 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set all of the elements of the vector to A.`。
- **L690 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L690 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L691 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L691 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L692 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_epi64x(long long __A) {`.
  **L692 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_epi64x(long long __A) {`。
- **L693 EN**: Returns from the current function with `_mm_set_epi64x(__A, __A)`.
  **L693 CN**: 以 `_mm_set_epi64x(__A, __A)` 从当前函数返回。
- **L694 EN**: Closes the current lexical scope or compound statement.
  **L694 CN**: 结束当前词法作用域或复合语句块。
- **L695 EN**: Blank line separating nearby declarations or logic blocks.
  **L695 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L696 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L696 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。

### Lines 697-720

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_epi64(__m64 __A) {
  return _mm_set_epi64(__A, __A);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_epi32(int __A) {
  return _mm_set_epi32(__A, __A, __A, __A);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_epi16(short __A) {
  return _mm_set_epi16(__A, __A, __A, __A, __A, __A, __A, __A);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_epi8(char __A) {
  return _mm_set_epi8(__A, __A, __A, __A, __A, __A, __A, __A, __A, __A, __A,
                      __A, __A, __A, __A, __A);
}

````
- **L697 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L697 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L698 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_epi64(__m64 __A) {`.
  **L698 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_epi64(__m64 __A) {`。
- **L699 EN**: Returns from the current function with `_mm_set_epi64(__A, __A)`.
  **L699 CN**: 以 `_mm_set_epi64(__A, __A)` 从当前函数返回。
- **L700 EN**: Closes the current lexical scope or compound statement.
  **L700 CN**: 结束当前词法作用域或复合语句块。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L702 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L703 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L703 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L704 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_epi32(int __A) {`.
  **L704 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_epi32(int __A) {`。
- **L705 EN**: Returns from the current function with `_mm_set_epi32(__A, __A, __A, __A)`.
  **L705 CN**: 以 `_mm_set_epi32(__A, __A, __A, __A)` 从当前函数返回。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L708 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L708 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L709 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L709 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L710 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_epi16(short __A) {`.
  **L710 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_epi16(short __A) {`。
- **L711 EN**: Returns from the current function with `_mm_set_epi16(__A, __A, __A, __A, __A, __A, __A, __A)`.
  **L711 CN**: 以 `_mm_set_epi16(__A, __A, __A, __A, __A, __A, __A, __A)` 从当前函数返回。
- **L712 EN**: Closes the current lexical scope or compound statement.
  **L712 CN**: 结束当前词法作用域或复合语句块。
- **L713 EN**: Blank line separating nearby declarations or logic blocks.
  **L713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L714 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L714 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L715 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L715 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L716 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_epi8(char __A) {`.
  **L716 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_epi8(char __A) {`。
- **L717 EN**: Returns from the current function with `_mm_set_epi8(__A, __A, __A, __A, __A, __A, __A, __A, __A, __A, __A,`.
  **L717 CN**: 以 `_mm_set_epi8(__A, __A, __A, __A, __A, __A, __A, __A, __A, __A, __A,` 从当前函数返回。
- **L718 EN**: Adds a standalone statement or declaration: `__A, __A, __A, __A, __A);`.
  **L718 CN**: 添加一条独立语句或声明：`__A, __A, __A, __A, __A);`。
- **L719 EN**: Closes the current lexical scope or compound statement.
  **L719 CN**: 结束当前词法作用域或复合语句块。
- **L720 EN**: Blank line separating nearby declarations or logic blocks.
  **L720 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 721-744

````c
/* Create a vector of Qi, where i is the element number.
   The parameter order is reversed from the _mm_set_epi* functions.  */
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_epi64(__m64 __q0, __m64 __q1) {
  return _mm_set_epi64(__q1, __q0);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_epi32(int __q0, int __q1, int __q2, int __q3) {
  return _mm_set_epi32(__q3, __q2, __q1, __q0);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_epi16(short __q0, short __q1, short __q2, short __q3, short __q4,
                   short __q5, short __q6, short __q7) {
  return _mm_set_epi16(__q7, __q6, __q5, __q4, __q3, __q2, __q1, __q0);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_epi8(char __q00, char __q01, char __q02, char __q03, char __q04,
````
- **L721 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector of Qi, where i is the element number.`.
  **L721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector of Qi, where i is the element number.`。
- **L722 EN**: Continues the surrounding expression or declaration: `The parameter order is reversed from the _mm_set_epi* functions.  */`.
  **L722 CN**: 继续构造周围的表达式或声明：`The parameter order is reversed from the _mm_set_epi* functions.  */`。
- **L723 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L723 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L724 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L724 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L725 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setr_epi64(__m64 __q0, __m64 __q1) {`.
  **L725 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setr_epi64(__m64 __q0, __m64 __q1) {`。
- **L726 EN**: Returns from the current function with `_mm_set_epi64(__q1, __q0)`.
  **L726 CN**: 以 `_mm_set_epi64(__q1, __q0)` 从当前函数返回。
- **L727 EN**: Closes the current lexical scope or compound statement.
  **L727 CN**: 结束当前词法作用域或复合语句块。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L729 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L729 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L730 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L730 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L731 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setr_epi32(int __q0, int __q1, int __q2, int __q3) {`.
  **L731 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setr_epi32(int __q0, int __q1, int __q2, int __q3) {`。
- **L732 EN**: Returns from the current function with `_mm_set_epi32(__q3, __q2, __q1, __q0)`.
  **L732 CN**: 以 `_mm_set_epi32(__q3, __q2, __q1, __q0)` 从当前函数返回。
- **L733 EN**: Closes the current lexical scope or compound statement.
  **L733 CN**: 结束当前词法作用域或复合语句块。
- **L734 EN**: Blank line separating nearby declarations or logic blocks.
  **L734 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L735 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L735 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L736 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L736 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_setr_epi16(short __q0, short __q1, short __q2, short __q3, short __q4,`.
  **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_setr_epi16(short __q0, short __q1, short __q2, short __q3, short __q4,`。
- **L738 EN**: Continues the surrounding expression or declaration: `short __q5, short __q6, short __q7) {`.
  **L738 CN**: 继续构造周围的表达式或声明：`short __q5, short __q6, short __q7) {`。
- **L739 EN**: Returns from the current function with `_mm_set_epi16(__q7, __q6, __q5, __q4, __q3, __q2, __q1, __q0)`.
  **L739 CN**: 以 `_mm_set_epi16(__q7, __q6, __q5, __q4, __q3, __q2, __q1, __q0)` 从当前函数返回。
- **L740 EN**: Closes the current lexical scope or compound statement.
  **L740 CN**: 结束当前词法作用域或复合语句块。
- **L741 EN**: Blank line separating nearby declarations or logic blocks.
  **L741 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L742 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L742 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L743 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L743 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_setr_epi8(char __q00, char __q01, char __q02, char __q03, char __q04,`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_setr_epi8(char __q00, char __q01, char __q02, char __q03, char __q04,`。

### Lines 745-768

````c
                  char __q05, char __q06, char __q07, char __q08, char __q09,
                  char __q10, char __q11, char __q12, char __q13, char __q14,
                  char __q15) {
  return _mm_set_epi8(__q15, __q14, __q13, __q12, __q11, __q10, __q09, __q08,
                      __q07, __q06, __q05, __q04, __q03, __q02, __q01, __q00);
}

/* Create a vector with element 0 as *P and the rest zero.  */
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load_si128(__m128i const *__P) {
  return *__P;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadu_si128(__m128i_u const *__P) {
  return (__m128i)(vec_vsx_ld(0, (signed int const *)__P));
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadl_epi64(__m128i_u const *__P) {
  return _mm_set_epi64((__m64)0LL, *(__m64 *)__P);
````
- **L745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `char __q05, char __q06, char __q07, char __q08, char __q09,`.
  **L745 CN**: 继续一个多行参数列表、初始化器或聚合项：`char __q05, char __q06, char __q07, char __q08, char __q09,`。
- **L746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `char __q10, char __q11, char __q12, char __q13, char __q14,`.
  **L746 CN**: 继续一个多行参数列表、初始化器或聚合项：`char __q10, char __q11, char __q12, char __q13, char __q14,`。
- **L747 EN**: Continues the surrounding expression or declaration: `char __q15) {`.
  **L747 CN**: 继续构造周围的表达式或声明：`char __q15) {`。
- **L748 EN**: Returns from the current function with `_mm_set_epi8(__q15, __q14, __q13, __q12, __q11, __q10, __q09, __q08,`.
  **L748 CN**: 以 `_mm_set_epi8(__q15, __q14, __q13, __q12, __q11, __q10, __q09, __q08,` 从当前函数返回。
- **L749 EN**: Adds a standalone statement or declaration: `__q07, __q06, __q05, __q04, __q03, __q02, __q01, __q00);`.
  **L749 CN**: 添加一条独立语句或声明：`__q07, __q06, __q05, __q04, __q03, __q02, __q01, __q00);`。
- **L750 EN**: Closes the current lexical scope or compound statement.
  **L750 CN**: 结束当前词法作用域或复合语句块。
- **L751 EN**: Blank line separating nearby declarations or logic blocks.
  **L751 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L752 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with element 0 as *P and the rest zero.`.
  **L752 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with element 0 as *P and the rest zero.`。
- **L753 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L753 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L754 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L754 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L755 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load_si128(__m128i const *__P) {`.
  **L755 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load_si128(__m128i const *__P) {`。
- **L756 EN**: Returns from the current function with `*__P`.
  **L756 CN**: 以 `*__P` 从当前函数返回。
- **L757 EN**: Closes the current lexical scope or compound statement.
  **L757 CN**: 结束当前词法作用域或复合语句块。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L759 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L759 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L760 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L760 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L761 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadu_si128(__m128i_u const *__P) {`.
  **L761 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadu_si128(__m128i_u const *__P) {`。
- **L762 EN**: Returns from the current function with `(__m128i)(vec_vsx_ld(0, (signed int const *)__P))`.
  **L762 CN**: 以 `(__m128i)(vec_vsx_ld(0, (signed int const *)__P))` 从当前函数返回。
- **L763 EN**: Closes the current lexical scope or compound statement.
  **L763 CN**: 结束当前词法作用域或复合语句块。
- **L764 EN**: Blank line separating nearby declarations or logic blocks.
  **L764 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L765 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L765 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L766 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L766 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L767 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadl_epi64(__m128i_u const *__P) {`.
  **L767 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadl_epi64(__m128i_u const *__P) {`。
- **L768 EN**: Returns from the current function with `_mm_set_epi64((__m64)0LL, *(__m64 *)__P)`.
  **L768 CN**: 以 `_mm_set_epi64((__m64)0LL, *(__m64 *)__P)` 从当前函数返回。

### Lines 769-792

````c
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store_si128(__m128i *__P, __m128i __B) {
  vec_st((__v16qu)__B, 0, (__v16qu *)__P);
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storeu_si128(__m128i_u *__P, __m128i __B) {
  *__P = __B;
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storel_epi64(__m128i_u *__P, __m128i __B) {
  *(long long *)__P = ((__v2di)__B)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_movepi64_pi64(__m128i_u __B) {
  return (__m64)((__v2di)__B)[0];
````
- **L769 EN**: Closes the current lexical scope or compound statement.
  **L769 CN**: 结束当前词法作用域或复合语句块。
- **L770 EN**: Blank line separating nearby declarations or logic blocks.
  **L770 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L771 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L771 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L772 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L772 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L773 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store_si128(__m128i *__P, __m128i __B) {`.
  **L773 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store_si128(__m128i *__P, __m128i __B) {`。
- **L774 EN**: Executes a call or declaration centered on `vec_st`.
  **L774 CN**: 执行以 `vec_st` 为核心的调用或声明。
- **L775 EN**: Closes the current lexical scope or compound statement.
  **L775 CN**: 结束当前词法作用域或复合语句块。
- **L776 EN**: Blank line separating nearby declarations or logic blocks.
  **L776 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L777 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L777 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L778 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L778 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L779 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storeu_si128(__m128i_u *__P, __m128i __B) {`.
  **L779 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storeu_si128(__m128i_u *__P, __m128i __B) {`。
- **L780 EN**: Comment explains nearby logic, constraints, or intent: `__P __B;`.
  **L780 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__P __B;`。
- **L781 EN**: Closes the current lexical scope or compound statement.
  **L781 CN**: 结束当前词法作用域或复合语句块。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L783 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L783 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L784 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L784 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L785 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storel_epi64(__m128i_u *__P, __m128i __B) {`.
  **L785 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storel_epi64(__m128i_u *__P, __m128i __B) {`。
- **L786 EN**: Comment explains nearby logic, constraints, or intent: `(long long *)__P ((__v2di)__B)[0];`.
  **L786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(long long *)__P ((__v2di)__B)[0];`。
- **L787 EN**: Closes the current lexical scope or compound statement.
  **L787 CN**: 结束当前词法作用域或复合语句块。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L789 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L789 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L790 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L790 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L791 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movepi64_pi64(__m128i_u __B) {`.
  **L791 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movepi64_pi64(__m128i_u __B) {`。
- **L792 EN**: Returns from the current function with `(__m64)((__v2di)__B)[0]`.
  **L792 CN**: 以 `(__m64)((__v2di)__B)[0]` 从当前函数返回。

### Lines 793-816

````c
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_movpi64_epi64(__m64 __A) {
  return _mm_set_epi64((__m64)0LL, __A);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_move_epi64(__m128i __A) {
  return _mm_set_epi64((__m64)0LL, (__m64)__A[0]);
}

/* Create an undefined vector.  */
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_undefined_si128(void) {
  __m128i __Y = __Y;
  return __Y;
}

/* Create a vector of zeros.  */
extern __inline __m128i
````
- **L793 EN**: Closes the current lexical scope or compound statement.
  **L793 CN**: 结束当前词法作用域或复合语句块。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L795 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L795 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L796 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L796 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L797 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movpi64_epi64(__m64 __A) {`.
  **L797 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movpi64_epi64(__m64 __A) {`。
- **L798 EN**: Returns from the current function with `_mm_set_epi64((__m64)0LL, __A)`.
  **L798 CN**: 以 `_mm_set_epi64((__m64)0LL, __A)` 从当前函数返回。
- **L799 EN**: Closes the current lexical scope or compound statement.
  **L799 CN**: 结束当前词法作用域或复合语句块。
- **L800 EN**: Blank line separating nearby declarations or logic blocks.
  **L800 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L801 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L801 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L802 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L802 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L803 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_move_epi64(__m128i __A) {`.
  **L803 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_move_epi64(__m128i __A) {`。
- **L804 EN**: Returns from the current function with `_mm_set_epi64((__m64)0LL, (__m64)__A[0])`.
  **L804 CN**: 以 `_mm_set_epi64((__m64)0LL, (__m64)__A[0])` 从当前函数返回。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L807 EN**: Comment explains nearby logic, constraints, or intent: `Create an undefined vector.`.
  **L807 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create an undefined vector.`。
- **L808 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L808 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L809 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L809 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L810 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_undefined_si128(void) {`.
  **L810 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_undefined_si128(void) {`。
- **L811 EN**: Initializes variable `__Y` from the expression on the right-hand side.
  **L811 CN**: 使用右侧表达式初始化变量 `__Y`。
- **L812 EN**: Returns from the current function with `__Y`.
  **L812 CN**: 以 `__Y` 从当前函数返回。
- **L813 EN**: Closes the current lexical scope or compound statement.
  **L813 CN**: 结束当前词法作用域或复合语句块。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector of zeros.`.
  **L815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector of zeros.`。
- **L816 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L816 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。

### Lines 817-840

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setzero_si128(void) {
  return __extension__(__m128i)(__v4si){0, 0, 0, 0};
}

#ifdef _ARCH_PWR8
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepi32_pd(__m128i __A) {
  __v2di __val;
  /* For LE need to generate Vector Unpack Low Signed Word.
     Which is generated from unpackh.  */
  __val = (__v2di)vec_unpackh((__v4si)__A);

  return (__m128d)vec_ctf(__val, 0);
}
#endif

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepi32_ps(__m128i __A) {
  return ((__m128)vec_ctf((__v4si)__A, 0));
}

````
- **L817 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L817 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L818 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setzero_si128(void) {`.
  **L818 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setzero_si128(void) {`。
- **L819 EN**: Returns from the current function with `__extension__(__m128i)(__v4si){0, 0, 0, 0}`.
  **L819 CN**: 以 `__extension__(__m128i)(__v4si){0, 0, 0, 0}` 从当前函数返回。
- **L820 EN**: Closes the current lexical scope or compound statement.
  **L820 CN**: 结束当前词法作用域或复合语句块。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L822 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L822 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L823 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L823 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L824 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L824 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L825 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi32_pd(__m128i __A) {`.
  **L825 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi32_pd(__m128i __A) {`。
- **L826 EN**: Adds a standalone statement or declaration: `__v2di __val;`.
  **L826 CN**: 添加一条独立语句或声明：`__v2di __val;`。
- **L827 EN**: Comment explains nearby logic, constraints, or intent: `For LE need to generate Vector Unpack Low Signed Word.`.
  **L827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For LE need to generate Vector Unpack Low Signed Word.`。
- **L828 EN**: Continues the surrounding expression or declaration: `Which is generated from unpackh.  */`.
  **L828 CN**: 继续构造周围的表达式或声明：`Which is generated from unpackh.  */`。
- **L829 EN**: Executes a call or declaration centered on `=`.
  **L829 CN**: 执行以 `=` 为核心的调用或声明。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L831 EN**: Returns from the current function with `(__m128d)vec_ctf(__val, 0)`.
  **L831 CN**: 以 `(__m128d)vec_ctf(__val, 0)` 从当前函数返回。
- **L832 EN**: Closes the current lexical scope or compound statement.
  **L832 CN**: 结束当前词法作用域或复合语句块。
- **L833 EN**: Closes the current preprocessor conditional block.
  **L833 CN**: 结束当前预处理条件块。
- **L834 EN**: Blank line separating nearby declarations or logic blocks.
  **L834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L835 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L835 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L836 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L836 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L837 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi32_ps(__m128i __A) {`.
  **L837 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi32_ps(__m128i __A) {`。
- **L838 EN**: Returns from the current function with `((__m128)vec_ctf((__v4si)__A, 0))`.
  **L838 CN**: 以 `((__m128)vec_ctf((__v4si)__A, 0))` 从当前函数返回。
- **L839 EN**: Closes the current lexical scope or compound statement.
  **L839 CN**: 结束当前词法作用域或复合语句块。
- **L840 EN**: Blank line separating nearby declarations or logic blocks.
  **L840 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 841-864

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpd_epi32(__m128d __A) {
  __v2df __rounded = vec_rint(__A);
  __v4si __result, __temp;
  const __v4si __vzero = {0, 0, 0, 0};

  /* VSX Vector truncate Double-Precision to integer and Convert to
   Signed Integer Word format with Saturate.  */
  __asm__("xvcvdpsxws %x0,%x1" : "=wa"(__temp) : "wa"(__rounded) :);

#ifdef _ARCH_PWR8
#ifdef __LITTLE_ENDIAN__
  __temp = vec_mergeo(__temp, __temp);
#else
  __temp = vec_mergee(__temp, __temp);
#endif
  __result = (__v4si)vec_vpkudum((__vector long long)__temp,
                                 (__vector long long)__vzero);
#else
  {
    const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,
                              0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};
    __result = (__v4si)vec_perm((__v16qu)__temp, (__v16qu)__vzero, __pkperm);
````
- **L841 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L841 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L842 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L842 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L843 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpd_epi32(__m128d __A) {`.
  **L843 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpd_epi32(__m128d __A) {`。
- **L844 EN**: Initializes variable `__rounded` from the expression on the right-hand side.
  **L844 CN**: 使用右侧表达式初始化变量 `__rounded`。
- **L845 EN**: Adds a standalone statement or declaration: `__v4si __result, __temp;`.
  **L845 CN**: 添加一条独立语句或声明：`__v4si __result, __temp;`。
- **L846 EN**: Initializes variable `__vzero` from the expression on the right-hand side.
  **L846 CN**: 使用右侧表达式初始化变量 `__vzero`。
- **L847 EN**: Blank line separating nearby declarations or logic blocks.
  **L847 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L848 EN**: Comment explains nearby logic, constraints, or intent: `VSX Vector truncate Double-Precision to integer and Convert to`.
  **L848 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VSX Vector truncate Double-Precision to integer and Convert to`。
- **L849 EN**: Continues the surrounding expression or declaration: `Signed Integer Word format with Saturate.  */`.
  **L849 CN**: 继续构造周围的表达式或声明：`Signed Integer Word format with Saturate.  */`。
- **L850 EN**: Executes a call or declaration centered on `__asm__`.
  **L850 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L851 EN**: Blank line separating nearby declarations or logic blocks.
  **L851 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L852 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L852 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L853 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L853 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L854 EN**: Executes a call or declaration centered on `vec_mergeo`.
  **L854 CN**: 执行以 `vec_mergeo` 为核心的调用或声明。
- **L855 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L855 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L856 EN**: Executes a call or declaration centered on `vec_mergee`.
  **L856 CN**: 执行以 `vec_mergee` 为核心的调用或声明。
- **L857 EN**: Closes the current preprocessor conditional block.
  **L857 CN**: 结束当前预处理条件块。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__result = (__v4si)vec_vpkudum((__vector long long)__temp,`.
  **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`__result = (__v4si)vec_vpkudum((__vector long long)__temp,`。
- **L859 EN**: Executes a call or declaration centered on `statement`.
  **L859 CN**: 执行以 `statement` 为核心的调用或声明。
- **L860 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L860 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L861 EN**: Opens a new lexical scope or compound statement.
  **L861 CN**: 打开一个新的词法作用域或复合语句块。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,`。
- **L863 EN**: Adds a standalone statement or declaration: `0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};`.
  **L863 CN**: 添加一条独立语句或声明：`0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};`。
- **L864 EN**: Executes a call or declaration centered on `=`.
  **L864 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 865-888

````c
  }
#endif
  return (__m128i)__result;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpd_pi32(__m128d __A) {
  __m128i __result = _mm_cvtpd_epi32(__A);

  return (__m64)__result[0];
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpd_ps(__m128d __A) {
  __v4sf __result;
  __v4si __temp;
  const __v4si __vzero = {0, 0, 0, 0};

  __asm__("xvcvdpsp %x0,%x1" : "=wa"(__temp) : "wa"(__A) :);

#ifdef _ARCH_PWR8
#ifdef __LITTLE_ENDIAN__
````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Closes the current preprocessor conditional block.
  **L866 CN**: 结束当前预处理条件块。
- **L867 EN**: Returns from the current function with `(__m128i)__result`.
  **L867 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L868 EN**: Closes the current lexical scope or compound statement.
  **L868 CN**: 结束当前词法作用域或复合语句块。
- **L869 EN**: Blank line separating nearby declarations or logic blocks.
  **L869 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L870 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L870 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L871 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L871 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L872 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpd_pi32(__m128d __A) {`.
  **L872 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpd_pi32(__m128d __A) {`。
- **L873 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L873 CN**: 使用右侧表达式初始化变量 `__result`。
- **L874 EN**: Blank line separating nearby declarations or logic blocks.
  **L874 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L875 EN**: Returns from the current function with `(__m64)__result[0]`.
  **L875 CN**: 以 `(__m64)__result[0]` 从当前函数返回。
- **L876 EN**: Closes the current lexical scope or compound statement.
  **L876 CN**: 结束当前词法作用域或复合语句块。
- **L877 EN**: Blank line separating nearby declarations or logic blocks.
  **L877 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L878 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L878 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L879 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L879 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L880 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpd_ps(__m128d __A) {`.
  **L880 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpd_ps(__m128d __A) {`。
- **L881 EN**: Adds a standalone statement or declaration: `__v4sf __result;`.
  **L881 CN**: 添加一条独立语句或声明：`__v4sf __result;`。
- **L882 EN**: Adds a standalone statement or declaration: `__v4si __temp;`.
  **L882 CN**: 添加一条独立语句或声明：`__v4si __temp;`。
- **L883 EN**: Initializes variable `__vzero` from the expression on the right-hand side.
  **L883 CN**: 使用右侧表达式初始化变量 `__vzero`。
- **L884 EN**: Blank line separating nearby declarations or logic blocks.
  **L884 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L885 EN**: Executes a call or declaration centered on `__asm__`.
  **L885 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L886 EN**: Blank line separating nearby declarations or logic blocks.
  **L886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L887 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L887 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L888 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L888 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。

### Lines 889-912

````c
  __temp = vec_mergeo(__temp, __temp);
#else
  __temp = vec_mergee(__temp, __temp);
#endif
  __result = (__v4sf)vec_vpkudum((__vector long long)__temp,
                                 (__vector long long)__vzero);
#else
  {
    const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,
                              0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};
    __result = (__v4sf)vec_perm((__v16qu)__temp, (__v16qu)__vzero, __pkperm);
  }
#endif
  return ((__m128)__result);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttpd_epi32(__m128d __A) {
  __v4si __result;
  __v4si __temp;
  const __v4si __vzero = {0, 0, 0, 0};

  /* VSX Vector truncate Double-Precision to integer and Convert to
````
- **L889 EN**: Executes a call or declaration centered on `vec_mergeo`.
  **L889 CN**: 执行以 `vec_mergeo` 为核心的调用或声明。
- **L890 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L890 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L891 EN**: Executes a call or declaration centered on `vec_mergee`.
  **L891 CN**: 执行以 `vec_mergee` 为核心的调用或声明。
- **L892 EN**: Closes the current preprocessor conditional block.
  **L892 CN**: 结束当前预处理条件块。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__result = (__v4sf)vec_vpkudum((__vector long long)__temp,`.
  **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`__result = (__v4sf)vec_vpkudum((__vector long long)__temp,`。
- **L894 EN**: Executes a call or declaration centered on `statement`.
  **L894 CN**: 执行以 `statement` 为核心的调用或声明。
- **L895 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L895 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L896 EN**: Opens a new lexical scope or compound statement.
  **L896 CN**: 打开一个新的词法作用域或复合语句块。
- **L897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,`.
  **L897 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,`。
- **L898 EN**: Adds a standalone statement or declaration: `0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};`.
  **L898 CN**: 添加一条独立语句或声明：`0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};`。
- **L899 EN**: Executes a call or declaration centered on `=`.
  **L899 CN**: 执行以 `=` 为核心的调用或声明。
- **L900 EN**: Closes the current lexical scope or compound statement.
  **L900 CN**: 结束当前词法作用域或复合语句块。
- **L901 EN**: Closes the current preprocessor conditional block.
  **L901 CN**: 结束当前预处理条件块。
- **L902 EN**: Returns from the current function with `((__m128)__result)`.
  **L902 CN**: 以 `((__m128)__result)` 从当前函数返回。
- **L903 EN**: Closes the current lexical scope or compound statement.
  **L903 CN**: 结束当前词法作用域或复合语句块。
- **L904 EN**: Blank line separating nearby declarations or logic blocks.
  **L904 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L905 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L905 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L906 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L906 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L907 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttpd_epi32(__m128d __A) {`.
  **L907 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttpd_epi32(__m128d __A) {`。
- **L908 EN**: Adds a standalone statement or declaration: `__v4si __result;`.
  **L908 CN**: 添加一条独立语句或声明：`__v4si __result;`。
- **L909 EN**: Adds a standalone statement or declaration: `__v4si __temp;`.
  **L909 CN**: 添加一条独立语句或声明：`__v4si __temp;`。
- **L910 EN**: Initializes variable `__vzero` from the expression on the right-hand side.
  **L910 CN**: 使用右侧表达式初始化变量 `__vzero`。
- **L911 EN**: Blank line separating nearby declarations or logic blocks.
  **L911 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L912 EN**: Comment explains nearby logic, constraints, or intent: `VSX Vector truncate Double-Precision to integer and Convert to`.
  **L912 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VSX Vector truncate Double-Precision to integer and Convert to`。

### Lines 913-936

````c
   Signed Integer Word format with Saturate.  */
  __asm__("xvcvdpsxws %x0,%x1" : "=wa"(__temp) : "wa"(__A) :);

#ifdef _ARCH_PWR8
#ifdef __LITTLE_ENDIAN__
  __temp = vec_mergeo(__temp, __temp);
#else
  __temp = vec_mergee(__temp, __temp);
#endif
  __result = (__v4si)vec_vpkudum((__vector long long)__temp,
                                 (__vector long long)__vzero);
#else
  {
    const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,
                              0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};
    __result = (__v4si)vec_perm((__v16qu)__temp, (__v16qu)__vzero, __pkperm);
  }
#endif

  return ((__m128i)__result);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L913 EN**: Continues the surrounding expression or declaration: `Signed Integer Word format with Saturate.  */`.
  **L913 CN**: 继续构造周围的表达式或声明：`Signed Integer Word format with Saturate.  */`。
- **L914 EN**: Executes a call or declaration centered on `__asm__`.
  **L914 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L915 EN**: Blank line separating nearby declarations or logic blocks.
  **L915 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L916 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L916 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L917 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L917 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L918 EN**: Executes a call or declaration centered on `vec_mergeo`.
  **L918 CN**: 执行以 `vec_mergeo` 为核心的调用或声明。
- **L919 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L919 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L920 EN**: Executes a call or declaration centered on `vec_mergee`.
  **L920 CN**: 执行以 `vec_mergee` 为核心的调用或声明。
- **L921 EN**: Closes the current preprocessor conditional block.
  **L921 CN**: 结束当前预处理条件块。
- **L922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__result = (__v4si)vec_vpkudum((__vector long long)__temp,`.
  **L922 CN**: 继续一个多行参数列表、初始化器或聚合项：`__result = (__v4si)vec_vpkudum((__vector long long)__temp,`。
- **L923 EN**: Executes a call or declaration centered on `statement`.
  **L923 CN**: 执行以 `statement` 为核心的调用或声明。
- **L924 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L924 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L925 EN**: Opens a new lexical scope or compound statement.
  **L925 CN**: 打开一个新的词法作用域或复合语句块。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`const __v16qu __pkperm = {0x00, 0x01, 0x02, 0x03, 0x08, 0x09, 0x0a, 0x0b,`。
- **L927 EN**: Adds a standalone statement or declaration: `0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};`.
  **L927 CN**: 添加一条独立语句或声明：`0x14, 0x15, 0x16, 0x17, 0x1c, 0x1d, 0x1e, 0x1f};`。
- **L928 EN**: Executes a call or declaration centered on `=`.
  **L928 CN**: 执行以 `=` 为核心的调用或声明。
- **L929 EN**: Closes the current lexical scope or compound statement.
  **L929 CN**: 结束当前词法作用域或复合语句块。
- **L930 EN**: Closes the current preprocessor conditional block.
  **L930 CN**: 结束当前预处理条件块。
- **L931 EN**: Blank line separating nearby declarations or logic blocks.
  **L931 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L932 EN**: Returns from the current function with `((__m128i)__result)`.
  **L932 CN**: 以 `((__m128i)__result)` 从当前函数返回。
- **L933 EN**: Closes the current lexical scope or compound statement.
  **L933 CN**: 结束当前词法作用域或复合语句块。
- **L934 EN**: Blank line separating nearby declarations or logic blocks.
  **L934 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L935 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L935 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L936 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L936 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 937-960

````c
    _mm_cvttpd_pi32(__m128d __A) {
  __m128i __result = _mm_cvttpd_epi32(__A);

  return (__m64)__result[0];
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi128_si32(__m128i __A) {
  return ((__v4si)__A)[0];
}

#ifdef _ARCH_PWR8
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpi32_pd(__m64 __A) {
  __v4si __temp;
  __v2di __tmp2;
  __v4f __result;

  __temp = (__v4si)vec_splats(__A);
  __tmp2 = (__v2di)vec_unpackl(__temp);
  __result = vec_ctf((__vector signed long long)__tmp2, 0);
  return (__m128d)__result;
````
- **L937 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttpd_pi32(__m128d __A) {`.
  **L937 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttpd_pi32(__m128d __A) {`。
- **L938 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L938 CN**: 使用右侧表达式初始化变量 `__result`。
- **L939 EN**: Blank line separating nearby declarations or logic blocks.
  **L939 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L940 EN**: Returns from the current function with `(__m64)__result[0]`.
  **L940 CN**: 以 `(__m64)__result[0]` 从当前函数返回。
- **L941 EN**: Closes the current lexical scope or compound statement.
  **L941 CN**: 结束当前词法作用域或复合语句块。
- **L942 EN**: Blank line separating nearby declarations or logic blocks.
  **L942 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L943 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L943 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L944 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L944 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L945 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi128_si32(__m128i __A) {`.
  **L945 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi128_si32(__m128i __A) {`。
- **L946 EN**: Returns from the current function with `((__v4si)__A)[0]`.
  **L946 CN**: 以 `((__v4si)__A)[0]` 从当前函数返回。
- **L947 EN**: Closes the current lexical scope or compound statement.
  **L947 CN**: 结束当前词法作用域或复合语句块。
- **L948 EN**: Blank line separating nearby declarations or logic blocks.
  **L948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L949 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L949 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L950 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L950 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L951 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L951 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpi32_pd(__m64 __A) {`.
  **L952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpi32_pd(__m64 __A) {`。
- **L953 EN**: Adds a standalone statement or declaration: `__v4si __temp;`.
  **L953 CN**: 添加一条独立语句或声明：`__v4si __temp;`。
- **L954 EN**: Adds a standalone statement or declaration: `__v2di __tmp2;`.
  **L954 CN**: 添加一条独立语句或声明：`__v2di __tmp2;`。
- **L955 EN**: Adds a standalone statement or declaration: `__v4f __result;`.
  **L955 CN**: 添加一条独立语句或声明：`__v4f __result;`。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L957 EN**: Executes a call or declaration centered on `=`.
  **L957 CN**: 执行以 `=` 为核心的调用或声明。
- **L958 EN**: Executes a call or declaration centered on `=`.
  **L958 CN**: 执行以 `=` 为核心的调用或声明。
- **L959 EN**: Executes a call or declaration centered on `vec_ctf`.
  **L959 CN**: 执行以 `vec_ctf` 为核心的调用或声明。
- **L960 EN**: Returns from the current function with `(__m128d)__result`.
  **L960 CN**: 以 `(__m128d)__result` 从当前函数返回。

### Lines 961-984

````c
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtps_epi32(__m128 __A) {
  __v4sf __rounded;
  __v4si __result;

  __rounded = vec_rint((__v4sf)__A);
  __result = vec_cts(__rounded, 0);
  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttps_epi32(__m128 __A) {
  __v4si __result;

  __result = vec_cts((__v4sf)__A, 0);
  return (__m128i)__result;
}

extern __inline __m128d
````
- **L961 EN**: Closes the current lexical scope or compound statement.
  **L961 CN**: 结束当前词法作用域或复合语句块。
- **L962 EN**: Closes the current preprocessor conditional block.
  **L962 CN**: 结束当前预处理条件块。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L964 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L964 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L965 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L965 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L966 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtps_epi32(__m128 __A) {`.
  **L966 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtps_epi32(__m128 __A) {`。
- **L967 EN**: Adds a standalone statement or declaration: `__v4sf __rounded;`.
  **L967 CN**: 添加一条独立语句或声明：`__v4sf __rounded;`。
- **L968 EN**: Adds a standalone statement or declaration: `__v4si __result;`.
  **L968 CN**: 添加一条独立语句或声明：`__v4si __result;`。
- **L969 EN**: Blank line separating nearby declarations or logic blocks.
  **L969 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L970 EN**: Executes a call or declaration centered on `vec_rint`.
  **L970 CN**: 执行以 `vec_rint` 为核心的调用或声明。
- **L971 EN**: Executes a call or declaration centered on `vec_cts`.
  **L971 CN**: 执行以 `vec_cts` 为核心的调用或声明。
- **L972 EN**: Returns from the current function with `(__m128i)__result`.
  **L972 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L973 EN**: Closes the current lexical scope or compound statement.
  **L973 CN**: 结束当前词法作用域或复合语句块。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L975 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L976 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L976 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L977 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttps_epi32(__m128 __A) {`.
  **L977 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttps_epi32(__m128 __A) {`。
- **L978 EN**: Adds a standalone statement or declaration: `__v4si __result;`.
  **L978 CN**: 添加一条独立语句或声明：`__v4si __result;`。
- **L979 EN**: Blank line separating nearby declarations or logic blocks.
  **L979 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L980 EN**: Executes a call or declaration centered on `vec_cts`.
  **L980 CN**: 执行以 `vec_cts` 为核心的调用或声明。
- **L981 EN**: Returns from the current function with `(__m128i)__result`.
  **L981 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L982 EN**: Closes the current lexical scope or compound statement.
  **L982 CN**: 结束当前词法作用域或复合语句块。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L984 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L984 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。

### Lines 985-1008

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtps_pd(__m128 __A) {
  /* Check if vec_doubleh is defined by <altivec.h>. If so use that. */
#ifdef vec_doubleh
  return (__m128d)vec_doubleh((__v4sf)__A);
#else
  /* Otherwise the compiler is not current and so need to generate the
     equivalent code.  */
  __v4sf __a = (__v4sf)__A;
  __v4sf __temp;
  __v2df __result;
#ifdef __LITTLE_ENDIAN__
  /* The input float values are in elements {[0], [1]} but the convert
     instruction needs them in elements {[1], [3]}, So we use two
     shift left double vector word immediates to get the elements
     lined up.  */
  __temp = __builtin_vsx_xxsldwi(__a, __a, 3);
  __temp = __builtin_vsx_xxsldwi(__a, __temp, 2);
#else
  /* The input float values are in elements {[0], [1]} but the convert
     instruction needs them in elements {[0], [2]}, So we use two
     shift left double vector word immediates to get the elements
     lined up.  */
  __temp = vec_vmrghw(__a, __a);
````
- **L985 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L985 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L986 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtps_pd(__m128 __A) {`.
  **L986 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtps_pd(__m128 __A) {`。
- **L987 EN**: Comment explains nearby logic, constraints, or intent: `Check if vec_doubleh is defined by <altivec.h>. If so use that.`.
  **L987 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Check if vec_doubleh is defined by <altivec.h>. If so use that.`。
- **L988 EN**: Starts a preprocessor conditional block: `#ifdef vec_doubleh`.
  **L988 CN**: 开始一个预处理条件块：`#ifdef vec_doubleh`。
- **L989 EN**: Returns from the current function with `(__m128d)vec_doubleh((__v4sf)__A)`.
  **L989 CN**: 以 `(__m128d)vec_doubleh((__v4sf)__A)` 从当前函数返回。
- **L990 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L990 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L991 EN**: Comment explains nearby logic, constraints, or intent: `Otherwise the compiler is not current and so need to generate the`.
  **L991 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Otherwise the compiler is not current and so need to generate the`。
- **L992 EN**: Continues the surrounding expression or declaration: `equivalent code.  */`.
  **L992 CN**: 继续构造周围的表达式或声明：`equivalent code.  */`。
- **L993 EN**: Initializes variable `__a` from the expression on the right-hand side.
  **L993 CN**: 使用右侧表达式初始化变量 `__a`。
- **L994 EN**: Adds a standalone statement or declaration: `__v4sf __temp;`.
  **L994 CN**: 添加一条独立语句或声明：`__v4sf __temp;`。
- **L995 EN**: Adds a standalone statement or declaration: `__v2df __result;`.
  **L995 CN**: 添加一条独立语句或声明：`__v2df __result;`。
- **L996 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L996 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L997 EN**: Comment explains nearby logic, constraints, or intent: `The input float values are in elements {[0], [1]} but the convert`.
  **L997 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The input float values are in elements {[0], [1]} but the convert`。
- **L998 EN**: Continues the surrounding expression or declaration: `instruction needs them in elements {[1], [3]}, So we use two`.
  **L998 CN**: 继续构造周围的表达式或声明：`instruction needs them in elements {[1], [3]}, So we use two`。
- **L999 EN**: Continues the surrounding expression or declaration: `shift left double vector word immediates to get the elements`.
  **L999 CN**: 继续构造周围的表达式或声明：`shift left double vector word immediates to get the elements`。
- **L1000 EN**: Continues the surrounding expression or declaration: `lined up.  */`.
  **L1000 CN**: 继续构造周围的表达式或声明：`lined up.  */`。
- **L1001 EN**: Executes a call or declaration centered on `__builtin_vsx_xxsldwi`.
  **L1001 CN**: 执行以 `__builtin_vsx_xxsldwi` 为核心的调用或声明。
- **L1002 EN**: Executes a call or declaration centered on `__builtin_vsx_xxsldwi`.
  **L1002 CN**: 执行以 `__builtin_vsx_xxsldwi` 为核心的调用或声明。
- **L1003 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1003 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1004 EN**: Comment explains nearby logic, constraints, or intent: `The input float values are in elements {[0], [1]} but the convert`.
  **L1004 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The input float values are in elements {[0], [1]} but the convert`。
- **L1005 EN**: Continues the surrounding expression or declaration: `instruction needs them in elements {[0], [2]}, So we use two`.
  **L1005 CN**: 继续构造周围的表达式或声明：`instruction needs them in elements {[0], [2]}, So we use two`。
- **L1006 EN**: Continues the surrounding expression or declaration: `shift left double vector word immediates to get the elements`.
  **L1006 CN**: 继续构造周围的表达式或声明：`shift left double vector word immediates to get the elements`。
- **L1007 EN**: Continues the surrounding expression or declaration: `lined up.  */`.
  **L1007 CN**: 继续构造周围的表达式或声明：`lined up.  */`。
- **L1008 EN**: Executes a call or declaration centered on `vec_vmrghw`.
  **L1008 CN**: 执行以 `vec_vmrghw` 为核心的调用或声明。

### Lines 1009-1032

````c
#endif
  __asm__(" xvcvspdp %x0,%x1" : "=wa"(__result) : "wa"(__temp) :);
  return (__m128d)__result;
#endif
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsd_si32(__m128d __A) {
  __v2df __rounded = vec_rint((__v2df)__A);
  int __result = ((__v2df)__rounded)[0];

  return __result;
}
/* Intel intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsd_si64(__m128d __A) {
  __v2df __rounded = vec_rint((__v2df)__A);
  long long __result = ((__v2df)__rounded)[0];

  return __result;
}

````
- **L1009 EN**: Closes the current preprocessor conditional block.
  **L1009 CN**: 结束当前预处理条件块。
- **L1010 EN**: Executes a call or declaration centered on `__asm__`.
  **L1010 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L1011 EN**: Returns from the current function with `(__m128d)__result`.
  **L1011 CN**: 以 `(__m128d)__result` 从当前函数返回。
- **L1012 EN**: Closes the current preprocessor conditional block.
  **L1012 CN**: 结束当前预处理条件块。
- **L1013 EN**: Closes the current lexical scope or compound statement.
  **L1013 CN**: 结束当前词法作用域或复合语句块。
- **L1014 EN**: Blank line separating nearby declarations or logic blocks.
  **L1014 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1015 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1015 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1016 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1016 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1017 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsd_si32(__m128d __A) {`.
  **L1017 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsd_si32(__m128d __A) {`。
- **L1018 EN**: Initializes variable `__rounded` from the expression on the right-hand side.
  **L1018 CN**: 使用右侧表达式初始化变量 `__rounded`。
- **L1019 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1019 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1020 EN**: Blank line separating nearby declarations or logic blocks.
  **L1020 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1021 EN**: Returns from the current function with `__result`.
  **L1021 CN**: 以 `__result` 从当前函数返回。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L1023 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L1024 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L1024 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L1025 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1025 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1026 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsd_si64(__m128d __A) {`.
  **L1026 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsd_si64(__m128d __A) {`。
- **L1027 EN**: Initializes variable `__rounded` from the expression on the right-hand side.
  **L1027 CN**: 使用右侧表达式初始化变量 `__rounded`。
- **L1028 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1028 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1029 EN**: Blank line separating nearby declarations or logic blocks.
  **L1029 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1030 EN**: Returns from the current function with `__result`.
  **L1030 CN**: 以 `__result` 从当前函数返回。
- **L1031 EN**: Closes the current lexical scope or compound statement.
  **L1031 CN**: 结束当前词法作用域或复合语句块。
- **L1032 EN**: Blank line separating nearby declarations or logic blocks.
  **L1032 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1033-1056

````c
/* Microsoft intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsd_si64x(__m128d __A) {
  return _mm_cvtsd_si64((__v2df)__A);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttsd_si32(__m128d __A) {
  int __result = ((__v2df)__A)[0];

  return __result;
}

/* Intel intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttsd_si64(__m128d __A) {
  long long __result = ((__v2df)__A)[0];

  return __result;
}

````
- **L1033 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L1033 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L1034 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L1034 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L1035 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1035 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1036 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsd_si64x(__m128d __A) {`.
  **L1036 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsd_si64x(__m128d __A) {`。
- **L1037 EN**: Returns from the current function with `_mm_cvtsd_si64((__v2df)__A)`.
  **L1037 CN**: 以 `_mm_cvtsd_si64((__v2df)__A)` 从当前函数返回。
- **L1038 EN**: Closes the current lexical scope or compound statement.
  **L1038 CN**: 结束当前词法作用域或复合语句块。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1040 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1040 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1041 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1041 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1042 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttsd_si32(__m128d __A) {`.
  **L1042 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttsd_si32(__m128d __A) {`。
- **L1043 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1043 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1044 EN**: Blank line separating nearby declarations or logic blocks.
  **L1044 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1045 EN**: Returns from the current function with `__result`.
  **L1045 CN**: 以 `__result` 从当前函数返回。
- **L1046 EN**: Closes the current lexical scope or compound statement.
  **L1046 CN**: 结束当前词法作用域或复合语句块。
- **L1047 EN**: Blank line separating nearby declarations or logic blocks.
  **L1047 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1048 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L1048 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L1049 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L1049 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L1050 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1050 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1051 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttsd_si64(__m128d __A) {`.
  **L1051 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttsd_si64(__m128d __A) {`。
- **L1052 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1052 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1053 EN**: Blank line separating nearby declarations or logic blocks.
  **L1053 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1054 EN**: Returns from the current function with `__result`.
  **L1054 CN**: 以 `__result` 从当前函数返回。
- **L1055 EN**: Closes the current lexical scope or compound statement.
  **L1055 CN**: 结束当前词法作用域或复合语句块。
- **L1056 EN**: Blank line separating nearby declarations or logic blocks.
  **L1056 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1057-1080

````c
/* Microsoft intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttsd_si64x(__m128d __A) {
  return _mm_cvttsd_si64(__A);
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsd_ss(__m128 __A, __m128d __B) {
  __v4sf __result = (__v4sf)__A;

#ifdef __LITTLE_ENDIAN__
  __v4sf __temp_s;
  /* Copy double element[0] to element [1] for conversion.  */
  __v2df __temp_b = vec_splat((__v2df)__B, 0);

  /* Pre-rotate __A left 3 (logically right 1) elements.  */
  __result = __builtin_vsx_xxsldwi(__result, __result, 3);
  /* Convert double to single float scalar in a vector.  */
  __asm__("xscvdpsp %x0,%x1" : "=wa"(__temp_s) : "wa"(__temp_b) :);
  /* Shift the resulting scalar into vector element [0].  */
  __result = __builtin_vsx_xxsldwi(__result, __temp_s, 1);
#else
````
- **L1057 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L1057 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L1058 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L1058 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L1059 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1059 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1060 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttsd_si64x(__m128d __A) {`.
  **L1060 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttsd_si64x(__m128d __A) {`。
- **L1061 EN**: Returns from the current function with `_mm_cvttsd_si64(__A)`.
  **L1061 CN**: 以 `_mm_cvttsd_si64(__A)` 从当前函数返回。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1064 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1064 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1065 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1065 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1066 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsd_ss(__m128 __A, __m128d __B) {`.
  **L1066 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsd_ss(__m128 __A, __m128d __B) {`。
- **L1067 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1067 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1068 EN**: Blank line separating nearby declarations or logic blocks.
  **L1068 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1069 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1069 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1070 EN**: Adds a standalone statement or declaration: `__v4sf __temp_s;`.
  **L1070 CN**: 添加一条独立语句或声明：`__v4sf __temp_s;`。
- **L1071 EN**: Comment explains nearby logic, constraints, or intent: `Copy double element[0] to element [1] for conversion.`.
  **L1071 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Copy double element[0] to element [1] for conversion.`。
- **L1072 EN**: Initializes variable `__temp_b` from the expression on the right-hand side.
  **L1072 CN**: 使用右侧表达式初始化变量 `__temp_b`。
- **L1073 EN**: Blank line separating nearby declarations or logic blocks.
  **L1073 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1074 EN**: Comment explains nearby logic, constraints, or intent: `Pre-rotate __A left 3 (logically right 1) elements.`.
  **L1074 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pre-rotate __A left 3 (logically right 1) elements.`。
- **L1075 EN**: Executes a call or declaration centered on `__builtin_vsx_xxsldwi`.
  **L1075 CN**: 执行以 `__builtin_vsx_xxsldwi` 为核心的调用或声明。
- **L1076 EN**: Comment explains nearby logic, constraints, or intent: `Convert double to single float scalar in a vector.`.
  **L1076 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert double to single float scalar in a vector.`。
- **L1077 EN**: Executes a call or declaration centered on `__asm__`.
  **L1077 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L1078 EN**: Comment explains nearby logic, constraints, or intent: `Shift the resulting scalar into vector element [0].`.
  **L1078 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift the resulting scalar into vector element [0].`。
- **L1079 EN**: Executes a call or declaration centered on `__builtin_vsx_xxsldwi`.
  **L1079 CN**: 执行以 `__builtin_vsx_xxsldwi` 为核心的调用或声明。
- **L1080 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1080 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 1081-1104

````c
  __result[0] = ((__v2df)__B)[0];
#endif
  return (__m128)__result;
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi32_sd(__m128d __A, int __B) {
  __v2df __result = (__v2df)__A;
  double __db = __B;
  __result[0] = __db;
  return (__m128d)__result;
}

/* Intel intrinsic.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64_sd(__m128d __A, long long __B) {
  __v2df __result = (__v2df)__A;
  double __db = __B;
  __result[0] = __db;
  return (__m128d)__result;
}

````
- **L1081 EN**: Executes a call or declaration centered on `=`.
  **L1081 CN**: 执行以 `=` 为核心的调用或声明。
- **L1082 EN**: Closes the current preprocessor conditional block.
  **L1082 CN**: 结束当前预处理条件块。
- **L1083 EN**: Returns from the current function with `(__m128)__result`.
  **L1083 CN**: 以 `(__m128)__result` 从当前函数返回。
- **L1084 EN**: Closes the current lexical scope or compound statement.
  **L1084 CN**: 结束当前词法作用域或复合语句块。
- **L1085 EN**: Blank line separating nearby declarations or logic blocks.
  **L1085 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1086 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1086 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1087 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1087 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1088 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi32_sd(__m128d __A, int __B) {`.
  **L1088 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi32_sd(__m128d __A, int __B) {`。
- **L1089 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1089 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1090 EN**: Initializes variable `__db` from the expression on the right-hand side.
  **L1090 CN**: 使用右侧表达式初始化变量 `__db`。
- **L1091 EN**: Adds a standalone statement or declaration: `__result[0] = __db;`.
  **L1091 CN**: 添加一条独立语句或声明：`__result[0] = __db;`。
- **L1092 EN**: Returns from the current function with `(__m128d)__result`.
  **L1092 CN**: 以 `(__m128d)__result` 从当前函数返回。
- **L1093 EN**: Closes the current lexical scope or compound statement.
  **L1093 CN**: 结束当前词法作用域或复合语句块。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1095 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L1095 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L1096 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1096 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1097 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1097 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1098 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64_sd(__m128d __A, long long __B) {`.
  **L1098 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64_sd(__m128d __A, long long __B) {`。
- **L1099 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1099 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1100 EN**: Initializes variable `__db` from the expression on the right-hand side.
  **L1100 CN**: 使用右侧表达式初始化变量 `__db`。
- **L1101 EN**: Adds a standalone statement or declaration: `__result[0] = __db;`.
  **L1101 CN**: 添加一条独立语句或声明：`__result[0] = __db;`。
- **L1102 EN**: Returns from the current function with `(__m128d)__result`.
  **L1102 CN**: 以 `(__m128d)__result` 从当前函数返回。
- **L1103 EN**: Closes the current lexical scope or compound statement.
  **L1103 CN**: 结束当前词法作用域或复合语句块。
- **L1104 EN**: Blank line separating nearby declarations or logic blocks.
  **L1104 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1105-1128

````c
/* Microsoft intrinsic.  */
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64x_sd(__m128d __A, long long __B) {
  return _mm_cvtsi64_sd(__A, __B);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtss_sd(__m128d __A, __m128 __B) {
#ifdef __LITTLE_ENDIAN__
  /* Use splat to move element [0] into position for the convert. */
  __v4sf __temp = vec_splat((__v4sf)__B, 0);
  __v2df __res;
  /* Convert single float scalar to double in a vector.  */
  __asm__("xscvspdp %x0,%x1" : "=wa"(__res) : "wa"(__temp) :);
  return (__m128d)vec_mergel(__res, (__v2df)__A);
#else
  __v2df __res = (__v2df)__A;
  __res[0] = ((__v4sf)__B)[0];
  return (__m128d)__res;
#endif
}

````
- **L1105 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L1105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L1106 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1106 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1107 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1107 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1108 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64x_sd(__m128d __A, long long __B) {`.
  **L1108 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64x_sd(__m128d __A, long long __B) {`。
- **L1109 EN**: Returns from the current function with `_mm_cvtsi64_sd(__A, __B)`.
  **L1109 CN**: 以 `_mm_cvtsi64_sd(__A, __B)` 从当前函数返回。
- **L1110 EN**: Closes the current lexical scope or compound statement.
  **L1110 CN**: 结束当前词法作用域或复合语句块。
- **L1111 EN**: Blank line separating nearby declarations or logic blocks.
  **L1111 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1112 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1112 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1113 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1113 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1114 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtss_sd(__m128d __A, __m128 __B) {`.
  **L1114 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtss_sd(__m128d __A, __m128 __B) {`。
- **L1115 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1115 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1116 EN**: Comment explains nearby logic, constraints, or intent: `Use splat to move element [0] into position for the convert.`.
  **L1116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use splat to move element [0] into position for the convert.`。
- **L1117 EN**: Initializes variable `__temp` from the expression on the right-hand side.
  **L1117 CN**: 使用右侧表达式初始化变量 `__temp`。
- **L1118 EN**: Adds a standalone statement or declaration: `__v2df __res;`.
  **L1118 CN**: 添加一条独立语句或声明：`__v2df __res;`。
- **L1119 EN**: Comment explains nearby logic, constraints, or intent: `Convert single float scalar to double in a vector.`.
  **L1119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert single float scalar to double in a vector.`。
- **L1120 EN**: Executes a call or declaration centered on `__asm__`.
  **L1120 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L1121 EN**: Returns from the current function with `(__m128d)vec_mergel(__res, (__v2df)__A)`.
  **L1121 CN**: 以 `(__m128d)vec_mergel(__res, (__v2df)__A)` 从当前函数返回。
- **L1122 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1122 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1123 EN**: Initializes variable `__res` from the expression on the right-hand side.
  **L1123 CN**: 使用右侧表达式初始化变量 `__res`。
- **L1124 EN**: Executes a call or declaration centered on `=`.
  **L1124 CN**: 执行以 `=` 为核心的调用或声明。
- **L1125 EN**: Returns from the current function with `(__m128d)__res`.
  **L1125 CN**: 以 `(__m128d)__res` 从当前函数返回。
- **L1126 EN**: Closes the current preprocessor conditional block.
  **L1126 CN**: 结束当前预处理条件块。
- **L1127 EN**: Closes the current lexical scope or compound statement.
  **L1127 CN**: 结束当前词法作用域或复合语句块。
- **L1128 EN**: Blank line separating nearby declarations or logic blocks.
  **L1128 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1129-1152

````c
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_shuffle_pd(__m128d __A, __m128d __B, const int __mask) {
  __vector double __result;
  const int __litmsk = __mask & 0x3;

  if (__litmsk == 0)
    __result = vec_mergeh(__A, __B);
#if __GNUC__ < 6
  else if (__litmsk == 1)
    __result = vec_xxpermdi(__B, __A, 2);
  else if (__litmsk == 2)
    __result = vec_xxpermdi(__B, __A, 1);
#else
  else if (__litmsk == 1)
    __result = vec_xxpermdi(__A, __B, 2);
  else if (__litmsk == 2)
    __result = vec_xxpermdi(__A, __B, 1);
#endif
  else
    __result = vec_mergel(__A, __B);

  return __result;
}
````
- **L1129 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1129 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1130 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1130 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1131 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_shuffle_pd(__m128d __A, __m128d __B, const int __mask) {`.
  **L1131 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_shuffle_pd(__m128d __A, __m128d __B, const int __mask) {`。
- **L1132 EN**: Adds a standalone statement or declaration: `__vector double __result;`.
  **L1132 CN**: 添加一条独立语句或声明：`__vector double __result;`。
- **L1133 EN**: Initializes variable `__litmsk` from the expression on the right-hand side.
  **L1133 CN**: 使用右侧表达式初始化变量 `__litmsk`。
- **L1134 EN**: Blank line separating nearby declarations or logic blocks.
  **L1134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1135 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1135 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1136 EN**: Executes a call or declaration centered on `vec_mergeh`.
  **L1136 CN**: 执行以 `vec_mergeh` 为核心的调用或声明。
- **L1137 EN**: Starts a preprocessor conditional block: `#if __GNUC__ < 6`.
  **L1137 CN**: 开始一个预处理条件块：`#if __GNUC__ < 6`。
- **L1138 EN**: Starts the alternative branch of the preceding conditional.
  **L1138 CN**: 开始前一个条件语句的备选分支。
- **L1139 EN**: Executes a call or declaration centered on `vec_xxpermdi`.
  **L1139 CN**: 执行以 `vec_xxpermdi` 为核心的调用或声明。
- **L1140 EN**: Starts the alternative branch of the preceding conditional.
  **L1140 CN**: 开始前一个条件语句的备选分支。
- **L1141 EN**: Executes a call or declaration centered on `vec_xxpermdi`.
  **L1141 CN**: 执行以 `vec_xxpermdi` 为核心的调用或声明。
- **L1142 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1142 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1143 EN**: Starts the alternative branch of the preceding conditional.
  **L1143 CN**: 开始前一个条件语句的备选分支。
- **L1144 EN**: Executes a call or declaration centered on `vec_xxpermdi`.
  **L1144 CN**: 执行以 `vec_xxpermdi` 为核心的调用或声明。
- **L1145 EN**: Starts the alternative branch of the preceding conditional.
  **L1145 CN**: 开始前一个条件语句的备选分支。
- **L1146 EN**: Executes a call or declaration centered on `vec_xxpermdi`.
  **L1146 CN**: 执行以 `vec_xxpermdi` 为核心的调用或声明。
- **L1147 EN**: Closes the current preprocessor conditional block.
  **L1147 CN**: 结束当前预处理条件块。
- **L1148 EN**: Starts the alternative branch of the preceding conditional.
  **L1148 CN**: 开始前一个条件语句的备选分支。
- **L1149 EN**: Executes a call or declaration centered on `vec_mergel`.
  **L1149 CN**: 执行以 `vec_mergel` 为核心的调用或声明。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1151 EN**: Returns from the current function with `__result`.
  **L1151 CN**: 以 `__result` 从当前函数返回。
- **L1152 EN**: Closes the current lexical scope or compound statement.
  **L1152 CN**: 结束当前词法作用域或复合语句块。

### Lines 1153-1176

````c

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_pd(__m128d __A, __m128d __B) {
  return (__m128d)vec_mergel((__v2df)__A, (__v2df)__B);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_pd(__m128d __A, __m128d __B) {
  return (__m128d)vec_mergeh((__v2df)__A, (__v2df)__B);
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadh_pd(__m128d __A, double const *__B) {
  __v2df __result = (__v2df)__A;
  __result[1] = *__B;
  return (__m128d)__result;
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadl_pd(__m128d __A, double const *__B) {
````
- **L1153 EN**: Blank line separating nearby declarations or logic blocks.
  **L1153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1154 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1154 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1155 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1155 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_pd(__m128d __A, __m128d __B) {`.
  **L1156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_pd(__m128d __A, __m128d __B) {`。
- **L1157 EN**: Returns from the current function with `(__m128d)vec_mergel((__v2df)__A, (__v2df)__B)`.
  **L1157 CN**: 以 `(__m128d)vec_mergel((__v2df)__A, (__v2df)__B)` 从当前函数返回。
- **L1158 EN**: Closes the current lexical scope or compound statement.
  **L1158 CN**: 结束当前词法作用域或复合语句块。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1160 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1160 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1161 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1161 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1162 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_pd(__m128d __A, __m128d __B) {`.
  **L1162 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_pd(__m128d __A, __m128d __B) {`。
- **L1163 EN**: Returns from the current function with `(__m128d)vec_mergeh((__v2df)__A, (__v2df)__B)`.
  **L1163 CN**: 以 `(__m128d)vec_mergeh((__v2df)__A, (__v2df)__B)` 从当前函数返回。
- **L1164 EN**: Closes the current lexical scope or compound statement.
  **L1164 CN**: 结束当前词法作用域或复合语句块。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1166 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1166 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1167 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1167 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1168 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadh_pd(__m128d __A, double const *__B) {`.
  **L1168 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadh_pd(__m128d __A, double const *__B) {`。
- **L1169 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1169 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1170 EN**: Adds a standalone statement or declaration: `__result[1] = *__B;`.
  **L1170 CN**: 添加一条独立语句或声明：`__result[1] = *__B;`。
- **L1171 EN**: Returns from the current function with `(__m128d)__result`.
  **L1171 CN**: 以 `(__m128d)__result` 从当前函数返回。
- **L1172 EN**: Closes the current lexical scope or compound statement.
  **L1172 CN**: 结束当前词法作用域或复合语句块。
- **L1173 EN**: Blank line separating nearby declarations or logic blocks.
  **L1173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1174 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1174 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1175 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1175 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1176 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadl_pd(__m128d __A, double const *__B) {`.
  **L1176 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadl_pd(__m128d __A, double const *__B) {`。

### Lines 1177-1200

````c
  __v2df __result = (__v2df)__A;
  __result[0] = *__B;
  return (__m128d)__result;
}

#ifdef _ARCH_PWR8
/* Intrinsic functions that require PowerISA 2.07 minimum.  */

/* Creates a 2-bit mask from the most significant bits of the DPFP values.  */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_movemask_pd(__m128d __A) {
#ifdef _ARCH_PWR10
  return vec_extractm((__v2du)__A);
#else
  __vector unsigned long long __result;
  static const __vector unsigned int __perm_mask = {
#ifdef __LITTLE_ENDIAN__
      0x80800040, 0x80808080, 0x80808080, 0x80808080
#else
      0x80808080, 0x80808080, 0x80808080, 0x80804000
#endif
  };

````
- **L1177 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1177 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1178 EN**: Adds a standalone statement or declaration: `__result[0] = *__B;`.
  **L1178 CN**: 添加一条独立语句或声明：`__result[0] = *__B;`。
- **L1179 EN**: Returns from the current function with `(__m128d)__result`.
  **L1179 CN**: 以 `(__m128d)__result` 从当前函数返回。
- **L1180 EN**: Closes the current lexical scope or compound statement.
  **L1180 CN**: 结束当前词法作用域或复合语句块。
- **L1181 EN**: Blank line separating nearby declarations or logic blocks.
  **L1181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1182 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L1182 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L1183 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsic functions that require PowerISA 2.07 minimum.`.
  **L1183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsic functions that require PowerISA 2.07 minimum.`。
- **L1184 EN**: Blank line separating nearby declarations or logic blocks.
  **L1184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1185 EN**: Comment explains nearby logic, constraints, or intent: `Creates a 2-bit mask from the most significant bits of the DPFP values.`.
  **L1185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a 2-bit mask from the most significant bits of the DPFP values.`。
- **L1186 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1186 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1187 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1187 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1188 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movemask_pd(__m128d __A) {`.
  **L1188 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movemask_pd(__m128d __A) {`。
- **L1189 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR10`.
  **L1189 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR10`。
- **L1190 EN**: Returns from the current function with `vec_extractm((__v2du)__A)`.
  **L1190 CN**: 以 `vec_extractm((__v2du)__A)` 从当前函数返回。
- **L1191 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1191 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1192 EN**: Adds a standalone statement or declaration: `__vector unsigned long long __result;`.
  **L1192 CN**: 添加一条独立语句或声明：`__vector unsigned long long __result;`。
- **L1193 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned int __perm_mask = {`.
  **L1193 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned int __perm_mask = {`。
- **L1194 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1194 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1195 EN**: Continues the surrounding expression or declaration: `0x80800040, 0x80808080, 0x80808080, 0x80808080`.
  **L1195 CN**: 继续构造周围的表达式或声明：`0x80800040, 0x80808080, 0x80808080, 0x80808080`。
- **L1196 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1196 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1197 EN**: Continues the surrounding expression or declaration: `0x80808080, 0x80808080, 0x80808080, 0x80804000`.
  **L1197 CN**: 继续构造周围的表达式或声明：`0x80808080, 0x80808080, 0x80808080, 0x80804000`。
- **L1198 EN**: Closes the current preprocessor conditional block.
  **L1198 CN**: 结束当前预处理条件块。
- **L1199 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1199 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1200 EN**: Blank line separating nearby declarations or logic blocks.
  **L1200 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1201-1224

````c
  __result = ((__vector unsigned long long)vec_vbpermq(
      (__vector unsigned char)__A, (__vector unsigned char)__perm_mask));

#ifdef __LITTLE_ENDIAN__
  return __result[1];
#else
  return __result[0];
#endif
#endif /* !_ARCH_PWR10 */
}
#endif /* _ARCH_PWR8 */

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_packs_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_packs((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_packs_epi32(__m128i __A, __m128i __B) {
  return (__m128i)vec_packs((__v4si)__A, (__v4si)__B);
}

````
- **L1201 EN**: Continues logic associated with callable symbol `vec_vbpermq`.
  **L1201 CN**: 继续与可调用符号 `vec_vbpermq` 相关的逻辑。
- **L1202 EN**: Executes a call or declaration centered on `statement`.
  **L1202 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1203 EN**: Blank line separating nearby declarations or logic blocks.
  **L1203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1204 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1204 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1205 EN**: Returns from the current function with `__result[1]`.
  **L1205 CN**: 以 `__result[1]` 从当前函数返回。
- **L1206 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1206 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1207 EN**: Returns from the current function with `__result[0]`.
  **L1207 CN**: 以 `__result[0]` 从当前函数返回。
- **L1208 EN**: Closes the current preprocessor conditional block.
  **L1208 CN**: 结束当前预处理条件块。
- **L1209 EN**: Closes the current preprocessor conditional block.
  **L1209 CN**: 结束当前预处理条件块。
- **L1210 EN**: Closes the current lexical scope or compound statement.
  **L1210 CN**: 结束当前词法作用域或复合语句块。
- **L1211 EN**: Closes the current preprocessor conditional block.
  **L1211 CN**: 结束当前预处理条件块。
- **L1212 EN**: Blank line separating nearby declarations or logic blocks.
  **L1212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1213 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1213 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1214 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1214 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1215 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_packs_epi16(__m128i __A, __m128i __B) {`.
  **L1215 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_packs_epi16(__m128i __A, __m128i __B) {`。
- **L1216 EN**: Returns from the current function with `(__m128i)vec_packs((__v8hi)__A, (__v8hi)__B)`.
  **L1216 CN**: 以 `(__m128i)vec_packs((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1217 EN**: Closes the current lexical scope or compound statement.
  **L1217 CN**: 结束当前词法作用域或复合语句块。
- **L1218 EN**: Blank line separating nearby declarations or logic blocks.
  **L1218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1219 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1219 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1220 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1220 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1221 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_packs_epi32(__m128i __A, __m128i __B) {`.
  **L1221 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_packs_epi32(__m128i __A, __m128i __B) {`。
- **L1222 EN**: Returns from the current function with `(__m128i)vec_packs((__v4si)__A, (__v4si)__B)`.
  **L1222 CN**: 以 `(__m128i)vec_packs((__v4si)__A, (__v4si)__B)` 从当前函数返回。
- **L1223 EN**: Closes the current lexical scope or compound statement.
  **L1223 CN**: 结束当前词法作用域或复合语句块。
- **L1224 EN**: Blank line separating nearby declarations or logic blocks.
  **L1224 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1225-1248

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_packus_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_packsu((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_epi8(__m128i __A, __m128i __B) {
  return (__m128i)vec_mergel((__v16qu)__A, (__v16qu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_mergel((__v8hu)__A, (__v8hu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_epi32(__m128i __A, __m128i __B) {
  return (__m128i)vec_mergel((__v4su)__A, (__v4su)__B);
}

````
- **L1225 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1225 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1226 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1226 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_packus_epi16(__m128i __A, __m128i __B) {`.
  **L1227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_packus_epi16(__m128i __A, __m128i __B) {`。
- **L1228 EN**: Returns from the current function with `(__m128i)vec_packsu((__v8hi)__A, (__v8hi)__B)`.
  **L1228 CN**: 以 `(__m128i)vec_packsu((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1229 EN**: Closes the current lexical scope or compound statement.
  **L1229 CN**: 结束当前词法作用域或复合语句块。
- **L1230 EN**: Blank line separating nearby declarations or logic blocks.
  **L1230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1231 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1231 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1232 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1232 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1233 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_epi8(__m128i __A, __m128i __B) {`.
  **L1233 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_epi8(__m128i __A, __m128i __B) {`。
- **L1234 EN**: Returns from the current function with `(__m128i)vec_mergel((__v16qu)__A, (__v16qu)__B)`.
  **L1234 CN**: 以 `(__m128i)vec_mergel((__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L1235 EN**: Closes the current lexical scope or compound statement.
  **L1235 CN**: 结束当前词法作用域或复合语句块。
- **L1236 EN**: Blank line separating nearby declarations or logic blocks.
  **L1236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1237 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1237 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1238 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1238 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1239 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_epi16(__m128i __A, __m128i __B) {`.
  **L1239 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_epi16(__m128i __A, __m128i __B) {`。
- **L1240 EN**: Returns from the current function with `(__m128i)vec_mergel((__v8hu)__A, (__v8hu)__B)`.
  **L1240 CN**: 以 `(__m128i)vec_mergel((__v8hu)__A, (__v8hu)__B)` 从当前函数返回。
- **L1241 EN**: Closes the current lexical scope or compound statement.
  **L1241 CN**: 结束当前词法作用域或复合语句块。
- **L1242 EN**: Blank line separating nearby declarations or logic blocks.
  **L1242 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1243 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1243 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1244 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1244 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1245 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_epi32(__m128i __A, __m128i __B) {`.
  **L1245 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_epi32(__m128i __A, __m128i __B) {`。
- **L1246 EN**: Returns from the current function with `(__m128i)vec_mergel((__v4su)__A, (__v4su)__B)`.
  **L1246 CN**: 以 `(__m128i)vec_mergel((__v4su)__A, (__v4su)__B)` 从当前函数返回。
- **L1247 EN**: Closes the current lexical scope or compound statement.
  **L1247 CN**: 结束当前词法作用域或复合语句块。
- **L1248 EN**: Blank line separating nearby declarations or logic blocks.
  **L1248 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1249-1272

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_epi64(__m128i __A, __m128i __B) {
  return (__m128i)vec_mergel((__vector long long)__A, (__vector long long)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_epi8(__m128i __A, __m128i __B) {
  return (__m128i)vec_mergeh((__v16qu)__A, (__v16qu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_mergeh((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_epi32(__m128i __A, __m128i __B) {
  return (__m128i)vec_mergeh((__v4si)__A, (__v4si)__B);
}

````
- **L1249 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1249 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1250 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1250 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1251 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_epi64(__m128i __A, __m128i __B) {`.
  **L1251 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_epi64(__m128i __A, __m128i __B) {`。
- **L1252 EN**: Returns from the current function with `(__m128i)vec_mergel((__vector long long)__A, (__vector long long)__B)`.
  **L1252 CN**: 以 `(__m128i)vec_mergel((__vector long long)__A, (__vector long long)__B)` 从当前函数返回。
- **L1253 EN**: Closes the current lexical scope or compound statement.
  **L1253 CN**: 结束当前词法作用域或复合语句块。
- **L1254 EN**: Blank line separating nearby declarations or logic blocks.
  **L1254 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1255 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1255 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1256 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1256 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1257 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_epi8(__m128i __A, __m128i __B) {`.
  **L1257 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_epi8(__m128i __A, __m128i __B) {`。
- **L1258 EN**: Returns from the current function with `(__m128i)vec_mergeh((__v16qu)__A, (__v16qu)__B)`.
  **L1258 CN**: 以 `(__m128i)vec_mergeh((__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L1259 EN**: Closes the current lexical scope or compound statement.
  **L1259 CN**: 结束当前词法作用域或复合语句块。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1261 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1261 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1262 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1262 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1263 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_epi16(__m128i __A, __m128i __B) {`.
  **L1263 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_epi16(__m128i __A, __m128i __B) {`。
- **L1264 EN**: Returns from the current function with `(__m128i)vec_mergeh((__v8hi)__A, (__v8hi)__B)`.
  **L1264 CN**: 以 `(__m128i)vec_mergeh((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1265 EN**: Closes the current lexical scope or compound statement.
  **L1265 CN**: 结束当前词法作用域或复合语句块。
- **L1266 EN**: Blank line separating nearby declarations or logic blocks.
  **L1266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1267 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1267 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1268 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1268 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1269 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_epi32(__m128i __A, __m128i __B) {`.
  **L1269 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_epi32(__m128i __A, __m128i __B) {`。
- **L1270 EN**: Returns from the current function with `(__m128i)vec_mergeh((__v4si)__A, (__v4si)__B)`.
  **L1270 CN**: 以 `(__m128i)vec_mergeh((__v4si)__A, (__v4si)__B)` 从当前函数返回。
- **L1271 EN**: Closes the current lexical scope or compound statement.
  **L1271 CN**: 结束当前词法作用域或复合语句块。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1273-1296

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_epi64(__m128i __A, __m128i __B) {
  return (__m128i)vec_mergeh((__vector long long)__A, (__vector long long)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_epi8(__m128i __A, __m128i __B) {
  return (__m128i)((__v16qu)__A + (__v16qu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_epi16(__m128i __A, __m128i __B) {
  return (__m128i)((__v8hu)__A + (__v8hu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_epi32(__m128i __A, __m128i __B) {
  return (__m128i)((__v4su)__A + (__v4su)__B);
}

````
- **L1273 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1273 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1274 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1274 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1275 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_epi64(__m128i __A, __m128i __B) {`.
  **L1275 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_epi64(__m128i __A, __m128i __B) {`。
- **L1276 EN**: Returns from the current function with `(__m128i)vec_mergeh((__vector long long)__A, (__vector long long)__B)`.
  **L1276 CN**: 以 `(__m128i)vec_mergeh((__vector long long)__A, (__vector long long)__B)` 从当前函数返回。
- **L1277 EN**: Closes the current lexical scope or compound statement.
  **L1277 CN**: 结束当前词法作用域或复合语句块。
- **L1278 EN**: Blank line separating nearby declarations or logic blocks.
  **L1278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1279 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1279 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1280 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1280 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1281 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_epi8(__m128i __A, __m128i __B) {`.
  **L1281 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_epi8(__m128i __A, __m128i __B) {`。
- **L1282 EN**: Returns from the current function with `(__m128i)((__v16qu)__A + (__v16qu)__B)`.
  **L1282 CN**: 以 `(__m128i)((__v16qu)__A + (__v16qu)__B)` 从当前函数返回。
- **L1283 EN**: Closes the current lexical scope or compound statement.
  **L1283 CN**: 结束当前词法作用域或复合语句块。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1285 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1285 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1286 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1286 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1287 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_epi16(__m128i __A, __m128i __B) {`.
  **L1287 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_epi16(__m128i __A, __m128i __B) {`。
- **L1288 EN**: Returns from the current function with `(__m128i)((__v8hu)__A + (__v8hu)__B)`.
  **L1288 CN**: 以 `(__m128i)((__v8hu)__A + (__v8hu)__B)` 从当前函数返回。
- **L1289 EN**: Closes the current lexical scope or compound statement.
  **L1289 CN**: 结束当前词法作用域或复合语句块。
- **L1290 EN**: Blank line separating nearby declarations or logic blocks.
  **L1290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1291 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1291 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1292 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1292 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1293 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_epi32(__m128i __A, __m128i __B) {`.
  **L1293 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_epi32(__m128i __A, __m128i __B) {`。
- **L1294 EN**: Returns from the current function with `(__m128i)((__v4su)__A + (__v4su)__B)`.
  **L1294 CN**: 以 `(__m128i)((__v4su)__A + (__v4su)__B)` 从当前函数返回。
- **L1295 EN**: Closes the current lexical scope or compound statement.
  **L1295 CN**: 结束当前词法作用域或复合语句块。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1297-1320

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_epi64(__m128i __A, __m128i __B) {
  return (__m128i)((__v2du)__A + (__v2du)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_adds_epi8(__m128i __A, __m128i __B) {
  return (__m128i)vec_adds((__v16qi)__A, (__v16qi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_adds_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_adds((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_adds_epu8(__m128i __A, __m128i __B) {
  return (__m128i)vec_adds((__v16qu)__A, (__v16qu)__B);
}

````
- **L1297 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1297 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1298 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1298 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1299 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_epi64(__m128i __A, __m128i __B) {`.
  **L1299 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_epi64(__m128i __A, __m128i __B) {`。
- **L1300 EN**: Returns from the current function with `(__m128i)((__v2du)__A + (__v2du)__B)`.
  **L1300 CN**: 以 `(__m128i)((__v2du)__A + (__v2du)__B)` 从当前函数返回。
- **L1301 EN**: Closes the current lexical scope or compound statement.
  **L1301 CN**: 结束当前词法作用域或复合语句块。
- **L1302 EN**: Blank line separating nearby declarations or logic blocks.
  **L1302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1303 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1303 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1304 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1304 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1305 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_adds_epi8(__m128i __A, __m128i __B) {`.
  **L1305 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_adds_epi8(__m128i __A, __m128i __B) {`。
- **L1306 EN**: Returns from the current function with `(__m128i)vec_adds((__v16qi)__A, (__v16qi)__B)`.
  **L1306 CN**: 以 `(__m128i)vec_adds((__v16qi)__A, (__v16qi)__B)` 从当前函数返回。
- **L1307 EN**: Closes the current lexical scope or compound statement.
  **L1307 CN**: 结束当前词法作用域或复合语句块。
- **L1308 EN**: Blank line separating nearby declarations or logic blocks.
  **L1308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1309 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1309 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1310 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1310 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1311 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_adds_epi16(__m128i __A, __m128i __B) {`.
  **L1311 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_adds_epi16(__m128i __A, __m128i __B) {`。
- **L1312 EN**: Returns from the current function with `(__m128i)vec_adds((__v8hi)__A, (__v8hi)__B)`.
  **L1312 CN**: 以 `(__m128i)vec_adds((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1313 EN**: Closes the current lexical scope or compound statement.
  **L1313 CN**: 结束当前词法作用域或复合语句块。
- **L1314 EN**: Blank line separating nearby declarations or logic blocks.
  **L1314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1315 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1315 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1316 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1316 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1317 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_adds_epu8(__m128i __A, __m128i __B) {`.
  **L1317 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_adds_epu8(__m128i __A, __m128i __B) {`。
- **L1318 EN**: Returns from the current function with `(__m128i)vec_adds((__v16qu)__A, (__v16qu)__B)`.
  **L1318 CN**: 以 `(__m128i)vec_adds((__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L1319 EN**: Closes the current lexical scope or compound statement.
  **L1319 CN**: 结束当前词法作用域或复合语句块。
- **L1320 EN**: Blank line separating nearby declarations or logic blocks.
  **L1320 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1321-1344

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_adds_epu16(__m128i __A, __m128i __B) {
  return (__m128i)vec_adds((__v8hu)__A, (__v8hu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_epi8(__m128i __A, __m128i __B) {
  return (__m128i)((__v16qu)__A - (__v16qu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_epi16(__m128i __A, __m128i __B) {
  return (__m128i)((__v8hu)__A - (__v8hu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_epi32(__m128i __A, __m128i __B) {
  return (__m128i)((__v4su)__A - (__v4su)__B);
}

````
- **L1321 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1321 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1322 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1322 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1323 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_adds_epu16(__m128i __A, __m128i __B) {`.
  **L1323 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_adds_epu16(__m128i __A, __m128i __B) {`。
- **L1324 EN**: Returns from the current function with `(__m128i)vec_adds((__v8hu)__A, (__v8hu)__B)`.
  **L1324 CN**: 以 `(__m128i)vec_adds((__v8hu)__A, (__v8hu)__B)` 从当前函数返回。
- **L1325 EN**: Closes the current lexical scope or compound statement.
  **L1325 CN**: 结束当前词法作用域或复合语句块。
- **L1326 EN**: Blank line separating nearby declarations or logic blocks.
  **L1326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1327 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1327 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1328 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1328 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1329 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_epi8(__m128i __A, __m128i __B) {`.
  **L1329 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_epi8(__m128i __A, __m128i __B) {`。
- **L1330 EN**: Returns from the current function with `(__m128i)((__v16qu)__A - (__v16qu)__B)`.
  **L1330 CN**: 以 `(__m128i)((__v16qu)__A - (__v16qu)__B)` 从当前函数返回。
- **L1331 EN**: Closes the current lexical scope or compound statement.
  **L1331 CN**: 结束当前词法作用域或复合语句块。
- **L1332 EN**: Blank line separating nearby declarations or logic blocks.
  **L1332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1333 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1333 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1334 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1334 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1335 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_epi16(__m128i __A, __m128i __B) {`.
  **L1335 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_epi16(__m128i __A, __m128i __B) {`。
- **L1336 EN**: Returns from the current function with `(__m128i)((__v8hu)__A - (__v8hu)__B)`.
  **L1336 CN**: 以 `(__m128i)((__v8hu)__A - (__v8hu)__B)` 从当前函数返回。
- **L1337 EN**: Closes the current lexical scope or compound statement.
  **L1337 CN**: 结束当前词法作用域或复合语句块。
- **L1338 EN**: Blank line separating nearby declarations or logic blocks.
  **L1338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1339 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1339 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1340 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1340 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1341 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_epi32(__m128i __A, __m128i __B) {`.
  **L1341 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_epi32(__m128i __A, __m128i __B) {`。
- **L1342 EN**: Returns from the current function with `(__m128i)((__v4su)__A - (__v4su)__B)`.
  **L1342 CN**: 以 `(__m128i)((__v4su)__A - (__v4su)__B)` 从当前函数返回。
- **L1343 EN**: Closes the current lexical scope or compound statement.
  **L1343 CN**: 结束当前词法作用域或复合语句块。
- **L1344 EN**: Blank line separating nearby declarations or logic blocks.
  **L1344 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1345-1368

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_epi64(__m128i __A, __m128i __B) {
  return (__m128i)((__v2du)__A - (__v2du)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_subs_epi8(__m128i __A, __m128i __B) {
  return (__m128i)vec_subs((__v16qi)__A, (__v16qi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_subs_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_subs((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_subs_epu8(__m128i __A, __m128i __B) {
  return (__m128i)vec_subs((__v16qu)__A, (__v16qu)__B);
}

````
- **L1345 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1345 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1346 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1346 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1347 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_epi64(__m128i __A, __m128i __B) {`.
  **L1347 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_epi64(__m128i __A, __m128i __B) {`。
- **L1348 EN**: Returns from the current function with `(__m128i)((__v2du)__A - (__v2du)__B)`.
  **L1348 CN**: 以 `(__m128i)((__v2du)__A - (__v2du)__B)` 从当前函数返回。
- **L1349 EN**: Closes the current lexical scope or compound statement.
  **L1349 CN**: 结束当前词法作用域或复合语句块。
- **L1350 EN**: Blank line separating nearby declarations or logic blocks.
  **L1350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1351 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1351 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1352 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1352 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1353 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_subs_epi8(__m128i __A, __m128i __B) {`.
  **L1353 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_subs_epi8(__m128i __A, __m128i __B) {`。
- **L1354 EN**: Returns from the current function with `(__m128i)vec_subs((__v16qi)__A, (__v16qi)__B)`.
  **L1354 CN**: 以 `(__m128i)vec_subs((__v16qi)__A, (__v16qi)__B)` 从当前函数返回。
- **L1355 EN**: Closes the current lexical scope or compound statement.
  **L1355 CN**: 结束当前词法作用域或复合语句块。
- **L1356 EN**: Blank line separating nearby declarations or logic blocks.
  **L1356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1357 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1357 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1358 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1358 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1359 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_subs_epi16(__m128i __A, __m128i __B) {`.
  **L1359 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_subs_epi16(__m128i __A, __m128i __B) {`。
- **L1360 EN**: Returns from the current function with `(__m128i)vec_subs((__v8hi)__A, (__v8hi)__B)`.
  **L1360 CN**: 以 `(__m128i)vec_subs((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1361 EN**: Closes the current lexical scope or compound statement.
  **L1361 CN**: 结束当前词法作用域或复合语句块。
- **L1362 EN**: Blank line separating nearby declarations or logic blocks.
  **L1362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1363 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1363 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1364 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1364 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1365 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_subs_epu8(__m128i __A, __m128i __B) {`.
  **L1365 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_subs_epu8(__m128i __A, __m128i __B) {`。
- **L1366 EN**: Returns from the current function with `(__m128i)vec_subs((__v16qu)__A, (__v16qu)__B)`.
  **L1366 CN**: 以 `(__m128i)vec_subs((__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L1367 EN**: Closes the current lexical scope or compound statement.
  **L1367 CN**: 结束当前词法作用域或复合语句块。
- **L1368 EN**: Blank line separating nearby declarations or logic blocks.
  **L1368 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1369-1392

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_subs_epu16(__m128i __A, __m128i __B) {
  return (__m128i)vec_subs((__v8hu)__A, (__v8hu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_madd_epi16(__m128i __A, __m128i __B) {
  __vector signed int __zero = {0, 0, 0, 0};

  return (__m128i)vec_vmsumshm((__v8hi)__A, (__v8hi)__B, __zero);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mulhi_epi16(__m128i __A, __m128i __B) {
  __vector signed int __w0, __w1;

  __vector unsigned char __xform1 = {
#ifdef __LITTLE_ENDIAN__
      0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,
      0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F
#else
````
- **L1369 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1369 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1370 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1370 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1371 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_subs_epu16(__m128i __A, __m128i __B) {`.
  **L1371 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_subs_epu16(__m128i __A, __m128i __B) {`。
- **L1372 EN**: Returns from the current function with `(__m128i)vec_subs((__v8hu)__A, (__v8hu)__B)`.
  **L1372 CN**: 以 `(__m128i)vec_subs((__v8hu)__A, (__v8hu)__B)` 从当前函数返回。
- **L1373 EN**: Closes the current lexical scope or compound statement.
  **L1373 CN**: 结束当前词法作用域或复合语句块。
- **L1374 EN**: Blank line separating nearby declarations or logic blocks.
  **L1374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1375 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1375 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1376 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1376 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1377 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_madd_epi16(__m128i __A, __m128i __B) {`.
  **L1377 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_madd_epi16(__m128i __A, __m128i __B) {`。
- **L1378 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L1378 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L1379 EN**: Blank line separating nearby declarations or logic blocks.
  **L1379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1380 EN**: Returns from the current function with `(__m128i)vec_vmsumshm((__v8hi)__A, (__v8hi)__B, __zero)`.
  **L1380 CN**: 以 `(__m128i)vec_vmsumshm((__v8hi)__A, (__v8hi)__B, __zero)` 从当前函数返回。
- **L1381 EN**: Closes the current lexical scope or compound statement.
  **L1381 CN**: 结束当前词法作用域或复合语句块。
- **L1382 EN**: Blank line separating nearby declarations or logic blocks.
  **L1382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1383 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1383 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1384 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1384 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mulhi_epi16(__m128i __A, __m128i __B) {`.
  **L1385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mulhi_epi16(__m128i __A, __m128i __B) {`。
- **L1386 EN**: Adds a standalone statement or declaration: `__vector signed int __w0, __w1;`.
  **L1386 CN**: 添加一条独立语句或声明：`__vector signed int __w0, __w1;`。
- **L1387 EN**: Blank line separating nearby declarations or logic blocks.
  **L1387 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1388 EN**: Continues the surrounding expression or declaration: `__vector unsigned char __xform1 = {`.
  **L1388 CN**: 继续构造周围的表达式或声明：`__vector unsigned char __xform1 = {`。
- **L1389 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1389 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,`.
  **L1390 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,`。
- **L1391 EN**: Continues the surrounding expression or declaration: `0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F`.
  **L1391 CN**: 继续构造周围的表达式或声明：`0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F`。
- **L1392 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1392 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 1393-1416

````c
      0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x08,
      0x09, 0x18, 0x19, 0x0C, 0x0D, 0x1C, 0x1D
#endif
  };

  __w0 = vec_vmulesh((__v8hi)__A, (__v8hi)__B);
  __w1 = vec_vmulosh((__v8hi)__A, (__v8hi)__B);
  return (__m128i)vec_perm(__w0, __w1, __xform1);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mullo_epi16(__m128i __A, __m128i __B) {
  return (__m128i)((__v8hi)__A * (__v8hi)__B);
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mul_su32(__m64 __A, __m64 __B) {
  unsigned int __a = __A;
  unsigned int __b = __B;

  return ((__m64)__a * (__m64)__b);
}
````
- **L1393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x08,`.
  **L1393 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x08,`。
- **L1394 EN**: Continues the surrounding expression or declaration: `0x09, 0x18, 0x19, 0x0C, 0x0D, 0x1C, 0x1D`.
  **L1394 CN**: 继续构造周围的表达式或声明：`0x09, 0x18, 0x19, 0x0C, 0x0D, 0x1C, 0x1D`。
- **L1395 EN**: Closes the current preprocessor conditional block.
  **L1395 CN**: 结束当前预处理条件块。
- **L1396 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1396 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1397 EN**: Blank line separating nearby declarations or logic blocks.
  **L1397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1398 EN**: Executes a call or declaration centered on `vec_vmulesh`.
  **L1398 CN**: 执行以 `vec_vmulesh` 为核心的调用或声明。
- **L1399 EN**: Executes a call or declaration centered on `vec_vmulosh`.
  **L1399 CN**: 执行以 `vec_vmulosh` 为核心的调用或声明。
- **L1400 EN**: Returns from the current function with `(__m128i)vec_perm(__w0, __w1, __xform1)`.
  **L1400 CN**: 以 `(__m128i)vec_perm(__w0, __w1, __xform1)` 从当前函数返回。
- **L1401 EN**: Closes the current lexical scope or compound statement.
  **L1401 CN**: 结束当前词法作用域或复合语句块。
- **L1402 EN**: Blank line separating nearby declarations or logic blocks.
  **L1402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1403 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1403 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1404 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1404 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1405 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mullo_epi16(__m128i __A, __m128i __B) {`.
  **L1405 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mullo_epi16(__m128i __A, __m128i __B) {`。
- **L1406 EN**: Returns from the current function with `(__m128i)((__v8hi)__A * (__v8hi)__B)`.
  **L1406 CN**: 以 `(__m128i)((__v8hi)__A * (__v8hi)__B)` 从当前函数返回。
- **L1407 EN**: Closes the current lexical scope or compound statement.
  **L1407 CN**: 结束当前词法作用域或复合语句块。
- **L1408 EN**: Blank line separating nearby declarations or logic blocks.
  **L1408 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1409 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1409 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1410 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1410 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1411 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mul_su32(__m64 __A, __m64 __B) {`.
  **L1411 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mul_su32(__m64 __A, __m64 __B) {`。
- **L1412 EN**: Initializes variable `__a` from the expression on the right-hand side.
  **L1412 CN**: 使用右侧表达式初始化变量 `__a`。
- **L1413 EN**: Initializes variable `__b` from the expression on the right-hand side.
  **L1413 CN**: 使用右侧表达式初始化变量 `__b`。
- **L1414 EN**: Blank line separating nearby declarations or logic blocks.
  **L1414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1415 EN**: Returns from the current function with `((__m64)__a * (__m64)__b)`.
  **L1415 CN**: 以 `((__m64)__a * (__m64)__b)` 从当前函数返回。
- **L1416 EN**: Closes the current lexical scope or compound statement.
  **L1416 CN**: 结束当前词法作用域或复合语句块。

### Lines 1417-1440

````c

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mul_epu32(__m128i __A, __m128i __B) {
#if __GNUC__ < 8
  __v2du __result;

#ifdef __LITTLE_ENDIAN__
  /* VMX Vector Multiply Odd Unsigned Word.  */
  __asm__("vmulouw %0,%1,%2" : "=v"(__result) : "v"(__A), "v"(__B) :);
#else
  /* VMX Vector Multiply Even Unsigned Word.  */
  __asm__("vmuleuw %0,%1,%2" : "=v"(__result) : "v"(__A), "v"(__B) :);
#endif
  return (__m128i)__result;
#else
  return (__m128i)vec_mule((__v4su)__A, (__v4su)__B);
#endif
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L1417 EN**: Blank line separating nearby declarations or logic blocks.
  **L1417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1418 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L1418 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L1419 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1419 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1420 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1420 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1421 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mul_epu32(__m128i __A, __m128i __B) {`.
  **L1421 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mul_epu32(__m128i __A, __m128i __B) {`。
- **L1422 EN**: Starts a preprocessor conditional block: `#if __GNUC__ < 8`.
  **L1422 CN**: 开始一个预处理条件块：`#if __GNUC__ < 8`。
- **L1423 EN**: Adds a standalone statement or declaration: `__v2du __result;`.
  **L1423 CN**: 添加一条独立语句或声明：`__v2du __result;`。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1425 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1425 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1426 EN**: Comment explains nearby logic, constraints, or intent: `VMX Vector Multiply Odd Unsigned Word.`.
  **L1426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VMX Vector Multiply Odd Unsigned Word.`。
- **L1427 EN**: Executes a call or declaration centered on `__asm__`.
  **L1427 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L1428 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1428 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1429 EN**: Comment explains nearby logic, constraints, or intent: `VMX Vector Multiply Even Unsigned Word.`.
  **L1429 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VMX Vector Multiply Even Unsigned Word.`。
- **L1430 EN**: Executes a call or declaration centered on `__asm__`.
  **L1430 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L1431 EN**: Closes the current preprocessor conditional block.
  **L1431 CN**: 结束当前预处理条件块。
- **L1432 EN**: Returns from the current function with `(__m128i)__result`.
  **L1432 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1433 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1433 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1434 EN**: Returns from the current function with `(__m128i)vec_mule((__v4su)__A, (__v4su)__B)`.
  **L1434 CN**: 以 `(__m128i)vec_mule((__v4su)__A, (__v4su)__B)` 从当前函数返回。
- **L1435 EN**: Closes the current preprocessor conditional block.
  **L1435 CN**: 结束当前预处理条件块。
- **L1436 EN**: Closes the current lexical scope or compound statement.
  **L1436 CN**: 结束当前词法作用域或复合语句块。
- **L1437 EN**: Closes the current preprocessor conditional block.
  **L1437 CN**: 结束当前预处理条件块。
- **L1438 EN**: Blank line separating nearby declarations or logic blocks.
  **L1438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1439 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1439 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1440 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1440 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 1441-1464

````c
    _mm_slli_epi16(__m128i __A, int __B) {
  __v8hu __lshift;
  __v8hi __result = {0, 0, 0, 0, 0, 0, 0, 0};

  if (__B >= 0 && __B < 16) {
    if (__builtin_constant_p(__B))
      __lshift = (__v8hu)vec_splat_s16(__B);
    else
      __lshift = vec_splats((unsigned short)__B);

    __result = vec_sl((__v8hi)__A, __lshift);
  }

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_slli_epi32(__m128i __A, int __B) {
  __v4su __lshift;
  __v4si __result = {0, 0, 0, 0};

  if (__B >= 0 && __B < 32) {
    if (__builtin_constant_p(__B) && __B < 16)
````
- **L1441 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_slli_epi16(__m128i __A, int __B) {`.
  **L1441 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_slli_epi16(__m128i __A, int __B) {`。
- **L1442 EN**: Adds a standalone statement or declaration: `__v8hu __lshift;`.
  **L1442 CN**: 添加一条独立语句或声明：`__v8hu __lshift;`。
- **L1443 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1443 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1444 EN**: Blank line separating nearby declarations or logic blocks.
  **L1444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1445 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1445 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1446 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1446 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1447 EN**: Executes a call or declaration centered on `=`.
  **L1447 CN**: 执行以 `=` 为核心的调用或声明。
- **L1448 EN**: Starts the alternative branch of the preceding conditional.
  **L1448 CN**: 开始前一个条件语句的备选分支。
- **L1449 EN**: Executes a call or declaration centered on `vec_splats`.
  **L1449 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L1450 EN**: Blank line separating nearby declarations or logic blocks.
  **L1450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1451 EN**: Executes a call or declaration centered on `vec_sl`.
  **L1451 CN**: 执行以 `vec_sl` 为核心的调用或声明。
- **L1452 EN**: Closes the current lexical scope or compound statement.
  **L1452 CN**: 结束当前词法作用域或复合语句块。
- **L1453 EN**: Blank line separating nearby declarations or logic blocks.
  **L1453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1454 EN**: Returns from the current function with `(__m128i)__result`.
  **L1454 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1455 EN**: Closes the current lexical scope or compound statement.
  **L1455 CN**: 结束当前词法作用域或复合语句块。
- **L1456 EN**: Blank line separating nearby declarations or logic blocks.
  **L1456 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1457 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1457 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1458 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1458 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1459 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_slli_epi32(__m128i __A, int __B) {`.
  **L1459 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_slli_epi32(__m128i __A, int __B) {`。
- **L1460 EN**: Adds a standalone statement or declaration: `__v4su __lshift;`.
  **L1460 CN**: 添加一条独立语句或声明：`__v4su __lshift;`。
- **L1461 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1461 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1462 EN**: Blank line separating nearby declarations or logic blocks.
  **L1462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1463 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1463 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1464 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1464 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1465-1488

````c
      __lshift = (__v4su)vec_splat_s32(__B);
    else
      __lshift = vec_splats((unsigned int)__B);

    __result = vec_sl((__v4si)__A, __lshift);
  }

  return (__m128i)__result;
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_slli_epi64(__m128i __A, int __B) {
  __v2du __lshift;
  __v2di __result = {0, 0};

  if (__B >= 0 && __B < 64) {
    if (__builtin_constant_p(__B) && __B < 16)
      __lshift = (__v2du)vec_splat_s32(__B);
    else
      __lshift = (__v2du)vec_splats((unsigned int)__B);

    __result = vec_sl((__v2di)__A, __lshift);
````
- **L1465 EN**: Executes a call or declaration centered on `=`.
  **L1465 CN**: 执行以 `=` 为核心的调用或声明。
- **L1466 EN**: Starts the alternative branch of the preceding conditional.
  **L1466 CN**: 开始前一个条件语句的备选分支。
- **L1467 EN**: Executes a call or declaration centered on `vec_splats`.
  **L1467 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L1468 EN**: Blank line separating nearby declarations or logic blocks.
  **L1468 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1469 EN**: Executes a call or declaration centered on `vec_sl`.
  **L1469 CN**: 执行以 `vec_sl` 为核心的调用或声明。
- **L1470 EN**: Closes the current lexical scope or compound statement.
  **L1470 CN**: 结束当前词法作用域或复合语句块。
- **L1471 EN**: Blank line separating nearby declarations or logic blocks.
  **L1471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1472 EN**: Returns from the current function with `(__m128i)__result`.
  **L1472 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1473 EN**: Closes the current lexical scope or compound statement.
  **L1473 CN**: 结束当前词法作用域或复合语句块。
- **L1474 EN**: Blank line separating nearby declarations or logic blocks.
  **L1474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1475 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L1475 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L1476 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1476 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1477 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1477 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1478 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_slli_epi64(__m128i __A, int __B) {`.
  **L1478 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_slli_epi64(__m128i __A, int __B) {`。
- **L1479 EN**: Adds a standalone statement or declaration: `__v2du __lshift;`.
  **L1479 CN**: 添加一条独立语句或声明：`__v2du __lshift;`。
- **L1480 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1480 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1481 EN**: Blank line separating nearby declarations or logic blocks.
  **L1481 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1482 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1482 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1483 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1483 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1484 EN**: Executes a call or declaration centered on `=`.
  **L1484 CN**: 执行以 `=` 为核心的调用或声明。
- **L1485 EN**: Starts the alternative branch of the preceding conditional.
  **L1485 CN**: 开始前一个条件语句的备选分支。
- **L1486 EN**: Executes a call or declaration centered on `=`.
  **L1486 CN**: 执行以 `=` 为核心的调用或声明。
- **L1487 EN**: Blank line separating nearby declarations or logic blocks.
  **L1487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1488 EN**: Executes a call or declaration centered on `vec_sl`.
  **L1488 CN**: 执行以 `vec_sl` 为核心的调用或声明。

### Lines 1489-1512

````c
  }

  return (__m128i)__result;
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srai_epi16(__m128i __A, int __B) {
  __v8hu __rshift = {15, 15, 15, 15, 15, 15, 15, 15};
  __v8hi __result;

  if (__B < 16) {
    if (__builtin_constant_p(__B))
      __rshift = (__v8hu)vec_splat_s16(__B);
    else
      __rshift = vec_splats((unsigned short)__B);
  }
  __result = vec_sra((__v8hi)__A, __rshift);

  return (__m128i)__result;
}

extern __inline __m128i
````
- **L1489 EN**: Closes the current lexical scope or compound statement.
  **L1489 CN**: 结束当前词法作用域或复合语句块。
- **L1490 EN**: Blank line separating nearby declarations or logic blocks.
  **L1490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1491 EN**: Returns from the current function with `(__m128i)__result`.
  **L1491 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1492 EN**: Closes the current lexical scope or compound statement.
  **L1492 CN**: 结束当前词法作用域或复合语句块。
- **L1493 EN**: Closes the current preprocessor conditional block.
  **L1493 CN**: 结束当前预处理条件块。
- **L1494 EN**: Blank line separating nearby declarations or logic blocks.
  **L1494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1495 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1495 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1496 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1496 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1497 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srai_epi16(__m128i __A, int __B) {`.
  **L1497 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srai_epi16(__m128i __A, int __B) {`。
- **L1498 EN**: Initializes variable `__rshift` from the expression on the right-hand side.
  **L1498 CN**: 使用右侧表达式初始化变量 `__rshift`。
- **L1499 EN**: Adds a standalone statement or declaration: `__v8hi __result;`.
  **L1499 CN**: 添加一条独立语句或声明：`__v8hi __result;`。
- **L1500 EN**: Blank line separating nearby declarations or logic blocks.
  **L1500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1502 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1502 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1503 EN**: Executes a call or declaration centered on `=`.
  **L1503 CN**: 执行以 `=` 为核心的调用或声明。
- **L1504 EN**: Starts the alternative branch of the preceding conditional.
  **L1504 CN**: 开始前一个条件语句的备选分支。
- **L1505 EN**: Executes a call or declaration centered on `vec_splats`.
  **L1505 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L1506 EN**: Closes the current lexical scope or compound statement.
  **L1506 CN**: 结束当前词法作用域或复合语句块。
- **L1507 EN**: Executes a call or declaration centered on `vec_sra`.
  **L1507 CN**: 执行以 `vec_sra` 为核心的调用或声明。
- **L1508 EN**: Blank line separating nearby declarations or logic blocks.
  **L1508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1509 EN**: Returns from the current function with `(__m128i)__result`.
  **L1509 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1510 EN**: Closes the current lexical scope or compound statement.
  **L1510 CN**: 结束当前词法作用域或复合语句块。
- **L1511 EN**: Blank line separating nearby declarations or logic blocks.
  **L1511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1512 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1512 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。

### Lines 1513-1536

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srai_epi32(__m128i __A, int __B) {
  __v4su __rshift = {31, 31, 31, 31};
  __v4si __result;

  if (__B < 32) {
    if (__builtin_constant_p(__B)) {
      if (__B < 16)
        __rshift = (__v4su)vec_splat_s32(__B);
      else
        __rshift = (__v4su)vec_splats((unsigned int)__B);
    } else
      __rshift = vec_splats((unsigned int)__B);
  }
  __result = vec_sra((__v4si)__A, __rshift);

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_bslli_si128(__m128i __A, const int __N) {
  __v16qu __result;
  const __v16qu __zeros = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};
````
- **L1513 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1513 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1514 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srai_epi32(__m128i __A, int __B) {`.
  **L1514 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srai_epi32(__m128i __A, int __B) {`。
- **L1515 EN**: Initializes variable `__rshift` from the expression on the right-hand side.
  **L1515 CN**: 使用右侧表达式初始化变量 `__rshift`。
- **L1516 EN**: Adds a standalone statement or declaration: `__v4si __result;`.
  **L1516 CN**: 添加一条独立语句或声明：`__v4si __result;`。
- **L1517 EN**: Blank line separating nearby declarations or logic blocks.
  **L1517 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1518 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1518 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1519 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1519 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1520 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1520 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1521 EN**: Executes a call or declaration centered on `=`.
  **L1521 CN**: 执行以 `=` 为核心的调用或声明。
- **L1522 EN**: Starts the alternative branch of the preceding conditional.
  **L1522 CN**: 开始前一个条件语句的备选分支。
- **L1523 EN**: Executes a call or declaration centered on `=`.
  **L1523 CN**: 执行以 `=` 为核心的调用或声明。
- **L1524 EN**: Continues the surrounding expression or declaration: `} else`.
  **L1524 CN**: 继续构造周围的表达式或声明：`} else`。
- **L1525 EN**: Executes a call or declaration centered on `vec_splats`.
  **L1525 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L1526 EN**: Closes the current lexical scope or compound statement.
  **L1526 CN**: 结束当前词法作用域或复合语句块。
- **L1527 EN**: Executes a call or declaration centered on `vec_sra`.
  **L1527 CN**: 执行以 `vec_sra` 为核心的调用或声明。
- **L1528 EN**: Blank line separating nearby declarations or logic blocks.
  **L1528 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1529 EN**: Returns from the current function with `(__m128i)__result`.
  **L1529 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1530 EN**: Closes the current lexical scope or compound statement.
  **L1530 CN**: 结束当前词法作用域或复合语句块。
- **L1531 EN**: Blank line separating nearby declarations or logic blocks.
  **L1531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1532 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1532 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1533 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1533 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1534 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_bslli_si128(__m128i __A, const int __N) {`.
  **L1534 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_bslli_si128(__m128i __A, const int __N) {`。
- **L1535 EN**: Adds a standalone statement or declaration: `__v16qu __result;`.
  **L1535 CN**: 添加一条独立语句或声明：`__v16qu __result;`。
- **L1536 EN**: Initializes variable `__zeros` from the expression on the right-hand side.
  **L1536 CN**: 使用右侧表达式初始化变量 `__zeros`。

### Lines 1537-1560

````c

  if (__N < 16)
    __result = vec_sld((__v16qu)__A, __zeros, __N);
  else
    __result = __zeros;

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_bsrli_si128(__m128i __A, const int __N) {
  __v16qu __result;
  const __v16qu __zeros = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

  if (__N < 16)
#ifdef __LITTLE_ENDIAN__
    if (__builtin_constant_p(__N))
      /* Would like to use Vector Shift Left Double by Octet
         Immediate here to use the immediate form and avoid
         load of __N * 8 value into a separate VR.  */
      __result = vec_sld(__zeros, (__v16qu)__A, (16 - __N));
    else
#endif
````
- **L1537 EN**: Blank line separating nearby declarations or logic blocks.
  **L1537 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1538 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1538 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1539 EN**: Executes a call or declaration centered on `vec_sld`.
  **L1539 CN**: 执行以 `vec_sld` 为核心的调用或声明。
- **L1540 EN**: Starts the alternative branch of the preceding conditional.
  **L1540 CN**: 开始前一个条件语句的备选分支。
- **L1541 EN**: Adds a standalone statement or declaration: `__result = __zeros;`.
  **L1541 CN**: 添加一条独立语句或声明：`__result = __zeros;`。
- **L1542 EN**: Blank line separating nearby declarations or logic blocks.
  **L1542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1543 EN**: Returns from the current function with `(__m128i)__result`.
  **L1543 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1544 EN**: Closes the current lexical scope or compound statement.
  **L1544 CN**: 结束当前词法作用域或复合语句块。
- **L1545 EN**: Blank line separating nearby declarations or logic blocks.
  **L1545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1546 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1546 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1547 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1547 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1548 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_bsrli_si128(__m128i __A, const int __N) {`.
  **L1548 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_bsrli_si128(__m128i __A, const int __N) {`。
- **L1549 EN**: Adds a standalone statement or declaration: `__v16qu __result;`.
  **L1549 CN**: 添加一条独立语句或声明：`__v16qu __result;`。
- **L1550 EN**: Initializes variable `__zeros` from the expression on the right-hand side.
  **L1550 CN**: 使用右侧表达式初始化变量 `__zeros`。
- **L1551 EN**: Blank line separating nearby declarations or logic blocks.
  **L1551 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1552 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1552 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1553 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1553 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1554 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1554 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1555 EN**: Comment explains nearby logic, constraints, or intent: `Would like to use Vector Shift Left Double by Octet`.
  **L1555 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Would like to use Vector Shift Left Double by Octet`。
- **L1556 EN**: Continues the surrounding expression or declaration: `Immediate here to use the immediate form and avoid`.
  **L1556 CN**: 继续构造周围的表达式或声明：`Immediate here to use the immediate form and avoid`。
- **L1557 EN**: Continues the surrounding expression or declaration: `load of __N * 8 value into a separate VR.  */`.
  **L1557 CN**: 继续构造周围的表达式或声明：`load of __N * 8 value into a separate VR.  */`。
- **L1558 EN**: Executes a call or declaration centered on `vec_sld`.
  **L1558 CN**: 执行以 `vec_sld` 为核心的调用或声明。
- **L1559 EN**: Starts the alternative branch of the preceding conditional.
  **L1559 CN**: 开始前一个条件语句的备选分支。
- **L1560 EN**: Closes the current preprocessor conditional block.
  **L1560 CN**: 结束当前预处理条件块。

### Lines 1561-1584

````c
    {
      __v16qu __shift = vec_splats((unsigned char)(__N * 8));
#ifdef __LITTLE_ENDIAN__
      __result = vec_sro((__v16qu)__A, __shift);
#else
    __result = vec_slo((__v16qu)__A, __shift);
#endif
    }
  else
    __result = __zeros;

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srli_si128(__m128i __A, const int __N) {
  return _mm_bsrli_si128(__A, __N);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_slli_si128(__m128i __A, const int _imm5) {
  __v16qu __result;
````
- **L1561 EN**: Opens a new lexical scope or compound statement.
  **L1561 CN**: 打开一个新的词法作用域或复合语句块。
- **L1562 EN**: Initializes variable `__shift` from the expression on the right-hand side.
  **L1562 CN**: 使用右侧表达式初始化变量 `__shift`。
- **L1563 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1563 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1564 EN**: Executes a call or declaration centered on `vec_sro`.
  **L1564 CN**: 执行以 `vec_sro` 为核心的调用或声明。
- **L1565 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1565 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1566 EN**: Executes a call or declaration centered on `vec_slo`.
  **L1566 CN**: 执行以 `vec_slo` 为核心的调用或声明。
- **L1567 EN**: Closes the current preprocessor conditional block.
  **L1567 CN**: 结束当前预处理条件块。
- **L1568 EN**: Closes the current lexical scope or compound statement.
  **L1568 CN**: 结束当前词法作用域或复合语句块。
- **L1569 EN**: Starts the alternative branch of the preceding conditional.
  **L1569 CN**: 开始前一个条件语句的备选分支。
- **L1570 EN**: Adds a standalone statement or declaration: `__result = __zeros;`.
  **L1570 CN**: 添加一条独立语句或声明：`__result = __zeros;`。
- **L1571 EN**: Blank line separating nearby declarations or logic blocks.
  **L1571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1572 EN**: Returns from the current function with `(__m128i)__result`.
  **L1572 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1573 EN**: Closes the current lexical scope or compound statement.
  **L1573 CN**: 结束当前词法作用域或复合语句块。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1575 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1575 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1576 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1576 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1577 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srli_si128(__m128i __A, const int __N) {`.
  **L1577 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srli_si128(__m128i __A, const int __N) {`。
- **L1578 EN**: Returns from the current function with `_mm_bsrli_si128(__A, __N)`.
  **L1578 CN**: 以 `_mm_bsrli_si128(__A, __N)` 从当前函数返回。
- **L1579 EN**: Closes the current lexical scope or compound statement.
  **L1579 CN**: 结束当前词法作用域或复合语句块。
- **L1580 EN**: Blank line separating nearby declarations or logic blocks.
  **L1580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1581 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1581 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1582 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1582 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1583 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_slli_si128(__m128i __A, const int _imm5) {`.
  **L1583 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_slli_si128(__m128i __A, const int _imm5) {`。
- **L1584 EN**: Adds a standalone statement or declaration: `__v16qu __result;`.
  **L1584 CN**: 添加一条独立语句或声明：`__v16qu __result;`。

### Lines 1585-1608

````c
  const __v16qu __zeros = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

  if (_imm5 < 16)
#ifdef __LITTLE_ENDIAN__
    __result = vec_sld((__v16qu)__A, __zeros, _imm5);
#else
    __result = vec_sld(__zeros, (__v16qu)__A, (16 - _imm5));
#endif
  else
    __result = __zeros;

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))

    _mm_srli_epi16(__m128i __A, int __B) {
  __v8hu __rshift;
  __v8hi __result = {0, 0, 0, 0, 0, 0, 0, 0};

  if (__B < 16) {
    if (__builtin_constant_p(__B))
      __rshift = (__v8hu)vec_splat_s16(__B);
````
- **L1585 EN**: Initializes variable `__zeros` from the expression on the right-hand side.
  **L1585 CN**: 使用右侧表达式初始化变量 `__zeros`。
- **L1586 EN**: Blank line separating nearby declarations or logic blocks.
  **L1586 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1587 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1587 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1588 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1588 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1589 EN**: Executes a call or declaration centered on `vec_sld`.
  **L1589 CN**: 执行以 `vec_sld` 为核心的调用或声明。
- **L1590 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1590 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1591 EN**: Executes a call or declaration centered on `vec_sld`.
  **L1591 CN**: 执行以 `vec_sld` 为核心的调用或声明。
- **L1592 EN**: Closes the current preprocessor conditional block.
  **L1592 CN**: 结束当前预处理条件块。
- **L1593 EN**: Starts the alternative branch of the preceding conditional.
  **L1593 CN**: 开始前一个条件语句的备选分支。
- **L1594 EN**: Adds a standalone statement or declaration: `__result = __zeros;`.
  **L1594 CN**: 添加一条独立语句或声明：`__result = __zeros;`。
- **L1595 EN**: Blank line separating nearby declarations or logic blocks.
  **L1595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1596 EN**: Returns from the current function with `(__m128i)__result`.
  **L1596 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1597 EN**: Closes the current lexical scope or compound statement.
  **L1597 CN**: 结束当前词法作用域或复合语句块。
- **L1598 EN**: Blank line separating nearby declarations or logic blocks.
  **L1598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1599 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1599 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1600 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1600 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1601 EN**: Blank line separating nearby declarations or logic blocks.
  **L1601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1602 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srli_epi16(__m128i __A, int __B) {`.
  **L1602 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srli_epi16(__m128i __A, int __B) {`。
- **L1603 EN**: Adds a standalone statement or declaration: `__v8hu __rshift;`.
  **L1603 CN**: 添加一条独立语句或声明：`__v8hu __rshift;`。
- **L1604 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1604 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1605 EN**: Blank line separating nearby declarations or logic blocks.
  **L1605 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1606 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1606 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1607 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1607 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1608 EN**: Executes a call or declaration centered on `=`.
  **L1608 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 1609-1632

````c
    else
      __rshift = vec_splats((unsigned short)__B);

    __result = vec_sr((__v8hi)__A, __rshift);
  }

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srli_epi32(__m128i __A, int __B) {
  __v4su __rshift;
  __v4si __result = {0, 0, 0, 0};

  if (__B < 32) {
    if (__builtin_constant_p(__B)) {
      if (__B < 16)
        __rshift = (__v4su)vec_splat_s32(__B);
      else
        __rshift = (__v4su)vec_splats((unsigned int)__B);
    } else
      __rshift = vec_splats((unsigned int)__B);

````
- **L1609 EN**: Starts the alternative branch of the preceding conditional.
  **L1609 CN**: 开始前一个条件语句的备选分支。
- **L1610 EN**: Executes a call or declaration centered on `vec_splats`.
  **L1610 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L1611 EN**: Blank line separating nearby declarations or logic blocks.
  **L1611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1612 EN**: Executes a call or declaration centered on `vec_sr`.
  **L1612 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L1613 EN**: Closes the current lexical scope or compound statement.
  **L1613 CN**: 结束当前词法作用域或复合语句块。
- **L1614 EN**: Blank line separating nearby declarations or logic blocks.
  **L1614 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1615 EN**: Returns from the current function with `(__m128i)__result`.
  **L1615 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1616 EN**: Closes the current lexical scope or compound statement.
  **L1616 CN**: 结束当前词法作用域或复合语句块。
- **L1617 EN**: Blank line separating nearby declarations or logic blocks.
  **L1617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1618 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1618 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1619 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1619 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1620 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srli_epi32(__m128i __A, int __B) {`.
  **L1620 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srli_epi32(__m128i __A, int __B) {`。
- **L1621 EN**: Adds a standalone statement or declaration: `__v4su __rshift;`.
  **L1621 CN**: 添加一条独立语句或声明：`__v4su __rshift;`。
- **L1622 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1622 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1623 EN**: Blank line separating nearby declarations or logic blocks.
  **L1623 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1624 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1624 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1625 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1625 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1626 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1626 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1627 EN**: Executes a call or declaration centered on `=`.
  **L1627 CN**: 执行以 `=` 为核心的调用或声明。
- **L1628 EN**: Starts the alternative branch of the preceding conditional.
  **L1628 CN**: 开始前一个条件语句的备选分支。
- **L1629 EN**: Executes a call or declaration centered on `=`.
  **L1629 CN**: 执行以 `=` 为核心的调用或声明。
- **L1630 EN**: Continues the surrounding expression or declaration: `} else`.
  **L1630 CN**: 继续构造周围的表达式或声明：`} else`。
- **L1631 EN**: Executes a call or declaration centered on `vec_splats`.
  **L1631 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L1632 EN**: Blank line separating nearby declarations or logic blocks.
  **L1632 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1633-1656

````c
    __result = vec_sr((__v4si)__A, __rshift);
  }

  return (__m128i)__result;
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srli_epi64(__m128i __A, int __B) {
  __v2du __rshift;
  __v2di __result = {0, 0};

  if (__B < 64) {
    if (__builtin_constant_p(__B)) {
      if (__B < 16)
        __rshift = (__v2du)vec_splat_s32(__B);
      else
        __rshift = (__v2du)vec_splats((unsigned long long)__B);
    } else
      __rshift = (__v2du)vec_splats((unsigned int)__B);

    __result = vec_sr((__v2di)__A, __rshift);
  }
````
- **L1633 EN**: Executes a call or declaration centered on `vec_sr`.
  **L1633 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L1634 EN**: Closes the current lexical scope or compound statement.
  **L1634 CN**: 结束当前词法作用域或复合语句块。
- **L1635 EN**: Blank line separating nearby declarations or logic blocks.
  **L1635 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1636 EN**: Returns from the current function with `(__m128i)__result`.
  **L1636 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1637 EN**: Closes the current lexical scope or compound statement.
  **L1637 CN**: 结束当前词法作用域或复合语句块。
- **L1638 EN**: Blank line separating nearby declarations or logic blocks.
  **L1638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1639 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L1639 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L1640 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1640 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1641 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1641 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1642 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srli_epi64(__m128i __A, int __B) {`.
  **L1642 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srli_epi64(__m128i __A, int __B) {`。
- **L1643 EN**: Adds a standalone statement or declaration: `__v2du __rshift;`.
  **L1643 CN**: 添加一条独立语句或声明：`__v2du __rshift;`。
- **L1644 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1644 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1645 EN**: Blank line separating nearby declarations or logic blocks.
  **L1645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1646 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1646 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1647 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1647 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1648 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1648 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1649 EN**: Executes a call or declaration centered on `=`.
  **L1649 CN**: 执行以 `=` 为核心的调用或声明。
- **L1650 EN**: Starts the alternative branch of the preceding conditional.
  **L1650 CN**: 开始前一个条件语句的备选分支。
- **L1651 EN**: Executes a call or declaration centered on `=`.
  **L1651 CN**: 执行以 `=` 为核心的调用或声明。
- **L1652 EN**: Continues the surrounding expression or declaration: `} else`.
  **L1652 CN**: 继续构造周围的表达式或声明：`} else`。
- **L1653 EN**: Executes a call or declaration centered on `=`.
  **L1653 CN**: 执行以 `=` 为核心的调用或声明。
- **L1654 EN**: Blank line separating nearby declarations or logic blocks.
  **L1654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1655 EN**: Executes a call or declaration centered on `vec_sr`.
  **L1655 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L1656 EN**: Closes the current lexical scope or compound statement.
  **L1656 CN**: 结束当前词法作用域或复合语句块。

### Lines 1657-1680

````c

  return (__m128i)__result;
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sll_epi16(__m128i __A, __m128i __B) {
  __v8hu __lshift;
  __vector __bool short __shmask;
  const __v8hu __shmax = {15, 15, 15, 15, 15, 15, 15, 15};
  __v8hu __result;

#ifdef __LITTLE_ENDIAN__
  __lshift = vec_splat((__v8hu)__B, 0);
#else
  __lshift = vec_splat((__v8hu)__B, 3);
#endif
  __shmask = vec_cmple(__lshift, __shmax);
  __result = vec_sl((__v8hu)__A, __lshift);
  __result = vec_sel((__v8hu)__shmask, __result, __shmask);

  return (__m128i)__result;
}
````
- **L1657 EN**: Blank line separating nearby declarations or logic blocks.
  **L1657 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1658 EN**: Returns from the current function with `(__m128i)__result`.
  **L1658 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1659 EN**: Closes the current lexical scope or compound statement.
  **L1659 CN**: 结束当前词法作用域或复合语句块。
- **L1660 EN**: Closes the current preprocessor conditional block.
  **L1660 CN**: 结束当前预处理条件块。
- **L1661 EN**: Blank line separating nearby declarations or logic blocks.
  **L1661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1662 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1662 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1663 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1663 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1664 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sll_epi16(__m128i __A, __m128i __B) {`.
  **L1664 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sll_epi16(__m128i __A, __m128i __B) {`。
- **L1665 EN**: Adds a standalone statement or declaration: `__v8hu __lshift;`.
  **L1665 CN**: 添加一条独立语句或声明：`__v8hu __lshift;`。
- **L1666 EN**: Adds a standalone statement or declaration: `__vector __bool short __shmask;`.
  **L1666 CN**: 添加一条独立语句或声明：`__vector __bool short __shmask;`。
- **L1667 EN**: Initializes variable `__shmax` from the expression on the right-hand side.
  **L1667 CN**: 使用右侧表达式初始化变量 `__shmax`。
- **L1668 EN**: Adds a standalone statement or declaration: `__v8hu __result;`.
  **L1668 CN**: 添加一条独立语句或声明：`__v8hu __result;`。
- **L1669 EN**: Blank line separating nearby declarations or logic blocks.
  **L1669 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1670 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1670 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1671 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1671 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1672 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1672 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1673 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1673 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1674 EN**: Closes the current preprocessor conditional block.
  **L1674 CN**: 结束当前预处理条件块。
- **L1675 EN**: Executes a call or declaration centered on `vec_cmple`.
  **L1675 CN**: 执行以 `vec_cmple` 为核心的调用或声明。
- **L1676 EN**: Executes a call or declaration centered on `vec_sl`.
  **L1676 CN**: 执行以 `vec_sl` 为核心的调用或声明。
- **L1677 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1677 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1678 EN**: Blank line separating nearby declarations or logic blocks.
  **L1678 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1679 EN**: Returns from the current function with `(__m128i)__result`.
  **L1679 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1680 EN**: Closes the current lexical scope or compound statement.
  **L1680 CN**: 结束当前词法作用域或复合语句块。

### Lines 1681-1704

````c

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sll_epi32(__m128i __A, __m128i __B) {
  __v4su __lshift;
  __vector __bool int __shmask;
  const __v4su __shmax = {32, 32, 32, 32};
  __v4su __result;
#ifdef __LITTLE_ENDIAN__
  __lshift = vec_splat((__v4su)__B, 0);
#else
  __lshift = vec_splat((__v4su)__B, 1);
#endif
  __shmask = vec_cmplt(__lshift, __shmax);
  __result = vec_sl((__v4su)__A, __lshift);
  __result = vec_sel((__v4su)__shmask, __result, __shmask);

  return (__m128i)__result;
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sll_epi64(__m128i __A, __m128i __B) {
````
- **L1681 EN**: Blank line separating nearby declarations or logic blocks.
  **L1681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1682 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1682 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1683 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1683 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1684 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sll_epi32(__m128i __A, __m128i __B) {`.
  **L1684 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sll_epi32(__m128i __A, __m128i __B) {`。
- **L1685 EN**: Adds a standalone statement or declaration: `__v4su __lshift;`.
  **L1685 CN**: 添加一条独立语句或声明：`__v4su __lshift;`。
- **L1686 EN**: Adds a standalone statement or declaration: `__vector __bool int __shmask;`.
  **L1686 CN**: 添加一条独立语句或声明：`__vector __bool int __shmask;`。
- **L1687 EN**: Initializes variable `__shmax` from the expression on the right-hand side.
  **L1687 CN**: 使用右侧表达式初始化变量 `__shmax`。
- **L1688 EN**: Adds a standalone statement or declaration: `__v4su __result;`.
  **L1688 CN**: 添加一条独立语句或声明：`__v4su __result;`。
- **L1689 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1689 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1690 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1690 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1691 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1691 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1692 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1692 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1693 EN**: Closes the current preprocessor conditional block.
  **L1693 CN**: 结束当前预处理条件块。
- **L1694 EN**: Executes a call or declaration centered on `vec_cmplt`.
  **L1694 CN**: 执行以 `vec_cmplt` 为核心的调用或声明。
- **L1695 EN**: Executes a call or declaration centered on `vec_sl`.
  **L1695 CN**: 执行以 `vec_sl` 为核心的调用或声明。
- **L1696 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1696 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1697 EN**: Blank line separating nearby declarations or logic blocks.
  **L1697 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1698 EN**: Returns from the current function with `(__m128i)__result`.
  **L1698 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1699 EN**: Closes the current lexical scope or compound statement.
  **L1699 CN**: 结束当前词法作用域或复合语句块。
- **L1700 EN**: Blank line separating nearby declarations or logic blocks.
  **L1700 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1701 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L1701 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L1702 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1702 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1703 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1703 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1704 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sll_epi64(__m128i __A, __m128i __B) {`.
  **L1704 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sll_epi64(__m128i __A, __m128i __B) {`。

### Lines 1705-1728

````c
  __v2du __lshift;
  __vector __bool long long __shmask;
  const __v2du __shmax = {64, 64};
  __v2du __result;

  __lshift = vec_splat((__v2du)__B, 0);
  __shmask = vec_cmplt(__lshift, __shmax);
  __result = vec_sl((__v2du)__A, __lshift);
  __result = vec_sel((__v2du)__shmask, __result, __shmask);

  return (__m128i)__result;
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sra_epi16(__m128i __A, __m128i __B) {
  const __v8hu __rshmax = {15, 15, 15, 15, 15, 15, 15, 15};
  __v8hu __rshift;
  __v8hi __result;

#ifdef __LITTLE_ENDIAN__
  __rshift = vec_splat((__v8hu)__B, 0);
#else
````
- **L1705 EN**: Adds a standalone statement or declaration: `__v2du __lshift;`.
  **L1705 CN**: 添加一条独立语句或声明：`__v2du __lshift;`。
- **L1706 EN**: Adds a standalone statement or declaration: `__vector __bool long long __shmask;`.
  **L1706 CN**: 添加一条独立语句或声明：`__vector __bool long long __shmask;`。
- **L1707 EN**: Initializes variable `__shmax` from the expression on the right-hand side.
  **L1707 CN**: 使用右侧表达式初始化变量 `__shmax`。
- **L1708 EN**: Adds a standalone statement or declaration: `__v2du __result;`.
  **L1708 CN**: 添加一条独立语句或声明：`__v2du __result;`。
- **L1709 EN**: Blank line separating nearby declarations or logic blocks.
  **L1709 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1710 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1710 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1711 EN**: Executes a call or declaration centered on `vec_cmplt`.
  **L1711 CN**: 执行以 `vec_cmplt` 为核心的调用或声明。
- **L1712 EN**: Executes a call or declaration centered on `vec_sl`.
  **L1712 CN**: 执行以 `vec_sl` 为核心的调用或声明。
- **L1713 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1713 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1714 EN**: Blank line separating nearby declarations or logic blocks.
  **L1714 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1715 EN**: Returns from the current function with `(__m128i)__result`.
  **L1715 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1716 EN**: Closes the current lexical scope or compound statement.
  **L1716 CN**: 结束当前词法作用域或复合语句块。
- **L1717 EN**: Closes the current preprocessor conditional block.
  **L1717 CN**: 结束当前预处理条件块。
- **L1718 EN**: Blank line separating nearby declarations or logic blocks.
  **L1718 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1719 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1719 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1720 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1720 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1721 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sra_epi16(__m128i __A, __m128i __B) {`.
  **L1721 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sra_epi16(__m128i __A, __m128i __B) {`。
- **L1722 EN**: Initializes variable `__rshmax` from the expression on the right-hand side.
  **L1722 CN**: 使用右侧表达式初始化变量 `__rshmax`。
- **L1723 EN**: Adds a standalone statement or declaration: `__v8hu __rshift;`.
  **L1723 CN**: 添加一条独立语句或声明：`__v8hu __rshift;`。
- **L1724 EN**: Adds a standalone statement or declaration: `__v8hi __result;`.
  **L1724 CN**: 添加一条独立语句或声明：`__v8hi __result;`。
- **L1725 EN**: Blank line separating nearby declarations or logic blocks.
  **L1725 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1726 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1726 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1727 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1727 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1728 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1728 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 1729-1752

````c
  __rshift = vec_splat((__v8hu)__B, 3);
#endif
  __rshift = vec_min(__rshift, __rshmax);
  __result = vec_sra((__v8hi)__A, __rshift);

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sra_epi32(__m128i __A, __m128i __B) {
  const __v4su __rshmax = {31, 31, 31, 31};
  __v4su __rshift;
  __v4si __result;

#ifdef __LITTLE_ENDIAN__
  __rshift = vec_splat((__v4su)__B, 0);
#else
  __rshift = vec_splat((__v4su)__B, 1);
#endif
  __rshift = vec_min(__rshift, __rshmax);
  __result = vec_sra((__v4si)__A, __rshift);

  return (__m128i)__result;
````
- **L1729 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1729 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1730 EN**: Closes the current preprocessor conditional block.
  **L1730 CN**: 结束当前预处理条件块。
- **L1731 EN**: Executes a call or declaration centered on `vec_min`.
  **L1731 CN**: 执行以 `vec_min` 为核心的调用或声明。
- **L1732 EN**: Executes a call or declaration centered on `vec_sra`.
  **L1732 CN**: 执行以 `vec_sra` 为核心的调用或声明。
- **L1733 EN**: Blank line separating nearby declarations or logic blocks.
  **L1733 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1734 EN**: Returns from the current function with `(__m128i)__result`.
  **L1734 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1735 EN**: Closes the current lexical scope or compound statement.
  **L1735 CN**: 结束当前词法作用域或复合语句块。
- **L1736 EN**: Blank line separating nearby declarations or logic blocks.
  **L1736 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1737 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1737 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1738 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1738 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1739 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sra_epi32(__m128i __A, __m128i __B) {`.
  **L1739 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sra_epi32(__m128i __A, __m128i __B) {`。
- **L1740 EN**: Initializes variable `__rshmax` from the expression on the right-hand side.
  **L1740 CN**: 使用右侧表达式初始化变量 `__rshmax`。
- **L1741 EN**: Adds a standalone statement or declaration: `__v4su __rshift;`.
  **L1741 CN**: 添加一条独立语句或声明：`__v4su __rshift;`。
- **L1742 EN**: Adds a standalone statement or declaration: `__v4si __result;`.
  **L1742 CN**: 添加一条独立语句或声明：`__v4si __result;`。
- **L1743 EN**: Blank line separating nearby declarations or logic blocks.
  **L1743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1744 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1744 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1745 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1745 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1746 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1746 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1747 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1747 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1748 EN**: Closes the current preprocessor conditional block.
  **L1748 CN**: 结束当前预处理条件块。
- **L1749 EN**: Executes a call or declaration centered on `vec_min`.
  **L1749 CN**: 执行以 `vec_min` 为核心的调用或声明。
- **L1750 EN**: Executes a call or declaration centered on `vec_sra`.
  **L1750 CN**: 执行以 `vec_sra` 为核心的调用或声明。
- **L1751 EN**: Blank line separating nearby declarations or logic blocks.
  **L1751 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1752 EN**: Returns from the current function with `(__m128i)__result`.
  **L1752 CN**: 以 `(__m128i)__result` 从当前函数返回。

### Lines 1753-1776

````c
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srl_epi16(__m128i __A, __m128i __B) {
  __v8hu __rshift;
  __vector __bool short __shmask;
  const __v8hu __shmax = {15, 15, 15, 15, 15, 15, 15, 15};
  __v8hu __result;

#ifdef __LITTLE_ENDIAN__
  __rshift = vec_splat((__v8hu)__B, 0);
#else
  __rshift = vec_splat((__v8hu)__B, 3);
#endif
  __shmask = vec_cmple(__rshift, __shmax);
  __result = vec_sr((__v8hu)__A, __rshift);
  __result = vec_sel((__v8hu)__shmask, __result, __shmask);

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L1753 EN**: Closes the current lexical scope or compound statement.
  **L1753 CN**: 结束当前词法作用域或复合语句块。
- **L1754 EN**: Blank line separating nearby declarations or logic blocks.
  **L1754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1755 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1755 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1756 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1756 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1757 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srl_epi16(__m128i __A, __m128i __B) {`.
  **L1757 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srl_epi16(__m128i __A, __m128i __B) {`。
- **L1758 EN**: Adds a standalone statement or declaration: `__v8hu __rshift;`.
  **L1758 CN**: 添加一条独立语句或声明：`__v8hu __rshift;`。
- **L1759 EN**: Adds a standalone statement or declaration: `__vector __bool short __shmask;`.
  **L1759 CN**: 添加一条独立语句或声明：`__vector __bool short __shmask;`。
- **L1760 EN**: Initializes variable `__shmax` from the expression on the right-hand side.
  **L1760 CN**: 使用右侧表达式初始化变量 `__shmax`。
- **L1761 EN**: Adds a standalone statement or declaration: `__v8hu __result;`.
  **L1761 CN**: 添加一条独立语句或声明：`__v8hu __result;`。
- **L1762 EN**: Blank line separating nearby declarations or logic blocks.
  **L1762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1763 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1763 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1764 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1764 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1765 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1765 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1766 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1766 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1767 EN**: Closes the current preprocessor conditional block.
  **L1767 CN**: 结束当前预处理条件块。
- **L1768 EN**: Executes a call or declaration centered on `vec_cmple`.
  **L1768 CN**: 执行以 `vec_cmple` 为核心的调用或声明。
- **L1769 EN**: Executes a call or declaration centered on `vec_sr`.
  **L1769 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L1770 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1770 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1771 EN**: Blank line separating nearby declarations or logic blocks.
  **L1771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1772 EN**: Returns from the current function with `(__m128i)__result`.
  **L1772 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1773 EN**: Closes the current lexical scope or compound statement.
  **L1773 CN**: 结束当前词法作用域或复合语句块。
- **L1774 EN**: Blank line separating nearby declarations or logic blocks.
  **L1774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1775 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1775 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1776 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1776 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 1777-1800

````c
    _mm_srl_epi32(__m128i __A, __m128i __B) {
  __v4su __rshift;
  __vector __bool int __shmask;
  const __v4su __shmax = {32, 32, 32, 32};
  __v4su __result;

#ifdef __LITTLE_ENDIAN__
  __rshift = vec_splat((__v4su)__B, 0);
#else
  __rshift = vec_splat((__v4su)__B, 1);
#endif
  __shmask = vec_cmplt(__rshift, __shmax);
  __result = vec_sr((__v4su)__A, __rshift);
  __result = vec_sel((__v4su)__shmask, __result, __shmask);

  return (__m128i)__result;
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_srl_epi64(__m128i __A, __m128i __B) {
  __v2du __rshift;
  __vector __bool long long __shmask;
````
- **L1777 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srl_epi32(__m128i __A, __m128i __B) {`.
  **L1777 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srl_epi32(__m128i __A, __m128i __B) {`。
- **L1778 EN**: Adds a standalone statement or declaration: `__v4su __rshift;`.
  **L1778 CN**: 添加一条独立语句或声明：`__v4su __rshift;`。
- **L1779 EN**: Adds a standalone statement or declaration: `__vector __bool int __shmask;`.
  **L1779 CN**: 添加一条独立语句或声明：`__vector __bool int __shmask;`。
- **L1780 EN**: Initializes variable `__shmax` from the expression on the right-hand side.
  **L1780 CN**: 使用右侧表达式初始化变量 `__shmax`。
- **L1781 EN**: Adds a standalone statement or declaration: `__v4su __result;`.
  **L1781 CN**: 添加一条独立语句或声明：`__v4su __result;`。
- **L1782 EN**: Blank line separating nearby declarations or logic blocks.
  **L1782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1783 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1783 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1784 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1784 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1785 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1785 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1786 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1786 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1787 EN**: Closes the current preprocessor conditional block.
  **L1787 CN**: 结束当前预处理条件块。
- **L1788 EN**: Executes a call or declaration centered on `vec_cmplt`.
  **L1788 CN**: 执行以 `vec_cmplt` 为核心的调用或声明。
- **L1789 EN**: Executes a call or declaration centered on `vec_sr`.
  **L1789 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L1790 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1790 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1791 EN**: Blank line separating nearby declarations or logic blocks.
  **L1791 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1792 EN**: Returns from the current function with `(__m128i)__result`.
  **L1792 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1793 EN**: Closes the current lexical scope or compound statement.
  **L1793 CN**: 结束当前词法作用域或复合语句块。
- **L1794 EN**: Blank line separating nearby declarations or logic blocks.
  **L1794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1795 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L1795 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L1796 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1796 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1797 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1797 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1798 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_srl_epi64(__m128i __A, __m128i __B) {`.
  **L1798 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_srl_epi64(__m128i __A, __m128i __B) {`。
- **L1799 EN**: Adds a standalone statement or declaration: `__v2du __rshift;`.
  **L1799 CN**: 添加一条独立语句或声明：`__v2du __rshift;`。
- **L1800 EN**: Adds a standalone statement or declaration: `__vector __bool long long __shmask;`.
  **L1800 CN**: 添加一条独立语句或声明：`__vector __bool long long __shmask;`。

### Lines 1801-1824

````c
  const __v2du __shmax = {64, 64};
  __v2du __result;

  __rshift = vec_splat((__v2du)__B, 0);
  __shmask = vec_cmplt(__rshift, __shmax);
  __result = vec_sr((__v2du)__A, __rshift);
  __result = vec_sel((__v2du)__shmask, __result, __shmask);

  return (__m128i)__result;
}
#endif

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_and_pd(__m128d __A, __m128d __B) {
  return (vec_and((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_andnot_pd(__m128d __A, __m128d __B) {
  return (vec_andc((__v2df)__B, (__v2df)__A));
}

````
- **L1801 EN**: Initializes variable `__shmax` from the expression on the right-hand side.
  **L1801 CN**: 使用右侧表达式初始化变量 `__shmax`。
- **L1802 EN**: Adds a standalone statement or declaration: `__v2du __result;`.
  **L1802 CN**: 添加一条独立语句或声明：`__v2du __result;`。
- **L1803 EN**: Blank line separating nearby declarations or logic blocks.
  **L1803 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1804 EN**: Executes a call or declaration centered on `vec_splat`.
  **L1804 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L1805 EN**: Executes a call or declaration centered on `vec_cmplt`.
  **L1805 CN**: 执行以 `vec_cmplt` 为核心的调用或声明。
- **L1806 EN**: Executes a call or declaration centered on `vec_sr`.
  **L1806 CN**: 执行以 `vec_sr` 为核心的调用或声明。
- **L1807 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1807 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1808 EN**: Blank line separating nearby declarations or logic blocks.
  **L1808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1809 EN**: Returns from the current function with `(__m128i)__result`.
  **L1809 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1810 EN**: Closes the current lexical scope or compound statement.
  **L1810 CN**: 结束当前词法作用域或复合语句块。
- **L1811 EN**: Closes the current preprocessor conditional block.
  **L1811 CN**: 结束当前预处理条件块。
- **L1812 EN**: Blank line separating nearby declarations or logic blocks.
  **L1812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1813 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1813 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1814 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1814 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1815 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_and_pd(__m128d __A, __m128d __B) {`.
  **L1815 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_and_pd(__m128d __A, __m128d __B) {`。
- **L1816 EN**: Returns from the current function with `(vec_and((__v2df)__A, (__v2df)__B))`.
  **L1816 CN**: 以 `(vec_and((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L1817 EN**: Closes the current lexical scope or compound statement.
  **L1817 CN**: 结束当前词法作用域或复合语句块。
- **L1818 EN**: Blank line separating nearby declarations or logic blocks.
  **L1818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1819 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1819 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1820 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1820 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1821 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_andnot_pd(__m128d __A, __m128d __B) {`.
  **L1821 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_andnot_pd(__m128d __A, __m128d __B) {`。
- **L1822 EN**: Returns from the current function with `(vec_andc((__v2df)__B, (__v2df)__A))`.
  **L1822 CN**: 以 `(vec_andc((__v2df)__B, (__v2df)__A))` 从当前函数返回。
- **L1823 EN**: Closes the current lexical scope or compound statement.
  **L1823 CN**: 结束当前词法作用域或复合语句块。
- **L1824 EN**: Blank line separating nearby declarations or logic blocks.
  **L1824 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1825-1848

````c
extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_or_pd(__m128d __A, __m128d __B) {
  return (vec_or((__v2df)__A, (__v2df)__B));
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_xor_pd(__m128d __A, __m128d __B) {
  return (vec_xor((__v2df)__A, (__v2df)__B));
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_and_si128(__m128i __A, __m128i __B) {
  return (__m128i)vec_and((__v2di)__A, (__v2di)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_andnot_si128(__m128i __A, __m128i __B) {
  return (__m128i)vec_andc((__v2di)__B, (__v2di)__A);
}

````
- **L1825 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1825 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1826 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1826 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1827 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_or_pd(__m128d __A, __m128d __B) {`.
  **L1827 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_or_pd(__m128d __A, __m128d __B) {`。
- **L1828 EN**: Returns from the current function with `(vec_or((__v2df)__A, (__v2df)__B))`.
  **L1828 CN**: 以 `(vec_or((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L1829 EN**: Closes the current lexical scope or compound statement.
  **L1829 CN**: 结束当前词法作用域或复合语句块。
- **L1830 EN**: Blank line separating nearby declarations or logic blocks.
  **L1830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1831 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L1831 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L1832 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1832 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1833 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_xor_pd(__m128d __A, __m128d __B) {`.
  **L1833 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_xor_pd(__m128d __A, __m128d __B) {`。
- **L1834 EN**: Returns from the current function with `(vec_xor((__v2df)__A, (__v2df)__B))`.
  **L1834 CN**: 以 `(vec_xor((__v2df)__A, (__v2df)__B))` 从当前函数返回。
- **L1835 EN**: Closes the current lexical scope or compound statement.
  **L1835 CN**: 结束当前词法作用域或复合语句块。
- **L1836 EN**: Blank line separating nearby declarations or logic blocks.
  **L1836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1837 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1837 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1838 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1838 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1839 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_and_si128(__m128i __A, __m128i __B) {`.
  **L1839 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_and_si128(__m128i __A, __m128i __B) {`。
- **L1840 EN**: Returns from the current function with `(__m128i)vec_and((__v2di)__A, (__v2di)__B)`.
  **L1840 CN**: 以 `(__m128i)vec_and((__v2di)__A, (__v2di)__B)` 从当前函数返回。
- **L1841 EN**: Closes the current lexical scope or compound statement.
  **L1841 CN**: 结束当前词法作用域或复合语句块。
- **L1842 EN**: Blank line separating nearby declarations or logic blocks.
  **L1842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1843 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1843 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1844 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1844 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1845 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_andnot_si128(__m128i __A, __m128i __B) {`.
  **L1845 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_andnot_si128(__m128i __A, __m128i __B) {`。
- **L1846 EN**: Returns from the current function with `(__m128i)vec_andc((__v2di)__B, (__v2di)__A)`.
  **L1846 CN**: 以 `(__m128i)vec_andc((__v2di)__B, (__v2di)__A)` 从当前函数返回。
- **L1847 EN**: Closes the current lexical scope or compound statement.
  **L1847 CN**: 结束当前词法作用域或复合语句块。
- **L1848 EN**: Blank line separating nearby declarations or logic blocks.
  **L1848 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1849-1872

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_or_si128(__m128i __A, __m128i __B) {
  return (__m128i)vec_or((__v2di)__A, (__v2di)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_xor_si128(__m128i __A, __m128i __B) {
  return (__m128i)vec_xor((__v2di)__A, (__v2di)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_epi8(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmpeq((__v16qi)__A, (__v16qi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmpeq((__v8hi)__A, (__v8hi)__B);
}

````
- **L1849 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1849 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1850 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1850 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1851 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_or_si128(__m128i __A, __m128i __B) {`.
  **L1851 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_or_si128(__m128i __A, __m128i __B) {`。
- **L1852 EN**: Returns from the current function with `(__m128i)vec_or((__v2di)__A, (__v2di)__B)`.
  **L1852 CN**: 以 `(__m128i)vec_or((__v2di)__A, (__v2di)__B)` 从当前函数返回。
- **L1853 EN**: Closes the current lexical scope or compound statement.
  **L1853 CN**: 结束当前词法作用域或复合语句块。
- **L1854 EN**: Blank line separating nearby declarations or logic blocks.
  **L1854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1855 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1855 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1856 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1856 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1857 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_xor_si128(__m128i __A, __m128i __B) {`.
  **L1857 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_xor_si128(__m128i __A, __m128i __B) {`。
- **L1858 EN**: Returns from the current function with `(__m128i)vec_xor((__v2di)__A, (__v2di)__B)`.
  **L1858 CN**: 以 `(__m128i)vec_xor((__v2di)__A, (__v2di)__B)` 从当前函数返回。
- **L1859 EN**: Closes the current lexical scope or compound statement.
  **L1859 CN**: 结束当前词法作用域或复合语句块。
- **L1860 EN**: Blank line separating nearby declarations or logic blocks.
  **L1860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1861 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1861 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1862 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1862 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1863 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_epi8(__m128i __A, __m128i __B) {`.
  **L1863 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_epi8(__m128i __A, __m128i __B) {`。
- **L1864 EN**: Returns from the current function with `(__m128i)vec_cmpeq((__v16qi)__A, (__v16qi)__B)`.
  **L1864 CN**: 以 `(__m128i)vec_cmpeq((__v16qi)__A, (__v16qi)__B)` 从当前函数返回。
- **L1865 EN**: Closes the current lexical scope or compound statement.
  **L1865 CN**: 结束当前词法作用域或复合语句块。
- **L1866 EN**: Blank line separating nearby declarations or logic blocks.
  **L1866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1867 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1867 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1868 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1868 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1869 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_epi16(__m128i __A, __m128i __B) {`.
  **L1869 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_epi16(__m128i __A, __m128i __B) {`。
- **L1870 EN**: Returns from the current function with `(__m128i)vec_cmpeq((__v8hi)__A, (__v8hi)__B)`.
  **L1870 CN**: 以 `(__m128i)vec_cmpeq((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1871 EN**: Closes the current lexical scope or compound statement.
  **L1871 CN**: 结束当前词法作用域或复合语句块。
- **L1872 EN**: Blank line separating nearby declarations or logic blocks.
  **L1872 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1873-1896

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_epi32(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmpeq((__v4si)__A, (__v4si)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmplt_epi8(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmplt((__v16qi)__A, (__v16qi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmplt_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmplt((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmplt_epi32(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmplt((__v4si)__A, (__v4si)__B);
}

````
- **L1873 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1873 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1874 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1874 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1875 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_epi32(__m128i __A, __m128i __B) {`.
  **L1875 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_epi32(__m128i __A, __m128i __B) {`。
- **L1876 EN**: Returns from the current function with `(__m128i)vec_cmpeq((__v4si)__A, (__v4si)__B)`.
  **L1876 CN**: 以 `(__m128i)vec_cmpeq((__v4si)__A, (__v4si)__B)` 从当前函数返回。
- **L1877 EN**: Closes the current lexical scope or compound statement.
  **L1877 CN**: 结束当前词法作用域或复合语句块。
- **L1878 EN**: Blank line separating nearby declarations or logic blocks.
  **L1878 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1879 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1879 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1880 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1880 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1881 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmplt_epi8(__m128i __A, __m128i __B) {`.
  **L1881 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmplt_epi8(__m128i __A, __m128i __B) {`。
- **L1882 EN**: Returns from the current function with `(__m128i)vec_cmplt((__v16qi)__A, (__v16qi)__B)`.
  **L1882 CN**: 以 `(__m128i)vec_cmplt((__v16qi)__A, (__v16qi)__B)` 从当前函数返回。
- **L1883 EN**: Closes the current lexical scope or compound statement.
  **L1883 CN**: 结束当前词法作用域或复合语句块。
- **L1884 EN**: Blank line separating nearby declarations or logic blocks.
  **L1884 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1885 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1885 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1886 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1886 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1887 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmplt_epi16(__m128i __A, __m128i __B) {`.
  **L1887 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmplt_epi16(__m128i __A, __m128i __B) {`。
- **L1888 EN**: Returns from the current function with `(__m128i)vec_cmplt((__v8hi)__A, (__v8hi)__B)`.
  **L1888 CN**: 以 `(__m128i)vec_cmplt((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1889 EN**: Closes the current lexical scope or compound statement.
  **L1889 CN**: 结束当前词法作用域或复合语句块。
- **L1890 EN**: Blank line separating nearby declarations or logic blocks.
  **L1890 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1891 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1891 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1892 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1892 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1893 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmplt_epi32(__m128i __A, __m128i __B) {`.
  **L1893 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmplt_epi32(__m128i __A, __m128i __B) {`。
- **L1894 EN**: Returns from the current function with `(__m128i)vec_cmplt((__v4si)__A, (__v4si)__B)`.
  **L1894 CN**: 以 `(__m128i)vec_cmplt((__v4si)__A, (__v4si)__B)` 从当前函数返回。
- **L1895 EN**: Closes the current lexical scope or compound statement.
  **L1895 CN**: 结束当前词法作用域或复合语句块。
- **L1896 EN**: Blank line separating nearby declarations or logic blocks.
  **L1896 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1897-1920

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_epi8(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmpgt((__v16qi)__A, (__v16qi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmpgt((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_epi32(__m128i __A, __m128i __B) {
  return (__m128i)vec_cmpgt((__v4si)__A, (__v4si)__B);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_extract_epi16(__m128i const __A, int const __N) {
  return (unsigned short)((__v8hi)__A)[__N & 7];
}

````
- **L1897 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1897 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1898 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1898 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1899 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_epi8(__m128i __A, __m128i __B) {`.
  **L1899 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_epi8(__m128i __A, __m128i __B) {`。
- **L1900 EN**: Returns from the current function with `(__m128i)vec_cmpgt((__v16qi)__A, (__v16qi)__B)`.
  **L1900 CN**: 以 `(__m128i)vec_cmpgt((__v16qi)__A, (__v16qi)__B)` 从当前函数返回。
- **L1901 EN**: Closes the current lexical scope or compound statement.
  **L1901 CN**: 结束当前词法作用域或复合语句块。
- **L1902 EN**: Blank line separating nearby declarations or logic blocks.
  **L1902 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1903 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1903 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1904 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1904 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1905 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_epi16(__m128i __A, __m128i __B) {`.
  **L1905 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_epi16(__m128i __A, __m128i __B) {`。
- **L1906 EN**: Returns from the current function with `(__m128i)vec_cmpgt((__v8hi)__A, (__v8hi)__B)`.
  **L1906 CN**: 以 `(__m128i)vec_cmpgt((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1907 EN**: Closes the current lexical scope or compound statement.
  **L1907 CN**: 结束当前词法作用域或复合语句块。
- **L1908 EN**: Blank line separating nearby declarations or logic blocks.
  **L1908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1909 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1909 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1910 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1910 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1911 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_epi32(__m128i __A, __m128i __B) {`.
  **L1911 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_epi32(__m128i __A, __m128i __B) {`。
- **L1912 EN**: Returns from the current function with `(__m128i)vec_cmpgt((__v4si)__A, (__v4si)__B)`.
  **L1912 CN**: 以 `(__m128i)vec_cmpgt((__v4si)__A, (__v4si)__B)` 从当前函数返回。
- **L1913 EN**: Closes the current lexical scope or compound statement.
  **L1913 CN**: 结束当前词法作用域或复合语句块。
- **L1914 EN**: Blank line separating nearby declarations or logic blocks.
  **L1914 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1915 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1915 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1916 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1916 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1917 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_extract_epi16(__m128i const __A, int const __N) {`.
  **L1917 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_extract_epi16(__m128i const __A, int const __N) {`。
- **L1918 EN**: Returns from the current function with `(unsigned short)((__v8hi)__A)[__N & 7]`.
  **L1918 CN**: 以 `(unsigned short)((__v8hi)__A)[__N & 7]` 从当前函数返回。
- **L1919 EN**: Closes the current lexical scope or compound statement.
  **L1919 CN**: 结束当前词法作用域或复合语句块。
- **L1920 EN**: Blank line separating nearby declarations or logic blocks.
  **L1920 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1921-1944

````c
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_insert_epi16(__m128i const __A, int const __D, int const __N) {
  __v8hi __result = (__v8hi)__A;

  __result[(__N & 7)] = __D;

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_max((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_epu8(__m128i __A, __m128i __B) {
  return (__m128i)vec_max((__v16qu)__A, (__v16qu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L1921 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1921 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1922 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1922 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1923 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_insert_epi16(__m128i const __A, int const __D, int const __N) {`.
  **L1923 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_insert_epi16(__m128i const __A, int const __D, int const __N) {`。
- **L1924 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1924 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1925 EN**: Blank line separating nearby declarations or logic blocks.
  **L1925 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1926 EN**: Executes a call or declaration centered on `__result[`.
  **L1926 CN**: 执行以 `__result[` 为核心的调用或声明。
- **L1927 EN**: Blank line separating nearby declarations or logic blocks.
  **L1927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1928 EN**: Returns from the current function with `(__m128i)__result`.
  **L1928 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L1929 EN**: Closes the current lexical scope or compound statement.
  **L1929 CN**: 结束当前词法作用域或复合语句块。
- **L1930 EN**: Blank line separating nearby declarations or logic blocks.
  **L1930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1931 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1931 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1932 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1932 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1933 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epi16(__m128i __A, __m128i __B) {`.
  **L1933 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epi16(__m128i __A, __m128i __B) {`。
- **L1934 EN**: Returns from the current function with `(__m128i)vec_max((__v8hi)__A, (__v8hi)__B)`.
  **L1934 CN**: 以 `(__m128i)vec_max((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1935 EN**: Closes the current lexical scope or compound statement.
  **L1935 CN**: 结束当前词法作用域或复合语句块。
- **L1936 EN**: Blank line separating nearby declarations or logic blocks.
  **L1936 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1937 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1937 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1938 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1938 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1939 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epu8(__m128i __A, __m128i __B) {`.
  **L1939 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epu8(__m128i __A, __m128i __B) {`。
- **L1940 EN**: Returns from the current function with `(__m128i)vec_max((__v16qu)__A, (__v16qu)__B)`.
  **L1940 CN**: 以 `(__m128i)vec_max((__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L1941 EN**: Closes the current lexical scope or compound statement.
  **L1941 CN**: 结束当前词法作用域或复合语句块。
- **L1942 EN**: Blank line separating nearby declarations or logic blocks.
  **L1942 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1943 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1943 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1944 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1944 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 1945-1968

````c
    _mm_min_epi16(__m128i __A, __m128i __B) {
  return (__m128i)vec_min((__v8hi)__A, (__v8hi)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_epu8(__m128i __A, __m128i __B) {
  return (__m128i)vec_min((__v16qu)__A, (__v16qu)__B);
}

#ifdef _ARCH_PWR8
/* Intrinsic functions that require PowerISA 2.07 minimum.  */

/* Return a mask created from the most significant bit of each 8-bit
   element in A.  */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_movemask_epi8(__m128i __A) {
#ifdef _ARCH_PWR10
  return vec_extractm((__v16qu)__A);
#else
  __vector unsigned long long __result;
  static const __vector unsigned char __perm_mask = {
      0x78, 0x70, 0x68, 0x60, 0x58, 0x50, 0x48, 0x40,
````
- **L1945 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epi16(__m128i __A, __m128i __B) {`.
  **L1945 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epi16(__m128i __A, __m128i __B) {`。
- **L1946 EN**: Returns from the current function with `(__m128i)vec_min((__v8hi)__A, (__v8hi)__B)`.
  **L1946 CN**: 以 `(__m128i)vec_min((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1947 EN**: Closes the current lexical scope or compound statement.
  **L1947 CN**: 结束当前词法作用域或复合语句块。
- **L1948 EN**: Blank line separating nearby declarations or logic blocks.
  **L1948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1949 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1949 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1950 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1950 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1951 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epu8(__m128i __A, __m128i __B) {`.
  **L1951 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epu8(__m128i __A, __m128i __B) {`。
- **L1952 EN**: Returns from the current function with `(__m128i)vec_min((__v16qu)__A, (__v16qu)__B)`.
  **L1952 CN**: 以 `(__m128i)vec_min((__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L1953 EN**: Closes the current lexical scope or compound statement.
  **L1953 CN**: 结束当前词法作用域或复合语句块。
- **L1954 EN**: Blank line separating nearby declarations or logic blocks.
  **L1954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1955 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L1955 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L1956 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsic functions that require PowerISA 2.07 minimum.`.
  **L1956 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsic functions that require PowerISA 2.07 minimum.`。
- **L1957 EN**: Blank line separating nearby declarations or logic blocks.
  **L1957 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1958 EN**: Comment explains nearby logic, constraints, or intent: `Return a mask created from the most significant bit of each 8-bit`.
  **L1958 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Return a mask created from the most significant bit of each 8-bit`。
- **L1959 EN**: Continues the surrounding expression or declaration: `element in A.  */`.
  **L1959 CN**: 继续构造周围的表达式或声明：`element in A.  */`。
- **L1960 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1960 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1961 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1961 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1962 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movemask_epi8(__m128i __A) {`.
  **L1962 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movemask_epi8(__m128i __A) {`。
- **L1963 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR10`.
  **L1963 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR10`。
- **L1964 EN**: Returns from the current function with `vec_extractm((__v16qu)__A)`.
  **L1964 CN**: 以 `vec_extractm((__v16qu)__A)` 从当前函数返回。
- **L1965 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1965 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1966 EN**: Adds a standalone statement or declaration: `__vector unsigned long long __result;`.
  **L1966 CN**: 添加一条独立语句或声明：`__vector unsigned long long __result;`。
- **L1967 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned char __perm_mask = {`.
  **L1967 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned char __perm_mask = {`。
- **L1968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x78, 0x70, 0x68, 0x60, 0x58, 0x50, 0x48, 0x40,`.
  **L1968 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x78, 0x70, 0x68, 0x60, 0x58, 0x50, 0x48, 0x40,`。

### Lines 1969-1992

````c
      0x38, 0x30, 0x28, 0x20, 0x18, 0x10, 0x08, 0x00};

  __result = ((__vector unsigned long long)vec_vbpermq(
      (__vector unsigned char)__A, (__vector unsigned char)__perm_mask));

#ifdef __LITTLE_ENDIAN__
  return __result[1];
#else
  return __result[0];
#endif
#endif /* !_ARCH_PWR10 */
}
#endif /* _ARCH_PWR8 */

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mulhi_epu16(__m128i __A, __m128i __B) {
  __v4su __w0, __w1;
  __v16qu __xform1 = {
#ifdef __LITTLE_ENDIAN__
      0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,
      0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F
#else
      0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x08,
````
- **L1969 EN**: Adds a standalone statement or declaration: `0x38, 0x30, 0x28, 0x20, 0x18, 0x10, 0x08, 0x00};`.
  **L1969 CN**: 添加一条独立语句或声明：`0x38, 0x30, 0x28, 0x20, 0x18, 0x10, 0x08, 0x00};`。
- **L1970 EN**: Blank line separating nearby declarations or logic blocks.
  **L1970 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1971 EN**: Continues logic associated with callable symbol `vec_vbpermq`.
  **L1971 CN**: 继续与可调用符号 `vec_vbpermq` 相关的逻辑。
- **L1972 EN**: Executes a call or declaration centered on `statement`.
  **L1972 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1973 EN**: Blank line separating nearby declarations or logic blocks.
  **L1973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1974 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1974 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1975 EN**: Returns from the current function with `__result[1]`.
  **L1975 CN**: 以 `__result[1]` 从当前函数返回。
- **L1976 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1976 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1977 EN**: Returns from the current function with `__result[0]`.
  **L1977 CN**: 以 `__result[0]` 从当前函数返回。
- **L1978 EN**: Closes the current preprocessor conditional block.
  **L1978 CN**: 结束当前预处理条件块。
- **L1979 EN**: Closes the current preprocessor conditional block.
  **L1979 CN**: 结束当前预处理条件块。
- **L1980 EN**: Closes the current lexical scope or compound statement.
  **L1980 CN**: 结束当前词法作用域或复合语句块。
- **L1981 EN**: Closes the current preprocessor conditional block.
  **L1981 CN**: 结束当前预处理条件块。
- **L1982 EN**: Blank line separating nearby declarations or logic blocks.
  **L1982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1983 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L1983 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L1984 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1984 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1985 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mulhi_epu16(__m128i __A, __m128i __B) {`.
  **L1985 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mulhi_epu16(__m128i __A, __m128i __B) {`。
- **L1986 EN**: Adds a standalone statement or declaration: `__v4su __w0, __w1;`.
  **L1986 CN**: 添加一条独立语句或声明：`__v4su __w0, __w1;`。
- **L1987 EN**: Continues the surrounding expression or declaration: `__v16qu __xform1 = {`.
  **L1987 CN**: 继续构造周围的表达式或声明：`__v16qu __xform1 = {`。
- **L1988 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1988 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,`.
  **L1989 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,`。
- **L1990 EN**: Continues the surrounding expression or declaration: `0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F`.
  **L1990 CN**: 继续构造周围的表达式或声明：`0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F`。
- **L1991 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1991 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x08,`.
  **L1992 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x08,`。

### Lines 1993-2016

````c
      0x09, 0x18, 0x19, 0x0C, 0x0D, 0x1C, 0x1D
#endif
  };

  __w0 = vec_vmuleuh((__v8hu)__A, (__v8hu)__B);
  __w1 = vec_vmulouh((__v8hu)__A, (__v8hu)__B);
  return (__m128i)vec_perm(__w0, __w1, __xform1);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_shufflehi_epi16(__m128i __A, const int __mask) {
  unsigned long __element_selector_98 = __mask & 0x03;
  unsigned long __element_selector_BA = (__mask >> 2) & 0x03;
  unsigned long __element_selector_DC = (__mask >> 4) & 0x03;
  unsigned long __element_selector_FE = (__mask >> 6) & 0x03;
  static const unsigned short __permute_selectors[4] = {
#ifdef __LITTLE_ENDIAN__
      0x0908, 0x0B0A, 0x0D0C, 0x0F0E
#else
      0x0809, 0x0A0B, 0x0C0D, 0x0E0F
#endif
  };
  __v2du __pmask =
````
- **L1993 EN**: Continues the surrounding expression or declaration: `0x09, 0x18, 0x19, 0x0C, 0x0D, 0x1C, 0x1D`.
  **L1993 CN**: 继续构造周围的表达式或声明：`0x09, 0x18, 0x19, 0x0C, 0x0D, 0x1C, 0x1D`。
- **L1994 EN**: Closes the current preprocessor conditional block.
  **L1994 CN**: 结束当前预处理条件块。
- **L1995 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1995 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1996 EN**: Blank line separating nearby declarations or logic blocks.
  **L1996 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1997 EN**: Executes a call or declaration centered on `vec_vmuleuh`.
  **L1997 CN**: 执行以 `vec_vmuleuh` 为核心的调用或声明。
- **L1998 EN**: Executes a call or declaration centered on `vec_vmulouh`.
  **L1998 CN**: 执行以 `vec_vmulouh` 为核心的调用或声明。
- **L1999 EN**: Returns from the current function with `(__m128i)vec_perm(__w0, __w1, __xform1)`.
  **L1999 CN**: 以 `(__m128i)vec_perm(__w0, __w1, __xform1)` 从当前函数返回。
- **L2000 EN**: Closes the current lexical scope or compound statement.
  **L2000 CN**: 结束当前词法作用域或复合语句块。
- **L2001 EN**: Blank line separating nearby declarations or logic blocks.
  **L2001 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2002 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2002 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2003 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2003 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2004 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_shufflehi_epi16(__m128i __A, const int __mask) {`.
  **L2004 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_shufflehi_epi16(__m128i __A, const int __mask) {`。
- **L2005 EN**: Initializes variable `__element_selector_98` from the expression on the right-hand side.
  **L2005 CN**: 使用右侧表达式初始化变量 `__element_selector_98`。
- **L2006 EN**: Initializes variable `__element_selector_BA` from the expression on the right-hand side.
  **L2006 CN**: 使用右侧表达式初始化变量 `__element_selector_BA`。
- **L2007 EN**: Initializes variable `__element_selector_DC` from the expression on the right-hand side.
  **L2007 CN**: 使用右侧表达式初始化变量 `__element_selector_DC`。
- **L2008 EN**: Initializes variable `__element_selector_FE` from the expression on the right-hand side.
  **L2008 CN**: 使用右侧表达式初始化变量 `__element_selector_FE`。
- **L2009 EN**: Continues the surrounding expression or declaration: `static const unsigned short __permute_selectors[4] = {`.
  **L2009 CN**: 继续构造周围的表达式或声明：`static const unsigned short __permute_selectors[4] = {`。
- **L2010 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L2010 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L2011 EN**: Continues the surrounding expression or declaration: `0x0908, 0x0B0A, 0x0D0C, 0x0F0E`.
  **L2011 CN**: 继续构造周围的表达式或声明：`0x0908, 0x0B0A, 0x0D0C, 0x0F0E`。
- **L2012 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2012 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2013 EN**: Continues the surrounding expression or declaration: `0x0809, 0x0A0B, 0x0C0D, 0x0E0F`.
  **L2013 CN**: 继续构造周围的表达式或声明：`0x0809, 0x0A0B, 0x0C0D, 0x0E0F`。
- **L2014 EN**: Closes the current preprocessor conditional block.
  **L2014 CN**: 结束当前预处理条件块。
- **L2015 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2015 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2016 EN**: Continues the surrounding expression or declaration: `__v2du __pmask =`.
  **L2016 CN**: 继续构造周围的表达式或声明：`__v2du __pmask =`。

### Lines 2017-2040

````c
#ifdef __LITTLE_ENDIAN__
      {0x1716151413121110UL, 0UL};
#else
      {0x1011121314151617UL, 0UL};
#endif
  __m64_union __t;
  __v2du __a, __r;

  __t.as_short[0] = __permute_selectors[__element_selector_98];
  __t.as_short[1] = __permute_selectors[__element_selector_BA];
  __t.as_short[2] = __permute_selectors[__element_selector_DC];
  __t.as_short[3] = __permute_selectors[__element_selector_FE];
  __pmask[1] = __t.as_m64;
  __a = (__v2du)__A;
  __r = vec_perm(__a, __a, (__vector unsigned char)__pmask);
  return (__m128i)__r;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_shufflelo_epi16(__m128i __A, const int __mask) {
  unsigned long __element_selector_10 = __mask & 0x03;
  unsigned long __element_selector_32 = (__mask >> 2) & 0x03;
  unsigned long __element_selector_54 = (__mask >> 4) & 0x03;
````
- **L2017 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L2017 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L2018 EN**: Adds a standalone statement or declaration: `{0x1716151413121110UL, 0UL};`.
  **L2018 CN**: 添加一条独立语句或声明：`{0x1716151413121110UL, 0UL};`。
- **L2019 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2019 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2020 EN**: Adds a standalone statement or declaration: `{0x1011121314151617UL, 0UL};`.
  **L2020 CN**: 添加一条独立语句或声明：`{0x1011121314151617UL, 0UL};`。
- **L2021 EN**: Closes the current preprocessor conditional block.
  **L2021 CN**: 结束当前预处理条件块。
- **L2022 EN**: Adds a standalone statement or declaration: `__m64_union __t;`.
  **L2022 CN**: 添加一条独立语句或声明：`__m64_union __t;`。
- **L2023 EN**: Adds a standalone statement or declaration: `__v2du __a, __r;`.
  **L2023 CN**: 添加一条独立语句或声明：`__v2du __a, __r;`。
- **L2024 EN**: Blank line separating nearby declarations or logic blocks.
  **L2024 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2025 EN**: Adds a standalone statement or declaration: `__t.as_short[0] = __permute_selectors[__element_selector_98];`.
  **L2025 CN**: 添加一条独立语句或声明：`__t.as_short[0] = __permute_selectors[__element_selector_98];`。
- **L2026 EN**: Adds a standalone statement or declaration: `__t.as_short[1] = __permute_selectors[__element_selector_BA];`.
  **L2026 CN**: 添加一条独立语句或声明：`__t.as_short[1] = __permute_selectors[__element_selector_BA];`。
- **L2027 EN**: Adds a standalone statement or declaration: `__t.as_short[2] = __permute_selectors[__element_selector_DC];`.
  **L2027 CN**: 添加一条独立语句或声明：`__t.as_short[2] = __permute_selectors[__element_selector_DC];`。
- **L2028 EN**: Adds a standalone statement or declaration: `__t.as_short[3] = __permute_selectors[__element_selector_FE];`.
  **L2028 CN**: 添加一条独立语句或声明：`__t.as_short[3] = __permute_selectors[__element_selector_FE];`。
- **L2029 EN**: Adds a standalone statement or declaration: `__pmask[1] = __t.as_m64;`.
  **L2029 CN**: 添加一条独立语句或声明：`__pmask[1] = __t.as_m64;`。
- **L2030 EN**: Executes a call or declaration centered on `=`.
  **L2030 CN**: 执行以 `=` 为核心的调用或声明。
- **L2031 EN**: Executes a call or declaration centered on `vec_perm`.
  **L2031 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L2032 EN**: Returns from the current function with `(__m128i)__r`.
  **L2032 CN**: 以 `(__m128i)__r` 从当前函数返回。
- **L2033 EN**: Closes the current lexical scope or compound statement.
  **L2033 CN**: 结束当前词法作用域或复合语句块。
- **L2034 EN**: Blank line separating nearby declarations or logic blocks.
  **L2034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2035 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2035 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2036 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2036 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2037 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_shufflelo_epi16(__m128i __A, const int __mask) {`.
  **L2037 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_shufflelo_epi16(__m128i __A, const int __mask) {`。
- **L2038 EN**: Initializes variable `__element_selector_10` from the expression on the right-hand side.
  **L2038 CN**: 使用右侧表达式初始化变量 `__element_selector_10`。
- **L2039 EN**: Initializes variable `__element_selector_32` from the expression on the right-hand side.
  **L2039 CN**: 使用右侧表达式初始化变量 `__element_selector_32`。
- **L2040 EN**: Initializes variable `__element_selector_54` from the expression on the right-hand side.
  **L2040 CN**: 使用右侧表达式初始化变量 `__element_selector_54`。

### Lines 2041-2064

````c
  unsigned long __element_selector_76 = (__mask >> 6) & 0x03;
  static const unsigned short __permute_selectors[4] = {
#ifdef __LITTLE_ENDIAN__
      0x0100, 0x0302, 0x0504, 0x0706
#else
      0x0001, 0x0203, 0x0405, 0x0607
#endif
  };
  __v2du __pmask =
#ifdef __LITTLE_ENDIAN__
      {0UL, 0x1f1e1d1c1b1a1918UL};
#else
      {0UL, 0x18191a1b1c1d1e1fUL};
#endif
  __m64_union __t;
  __v2du __a, __r;
  __t.as_short[0] = __permute_selectors[__element_selector_10];
  __t.as_short[1] = __permute_selectors[__element_selector_32];
  __t.as_short[2] = __permute_selectors[__element_selector_54];
  __t.as_short[3] = __permute_selectors[__element_selector_76];
  __pmask[0] = __t.as_m64;
  __a = (__v2du)__A;
  __r = vec_perm(__a, __a, (__vector unsigned char)__pmask);
  return (__m128i)__r;
````
- **L2041 EN**: Initializes variable `__element_selector_76` from the expression on the right-hand side.
  **L2041 CN**: 使用右侧表达式初始化变量 `__element_selector_76`。
- **L2042 EN**: Continues the surrounding expression or declaration: `static const unsigned short __permute_selectors[4] = {`.
  **L2042 CN**: 继续构造周围的表达式或声明：`static const unsigned short __permute_selectors[4] = {`。
- **L2043 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L2043 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L2044 EN**: Continues the surrounding expression or declaration: `0x0100, 0x0302, 0x0504, 0x0706`.
  **L2044 CN**: 继续构造周围的表达式或声明：`0x0100, 0x0302, 0x0504, 0x0706`。
- **L2045 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2045 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2046 EN**: Continues the surrounding expression or declaration: `0x0001, 0x0203, 0x0405, 0x0607`.
  **L2046 CN**: 继续构造周围的表达式或声明：`0x0001, 0x0203, 0x0405, 0x0607`。
- **L2047 EN**: Closes the current preprocessor conditional block.
  **L2047 CN**: 结束当前预处理条件块。
- **L2048 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2048 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2049 EN**: Continues the surrounding expression or declaration: `__v2du __pmask =`.
  **L2049 CN**: 继续构造周围的表达式或声明：`__v2du __pmask =`。
- **L2050 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L2050 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L2051 EN**: Adds a standalone statement or declaration: `{0UL, 0x1f1e1d1c1b1a1918UL};`.
  **L2051 CN**: 添加一条独立语句或声明：`{0UL, 0x1f1e1d1c1b1a1918UL};`。
- **L2052 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2052 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2053 EN**: Adds a standalone statement or declaration: `{0UL, 0x18191a1b1c1d1e1fUL};`.
  **L2053 CN**: 添加一条独立语句或声明：`{0UL, 0x18191a1b1c1d1e1fUL};`。
- **L2054 EN**: Closes the current preprocessor conditional block.
  **L2054 CN**: 结束当前预处理条件块。
- **L2055 EN**: Adds a standalone statement or declaration: `__m64_union __t;`.
  **L2055 CN**: 添加一条独立语句或声明：`__m64_union __t;`。
- **L2056 EN**: Adds a standalone statement or declaration: `__v2du __a, __r;`.
  **L2056 CN**: 添加一条独立语句或声明：`__v2du __a, __r;`。
- **L2057 EN**: Adds a standalone statement or declaration: `__t.as_short[0] = __permute_selectors[__element_selector_10];`.
  **L2057 CN**: 添加一条独立语句或声明：`__t.as_short[0] = __permute_selectors[__element_selector_10];`。
- **L2058 EN**: Adds a standalone statement or declaration: `__t.as_short[1] = __permute_selectors[__element_selector_32];`.
  **L2058 CN**: 添加一条独立语句或声明：`__t.as_short[1] = __permute_selectors[__element_selector_32];`。
- **L2059 EN**: Adds a standalone statement or declaration: `__t.as_short[2] = __permute_selectors[__element_selector_54];`.
  **L2059 CN**: 添加一条独立语句或声明：`__t.as_short[2] = __permute_selectors[__element_selector_54];`。
- **L2060 EN**: Adds a standalone statement or declaration: `__t.as_short[3] = __permute_selectors[__element_selector_76];`.
  **L2060 CN**: 添加一条独立语句或声明：`__t.as_short[3] = __permute_selectors[__element_selector_76];`。
- **L2061 EN**: Adds a standalone statement or declaration: `__pmask[0] = __t.as_m64;`.
  **L2061 CN**: 添加一条独立语句或声明：`__pmask[0] = __t.as_m64;`。
- **L2062 EN**: Executes a call or declaration centered on `=`.
  **L2062 CN**: 执行以 `=` 为核心的调用或声明。
- **L2063 EN**: Executes a call or declaration centered on `vec_perm`.
  **L2063 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L2064 EN**: Returns from the current function with `(__m128i)__r`.
  **L2064 CN**: 以 `(__m128i)__r` 从当前函数返回。

### Lines 2065-2088

````c
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_shuffle_epi32(__m128i __A, const int __mask) {
  unsigned long __element_selector_10 = __mask & 0x03;
  unsigned long __element_selector_32 = (__mask >> 2) & 0x03;
  unsigned long __element_selector_54 = (__mask >> 4) & 0x03;
  unsigned long __element_selector_76 = (__mask >> 6) & 0x03;
  static const unsigned int __permute_selectors[4] = {
#ifdef __LITTLE_ENDIAN__
      0x03020100, 0x07060504, 0x0B0A0908, 0x0F0E0D0C
#else
      0x00010203, 0x04050607, 0x08090A0B, 0x0C0D0E0F
#endif
  };
  __v4su __t;

  __t[0] = __permute_selectors[__element_selector_10];
  __t[1] = __permute_selectors[__element_selector_32];
  __t[2] = __permute_selectors[__element_selector_54] + 0x10101010;
  __t[3] = __permute_selectors[__element_selector_76] + 0x10101010;
  return (__m128i)vec_perm((__v4si)__A, (__v4si)__A,
                           (__vector unsigned char)__t);
````
- **L2065 EN**: Closes the current lexical scope or compound statement.
  **L2065 CN**: 结束当前词法作用域或复合语句块。
- **L2066 EN**: Blank line separating nearby declarations or logic blocks.
  **L2066 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2067 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2067 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2068 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2068 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2069 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_shuffle_epi32(__m128i __A, const int __mask) {`.
  **L2069 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_shuffle_epi32(__m128i __A, const int __mask) {`。
- **L2070 EN**: Initializes variable `__element_selector_10` from the expression on the right-hand side.
  **L2070 CN**: 使用右侧表达式初始化变量 `__element_selector_10`。
- **L2071 EN**: Initializes variable `__element_selector_32` from the expression on the right-hand side.
  **L2071 CN**: 使用右侧表达式初始化变量 `__element_selector_32`。
- **L2072 EN**: Initializes variable `__element_selector_54` from the expression on the right-hand side.
  **L2072 CN**: 使用右侧表达式初始化变量 `__element_selector_54`。
- **L2073 EN**: Initializes variable `__element_selector_76` from the expression on the right-hand side.
  **L2073 CN**: 使用右侧表达式初始化变量 `__element_selector_76`。
- **L2074 EN**: Continues the surrounding expression or declaration: `static const unsigned int __permute_selectors[4] = {`.
  **L2074 CN**: 继续构造周围的表达式或声明：`static const unsigned int __permute_selectors[4] = {`。
- **L2075 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L2075 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L2076 EN**: Continues the surrounding expression or declaration: `0x03020100, 0x07060504, 0x0B0A0908, 0x0F0E0D0C`.
  **L2076 CN**: 继续构造周围的表达式或声明：`0x03020100, 0x07060504, 0x0B0A0908, 0x0F0E0D0C`。
- **L2077 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2077 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2078 EN**: Continues the surrounding expression or declaration: `0x00010203, 0x04050607, 0x08090A0B, 0x0C0D0E0F`.
  **L2078 CN**: 继续构造周围的表达式或声明：`0x00010203, 0x04050607, 0x08090A0B, 0x0C0D0E0F`。
- **L2079 EN**: Closes the current preprocessor conditional block.
  **L2079 CN**: 结束当前预处理条件块。
- **L2080 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2080 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2081 EN**: Adds a standalone statement or declaration: `__v4su __t;`.
  **L2081 CN**: 添加一条独立语句或声明：`__v4su __t;`。
- **L2082 EN**: Blank line separating nearby declarations or logic blocks.
  **L2082 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2083 EN**: Adds a standalone statement or declaration: `__t[0] = __permute_selectors[__element_selector_10];`.
  **L2083 CN**: 添加一条独立语句或声明：`__t[0] = __permute_selectors[__element_selector_10];`。
- **L2084 EN**: Adds a standalone statement or declaration: `__t[1] = __permute_selectors[__element_selector_32];`.
  **L2084 CN**: 添加一条独立语句或声明：`__t[1] = __permute_selectors[__element_selector_32];`。
- **L2085 EN**: Adds a standalone statement or declaration: `__t[2] = __permute_selectors[__element_selector_54] + 0x10101010;`.
  **L2085 CN**: 添加一条独立语句或声明：`__t[2] = __permute_selectors[__element_selector_54] + 0x10101010;`。
- **L2086 EN**: Adds a standalone statement or declaration: `__t[3] = __permute_selectors[__element_selector_76] + 0x10101010;`.
  **L2086 CN**: 添加一条独立语句或声明：`__t[3] = __permute_selectors[__element_selector_76] + 0x10101010;`。
- **L2087 EN**: Returns from the current function with `(__m128i)vec_perm((__v4si)__A, (__v4si)__A,`.
  **L2087 CN**: 以 `(__m128i)vec_perm((__v4si)__A, (__v4si)__A,` 从当前函数返回。
- **L2088 EN**: Executes a call or declaration centered on `statement`.
  **L2088 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 2089-2112

````c
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_maskmoveu_si128(__m128i __A, __m128i __B, char *__C) {
  __v2du __hibit = {0x7f7f7f7f7f7f7f7fUL, 0x7f7f7f7f7f7f7f7fUL};
  __v16qu __mask, __tmp;
  __m128i_u *__p = (__m128i_u *)__C;

  __tmp = (__v16qu)_mm_loadu_si128(__p);
  __mask = (__v16qu)vec_cmpgt((__v16qu)__B, (__v16qu)__hibit);
  __tmp = vec_sel(__tmp, (__v16qu)__A, __mask);
  _mm_storeu_si128(__p, (__m128i)__tmp);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_avg_epu8(__m128i __A, __m128i __B) {
  return (__m128i)vec_avg((__v16qu)__A, (__v16qu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_avg_epu16(__m128i __A, __m128i __B) {
````
- **L2089 EN**: Closes the current lexical scope or compound statement.
  **L2089 CN**: 结束当前词法作用域或复合语句块。
- **L2090 EN**: Blank line separating nearby declarations or logic blocks.
  **L2090 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2091 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L2091 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L2092 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2092 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2093 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskmoveu_si128(__m128i __A, __m128i __B, char *__C) {`.
  **L2093 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskmoveu_si128(__m128i __A, __m128i __B, char *__C) {`。
- **L2094 EN**: Initializes variable `__hibit` from the expression on the right-hand side.
  **L2094 CN**: 使用右侧表达式初始化变量 `__hibit`。
- **L2095 EN**: Adds a standalone statement or declaration: `__v16qu __mask, __tmp;`.
  **L2095 CN**: 添加一条独立语句或声明：`__v16qu __mask, __tmp;`。
- **L2096 EN**: Executes a call or declaration centered on `=`.
  **L2096 CN**: 执行以 `=` 为核心的调用或声明。
- **L2097 EN**: Blank line separating nearby declarations or logic blocks.
  **L2097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2098 EN**: Executes a call or declaration centered on `=`.
  **L2098 CN**: 执行以 `=` 为核心的调用或声明。
- **L2099 EN**: Executes a call or declaration centered on `=`.
  **L2099 CN**: 执行以 `=` 为核心的调用或声明。
- **L2100 EN**: Executes a call or declaration centered on `vec_sel`.
  **L2100 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L2101 EN**: Executes a call or declaration centered on `_mm_storeu_si128`.
  **L2101 CN**: 执行以 `_mm_storeu_si128` 为核心的调用或声明。
- **L2102 EN**: Closes the current lexical scope or compound statement.
  **L2102 CN**: 结束当前词法作用域或复合语句块。
- **L2103 EN**: Blank line separating nearby declarations or logic blocks.
  **L2103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2104 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2104 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2105 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2105 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2106 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_avg_epu8(__m128i __A, __m128i __B) {`.
  **L2106 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_avg_epu8(__m128i __A, __m128i __B) {`。
- **L2107 EN**: Returns from the current function with `(__m128i)vec_avg((__v16qu)__A, (__v16qu)__B)`.
  **L2107 CN**: 以 `(__m128i)vec_avg((__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L2108 EN**: Closes the current lexical scope or compound statement.
  **L2108 CN**: 结束当前词法作用域或复合语句块。
- **L2109 EN**: Blank line separating nearby declarations or logic blocks.
  **L2109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2110 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2110 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2111 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2111 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2112 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_avg_epu16(__m128i __A, __m128i __B) {`.
  **L2112 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_avg_epu16(__m128i __A, __m128i __B) {`。

### Lines 2113-2136

````c
  return (__m128i)vec_avg((__v8hu)__A, (__v8hu)__B);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sad_epu8(__m128i __A, __m128i __B) {
  __v16qu __a, __b;
  __v16qu __vabsdiff;
  __v4si __vsum;
  const __v4su __zero = {0, 0, 0, 0};
  __v4si __result;

  __a = (__v16qu)__A;
  __b = (__v16qu)__B;
#ifndef _ARCH_PWR9
  __v16qu __vmin = vec_min(__a, __b);
  __v16qu __vmax = vec_max(__a, __b);
  __vabsdiff = vec_sub(__vmax, __vmin);
#else
  __vabsdiff = vec_absd(__a, __b);
#endif
  /* Sum four groups of bytes into integers.  */
  __vsum = (__vector signed int)vec_sum4s(__vabsdiff, __zero);
#ifdef __LITTLE_ENDIAN__
````
- **L2113 EN**: Returns from the current function with `(__m128i)vec_avg((__v8hu)__A, (__v8hu)__B)`.
  **L2113 CN**: 以 `(__m128i)vec_avg((__v8hu)__A, (__v8hu)__B)` 从当前函数返回。
- **L2114 EN**: Closes the current lexical scope or compound statement.
  **L2114 CN**: 结束当前词法作用域或复合语句块。
- **L2115 EN**: Blank line separating nearby declarations or logic blocks.
  **L2115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2116 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2116 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2117 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2117 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2118 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sad_epu8(__m128i __A, __m128i __B) {`.
  **L2118 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sad_epu8(__m128i __A, __m128i __B) {`。
- **L2119 EN**: Adds a standalone statement or declaration: `__v16qu __a, __b;`.
  **L2119 CN**: 添加一条独立语句或声明：`__v16qu __a, __b;`。
- **L2120 EN**: Adds a standalone statement or declaration: `__v16qu __vabsdiff;`.
  **L2120 CN**: 添加一条独立语句或声明：`__v16qu __vabsdiff;`。
- **L2121 EN**: Adds a standalone statement or declaration: `__v4si __vsum;`.
  **L2121 CN**: 添加一条独立语句或声明：`__v4si __vsum;`。
- **L2122 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L2122 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L2123 EN**: Adds a standalone statement or declaration: `__v4si __result;`.
  **L2123 CN**: 添加一条独立语句或声明：`__v4si __result;`。
- **L2124 EN**: Blank line separating nearby declarations or logic blocks.
  **L2124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2125 EN**: Executes a call or declaration centered on `=`.
  **L2125 CN**: 执行以 `=` 为核心的调用或声明。
- **L2126 EN**: Executes a call or declaration centered on `=`.
  **L2126 CN**: 执行以 `=` 为核心的调用或声明。
- **L2127 EN**: Starts a preprocessor conditional block: `#ifndef _ARCH_PWR9`.
  **L2127 CN**: 开始一个预处理条件块：`#ifndef _ARCH_PWR9`。
- **L2128 EN**: Initializes variable `__vmin` from the expression on the right-hand side.
  **L2128 CN**: 使用右侧表达式初始化变量 `__vmin`。
- **L2129 EN**: Initializes variable `__vmax` from the expression on the right-hand side.
  **L2129 CN**: 使用右侧表达式初始化变量 `__vmax`。
- **L2130 EN**: Executes a call or declaration centered on `vec_sub`.
  **L2130 CN**: 执行以 `vec_sub` 为核心的调用或声明。
- **L2131 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2131 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2132 EN**: Executes a call or declaration centered on `vec_absd`.
  **L2132 CN**: 执行以 `vec_absd` 为核心的调用或声明。
- **L2133 EN**: Closes the current preprocessor conditional block.
  **L2133 CN**: 结束当前预处理条件块。
- **L2134 EN**: Comment explains nearby logic, constraints, or intent: `Sum four groups of bytes into integers.`.
  **L2134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sum four groups of bytes into integers.`。
- **L2135 EN**: Executes a call or declaration centered on `=`.
  **L2135 CN**: 执行以 `=` 为核心的调用或声明。
- **L2136 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L2136 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。

### Lines 2137-2160

````c
  /* Sum across four integers with two integer results.  */
  __asm__("vsum2sws %0,%1,%2" : "=v"(__result) : "v"(__vsum), "v"(__zero));
  /* Note: vec_sum2s could be used here, but on little-endian, vector
     shifts are added that are not needed for this use-case.
     A vector shift to correctly position the 32-bit integer results
     (currently at [0] and [2]) to [1] and [3] would then need to be
     swapped back again since the desired results are two 64-bit
     integers ([1]|[0] and [3]|[2]).  Thus, no shift is performed.  */
#else
  /* Sum across four integers with two integer results.  */
  __result = vec_sum2s(__vsum, (__vector signed int)__zero);
  /* Rotate the sums into the correct position.  */
  __result = vec_sld(__result, __result, 6);
#endif
  return (__m128i)__result;
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_stream_si32(int *__A, int __B) {
  /* Use the data cache block touch for store transient.  */
  __asm__("dcbtstt 0,%0" : : "b"(__A) : "memory");
  *__A = __B;
}
````
- **L2137 EN**: Comment explains nearby logic, constraints, or intent: `Sum across four integers with two integer results.`.
  **L2137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sum across four integers with two integer results.`。
- **L2138 EN**: Executes a call or declaration centered on `__asm__`.
  **L2138 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L2139 EN**: Comment highlights an implementation note: `Note: vec_sum2s could be used here, but on little-endian, vector`.
  **L2139 CN**: 注释强调一条实现说明：`Note: vec_sum2s could be used here, but on little-endian, vector`。
- **L2140 EN**: Continues the surrounding expression or declaration: `shifts are added that are not needed for this use-case.`.
  **L2140 CN**: 继续构造周围的表达式或声明：`shifts are added that are not needed for this use-case.`。
- **L2141 EN**: Continues the surrounding expression or declaration: `A vector shift to correctly position the 32-bit integer results`.
  **L2141 CN**: 继续构造周围的表达式或声明：`A vector shift to correctly position the 32-bit integer results`。
- **L2142 EN**: Continues the surrounding expression or declaration: `(currently at [0] and [2]) to [1] and [3] would then need to be`.
  **L2142 CN**: 继续构造周围的表达式或声明：`(currently at [0] and [2]) to [1] and [3] would then need to be`。
- **L2143 EN**: Continues the surrounding expression or declaration: `swapped back again since the desired results are two 64-bit`.
  **L2143 CN**: 继续构造周围的表达式或声明：`swapped back again since the desired results are two 64-bit`。
- **L2144 EN**: Continues logic associated with callable symbol `integers`.
  **L2144 CN**: 继续与可调用符号 `integers` 相关的逻辑。
- **L2145 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2145 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2146 EN**: Comment explains nearby logic, constraints, or intent: `Sum across four integers with two integer results.`.
  **L2146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sum across four integers with two integer results.`。
- **L2147 EN**: Executes a call or declaration centered on `vec_sum2s`.
  **L2147 CN**: 执行以 `vec_sum2s` 为核心的调用或声明。
- **L2148 EN**: Comment explains nearby logic, constraints, or intent: `Rotate the sums into the correct position.`.
  **L2148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rotate the sums into the correct position.`。
- **L2149 EN**: Executes a call or declaration centered on `vec_sld`.
  **L2149 CN**: 执行以 `vec_sld` 为核心的调用或声明。
- **L2150 EN**: Closes the current preprocessor conditional block.
  **L2150 CN**: 结束当前预处理条件块。
- **L2151 EN**: Returns from the current function with `(__m128i)__result`.
  **L2151 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L2152 EN**: Closes the current lexical scope or compound statement.
  **L2152 CN**: 结束当前词法作用域或复合语句块。
- **L2153 EN**: Blank line separating nearby declarations or logic blocks.
  **L2153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2154 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L2154 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L2155 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2155 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_stream_si32(int *__A, int __B) {`.
  **L2156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_stream_si32(int *__A, int __B) {`。
- **L2157 EN**: Comment explains nearby logic, constraints, or intent: `Use the data cache block touch for store transient.`.
  **L2157 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use the data cache block touch for store transient.`。
- **L2158 EN**: Executes a call or declaration centered on `__asm__`.
  **L2158 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L2159 EN**: Comment explains nearby logic, constraints, or intent: `__A __B;`.
  **L2159 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__A __B;`。
- **L2160 EN**: Closes the current lexical scope or compound statement.
  **L2160 CN**: 结束当前词法作用域或复合语句块。

### Lines 2161-2184

````c

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_stream_si64(long long int *__A, long long int __B) {
  /* Use the data cache block touch for store transient.  */
  __asm__("	dcbtstt	0,%0" : : "b"(__A) : "memory");
  *__A = __B;
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_stream_si128(__m128i *__A, __m128i __B) {
  /* Use the data cache block touch for store transient.  */
  __asm__("dcbtstt 0,%0" : : "b"(__A) : "memory");
  *__A = __B;
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_stream_pd(double *__A, __m128d __B) {
  /* Use the data cache block touch for store transient.  */
  __asm__("dcbtstt 0,%0" : : "b"(__A) : "memory");
  *(__m128d *)__A = __B;
}
````
- **L2161 EN**: Blank line separating nearby declarations or logic blocks.
  **L2161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2162 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L2162 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L2163 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2163 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2164 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_stream_si64(long long int *__A, long long int __B) {`.
  **L2164 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_stream_si64(long long int *__A, long long int __B) {`。
- **L2165 EN**: Comment explains nearby logic, constraints, or intent: `Use the data cache block touch for store transient.`.
  **L2165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use the data cache block touch for store transient.`。
- **L2166 EN**: Executes a call or declaration centered on `__asm__`.
  **L2166 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L2167 EN**: Comment explains nearby logic, constraints, or intent: `__A __B;`.
  **L2167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__A __B;`。
- **L2168 EN**: Closes the current lexical scope or compound statement.
  **L2168 CN**: 结束当前词法作用域或复合语句块。
- **L2169 EN**: Blank line separating nearby declarations or logic blocks.
  **L2169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2170 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L2170 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L2171 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2171 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2172 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_stream_si128(__m128i *__A, __m128i __B) {`.
  **L2172 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_stream_si128(__m128i *__A, __m128i __B) {`。
- **L2173 EN**: Comment explains nearby logic, constraints, or intent: `Use the data cache block touch for store transient.`.
  **L2173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use the data cache block touch for store transient.`。
- **L2174 EN**: Executes a call or declaration centered on `__asm__`.
  **L2174 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L2175 EN**: Comment explains nearby logic, constraints, or intent: `__A __B;`.
  **L2175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__A __B;`。
- **L2176 EN**: Closes the current lexical scope or compound statement.
  **L2176 CN**: 结束当前词法作用域或复合语句块。
- **L2177 EN**: Blank line separating nearby declarations or logic blocks.
  **L2177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2178 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L2178 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L2179 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2179 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2180 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_stream_pd(double *__A, __m128d __B) {`.
  **L2180 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_stream_pd(double *__A, __m128d __B) {`。
- **L2181 EN**: Comment explains nearby logic, constraints, or intent: `Use the data cache block touch for store transient.`.
  **L2181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use the data cache block touch for store transient.`。
- **L2182 EN**: Executes a call or declaration centered on `__asm__`.
  **L2182 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L2183 EN**: Comment explains nearby logic, constraints, or intent: `(__m128d *)__A __B;`.
  **L2183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__m128d *)__A __B;`。
- **L2184 EN**: Closes the current lexical scope or compound statement.
  **L2184 CN**: 结束当前词法作用域或复合语句块。

### Lines 2185-2208

````c

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_clflush(void const *__A) {
  /* Use the data cache block flush.  */
  __asm__("dcbf 0,%0" : : "b"(__A) : "memory");
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_lfence(void) {
  /* Use light weight sync for load to load ordering.  */
  __atomic_thread_fence(__ATOMIC_RELEASE);
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mfence(void) {
  /* Use heavy weight sync for any to any ordering.  */
  __atomic_thread_fence(__ATOMIC_SEQ_CST);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L2185 EN**: Blank line separating nearby declarations or logic blocks.
  **L2185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2186 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L2186 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L2187 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2187 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2188 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_clflush(void const *__A) {`.
  **L2188 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_clflush(void const *__A) {`。
- **L2189 EN**: Comment explains nearby logic, constraints, or intent: `Use the data cache block flush.`.
  **L2189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use the data cache block flush.`。
- **L2190 EN**: Executes a call or declaration centered on `__asm__`.
  **L2190 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L2191 EN**: Closes the current lexical scope or compound statement.
  **L2191 CN**: 结束当前词法作用域或复合语句块。
- **L2192 EN**: Blank line separating nearby declarations or logic blocks.
  **L2192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2193 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L2193 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L2194 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2194 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2195 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_lfence(void) {`.
  **L2195 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_lfence(void) {`。
- **L2196 EN**: Comment explains nearby logic, constraints, or intent: `Use light weight sync for load to load ordering.`.
  **L2196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use light weight sync for load to load ordering.`。
- **L2197 EN**: Executes a call or declaration centered on `__atomic_thread_fence`.
  **L2197 CN**: 执行以 `__atomic_thread_fence` 为核心的调用或声明。
- **L2198 EN**: Closes the current lexical scope or compound statement.
  **L2198 CN**: 结束当前词法作用域或复合语句块。
- **L2199 EN**: Blank line separating nearby declarations or logic blocks.
  **L2199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2200 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L2200 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L2201 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2201 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2202 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mfence(void) {`.
  **L2202 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mfence(void) {`。
- **L2203 EN**: Comment explains nearby logic, constraints, or intent: `Use heavy weight sync for any to any ordering.`.
  **L2203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use heavy weight sync for any to any ordering.`。
- **L2204 EN**: Executes a call or declaration centered on `__atomic_thread_fence`.
  **L2204 CN**: 执行以 `__atomic_thread_fence` 为核心的调用或声明。
- **L2205 EN**: Closes the current lexical scope or compound statement.
  **L2205 CN**: 结束当前词法作用域或复合语句块。
- **L2206 EN**: Blank line separating nearby declarations or logic blocks.
  **L2206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2207 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2207 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2208 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2208 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 2209-2232

````c
    _mm_cvtsi32_si128(int __A) {
  return _mm_set_epi32(0, 0, 0, __A);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64_si128(long long __A) {
  return __extension__(__m128i)(__v2di){__A, 0LL};
}

/* Microsoft intrinsic.  */
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64x_si128(long long __A) {
  return __extension__(__m128i)(__v2di){__A, 0LL};
}

/* Casts between various SP, DP, INT vector types.  Note that these do no
   conversion of values, they just change the type.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_castpd_ps(__m128d __A) {
  return (__m128)__A;
}
````
- **L2209 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi32_si128(int __A) {`.
  **L2209 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi32_si128(int __A) {`。
- **L2210 EN**: Returns from the current function with `_mm_set_epi32(0, 0, 0, __A)`.
  **L2210 CN**: 以 `_mm_set_epi32(0, 0, 0, __A)` 从当前函数返回。
- **L2211 EN**: Closes the current lexical scope or compound statement.
  **L2211 CN**: 结束当前词法作用域或复合语句块。
- **L2212 EN**: Blank line separating nearby declarations or logic blocks.
  **L2212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2213 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2213 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2214 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2214 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2215 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64_si128(long long __A) {`.
  **L2215 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64_si128(long long __A) {`。
- **L2216 EN**: Returns from the current function with `__extension__(__m128i)(__v2di){__A, 0LL}`.
  **L2216 CN**: 以 `__extension__(__m128i)(__v2di){__A, 0LL}` 从当前函数返回。
- **L2217 EN**: Closes the current lexical scope or compound statement.
  **L2217 CN**: 结束当前词法作用域或复合语句块。
- **L2218 EN**: Blank line separating nearby declarations or logic blocks.
  **L2218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2219 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L2219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L2220 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2220 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2221 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2221 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2222 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64x_si128(long long __A) {`.
  **L2222 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64x_si128(long long __A) {`。
- **L2223 EN**: Returns from the current function with `__extension__(__m128i)(__v2di){__A, 0LL}`.
  **L2223 CN**: 以 `__extension__(__m128i)(__v2di){__A, 0LL}` 从当前函数返回。
- **L2224 EN**: Closes the current lexical scope or compound statement.
  **L2224 CN**: 结束当前词法作用域或复合语句块。
- **L2225 EN**: Blank line separating nearby declarations or logic blocks.
  **L2225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2226 EN**: Comment highlights an implementation note: `Casts between various SP, DP, INT vector types. Note that these do no`.
  **L2226 CN**: 注释强调一条实现说明：`Casts between various SP, DP, INT vector types. Note that these do no`。
- **L2227 EN**: Continues the surrounding expression or declaration: `conversion of values, they just change the type.  */`.
  **L2227 CN**: 继续构造周围的表达式或声明：`conversion of values, they just change the type.  */`。
- **L2228 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L2228 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L2229 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2229 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2230 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_castpd_ps(__m128d __A) {`.
  **L2230 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_castpd_ps(__m128d __A) {`。
- **L2231 EN**: Returns from the current function with `(__m128)__A`.
  **L2231 CN**: 以 `(__m128)__A` 从当前函数返回。
- **L2232 EN**: Closes the current lexical scope or compound statement.
  **L2232 CN**: 结束当前词法作用域或复合语句块。

### Lines 2233-2256

````c

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_castpd_si128(__m128d __A) {
  return (__m128i)__A;
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_castps_pd(__m128 __A) {
  return (__m128d)__A;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_castps_si128(__m128 __A) {
  return (__m128i)__A;
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_castsi128_ps(__m128i __A) {
  return (__m128)__A;
}
````
- **L2233 EN**: Blank line separating nearby declarations or logic blocks.
  **L2233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2234 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2234 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2235 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2235 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2236 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_castpd_si128(__m128d __A) {`.
  **L2236 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_castpd_si128(__m128d __A) {`。
- **L2237 EN**: Returns from the current function with `(__m128i)__A`.
  **L2237 CN**: 以 `(__m128i)__A` 从当前函数返回。
- **L2238 EN**: Closes the current lexical scope or compound statement.
  **L2238 CN**: 结束当前词法作用域或复合语句块。
- **L2239 EN**: Blank line separating nearby declarations or logic blocks.
  **L2239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2240 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L2240 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L2241 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2241 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2242 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_castps_pd(__m128 __A) {`.
  **L2242 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_castps_pd(__m128 __A) {`。
- **L2243 EN**: Returns from the current function with `(__m128d)__A`.
  **L2243 CN**: 以 `(__m128d)__A` 从当前函数返回。
- **L2244 EN**: Closes the current lexical scope or compound statement.
  **L2244 CN**: 结束当前词法作用域或复合语句块。
- **L2245 EN**: Blank line separating nearby declarations or logic blocks.
  **L2245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2246 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L2246 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L2247 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2247 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2248 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_castps_si128(__m128 __A) {`.
  **L2248 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_castps_si128(__m128 __A) {`。
- **L2249 EN**: Returns from the current function with `(__m128i)__A`.
  **L2249 CN**: 以 `(__m128i)__A` 从当前函数返回。
- **L2250 EN**: Closes the current lexical scope or compound statement.
  **L2250 CN**: 结束当前词法作用域或复合语句块。
- **L2251 EN**: Blank line separating nearby declarations or logic blocks.
  **L2251 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2252 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L2252 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L2253 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2253 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2254 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_castsi128_ps(__m128i __A) {`.
  **L2254 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_castsi128_ps(__m128i __A) {`。
- **L2255 EN**: Returns from the current function with `(__m128)__A`.
  **L2255 CN**: 以 `(__m128)__A` 从当前函数返回。
- **L2256 EN**: Closes the current lexical scope or compound statement.
  **L2256 CN**: 结束当前词法作用域或复合语句块。

### Lines 2257-2269

````c

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_castsi128_pd(__m128i __A) {
  return (__m128d)__A;
}

#else
#include_next <emmintrin.h>
#endif /* defined(__powerpc64__) &&                                            \
        *   (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX)) */

#endif /* EMMINTRIN_H_ */
````
- **L2257 EN**: Blank line separating nearby declarations or logic blocks.
  **L2257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2258 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L2258 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L2259 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L2259 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L2260 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_castsi128_pd(__m128i __A) {`.
  **L2260 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_castsi128_pd(__m128i __A) {`。
- **L2261 EN**: Returns from the current function with `(__m128d)__A`.
  **L2261 CN**: 以 `(__m128d)__A` 从当前函数返回。
- **L2262 EN**: Closes the current lexical scope or compound statement.
  **L2262 CN**: 结束当前词法作用域或复合语句块。
- **L2263 EN**: Blank line separating nearby declarations or logic blocks.
  **L2263 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2264 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2264 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2265 EN**: Includes <emmintrin.h> to access related header declarations.
  **L2265 CN**: 引入 <emmintrin.h> 以使用相关头文件声明。
- **L2266 EN**: Closes the current preprocessor conditional block.
  **L2266 CN**: 结束当前预处理条件块。
- **L2267 EN**: Comment explains nearby logic, constraints, or intent: `(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`.
  **L2267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`。
- **L2268 EN**: Blank line separating nearby declarations or logic blocks.
  **L2268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2269 EN**: Closes the current preprocessor conditional block.
  **L2269 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **PowerPC vector or system interfaces / PowerPC 向量或系统接口**
- **wrapper forwarding layers / 包装转发层**
- **Header composition through includes / 通过 include 组合头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `altivec.h`: Provides related header declarations. / 提供相关头文件声明。
  - `xmmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
  - `emmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `NO_WARN_X86_INTRINSICS`, `EMMINTRIN_H_`, `__powerpc64__`, `__linux__`, `__FreeBSD__`, `_AIX`, `_ARCH_PWR8`, `__LITTLE_ENDIAN__`, `vec_doubleh`, `_ARCH_PWR10`, `_ARCH_PWR9`
- **External builtins / 外部 builtin**: `__builtin_vsx_xxsldwi`, `__builtin_constant_p`
