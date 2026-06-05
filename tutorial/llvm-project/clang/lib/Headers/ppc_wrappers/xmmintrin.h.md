# xmmintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/ppc_wrappers/xmmintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implementation of SSE intrinsics on PowerPC.
- **Purpose (CN)**: 该头文件主要作用是：Implementation of SSE intrinsics on PowerPC。
- **Line Count / 行数**: 1827

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- xmmintrin.h - Implementation of SSE intrinsics on PowerPC --------===
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

   Since X86 SSE intrinsics mainly handles __m128 type, PowerPC
   VMX/VSX ISA is a good match for vector float SIMD operations.
   However scalar float operations in vector (XMM) registers require
   the POWER8 VSX ISA (2.07) level. There are differences for data
   format and placement of float scalars in the vector register, which
   require extra steps to match SSE scalar float semantics on POWER.

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
- **L17 EN**: Continues the surrounding expression or declaration: `Since X86 SSE intrinsics mainly handles __m128 type, PowerPC`.
  **L17 CN**: 继续构造周围的表达式或声明：`Since X86 SSE intrinsics mainly handles __m128 type, PowerPC`。
- **L18 EN**: Continues the surrounding expression or declaration: `VMX/VSX ISA is a good match for vector float SIMD operations.`.
  **L18 CN**: 继续构造周围的表达式或声明：`VMX/VSX ISA is a good match for vector float SIMD operations.`。
- **L19 EN**: Continues logic associated with callable symbol `vector`.
  **L19 CN**: 继续与可调用符号 `vector` 相关的逻辑。
- **L20 EN**: Continues logic associated with callable symbol `ISA`.
  **L20 CN**: 继续与可调用符号 `ISA` 相关的逻辑。
- **L21 EN**: Continues the surrounding expression or declaration: `format and placement of float scalars in the vector register, which`.
  **L21 CN**: 继续构造周围的表达式或声明：`format and placement of float scalars in the vector register, which`。
- **L22 EN**: Continues the surrounding expression or declaration: `require extra steps to match SSE scalar float semantics on POWER.`.
  **L22 CN**: 继续构造周围的表达式或声明：`require extra steps to match SSE scalar float semantics on POWER.`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Continues the surrounding expression or declaration: `It should be noted that there's much difference between X86_64's`.
  **L24 CN**: 继续构造周围的表达式或声明：`It should be noted that there's much difference between X86_64's`。

### Lines 25-48

````c
   MXSCR and PowerISA's FPSCR/VSCR registers. It's recommended to use
   portable <fenv.h> instead of access MXSCR directly.

   Most SSE scalar float intrinsic operations can be performed more
   efficiently as C language float scalar operations or optimized to
   use vector SIMD operations. We recommend this for new applications. */
#error                                                                         \
    "Please read comment above. Use -DNO_WARN_X86_INTRINSICS to disable this error."
#endif

#ifndef XMMINTRIN_H_
#define XMMINTRIN_H_

#if defined(__powerpc64__) &&                                                  \
    (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))

/* Define four value permute mask */
#define _MM_SHUFFLE(w, x, y, z) (((w) << 6) | ((x) << 4) | ((y) << 2) | (z))

#include <altivec.h>

/* Avoid collisions between altivec.h and strict adherence to C++ and
   C11 standards.  This should eventually be done inside altivec.h itself,
   but only after testing a full distro build.  */
````
- **L25 EN**: Continues the surrounding expression or declaration: `MXSCR and PowerISA's FPSCR/VSCR registers. It's recommended to use`.
  **L25 CN**: 继续构造周围的表达式或声明：`MXSCR and PowerISA's FPSCR/VSCR registers. It's recommended to use`。
- **L26 EN**: Continues the surrounding expression or declaration: `portable <fenv.h> instead of access MXSCR directly.`.
  **L26 CN**: 继续构造周围的表达式或声明：`portable <fenv.h> instead of access MXSCR directly.`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Continues the surrounding expression or declaration: `Most SSE scalar float intrinsic operations can be performed more`.
  **L28 CN**: 继续构造周围的表达式或声明：`Most SSE scalar float intrinsic operations can be performed more`。
- **L29 EN**: Continues the surrounding expression or declaration: `efficiently as C language float scalar operations or optimized to`.
  **L29 CN**: 继续构造周围的表达式或声明：`efficiently as C language float scalar operations or optimized to`。
- **L30 EN**: Continues the surrounding expression or declaration: `use vector SIMD operations. We recommend this for new applications. */`.
  **L30 CN**: 继续构造周围的表达式或声明：`use vector SIMD operations. We recommend this for new applications. */`。
- **L31 EN**: Emits a compilation error for an unsupported configuration: `#error                                                                         \`.
  **L31 CN**: 为不受支持的配置触发编译错误：`#error                                                                         \`。
- **L32 EN**: Continues the surrounding expression or declaration: `"Please read comment above. Use -DNO_WARN_X86_INTRINSICS to disable this error."`.
  **L32 CN**: 继续构造周围的表达式或声明：`"Please read comment above. Use -DNO_WARN_X86_INTRINSICS to disable this error."`。
- **L33 EN**: Closes the current preprocessor conditional block.
  **L33 CN**: 结束当前预处理条件块。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Starts a preprocessor conditional block: `#ifndef XMMINTRIN_H_`.
  **L35 CN**: 开始一个预处理条件块：`#ifndef XMMINTRIN_H_`。
- **L36 EN**: Defines macro `XMMINTRIN_H_` for conditional compilation, shorthand, or API generation.
  **L36 CN**: 定义宏 `XMMINTRIN_H_`，用于条件编译、简写或 API 生成。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Starts a preprocessor conditional block: `#if defined(__powerpc64__) &&                                                  \`.
  **L38 CN**: 开始一个预处理条件块：`#if defined(__powerpc64__) &&                                                  \`。
- **L39 EN**: Continues logic associated with callable symbol `defined`.
  **L39 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L41 EN**: Comment explains nearby logic, constraints, or intent: `Define four value permute mask`.
  **L41 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define four value permute mask`。
- **L42 EN**: Defines macro `_MM_SHUFFLE(w, x, y, z)` for conditional compilation, shorthand, or API generation.
  **L42 CN**: 定义宏 `_MM_SHUFFLE(w, x, y, z)`，用于条件编译、简写或 API 生成。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L44 EN**: Includes <altivec.h> to access related header declarations.
  **L44 CN**: 引入 <altivec.h> 以使用相关头文件声明。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L46 EN**: Comment explains nearby logic, constraints, or intent: `Avoid collisions between altivec.h and strict adherence to C++ and`.
  **L46 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Avoid collisions between altivec.h and strict adherence to C++ and`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C11 standards.  This should eventually be done inside altivec.h itself,`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`C11 standards.  This should eventually be done inside altivec.h itself,`。
- **L48 EN**: Continues the surrounding expression or declaration: `but only after testing a full distro build.  */`.
  **L48 CN**: 继续构造周围的表达式或声明：`but only after testing a full distro build.  */`。

### Lines 49-72

````c
#if defined(__STRICT_ANSI__) &&                                                \
    (defined(__cplusplus) ||                                                   \
     (defined(__STDC_VERSION__) && __STDC_VERSION__ >= 201112L))
#undef vector
#undef pixel
#undef bool
#endif

/* We need type definitions from the MMX header file.  */
#include <mmintrin.h>

/* Get _mm_malloc () and _mm_free ().  */
#if __STDC_HOSTED__
#include <mm_malloc.h>
#endif

/* The Intel API is flexible enough that we must allow aliasing with other
   vector types, and their scalar components.  */
typedef vector float __m128 __attribute__((__may_alias__));

/* Unaligned version of the same type.  */
typedef vector float __m128_u __attribute__((__may_alias__, __aligned__(1)));

/* Internal data types for implementing the intrinsics.  */
````
- **L49 EN**: Starts a preprocessor conditional block: `#if defined(__STRICT_ANSI__) &&                                                \`.
  **L49 CN**: 开始一个预处理条件块：`#if defined(__STRICT_ANSI__) &&                                                \`。
- **L50 EN**: Continues logic associated with callable symbol `defined`.
  **L50 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L51 EN**: Continues logic associated with callable symbol `defined`.
  **L51 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L52 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef vector`.
  **L52 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef vector`。
- **L53 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef pixel`.
  **L53 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef pixel`。
- **L54 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef bool`.
  **L54 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef bool`。
- **L55 EN**: Closes the current preprocessor conditional block.
  **L55 CN**: 结束当前预处理条件块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `We need type definitions from the MMX header file.`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`We need type definitions from the MMX header file.`。
- **L58 EN**: Includes <mmintrin.h> to access related header declarations.
  **L58 CN**: 引入 <mmintrin.h> 以使用相关头文件声明。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L60 EN**: Comment explains nearby logic, constraints, or intent: `Get _mm_malloc () and _mm_free ().`.
  **L60 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Get _mm_malloc () and _mm_free ().`。
- **L61 EN**: Starts a preprocessor conditional block: `#if __STDC_HOSTED__`.
  **L61 CN**: 开始一个预处理条件块：`#if __STDC_HOSTED__`。
- **L62 EN**: Includes <mm_malloc.h> to access related header declarations.
  **L62 CN**: 引入 <mm_malloc.h> 以使用相关头文件声明。
- **L63 EN**: Closes the current preprocessor conditional block.
  **L63 CN**: 结束当前预处理条件块。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `The Intel API is flexible enough that we must allow aliasing with other`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The Intel API is flexible enough that we must allow aliasing with other`。
- **L66 EN**: Continues the surrounding expression or declaration: `vector types, and their scalar components.  */`.
  **L66 CN**: 继续构造周围的表达式或声明：`vector types, and their scalar components.  */`。
- **L67 EN**: Introduces an alias or helper declaration: `typedef vector float __m128 __attribute__((__may_alias__));`.
  **L67 CN**: 引入一条别名或辅助声明：`typedef vector float __m128 __attribute__((__may_alias__));`。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L69 EN**: Comment explains nearby logic, constraints, or intent: `Unaligned version of the same type.`.
  **L69 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unaligned version of the same type.`。
- **L70 EN**: Introduces an alias or helper declaration: `typedef vector float __m128_u __attribute__((__may_alias__, __aligned__(1)));`.
  **L70 CN**: 引入一条别名或辅助声明：`typedef vector float __m128_u __attribute__((__may_alias__, __aligned__(1)));`。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Comment explains nearby logic, constraints, or intent: `Internal data types for implementing the intrinsics.`.
  **L72 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Internal data types for implementing the intrinsics.`。

### Lines 73-96

````c
typedef vector float __v4sf;

/* Create an undefined vector.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_undefined_ps(void) {
  __m128 __Y = __Y;
  return __Y;
}

/* Create a vector of zeros.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setzero_ps(void) {
  return __extension__(__m128){0.0f, 0.0f, 0.0f, 0.0f};
}

/* Load four SPFP values from P.  The address must be 16-byte aligned.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load_ps(float const *__P) {
  return ((__m128)vec_ld(0, (__v4sf *)__P));
}

````
- **L73 EN**: Introduces an alias or helper declaration: `typedef vector float __v4sf;`.
  **L73 CN**: 引入一条别名或辅助声明：`typedef vector float __v4sf;`。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `Create an undefined vector.`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create an undefined vector.`。
- **L76 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L76 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L77 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L77 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L78 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_undefined_ps(void) {`.
  **L78 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_undefined_ps(void) {`。
- **L79 EN**: Initializes variable `__Y` from the expression on the right-hand side.
  **L79 CN**: 使用右侧表达式初始化变量 `__Y`。
- **L80 EN**: Returns from the current function with `__Y`.
  **L80 CN**: 以 `__Y` 从当前函数返回。
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L83 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector of zeros.`.
  **L83 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector of zeros.`。
- **L84 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L84 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L85 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L85 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L86 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setzero_ps(void) {`.
  **L86 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setzero_ps(void) {`。
- **L87 EN**: Returns from the current function with `__extension__(__m128){0.0f, 0.0f, 0.0f, 0.0f}`.
  **L87 CN**: 以 `__extension__(__m128){0.0f, 0.0f, 0.0f, 0.0f}` 从当前函数返回。
- **L88 EN**: Closes the current lexical scope or compound statement.
  **L88 CN**: 结束当前词法作用域或复合语句块。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L90 EN**: Comment explains nearby logic, constraints, or intent: `Load four SPFP values from P. The address must be 16-byte aligned.`.
  **L90 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load four SPFP values from P. The address must be 16-byte aligned.`。
- **L91 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L91 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L92 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L92 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L93 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load_ps(float const *__P) {`.
  **L93 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load_ps(float const *__P) {`。
- **L94 EN**: Returns from the current function with `((__m128)vec_ld(0, (__v4sf *)__P))`.
  **L94 CN**: 以 `((__m128)vec_ld(0, (__v4sf *)__P))` 从当前函数返回。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 97-120

````c
/* Load four SPFP values from P.  The address need not be 16-byte aligned.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadu_ps(float const *__P) {
  return (vec_vsx_ld(0, __P));
}

/* Load four SPFP values in reverse order.  The address must be aligned.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadr_ps(float const *__P) {
  __v4sf __tmp;
  __m128 __result;
  static const __vector unsigned char __permute_vector = {
      0x1C, 0x1D, 0x1E, 0x1F, 0x18, 0x19, 0x1A, 0x1B,
      0x14, 0x15, 0x16, 0x17, 0x10, 0x11, 0x12, 0x13};

  __tmp = vec_ld(0, (__v4sf *)__P);
  __result = (__m128)vec_perm(__tmp, __tmp, __permute_vector);
  return __result;
}

/* Create a vector with all four elements equal to F.  */
extern __inline __m128
````
- **L97 EN**: Comment explains nearby logic, constraints, or intent: `Load four SPFP values from P. The address need not be 16-byte aligned.`.
  **L97 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load four SPFP values from P. The address need not be 16-byte aligned.`。
- **L98 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L98 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L99 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L99 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L100 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadu_ps(float const *__P) {`.
  **L100 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadu_ps(float const *__P) {`。
- **L101 EN**: Returns from the current function with `(vec_vsx_ld(0, __P))`.
  **L101 CN**: 以 `(vec_vsx_ld(0, __P))` 从当前函数返回。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L104 EN**: Comment explains nearby logic, constraints, or intent: `Load four SPFP values in reverse order. The address must be aligned.`.
  **L104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load four SPFP values in reverse order. The address must be aligned.`。
- **L105 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L105 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L106 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L106 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L107 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadr_ps(float const *__P) {`.
  **L107 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadr_ps(float const *__P) {`。
- **L108 EN**: Adds a standalone statement or declaration: `__v4sf __tmp;`.
  **L108 CN**: 添加一条独立语句或声明：`__v4sf __tmp;`。
- **L109 EN**: Adds a standalone statement or declaration: `__m128 __result;`.
  **L109 CN**: 添加一条独立语句或声明：`__m128 __result;`。
- **L110 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned char __permute_vector = {`.
  **L110 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned char __permute_vector = {`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1C, 0x1D, 0x1E, 0x1F, 0x18, 0x19, 0x1A, 0x1B,`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1C, 0x1D, 0x1E, 0x1F, 0x18, 0x19, 0x1A, 0x1B,`。
- **L112 EN**: Adds a standalone statement or declaration: `0x14, 0x15, 0x16, 0x17, 0x10, 0x11, 0x12, 0x13};`.
  **L112 CN**: 添加一条独立语句或声明：`0x14, 0x15, 0x16, 0x17, 0x10, 0x11, 0x12, 0x13};`。
- **L113 EN**: Blank line separating nearby declarations or logic blocks.
  **L113 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L114 EN**: Executes a call or declaration centered on `vec_ld`.
  **L114 CN**: 执行以 `vec_ld` 为核心的调用或声明。
- **L115 EN**: Executes a call or declaration centered on `=`.
  **L115 CN**: 执行以 `=` 为核心的调用或声明。
- **L116 EN**: Returns from the current function with `__result`.
  **L116 CN**: 以 `__result` 从当前函数返回。
- **L117 EN**: Closes the current lexical scope or compound statement.
  **L117 CN**: 结束当前词法作用域或复合语句块。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L119 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with all four elements equal to F.`.
  **L119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with all four elements equal to F.`。
- **L120 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L120 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。

### Lines 121-144

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set1_ps(float __F) {
  return __extension__(__m128)(__v4sf){__F, __F, __F, __F};
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_ps1(float __F) {
  return _mm_set1_ps(__F);
}

/* Create the vector [Z Y X W].  */
extern __inline __m128 __attribute__((__gnu_inline__, __always_inline__,
                                      __artificial__))
_mm_set_ps(const float __Z, const float __Y, const float __X, const float __W) {
  return __extension__(__m128)(__v4sf){__W, __X, __Y, __Z};
}

/* Create the vector [W X Y Z].  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_setr_ps(float __Z, float __Y, float __X, float __W) {
  return __extension__(__m128)(__v4sf){__Z, __Y, __X, __W};
}
````
- **L121 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L121 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L122 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set1_ps(float __F) {`.
  **L122 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set1_ps(float __F) {`。
- **L123 EN**: Returns from the current function with `__extension__(__m128)(__v4sf){__F, __F, __F, __F}`.
  **L123 CN**: 以 `__extension__(__m128)(__v4sf){__F, __F, __F, __F}` 从当前函数返回。
- **L124 EN**: Closes the current lexical scope or compound statement.
  **L124 CN**: 结束当前词法作用域或复合语句块。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L126 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L126 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L127 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L127 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L128 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_ps1(float __F) {`.
  **L128 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_ps1(float __F) {`。
- **L129 EN**: Returns from the current function with `_mm_set1_ps(__F)`.
  **L129 CN**: 以 `_mm_set1_ps(__F)` 从当前函数返回。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L132 EN**: Comment explains nearby logic, constraints, or intent: `Create the vector [Z Y X W].`.
  **L132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create the vector [Z Y X W].`。
- **L133 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `extern __inline __m128 __attribute__((__gnu_inline__, __always_inline__,`.
  **L133 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`extern __inline __m128 __attribute__((__gnu_inline__, __always_inline__,`。
- **L134 EN**: Continues the surrounding expression or declaration: `__artificial__))`.
  **L134 CN**: 继续构造周围的表达式或声明：`__artificial__))`。
- **L135 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_ps(const float __Z, const float __Y, const float __X, const float __W) {`.
  **L135 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_ps(const float __Z, const float __Y, const float __X, const float __W) {`。
- **L136 EN**: Returns from the current function with `__extension__(__m128)(__v4sf){__W, __X, __Y, __Z}`.
  **L136 CN**: 以 `__extension__(__m128)(__v4sf){__W, __X, __Y, __Z}` 从当前函数返回。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L139 EN**: Comment explains nearby logic, constraints, or intent: `Create the vector [W X Y Z].`.
  **L139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create the vector [W X Y Z].`。
- **L140 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L140 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L141 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L141 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_setr_ps(float __Z, float __Y, float __X, float __W) {`.
  **L142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_setr_ps(float __Z, float __Y, float __X, float __W) {`。
- **L143 EN**: Returns from the current function with `__extension__(__m128)(__v4sf){__Z, __Y, __X, __W}`.
  **L143 CN**: 以 `__extension__(__m128)(__v4sf){__Z, __Y, __X, __W}` 从当前函数返回。
- **L144 EN**: Closes the current lexical scope or compound statement.
  **L144 CN**: 结束当前词法作用域或复合语句块。

### Lines 145-168

````c

/* Store four SPFP values.  The address must be 16-byte aligned.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store_ps(float *__P, __m128 __A) {
  vec_st((__v4sf)__A, 0, (__v4sf *)__P);
}

/* Store four SPFP values.  The address need not be 16-byte aligned.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storeu_ps(float *__P, __m128 __A) {
  *(__m128_u *)__P = __A;
}

/* Store four SPFP values in reverse order.  The address must be aligned.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storer_ps(float *__P, __m128 __A) {
  __v4sf __tmp;
  static const __vector unsigned char __permute_vector = {
      0x1C, 0x1D, 0x1E, 0x1F, 0x18, 0x19, 0x1A, 0x1B,
      0x14, 0x15, 0x16, 0x17, 0x10, 0x11, 0x12, 0x13};

````
- **L145 EN**: Blank line separating nearby declarations or logic blocks.
  **L145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L146 EN**: Comment explains nearby logic, constraints, or intent: `Store four SPFP values. The address must be 16-byte aligned.`.
  **L146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store four SPFP values. The address must be 16-byte aligned.`。
- **L147 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L147 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L148 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L148 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store_ps(float *__P, __m128 __A) {`.
  **L149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store_ps(float *__P, __m128 __A) {`。
- **L150 EN**: Executes a call or declaration centered on `vec_st`.
  **L150 CN**: 执行以 `vec_st` 为核心的调用或声明。
- **L151 EN**: Closes the current lexical scope or compound statement.
  **L151 CN**: 结束当前词法作用域或复合语句块。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L153 EN**: Comment explains nearby logic, constraints, or intent: `Store four SPFP values. The address need not be 16-byte aligned.`.
  **L153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store four SPFP values. The address need not be 16-byte aligned.`。
- **L154 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L154 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L155 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L155 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storeu_ps(float *__P, __m128 __A) {`.
  **L156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storeu_ps(float *__P, __m128 __A) {`。
- **L157 EN**: Comment explains nearby logic, constraints, or intent: `(__m128_u *)__P __A;`.
  **L157 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__m128_u *)__P __A;`。
- **L158 EN**: Closes the current lexical scope or compound statement.
  **L158 CN**: 结束当前词法作用域或复合语句块。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L160 EN**: Comment explains nearby logic, constraints, or intent: `Store four SPFP values in reverse order. The address must be aligned.`.
  **L160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store four SPFP values in reverse order. The address must be aligned.`。
- **L161 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L161 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L162 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L162 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storer_ps(float *__P, __m128 __A) {`.
  **L163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storer_ps(float *__P, __m128 __A) {`。
- **L164 EN**: Adds a standalone statement or declaration: `__v4sf __tmp;`.
  **L164 CN**: 添加一条独立语句或声明：`__v4sf __tmp;`。
- **L165 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned char __permute_vector = {`.
  **L165 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned char __permute_vector = {`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1C, 0x1D, 0x1E, 0x1F, 0x18, 0x19, 0x1A, 0x1B,`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1C, 0x1D, 0x1E, 0x1F, 0x18, 0x19, 0x1A, 0x1B,`。
- **L167 EN**: Adds a standalone statement or declaration: `0x14, 0x15, 0x16, 0x17, 0x10, 0x11, 0x12, 0x13};`.
  **L167 CN**: 添加一条独立语句或声明：`0x14, 0x15, 0x16, 0x17, 0x10, 0x11, 0x12, 0x13};`。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 169-192

````c
  __tmp = (__m128)vec_perm(__A, __A, __permute_vector);

  _mm_store_ps(__P, __tmp);
}

/* Store the lower SPFP value across four words.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store1_ps(float *__P, __m128 __A) {
  __v4sf __va = vec_splat((__v4sf)__A, 0);
  _mm_store_ps(__P, __va);
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store_ps1(float *__P, __m128 __A) {
  _mm_store1_ps(__P, __A);
}

/* Create a vector with element 0 as F and the rest zero.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_set_ss(float __F) {
  return __extension__(__m128)(__v4sf){__F, 0.0f, 0.0f, 0.0f};
````
- **L169 EN**: Executes a call or declaration centered on `=`.
  **L169 CN**: 执行以 `=` 为核心的调用或声明。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L171 EN**: Executes a call or declaration centered on `_mm_store_ps`.
  **L171 CN**: 执行以 `_mm_store_ps` 为核心的调用或声明。
- **L172 EN**: Closes the current lexical scope or compound statement.
  **L172 CN**: 结束当前词法作用域或复合语句块。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L174 EN**: Comment explains nearby logic, constraints, or intent: `Store the lower SPFP value across four words.`.
  **L174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store the lower SPFP value across four words.`。
- **L175 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L175 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L176 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L176 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L177 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store1_ps(float *__P, __m128 __A) {`.
  **L177 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store1_ps(float *__P, __m128 __A) {`。
- **L178 EN**: Initializes variable `__va` from the expression on the right-hand side.
  **L178 CN**: 使用右侧表达式初始化变量 `__va`。
- **L179 EN**: Executes a call or declaration centered on `_mm_store_ps`.
  **L179 CN**: 执行以 `_mm_store_ps` 为核心的调用或声明。
- **L180 EN**: Closes the current lexical scope or compound statement.
  **L180 CN**: 结束当前词法作用域或复合语句块。
- **L181 EN**: Blank line separating nearby declarations or logic blocks.
  **L181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L182 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L182 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L183 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L183 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L184 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store_ps1(float *__P, __m128 __A) {`.
  **L184 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store_ps1(float *__P, __m128 __A) {`。
- **L185 EN**: Executes a call or declaration centered on `_mm_store1_ps`.
  **L185 CN**: 执行以 `_mm_store1_ps` 为核心的调用或声明。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L188 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with element 0 as F and the rest zero.`.
  **L188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with element 0 as F and the rest zero.`。
- **L189 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L189 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L190 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L190 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L191 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_set_ss(float __F) {`.
  **L191 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_set_ss(float __F) {`。
- **L192 EN**: Returns from the current function with `__extension__(__m128)(__v4sf){__F, 0.0f, 0.0f, 0.0f}`.
  **L192 CN**: 以 `__extension__(__m128)(__v4sf){__F, 0.0f, 0.0f, 0.0f}` 从当前函数返回。

### Lines 193-216

````c
}

/* Sets the low SPFP value of A from the low value of B.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_move_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};

  return (vec_sel((__v4sf)__A, (__v4sf)__B, __mask));
}

/* Create a vector with element 0 as *P and the rest zero.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load_ss(float const *__P) {
  return _mm_set_ss(*__P);
}

/* Stores the lower SPFP value.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_store_ss(float *__P, __m128 __A) {
  *__P = ((__v4sf)__A)[0];
}
````
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L195 EN**: Comment explains nearby logic, constraints, or intent: `Sets the low SPFP value of A from the low value of B.`.
  **L195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sets the low SPFP value of A from the low value of B.`。
- **L196 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L196 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L197 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L197 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L198 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_move_ss(__m128 __A, __m128 __B) {`.
  **L198 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_move_ss(__m128 __A, __m128 __B) {`。
- **L199 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L199 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L201 EN**: Returns from the current function with `(vec_sel((__v4sf)__A, (__v4sf)__B, __mask))`.
  **L201 CN**: 以 `(vec_sel((__v4sf)__A, (__v4sf)__B, __mask))` 从当前函数返回。
- **L202 EN**: Closes the current lexical scope or compound statement.
  **L202 CN**: 结束当前词法作用域或复合语句块。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L204 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with element 0 as *P and the rest zero.`.
  **L204 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with element 0 as *P and the rest zero.`。
- **L205 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L205 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L206 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L206 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L207 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load_ss(float const *__P) {`.
  **L207 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load_ss(float const *__P) {`。
- **L208 EN**: Returns from the current function with `_mm_set_ss(*__P)`.
  **L208 CN**: 以 `_mm_set_ss(*__P)` 从当前函数返回。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L211 EN**: Comment explains nearby logic, constraints, or intent: `Stores the lower SPFP value.`.
  **L211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores the lower SPFP value.`。
- **L212 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L212 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L213 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L213 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L214 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_store_ss(float *__P, __m128 __A) {`.
  **L214 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_store_ss(float *__P, __m128 __A) {`。
- **L215 EN**: Comment explains nearby logic, constraints, or intent: `__P ((__v4sf)__A)[0];`.
  **L215 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__P ((__v4sf)__A)[0];`。
- **L216 EN**: Closes the current lexical scope or compound statement.
  **L216 CN**: 结束当前词法作用域或复合语句块。

### Lines 217-240

````c

/* Perform the respective operation on the lower SPFP (single-precision
   floating-point) values of A and B; the upper three SPFP values are
   passed through from A.  */

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_ss(__m128 __A, __m128 __B) {
#ifdef _ARCH_PWR7
  __m128 __a, __b, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower double)
     results. So to insure we don't generate spurious exceptions
     (from the upper double values) we splat the lower double
     before we to the operation.  */
  __a = vec_splat(__A, 0);
  __b = vec_splat(__B, 0);
  __c = __a + __b;
  /* Then we merge the lower float result with the original upper
     float elements from __A.  */
  return (vec_sel(__A, __c, __mask));
#else
  __A[0] = __A[0] + __B[0];
  return (__A);
````
- **L217 EN**: Blank line separating nearby declarations or logic blocks.
  **L217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L218 EN**: Comment explains nearby logic, constraints, or intent: `Perform the respective operation on the lower SPFP (single-precision`.
  **L218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Perform the respective operation on the lower SPFP (single-precision`。
- **L219 EN**: Continues the surrounding expression or declaration: `floating-point) values of A and B; the upper three SPFP values are`.
  **L219 CN**: 继续构造周围的表达式或声明：`floating-point) values of A and B; the upper three SPFP values are`。
- **L220 EN**: Continues the surrounding expression or declaration: `passed through from A.  */`.
  **L220 CN**: 继续构造周围的表达式或声明：`passed through from A.  */`。
- **L221 EN**: Blank line separating nearby declarations or logic blocks.
  **L221 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L222 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L222 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L223 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L223 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L224 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_ss(__m128 __A, __m128 __B) {`.
  **L224 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_ss(__m128 __A, __m128 __B) {`。
- **L225 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR7`.
  **L225 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR7`。
- **L226 EN**: Adds a standalone statement or declaration: `__m128 __a, __b, __c;`.
  **L226 CN**: 添加一条独立语句或声明：`__m128 __a, __b, __c;`。
- **L227 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L227 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L228 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower double)`.
  **L228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower double)`。
- **L229 EN**: Continues the surrounding expression or declaration: `results. So to insure we don't generate spurious exceptions`.
  **L229 CN**: 继续构造周围的表达式或声明：`results. So to insure we don't generate spurious exceptions`。
- **L230 EN**: Continues the surrounding expression or declaration: `(from the upper double values) we splat the lower double`.
  **L230 CN**: 继续构造周围的表达式或声明：`(from the upper double values) we splat the lower double`。
- **L231 EN**: Continues the surrounding expression or declaration: `before we to the operation.  */`.
  **L231 CN**: 继续构造周围的表达式或声明：`before we to the operation.  */`。
- **L232 EN**: Executes a call or declaration centered on `vec_splat`.
  **L232 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L233 EN**: Executes a call or declaration centered on `vec_splat`.
  **L233 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L234 EN**: Adds a standalone statement or declaration: `__c = __a + __b;`.
  **L234 CN**: 添加一条独立语句或声明：`__c = __a + __b;`。
- **L235 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L236 EN**: Continues the surrounding expression or declaration: `float elements from __A.  */`.
  **L236 CN**: 继续构造周围的表达式或声明：`float elements from __A.  */`。
- **L237 EN**: Returns from the current function with `(vec_sel(__A, __c, __mask))`.
  **L237 CN**: 以 `(vec_sel(__A, __c, __mask))` 从当前函数返回。
- **L238 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L238 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L239 EN**: Adds a standalone statement or declaration: `__A[0] = __A[0] + __B[0];`.
  **L239 CN**: 添加一条独立语句或声明：`__A[0] = __A[0] + __B[0];`。
- **L240 EN**: Returns from the current function with `(__A)`.
  **L240 CN**: 以 `(__A)` 从当前函数返回。

### Lines 241-264

````c
#endif
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_ss(__m128 __A, __m128 __B) {
#ifdef _ARCH_PWR7
  __m128 __a, __b, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower double)
     results. So to insure we don't generate spurious exceptions
     (from the upper double values) we splat the lower double
     before we to the operation.  */
  __a = vec_splat(__A, 0);
  __b = vec_splat(__B, 0);
  __c = __a - __b;
  /* Then we merge the lower float result with the original upper
     float elements from __A.  */
  return (vec_sel(__A, __c, __mask));
#else
  __A[0] = __A[0] - __B[0];
  return (__A);
#endif
}
````
- **L241 EN**: Closes the current preprocessor conditional block.
  **L241 CN**: 结束当前预处理条件块。
- **L242 EN**: Closes the current lexical scope or compound statement.
  **L242 CN**: 结束当前词法作用域或复合语句块。
- **L243 EN**: Blank line separating nearby declarations or logic blocks.
  **L243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L244 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L244 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L245 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L245 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L246 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_ss(__m128 __A, __m128 __B) {`.
  **L246 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_ss(__m128 __A, __m128 __B) {`。
- **L247 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR7`.
  **L247 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR7`。
- **L248 EN**: Adds a standalone statement or declaration: `__m128 __a, __b, __c;`.
  **L248 CN**: 添加一条独立语句或声明：`__m128 __a, __b, __c;`。
- **L249 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L249 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L250 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower double)`.
  **L250 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower double)`。
- **L251 EN**: Continues the surrounding expression or declaration: `results. So to insure we don't generate spurious exceptions`.
  **L251 CN**: 继续构造周围的表达式或声明：`results. So to insure we don't generate spurious exceptions`。
- **L252 EN**: Continues the surrounding expression or declaration: `(from the upper double values) we splat the lower double`.
  **L252 CN**: 继续构造周围的表达式或声明：`(from the upper double values) we splat the lower double`。
- **L253 EN**: Continues the surrounding expression or declaration: `before we to the operation.  */`.
  **L253 CN**: 继续构造周围的表达式或声明：`before we to the operation.  */`。
- **L254 EN**: Executes a call or declaration centered on `vec_splat`.
  **L254 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L255 EN**: Executes a call or declaration centered on `vec_splat`.
  **L255 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L256 EN**: Adds a standalone statement or declaration: `__c = __a - __b;`.
  **L256 CN**: 添加一条独立语句或声明：`__c = __a - __b;`。
- **L257 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L258 EN**: Continues the surrounding expression or declaration: `float elements from __A.  */`.
  **L258 CN**: 继续构造周围的表达式或声明：`float elements from __A.  */`。
- **L259 EN**: Returns from the current function with `(vec_sel(__A, __c, __mask))`.
  **L259 CN**: 以 `(vec_sel(__A, __c, __mask))` 从当前函数返回。
- **L260 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L260 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L261 EN**: Adds a standalone statement or declaration: `__A[0] = __A[0] - __B[0];`.
  **L261 CN**: 添加一条独立语句或声明：`__A[0] = __A[0] - __B[0];`。
- **L262 EN**: Returns from the current function with `(__A)`.
  **L262 CN**: 以 `(__A)` 从当前函数返回。
- **L263 EN**: Closes the current preprocessor conditional block.
  **L263 CN**: 结束当前预处理条件块。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。

### Lines 265-288

````c

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mul_ss(__m128 __A, __m128 __B) {
#ifdef _ARCH_PWR7
  __m128 __a, __b, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower double)
     results. So to insure we don't generate spurious exceptions
     (from the upper double values) we splat the lower double
     before we to the operation.  */
  __a = vec_splat(__A, 0);
  __b = vec_splat(__B, 0);
  __c = __a * __b;
  /* Then we merge the lower float result with the original upper
     float elements from __A.  */
  return (vec_sel(__A, __c, __mask));
#else
  __A[0] = __A[0] * __B[0];
  return (__A);
#endif
}

extern __inline __m128
````
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L266 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L266 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L267 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L267 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L268 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mul_ss(__m128 __A, __m128 __B) {`.
  **L268 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mul_ss(__m128 __A, __m128 __B) {`。
- **L269 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR7`.
  **L269 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR7`。
- **L270 EN**: Adds a standalone statement or declaration: `__m128 __a, __b, __c;`.
  **L270 CN**: 添加一条独立语句或声明：`__m128 __a, __b, __c;`。
- **L271 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L271 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L272 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower double)`.
  **L272 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower double)`。
- **L273 EN**: Continues the surrounding expression or declaration: `results. So to insure we don't generate spurious exceptions`.
  **L273 CN**: 继续构造周围的表达式或声明：`results. So to insure we don't generate spurious exceptions`。
- **L274 EN**: Continues the surrounding expression or declaration: `(from the upper double values) we splat the lower double`.
  **L274 CN**: 继续构造周围的表达式或声明：`(from the upper double values) we splat the lower double`。
- **L275 EN**: Continues the surrounding expression or declaration: `before we to the operation.  */`.
  **L275 CN**: 继续构造周围的表达式或声明：`before we to the operation.  */`。
- **L276 EN**: Executes a call or declaration centered on `vec_splat`.
  **L276 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L277 EN**: Executes a call or declaration centered on `vec_splat`.
  **L277 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L278 EN**: Adds a standalone statement or declaration: `__c = __a * __b;`.
  **L278 CN**: 添加一条独立语句或声明：`__c = __a * __b;`。
- **L279 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L280 EN**: Continues the surrounding expression or declaration: `float elements from __A.  */`.
  **L280 CN**: 继续构造周围的表达式或声明：`float elements from __A.  */`。
- **L281 EN**: Returns from the current function with `(vec_sel(__A, __c, __mask))`.
  **L281 CN**: 以 `(vec_sel(__A, __c, __mask))` 从当前函数返回。
- **L282 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L282 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L283 EN**: Adds a standalone statement or declaration: `__A[0] = __A[0] * __B[0];`.
  **L283 CN**: 添加一条独立语句或声明：`__A[0] = __A[0] * __B[0];`。
- **L284 EN**: Returns from the current function with `(__A)`.
  **L284 CN**: 以 `(__A)` 从当前函数返回。
- **L285 EN**: Closes the current preprocessor conditional block.
  **L285 CN**: 结束当前预处理条件块。
- **L286 EN**: Closes the current lexical scope or compound statement.
  **L286 CN**: 结束当前词法作用域或复合语句块。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L288 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L288 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。

### Lines 289-312

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_div_ss(__m128 __A, __m128 __B) {
#ifdef _ARCH_PWR7
  __m128 __a, __b, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower double)
     results. So to insure we don't generate spurious exceptions
     (from the upper double values) we splat the lower double
     before we to the operation.  */
  __a = vec_splat(__A, 0);
  __b = vec_splat(__B, 0);
  __c = __a / __b;
  /* Then we merge the lower float result with the original upper
     float elements from __A.  */
  return (vec_sel(__A, __c, __mask));
#else
  __A[0] = __A[0] / __B[0];
  return (__A);
#endif
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sqrt_ss(__m128 __A) {
````
- **L289 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L289 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L290 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_div_ss(__m128 __A, __m128 __B) {`.
  **L290 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_div_ss(__m128 __A, __m128 __B) {`。
- **L291 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR7`.
  **L291 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR7`。
- **L292 EN**: Adds a standalone statement or declaration: `__m128 __a, __b, __c;`.
  **L292 CN**: 添加一条独立语句或声明：`__m128 __a, __b, __c;`。
- **L293 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L293 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L294 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower double)`.
  **L294 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower double)`。
- **L295 EN**: Continues the surrounding expression or declaration: `results. So to insure we don't generate spurious exceptions`.
  **L295 CN**: 继续构造周围的表达式或声明：`results. So to insure we don't generate spurious exceptions`。
- **L296 EN**: Continues the surrounding expression or declaration: `(from the upper double values) we splat the lower double`.
  **L296 CN**: 继续构造周围的表达式或声明：`(from the upper double values) we splat the lower double`。
- **L297 EN**: Continues the surrounding expression or declaration: `before we to the operation.  */`.
  **L297 CN**: 继续构造周围的表达式或声明：`before we to the operation.  */`。
- **L298 EN**: Executes a call or declaration centered on `vec_splat`.
  **L298 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L299 EN**: Executes a call or declaration centered on `vec_splat`.
  **L299 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L300 EN**: Adds a standalone statement or declaration: `__c = __a / __b;`.
  **L300 CN**: 添加一条独立语句或声明：`__c = __a / __b;`。
- **L301 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L302 EN**: Continues the surrounding expression or declaration: `float elements from __A.  */`.
  **L302 CN**: 继续构造周围的表达式或声明：`float elements from __A.  */`。
- **L303 EN**: Returns from the current function with `(vec_sel(__A, __c, __mask))`.
  **L303 CN**: 以 `(vec_sel(__A, __c, __mask))` 从当前函数返回。
- **L304 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L304 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L305 EN**: Adds a standalone statement or declaration: `__A[0] = __A[0] / __B[0];`.
  **L305 CN**: 添加一条独立语句或声明：`__A[0] = __A[0] / __B[0];`。
- **L306 EN**: Returns from the current function with `(__A)`.
  **L306 CN**: 以 `(__A)` 从当前函数返回。
- **L307 EN**: Closes the current preprocessor conditional block.
  **L307 CN**: 结束当前预处理条件块。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L310 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L311 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L311 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L312 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sqrt_ss(__m128 __A) {`.
  **L312 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sqrt_ss(__m128 __A) {`。

### Lines 313-336

````c
  __m128 __a, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower double)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper double values) we splat the lower double
   * before we to the operation. */
  __a = vec_splat(__A, 0);
  __c = vec_sqrt(__a);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return (vec_sel(__A, __c, __mask));
}

/* Perform the respective operation on the four SPFP values in A and B.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_add_ps(__m128 __A, __m128 __B) {
  return (__m128)((__v4sf)__A + (__v4sf)__B);
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sub_ps(__m128 __A, __m128 __B) {
  return (__m128)((__v4sf)__A - (__v4sf)__B);
````
- **L313 EN**: Adds a standalone statement or declaration: `__m128 __a, __c;`.
  **L313 CN**: 添加一条独立语句或声明：`__m128 __a, __c;`。
- **L314 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L314 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L315 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower double)`.
  **L315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower double)`。
- **L316 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L317 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper double values) we splat the lower double`.
  **L317 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper double values) we splat the lower double`。
- **L318 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L318 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L319 EN**: Executes a call or declaration centered on `vec_splat`.
  **L319 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L320 EN**: Executes a call or declaration centered on `vec_sqrt`.
  **L320 CN**: 执行以 `vec_sqrt` 为核心的调用或声明。
- **L321 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L321 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L322 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L322 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L323 EN**: Returns from the current function with `(vec_sel(__A, __c, __mask))`.
  **L323 CN**: 以 `(vec_sel(__A, __c, __mask))` 从当前函数返回。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L326 EN**: Comment explains nearby logic, constraints, or intent: `Perform the respective operation on the four SPFP values in A and B.`.
  **L326 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Perform the respective operation on the four SPFP values in A and B.`。
- **L327 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L327 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L328 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L328 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L329 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_ps(__m128 __A, __m128 __B) {`.
  **L329 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_ps(__m128 __A, __m128 __B) {`。
- **L330 EN**: Returns from the current function with `(__m128)((__v4sf)__A + (__v4sf)__B)`.
  **L330 CN**: 以 `(__m128)((__v4sf)__A + (__v4sf)__B)` 从当前函数返回。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L333 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L334 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L334 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L335 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sub_ps(__m128 __A, __m128 __B) {`.
  **L335 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sub_ps(__m128 __A, __m128 __B) {`。
- **L336 EN**: Returns from the current function with `(__m128)((__v4sf)__A - (__v4sf)__B)`.
  **L336 CN**: 以 `(__m128)((__v4sf)__A - (__v4sf)__B)` 从当前函数返回。

### Lines 337-360

````c
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mul_ps(__m128 __A, __m128 __B) {
  return (__m128)((__v4sf)__A * (__v4sf)__B);
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_div_ps(__m128 __A, __m128 __B) {
  return (__m128)((__v4sf)__A / (__v4sf)__B);
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sqrt_ps(__m128 __A) {
  return (vec_sqrt((__v4sf)__A));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_rcp_ps(__m128 __A) {
  return (vec_re((__v4sf)__A));
````
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L339 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L340 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L340 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L341 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mul_ps(__m128 __A, __m128 __B) {`.
  **L341 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mul_ps(__m128 __A, __m128 __B) {`。
- **L342 EN**: Returns from the current function with `(__m128)((__v4sf)__A * (__v4sf)__B)`.
  **L342 CN**: 以 `(__m128)((__v4sf)__A * (__v4sf)__B)` 从当前函数返回。
- **L343 EN**: Closes the current lexical scope or compound statement.
  **L343 CN**: 结束当前词法作用域或复合语句块。
- **L344 EN**: Blank line separating nearby declarations or logic blocks.
  **L344 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L345 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L345 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L346 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L346 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L347 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_div_ps(__m128 __A, __m128 __B) {`.
  **L347 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_div_ps(__m128 __A, __m128 __B) {`。
- **L348 EN**: Returns from the current function with `(__m128)((__v4sf)__A / (__v4sf)__B)`.
  **L348 CN**: 以 `(__m128)((__v4sf)__A / (__v4sf)__B)` 从当前函数返回。
- **L349 EN**: Closes the current lexical scope or compound statement.
  **L349 CN**: 结束当前词法作用域或复合语句块。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L351 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L351 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L352 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L352 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L353 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sqrt_ps(__m128 __A) {`.
  **L353 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sqrt_ps(__m128 __A) {`。
- **L354 EN**: Returns from the current function with `(vec_sqrt((__v4sf)__A))`.
  **L354 CN**: 以 `(vec_sqrt((__v4sf)__A))` 从当前函数返回。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L357 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L357 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L358 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L358 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L359 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_rcp_ps(__m128 __A) {`.
  **L359 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_rcp_ps(__m128 __A) {`。
- **L360 EN**: Returns from the current function with `(vec_re((__v4sf)__A))`.
  **L360 CN**: 以 `(vec_re((__v4sf)__A))` 从当前函数返回。

### Lines 361-384

````c
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_rsqrt_ps(__m128 __A) {
  return (vec_rsqrte(__A));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_rcp_ss(__m128 __A) {
  __m128 __a, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower double)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper double values) we splat the lower double
   * before we to the operation. */
  __a = vec_splat(__A, 0);
  __c = _mm_rcp_ps(__a);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return (vec_sel(__A, __c, __mask));
}

````
- **L361 EN**: Closes the current lexical scope or compound statement.
  **L361 CN**: 结束当前词法作用域或复合语句块。
- **L362 EN**: Blank line separating nearby declarations or logic blocks.
  **L362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L363 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L363 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L364 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L364 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L365 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_rsqrt_ps(__m128 __A) {`.
  **L365 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_rsqrt_ps(__m128 __A) {`。
- **L366 EN**: Returns from the current function with `(vec_rsqrte(__A))`.
  **L366 CN**: 以 `(vec_rsqrte(__A))` 从当前函数返回。
- **L367 EN**: Closes the current lexical scope or compound statement.
  **L367 CN**: 结束当前词法作用域或复合语句块。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L369 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L369 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L370 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L370 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L371 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_rcp_ss(__m128 __A) {`.
  **L371 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_rcp_ss(__m128 __A) {`。
- **L372 EN**: Adds a standalone statement or declaration: `__m128 __a, __c;`.
  **L372 CN**: 添加一条独立语句或声明：`__m128 __a, __c;`。
- **L373 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L373 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L374 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower double)`.
  **L374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower double)`。
- **L375 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L376 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper double values) we splat the lower double`.
  **L376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper double values) we splat the lower double`。
- **L377 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L377 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L378 EN**: Executes a call or declaration centered on `vec_splat`.
  **L378 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L379 EN**: Executes a call or declaration centered on `_mm_rcp_ps`.
  **L379 CN**: 执行以 `_mm_rcp_ps` 为核心的调用或声明。
- **L380 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L380 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L381 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L382 EN**: Returns from the current function with `(vec_sel(__A, __c, __mask))`.
  **L382 CN**: 以 `(vec_sel(__A, __c, __mask))` 从当前函数返回。
- **L383 EN**: Closes the current lexical scope or compound statement.
  **L383 CN**: 结束当前词法作用域或复合语句块。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````c
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_rsqrt_ss(__m128 __A) {
  __m128 __a, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower double)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper double values) we splat the lower double
   * before we to the operation. */
  __a = vec_splat(__A, 0);
  __c = vec_rsqrte(__a);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return (vec_sel(__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_ss(__m128 __A, __m128 __B) {
  __v4sf __a, __b, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower float)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper float values) we splat the lower float
````
- **L385 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L385 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L386 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L386 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L387 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_rsqrt_ss(__m128 __A) {`.
  **L387 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_rsqrt_ss(__m128 __A) {`。
- **L388 EN**: Adds a standalone statement or declaration: `__m128 __a, __c;`.
  **L388 CN**: 添加一条独立语句或声明：`__m128 __a, __c;`。
- **L389 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L389 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L390 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower double)`.
  **L390 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower double)`。
- **L391 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L391 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L392 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper double values) we splat the lower double`.
  **L392 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper double values) we splat the lower double`。
- **L393 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L394 EN**: Executes a call or declaration centered on `vec_splat`.
  **L394 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L395 EN**: Executes a call or declaration centered on `vec_rsqrte`.
  **L395 CN**: 执行以 `vec_rsqrte` 为核心的调用或声明。
- **L396 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L397 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L397 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L398 EN**: Returns from the current function with `(vec_sel(__A, __c, __mask))`.
  **L398 CN**: 以 `(vec_sel(__A, __c, __mask))` 从当前函数返回。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L401 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L401 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L402 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L402 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L403 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_ss(__m128 __A, __m128 __B) {`.
  **L403 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_ss(__m128 __A, __m128 __B) {`。
- **L404 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L404 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L405 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L405 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L406 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower float)`.
  **L406 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower float)`。
- **L407 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L407 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L408 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper float values) we splat the lower float`.
  **L408 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper float values) we splat the lower float`。

### Lines 409-432

````c
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = vec_min(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return (vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_ss(__m128 __A, __m128 __B) {
  __v4sf __a, __b, __c;
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  /* PowerISA VSX does not allow partial (for just lower float)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper float values) we splat the lower float
   * before we to the operation. */
  __a = vec_splat(__A, 0);
  __b = vec_splat(__B, 0);
  __c = vec_max(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return (vec_sel((__v4sf)__A, __c, __mask));
````
- **L409 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L409 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L410 EN**: Executes a call or declaration centered on `vec_splat`.
  **L410 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L411 EN**: Executes a call or declaration centered on `vec_splat`.
  **L411 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L412 EN**: Executes a call or declaration centered on `vec_min`.
  **L412 CN**: 执行以 `vec_min` 为核心的调用或声明。
- **L413 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L413 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L415 EN**: Returns from the current function with `(vec_sel((__v4sf)__A, __c, __mask))`.
  **L415 CN**: 以 `(vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L416 EN**: Closes the current lexical scope or compound statement.
  **L416 CN**: 结束当前词法作用域或复合语句块。
- **L417 EN**: Blank line separating nearby declarations or logic blocks.
  **L417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L418 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L418 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L419 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L419 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L420 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_ss(__m128 __A, __m128 __B) {`.
  **L420 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_ss(__m128 __A, __m128 __B) {`。
- **L421 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L421 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L422 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L422 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L423 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VSX does not allow partial (for just lower float)`.
  **L423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VSX does not allow partial (for just lower float)`。
- **L424 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L425 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper float values) we splat the lower float`.
  **L425 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper float values) we splat the lower float`。
- **L426 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L427 EN**: Executes a call or declaration centered on `vec_splat`.
  **L427 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L428 EN**: Executes a call or declaration centered on `vec_splat`.
  **L428 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L429 EN**: Executes a call or declaration centered on `vec_max`.
  **L429 CN**: 执行以 `vec_max` 为核心的调用或声明。
- **L430 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L430 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L431 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L432 EN**: Returns from the current function with `(vec_sel((__v4sf)__A, __c, __mask))`.
  **L432 CN**: 以 `(vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。

### Lines 433-456

````c
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_ps(__m128 __A, __m128 __B) {
  __vector __bool int __m = vec_cmpgt((__v4sf)__B, (__v4sf)__A);
  return vec_sel(__B, __A, __m);
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_ps(__m128 __A, __m128 __B) {
  __vector __bool int __m = vec_cmpgt((__v4sf)__A, (__v4sf)__B);
  return vec_sel(__B, __A, __m);
}

/* Perform logical bit-wise operations on 128-bit values.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_and_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_and((__v4sf)__A, (__v4sf)__B));
  //  return __builtin_ia32_andps (__A, __B);
}

````
- **L433 EN**: Closes the current lexical scope or compound statement.
  **L433 CN**: 结束当前词法作用域或复合语句块。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L435 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L435 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L436 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L436 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L437 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_ps(__m128 __A, __m128 __B) {`.
  **L437 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_ps(__m128 __A, __m128 __B) {`。
- **L438 EN**: Initializes variable `__m` from the expression on the right-hand side.
  **L438 CN**: 使用右侧表达式初始化变量 `__m`。
- **L439 EN**: Returns from the current function with `vec_sel(__B, __A, __m)`.
  **L439 CN**: 以 `vec_sel(__B, __A, __m)` 从当前函数返回。
- **L440 EN**: Closes the current lexical scope or compound statement.
  **L440 CN**: 结束当前词法作用域或复合语句块。
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L442 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L442 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L443 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L443 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L444 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_ps(__m128 __A, __m128 __B) {`.
  **L444 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_ps(__m128 __A, __m128 __B) {`。
- **L445 EN**: Initializes variable `__m` from the expression on the right-hand side.
  **L445 CN**: 使用右侧表达式初始化变量 `__m`。
- **L446 EN**: Returns from the current function with `vec_sel(__B, __A, __m)`.
  **L446 CN**: 以 `vec_sel(__B, __A, __m)` 从当前函数返回。
- **L447 EN**: Closes the current lexical scope or compound statement.
  **L447 CN**: 结束当前词法作用域或复合语句块。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L449 EN**: Comment explains nearby logic, constraints, or intent: `Perform logical bit-wise operations on 128-bit values.`.
  **L449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Perform logical bit-wise operations on 128-bit values.`。
- **L450 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L450 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L451 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L451 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L452 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_and_ps(__m128 __A, __m128 __B) {`.
  **L452 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_and_ps(__m128 __A, __m128 __B) {`。
- **L453 EN**: Returns from the current function with `((__m128)vec_and((__v4sf)__A, (__v4sf)__B))`.
  **L453 CN**: 以 `((__m128)vec_and((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L454 EN**: Comment explains nearby logic, constraints, or intent: `return __builtin_ia32_andps (__A, __B);`.
  **L454 CN**: 注释解释附近代码的逻辑、约束或设计意图：`return __builtin_ia32_andps (__A, __B);`。
- **L455 EN**: Closes the current lexical scope or compound statement.
  **L455 CN**: 结束当前词法作用域或复合语句块。
- **L456 EN**: Blank line separating nearby declarations or logic blocks.
  **L456 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 457-480

````c
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_andnot_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_andc((__v4sf)__B, (__v4sf)__A));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_or_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_or((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_xor_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_xor((__v4sf)__A, (__v4sf)__B));
}

/* Perform a comparison on the four SPFP values of A and B.  For each
   element, if the comparison is true, place a mask of all ones in the
   result, otherwise a mask of zeros.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_ps(__m128 __A, __m128 __B) {
````
- **L457 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L457 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L458 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L458 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L459 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_andnot_ps(__m128 __A, __m128 __B) {`.
  **L459 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_andnot_ps(__m128 __A, __m128 __B) {`。
- **L460 EN**: Returns from the current function with `((__m128)vec_andc((__v4sf)__B, (__v4sf)__A))`.
  **L460 CN**: 以 `((__m128)vec_andc((__v4sf)__B, (__v4sf)__A))` 从当前函数返回。
- **L461 EN**: Closes the current lexical scope or compound statement.
  **L461 CN**: 结束当前词法作用域或复合语句块。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L463 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L463 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L464 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L464 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L465 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_or_ps(__m128 __A, __m128 __B) {`.
  **L465 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_or_ps(__m128 __A, __m128 __B) {`。
- **L466 EN**: Returns from the current function with `((__m128)vec_or((__v4sf)__A, (__v4sf)__B))`.
  **L466 CN**: 以 `((__m128)vec_or((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L467 EN**: Closes the current lexical scope or compound statement.
  **L467 CN**: 结束当前词法作用域或复合语句块。
- **L468 EN**: Blank line separating nearby declarations or logic blocks.
  **L468 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L469 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L469 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L470 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L470 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L471 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_xor_ps(__m128 __A, __m128 __B) {`.
  **L471 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_xor_ps(__m128 __A, __m128 __B) {`。
- **L472 EN**: Returns from the current function with `((__m128)vec_xor((__v4sf)__A, (__v4sf)__B))`.
  **L472 CN**: 以 `((__m128)vec_xor((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L473 EN**: Closes the current lexical scope or compound statement.
  **L473 CN**: 结束当前词法作用域或复合语句块。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Comment explains nearby logic, constraints, or intent: `Perform a comparison on the four SPFP values of A and B. For each`.
  **L475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Perform a comparison on the four SPFP values of A and B. For each`。
- **L476 EN**: Continues the surrounding expression or declaration: `element, if the comparison is true, place a mask of all ones in the`.
  **L476 CN**: 继续构造周围的表达式或声明：`element, if the comparison is true, place a mask of all ones in the`。
- **L477 EN**: Continues the surrounding expression or declaration: `result, otherwise a mask of zeros.  */`.
  **L477 CN**: 继续构造周围的表达式或声明：`result, otherwise a mask of zeros.  */`。
- **L478 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L478 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L479 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L479 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L480 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_ps(__m128 __A, __m128 __B) {`.
  **L480 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_ps(__m128 __A, __m128 __B) {`。

### Lines 481-504

````c
  return ((__m128)vec_cmpeq((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmplt_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_cmplt((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmple_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_cmple((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_cmpgt((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpge_ps(__m128 __A, __m128 __B) {
````
- **L481 EN**: Returns from the current function with `((__m128)vec_cmpeq((__v4sf)__A, (__v4sf)__B))`.
  **L481 CN**: 以 `((__m128)vec_cmpeq((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L482 EN**: Closes the current lexical scope or compound statement.
  **L482 CN**: 结束当前词法作用域或复合语句块。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L484 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L484 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L485 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L485 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L486 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmplt_ps(__m128 __A, __m128 __B) {`.
  **L486 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmplt_ps(__m128 __A, __m128 __B) {`。
- **L487 EN**: Returns from the current function with `((__m128)vec_cmplt((__v4sf)__A, (__v4sf)__B))`.
  **L487 CN**: 以 `((__m128)vec_cmplt((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L488 EN**: Closes the current lexical scope or compound statement.
  **L488 CN**: 结束当前词法作用域或复合语句块。
- **L489 EN**: Blank line separating nearby declarations or logic blocks.
  **L489 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L490 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L490 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L491 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L491 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L492 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmple_ps(__m128 __A, __m128 __B) {`.
  **L492 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmple_ps(__m128 __A, __m128 __B) {`。
- **L493 EN**: Returns from the current function with `((__m128)vec_cmple((__v4sf)__A, (__v4sf)__B))`.
  **L493 CN**: 以 `((__m128)vec_cmple((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L494 EN**: Closes the current lexical scope or compound statement.
  **L494 CN**: 结束当前词法作用域或复合语句块。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L496 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L496 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L497 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L497 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L498 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_ps(__m128 __A, __m128 __B) {`.
  **L498 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_ps(__m128 __A, __m128 __B) {`。
- **L499 EN**: Returns from the current function with `((__m128)vec_cmpgt((__v4sf)__A, (__v4sf)__B))`.
  **L499 CN**: 以 `((__m128)vec_cmpgt((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L500 EN**: Closes the current lexical scope or compound statement.
  **L500 CN**: 结束当前词法作用域或复合语句块。
- **L501 EN**: Blank line separating nearby declarations or logic blocks.
  **L501 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L502 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L502 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L503 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L503 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L504 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpge_ps(__m128 __A, __m128 __B) {`.
  **L504 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpge_ps(__m128 __A, __m128 __B) {`。

### Lines 505-528

````c
  return ((__m128)vec_cmpge((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpneq_ps(__m128 __A, __m128 __B) {
  __v4sf __temp = (__v4sf)vec_cmpeq((__v4sf)__A, (__v4sf)__B);
  return ((__m128)vec_nor(__temp, __temp));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnlt_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_cmpge((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnle_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_cmpgt((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L505 EN**: Returns from the current function with `((__m128)vec_cmpge((__v4sf)__A, (__v4sf)__B))`.
  **L505 CN**: 以 `((__m128)vec_cmpge((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L506 EN**: Closes the current lexical scope or compound statement.
  **L506 CN**: 结束当前词法作用域或复合语句块。
- **L507 EN**: Blank line separating nearby declarations or logic blocks.
  **L507 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L508 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L508 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L509 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L509 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L510 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpneq_ps(__m128 __A, __m128 __B) {`.
  **L510 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpneq_ps(__m128 __A, __m128 __B) {`。
- **L511 EN**: Initializes variable `__temp` from the expression on the right-hand side.
  **L511 CN**: 使用右侧表达式初始化变量 `__temp`。
- **L512 EN**: Returns from the current function with `((__m128)vec_nor(__temp, __temp))`.
  **L512 CN**: 以 `((__m128)vec_nor(__temp, __temp))` 从当前函数返回。
- **L513 EN**: Closes the current lexical scope or compound statement.
  **L513 CN**: 结束当前词法作用域或复合语句块。
- **L514 EN**: Blank line separating nearby declarations or logic blocks.
  **L514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L515 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L515 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L516 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L516 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L517 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnlt_ps(__m128 __A, __m128 __B) {`.
  **L517 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnlt_ps(__m128 __A, __m128 __B) {`。
- **L518 EN**: Returns from the current function with `((__m128)vec_cmpge((__v4sf)__A, (__v4sf)__B))`.
  **L518 CN**: 以 `((__m128)vec_cmpge((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L519 EN**: Closes the current lexical scope or compound statement.
  **L519 CN**: 结束当前词法作用域或复合语句块。
- **L520 EN**: Blank line separating nearby declarations or logic blocks.
  **L520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L521 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L521 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L522 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L522 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L523 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnle_ps(__m128 __A, __m128 __B) {`.
  **L523 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnle_ps(__m128 __A, __m128 __B) {`。
- **L524 EN**: Returns from the current function with `((__m128)vec_cmpgt((__v4sf)__A, (__v4sf)__B))`.
  **L524 CN**: 以 `((__m128)vec_cmpgt((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L525 EN**: Closes the current lexical scope or compound statement.
  **L525 CN**: 结束当前词法作用域或复合语句块。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L527 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L527 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L528 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L528 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 529-552

````c
    _mm_cmpngt_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_cmple((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnge_ps(__m128 __A, __m128 __B) {
  return ((__m128)vec_cmplt((__v4sf)__A, (__v4sf)__B));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpord_ps(__m128 __A, __m128 __B) {
  __vector unsigned int __a, __b;
  __vector unsigned int __c, __d;
  static const __vector unsigned int __float_exp_mask = {
      0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};

  __a = (__vector unsigned int)vec_abs((__v4sf)__A);
  __b = (__vector unsigned int)vec_abs((__v4sf)__B);
  __c = (__vector unsigned int)vec_cmpgt(__float_exp_mask, __a);
  __d = (__vector unsigned int)vec_cmpgt(__float_exp_mask, __b);
  return ((__m128)vec_and(__c, __d));
}
````
- **L529 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpngt_ps(__m128 __A, __m128 __B) {`.
  **L529 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpngt_ps(__m128 __A, __m128 __B) {`。
- **L530 EN**: Returns from the current function with `((__m128)vec_cmple((__v4sf)__A, (__v4sf)__B))`.
  **L530 CN**: 以 `((__m128)vec_cmple((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L531 EN**: Closes the current lexical scope or compound statement.
  **L531 CN**: 结束当前词法作用域或复合语句块。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L533 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L533 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L534 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L534 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L535 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnge_ps(__m128 __A, __m128 __B) {`.
  **L535 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnge_ps(__m128 __A, __m128 __B) {`。
- **L536 EN**: Returns from the current function with `((__m128)vec_cmplt((__v4sf)__A, (__v4sf)__B))`.
  **L536 CN**: 以 `((__m128)vec_cmplt((__v4sf)__A, (__v4sf)__B))` 从当前函数返回。
- **L537 EN**: Closes the current lexical scope or compound statement.
  **L537 CN**: 结束当前词法作用域或复合语句块。
- **L538 EN**: Blank line separating nearby declarations or logic blocks.
  **L538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L539 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L539 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L540 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L540 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L541 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpord_ps(__m128 __A, __m128 __B) {`.
  **L541 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpord_ps(__m128 __A, __m128 __B) {`。
- **L542 EN**: Adds a standalone statement or declaration: `__vector unsigned int __a, __b;`.
  **L542 CN**: 添加一条独立语句或声明：`__vector unsigned int __a, __b;`。
- **L543 EN**: Adds a standalone statement or declaration: `__vector unsigned int __c, __d;`.
  **L543 CN**: 添加一条独立语句或声明：`__vector unsigned int __c, __d;`。
- **L544 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned int __float_exp_mask = {`.
  **L544 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned int __float_exp_mask = {`。
- **L545 EN**: Adds a standalone statement or declaration: `0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};`.
  **L545 CN**: 添加一条独立语句或声明：`0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};`。
- **L546 EN**: Blank line separating nearby declarations or logic blocks.
  **L546 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L547 EN**: Executes a call or declaration centered on `=`.
  **L547 CN**: 执行以 `=` 为核心的调用或声明。
- **L548 EN**: Executes a call or declaration centered on `=`.
  **L548 CN**: 执行以 `=` 为核心的调用或声明。
- **L549 EN**: Executes a call or declaration centered on `=`.
  **L549 CN**: 执行以 `=` 为核心的调用或声明。
- **L550 EN**: Executes a call or declaration centered on `=`.
  **L550 CN**: 执行以 `=` 为核心的调用或声明。
- **L551 EN**: Returns from the current function with `((__m128)vec_and(__c, __d))`.
  **L551 CN**: 以 `((__m128)vec_and(__c, __d))` 从当前函数返回。
- **L552 EN**: Closes the current lexical scope or compound statement.
  **L552 CN**: 结束当前词法作用域或复合语句块。

### Lines 553-576

````c

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpunord_ps(__m128 __A, __m128 __B) {
  __vector unsigned int __a, __b;
  __vector unsigned int __c, __d;
  static const __vector unsigned int __float_exp_mask = {
      0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};

  __a = (__vector unsigned int)vec_abs((__v4sf)__A);
  __b = (__vector unsigned int)vec_abs((__v4sf)__B);
  __c = (__vector unsigned int)vec_cmpgt(__a, __float_exp_mask);
  __d = (__vector unsigned int)vec_cmpgt(__b, __float_exp_mask);
  return ((__m128)vec_or(__c, __d));
}

/* Perform a comparison on the lower SPFP values of A and B.  If the
   comparison is true, place a mask of all ones in the result, otherwise a
   mask of zeros.  The upper three SPFP values are passed through from A.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
````
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L554 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L554 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L555 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L555 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L556 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpunord_ps(__m128 __A, __m128 __B) {`.
  **L556 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpunord_ps(__m128 __A, __m128 __B) {`。
- **L557 EN**: Adds a standalone statement or declaration: `__vector unsigned int __a, __b;`.
  **L557 CN**: 添加一条独立语句或声明：`__vector unsigned int __a, __b;`。
- **L558 EN**: Adds a standalone statement or declaration: `__vector unsigned int __c, __d;`.
  **L558 CN**: 添加一条独立语句或声明：`__vector unsigned int __c, __d;`。
- **L559 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned int __float_exp_mask = {`.
  **L559 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned int __float_exp_mask = {`。
- **L560 EN**: Adds a standalone statement or declaration: `0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};`.
  **L560 CN**: 添加一条独立语句或声明：`0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};`。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L562 EN**: Executes a call or declaration centered on `=`.
  **L562 CN**: 执行以 `=` 为核心的调用或声明。
- **L563 EN**: Executes a call or declaration centered on `=`.
  **L563 CN**: 执行以 `=` 为核心的调用或声明。
- **L564 EN**: Executes a call or declaration centered on `=`.
  **L564 CN**: 执行以 `=` 为核心的调用或声明。
- **L565 EN**: Executes a call or declaration centered on `=`.
  **L565 CN**: 执行以 `=` 为核心的调用或声明。
- **L566 EN**: Returns from the current function with `((__m128)vec_or(__c, __d))`.
  **L566 CN**: 以 `((__m128)vec_or(__c, __d))` 从当前函数返回。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L569 EN**: Comment explains nearby logic, constraints, or intent: `Perform a comparison on the lower SPFP values of A and B. If the`.
  **L569 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Perform a comparison on the lower SPFP values of A and B. If the`。
- **L570 EN**: Continues the surrounding expression or declaration: `comparison is true, place a mask of all ones in the result, otherwise a`.
  **L570 CN**: 继续构造周围的表达式或声明：`comparison is true, place a mask of all ones in the result, otherwise a`。
- **L571 EN**: Continues the surrounding expression or declaration: `mask of zeros.  The upper three SPFP values are passed through from A.  */`.
  **L571 CN**: 继续构造周围的表达式或声明：`mask of zeros.  The upper three SPFP values are passed through from A.  */`。
- **L572 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L572 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L573 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L573 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L574 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_ss(__m128 __A, __m128 __B) {`.
  **L574 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_ss(__m128 __A, __m128 __B) {`。
- **L575 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L575 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L576 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L576 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。

### Lines 577-600

````c
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmpeq(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmplt_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmplt(__a, __b);
````
- **L577 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L577 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L578 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L578 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L579 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L579 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L580 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L580 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L581 EN**: Executes a call or declaration centered on `vec_splat`.
  **L581 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L582 EN**: Executes a call or declaration centered on `vec_splat`.
  **L582 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L583 EN**: Executes a call or declaration centered on `=`.
  **L583 CN**: 执行以 `=` 为核心的调用或声明。
- **L584 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L584 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L585 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L585 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L586 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L586 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L587 EN**: Closes the current lexical scope or compound statement.
  **L587 CN**: 结束当前词法作用域或复合语句块。
- **L588 EN**: Blank line separating nearby declarations or logic blocks.
  **L588 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L589 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L589 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L590 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L590 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L591 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmplt_ss(__m128 __A, __m128 __B) {`.
  **L591 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmplt_ss(__m128 __A, __m128 __B) {`。
- **L592 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L592 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L593 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L593 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L594 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L594 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L595 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L595 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L596 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L596 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L597 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L597 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L598 EN**: Executes a call or declaration centered on `vec_splat`.
  **L598 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L599 EN**: Executes a call or declaration centered on `vec_splat`.
  **L599 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L600 EN**: Executes a call or declaration centered on `=`.
  **L600 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 601-624

````c
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmple_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmple(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L601 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L602 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L603 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L603 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L604 EN**: Closes the current lexical scope or compound statement.
  **L604 CN**: 结束当前词法作用域或复合语句块。
- **L605 EN**: Blank line separating nearby declarations or logic blocks.
  **L605 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L606 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L606 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L607 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L607 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L608 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmple_ss(__m128 __A, __m128 __B) {`.
  **L608 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmple_ss(__m128 __A, __m128 __B) {`。
- **L609 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L609 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L610 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L610 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L611 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L611 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L612 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L613 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L613 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L614 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L614 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L615 EN**: Executes a call or declaration centered on `vec_splat`.
  **L615 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L616 EN**: Executes a call or declaration centered on `vec_splat`.
  **L616 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L617 EN**: Executes a call or declaration centered on `=`.
  **L617 CN**: 执行以 `=` 为核心的调用或声明。
- **L618 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L618 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L619 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L619 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L620 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L620 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L621 EN**: Closes the current lexical scope or compound statement.
  **L621 CN**: 结束当前词法作用域或复合语句块。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L623 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L623 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L624 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L624 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 625-648

````c
    _mm_cmpgt_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmpgt(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpge_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
````
- **L625 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_ss(__m128 __A, __m128 __B) {`.
  **L625 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_ss(__m128 __A, __m128 __B) {`。
- **L626 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L626 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L627 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L627 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L628 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L628 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L629 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L629 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L630 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L630 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L631 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L632 EN**: Executes a call or declaration centered on `vec_splat`.
  **L632 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L633 EN**: Executes a call or declaration centered on `vec_splat`.
  **L633 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L634 EN**: Executes a call or declaration centered on `=`.
  **L634 CN**: 执行以 `=` 为核心的调用或声明。
- **L635 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L635 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L636 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L636 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L637 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L637 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Blank line separating nearby declarations or logic blocks.
  **L639 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L640 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L640 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L641 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L641 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L642 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpge_ss(__m128 __A, __m128 __B) {`.
  **L642 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpge_ss(__m128 __A, __m128 __B) {`。
- **L643 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L643 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L644 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L644 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L645 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L645 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L646 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L646 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L647 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L647 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L648 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L648 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。

### Lines 649-672

````c
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmpge(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpneq_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmpeq(__a, __b);
  __c = vec_nor(__c, __c);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
````
- **L649 EN**: Executes a call or declaration centered on `vec_splat`.
  **L649 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L650 EN**: Executes a call or declaration centered on `vec_splat`.
  **L650 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L651 EN**: Executes a call or declaration centered on `=`.
  **L651 CN**: 执行以 `=` 为核心的调用或声明。
- **L652 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L653 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L653 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L654 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L654 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L655 EN**: Closes the current lexical scope or compound statement.
  **L655 CN**: 结束当前词法作用域或复合语句块。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L657 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L658 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L658 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L659 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpneq_ss(__m128 __A, __m128 __B) {`.
  **L659 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpneq_ss(__m128 __A, __m128 __B) {`。
- **L660 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L660 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L661 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L661 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L662 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L662 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L663 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L663 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L664 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L665 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L665 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L666 EN**: Executes a call or declaration centered on `vec_splat`.
  **L666 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L667 EN**: Executes a call or declaration centered on `vec_splat`.
  **L667 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L668 EN**: Executes a call or declaration centered on `=`.
  **L668 CN**: 执行以 `=` 为核心的调用或声明。
- **L669 EN**: Executes a call or declaration centered on `vec_nor`.
  **L669 CN**: 执行以 `vec_nor` 为核心的调用或声明。
- **L670 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L670 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L671 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L671 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L672 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L672 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。

### Lines 673-696

````c
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnlt_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmpge(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnle_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L675 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L675 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L676 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L676 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L677 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnlt_ss(__m128 __A, __m128 __B) {`.
  **L677 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnlt_ss(__m128 __A, __m128 __B) {`。
- **L678 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L678 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L679 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L679 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L680 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L680 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L681 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L681 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L682 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L682 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L683 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L683 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L684 EN**: Executes a call or declaration centered on `vec_splat`.
  **L684 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L685 EN**: Executes a call or declaration centered on `vec_splat`.
  **L685 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L686 EN**: Executes a call or declaration centered on `=`.
  **L686 CN**: 执行以 `=` 为核心的调用或声明。
- **L687 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L687 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L688 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L688 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L689 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L689 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L690 EN**: Closes the current lexical scope or compound statement.
  **L690 CN**: 结束当前词法作用域或复合语句块。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L692 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L692 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L693 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L693 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L694 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnle_ss(__m128 __A, __m128 __B) {`.
  **L694 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnle_ss(__m128 __A, __m128 __B) {`。
- **L695 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L695 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L696 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L696 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。

### Lines 697-720

````c
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmpgt(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpngt_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we to the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmple(__a, __b);
````
- **L697 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L697 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L698 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L698 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L699 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L699 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L700 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L700 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L701 EN**: Executes a call or declaration centered on `vec_splat`.
  **L701 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L702 EN**: Executes a call or declaration centered on `vec_splat`.
  **L702 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L703 EN**: Executes a call or declaration centered on `=`.
  **L703 CN**: 执行以 `=` 为核心的调用或声明。
- **L704 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L704 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L705 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L706 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L706 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L709 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L709 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L710 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L710 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L711 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpngt_ss(__m128 __A, __m128 __B) {`.
  **L711 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpngt_ss(__m128 __A, __m128 __B) {`。
- **L712 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L712 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L713 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L713 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L714 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L715 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L715 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L716 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L716 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L717 EN**: Comment explains nearby logic, constraints, or intent: `before we to the operation.`.
  **L717 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we to the operation.`。
- **L718 EN**: Executes a call or declaration centered on `vec_splat`.
  **L718 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L719 EN**: Executes a call or declaration centered on `vec_splat`.
  **L719 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L720 EN**: Executes a call or declaration centered on `=`.
  **L720 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 721-744

````c
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpnge_ss(__m128 __A, __m128 __B) {
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};
  __v4sf __a, __b, __c;
  /* PowerISA VMX does not allow partial (for just element 0)
   * results. So to insure we don't generate spurious exceptions
   * (from the upper elements) we splat the lower float
   * before we do the operation. */
  __a = vec_splat((__v4sf)__A, 0);
  __b = vec_splat((__v4sf)__B, 0);
  __c = (__v4sf)vec_cmplt(__a, __b);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, __c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L721 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L722 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L722 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L723 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L723 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L724 EN**: Closes the current lexical scope or compound statement.
  **L724 CN**: 结束当前词法作用域或复合语句块。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L726 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L726 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L727 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L727 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L728 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpnge_ss(__m128 __A, __m128 __B) {`.
  **L728 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpnge_ss(__m128 __A, __m128 __B) {`。
- **L729 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L729 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L730 EN**: Adds a standalone statement or declaration: `__v4sf __a, __b, __c;`.
  **L730 CN**: 添加一条独立语句或声明：`__v4sf __a, __b, __c;`。
- **L731 EN**: Comment explains nearby logic, constraints, or intent: `PowerISA VMX does not allow partial (for just element 0)`.
  **L731 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PowerISA VMX does not allow partial (for just element 0)`。
- **L732 EN**: Comment explains nearby logic, constraints, or intent: `results. So to insure we don't generate spurious exceptions`.
  **L732 CN**: 注释解释附近代码的逻辑、约束或设计意图：`results. So to insure we don't generate spurious exceptions`。
- **L733 EN**: Comment explains nearby logic, constraints, or intent: `(from the upper elements) we splat the lower float`.
  **L733 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(from the upper elements) we splat the lower float`。
- **L734 EN**: Comment explains nearby logic, constraints, or intent: `before we do the operation.`.
  **L734 CN**: 注释解释附近代码的逻辑、约束或设计意图：`before we do the operation.`。
- **L735 EN**: Executes a call or declaration centered on `vec_splat`.
  **L735 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L736 EN**: Executes a call or declaration centered on `vec_splat`.
  **L736 CN**: 执行以 `vec_splat` 为核心的调用或声明。
- **L737 EN**: Executes a call or declaration centered on `=`.
  **L737 CN**: 执行以 `=` 为核心的调用或声明。
- **L738 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L738 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L739 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L740 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, __c, __mask))`.
  **L740 CN**: 以 `((__m128)vec_sel((__v4sf)__A, __c, __mask))` 从当前函数返回。
- **L741 EN**: Closes the current lexical scope or compound statement.
  **L741 CN**: 结束当前词法作用域或复合语句块。
- **L742 EN**: Blank line separating nearby declarations or logic blocks.
  **L742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L743 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L743 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L744 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L744 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 745-768

````c
    _mm_cmpord_ss(__m128 __A, __m128 __B) {
  __vector unsigned int __a, __b;
  __vector unsigned int __c, __d;
  static const __vector unsigned int __float_exp_mask = {
      0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};

  __a = (__vector unsigned int)vec_abs((__v4sf)__A);
  __b = (__vector unsigned int)vec_abs((__v4sf)__B);
  __c = (__vector unsigned int)vec_cmpgt(__float_exp_mask, __a);
  __d = (__vector unsigned int)vec_cmpgt(__float_exp_mask, __b);
  __c = vec_and(__c, __d);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, (__v4sf)__c, __mask));
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpunord_ss(__m128 __A, __m128 __B) {
  __vector unsigned int __a, __b;
  __vector unsigned int __c, __d;
  static const __vector unsigned int __float_exp_mask = {
      0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};
````
- **L745 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpord_ss(__m128 __A, __m128 __B) {`.
  **L745 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpord_ss(__m128 __A, __m128 __B) {`。
- **L746 EN**: Adds a standalone statement or declaration: `__vector unsigned int __a, __b;`.
  **L746 CN**: 添加一条独立语句或声明：`__vector unsigned int __a, __b;`。
- **L747 EN**: Adds a standalone statement or declaration: `__vector unsigned int __c, __d;`.
  **L747 CN**: 添加一条独立语句或声明：`__vector unsigned int __c, __d;`。
- **L748 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned int __float_exp_mask = {`.
  **L748 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned int __float_exp_mask = {`。
- **L749 EN**: Adds a standalone statement or declaration: `0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};`.
  **L749 CN**: 添加一条独立语句或声明：`0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};`。
- **L750 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L750 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L751 EN**: Blank line separating nearby declarations or logic blocks.
  **L751 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L752 EN**: Executes a call or declaration centered on `=`.
  **L752 CN**: 执行以 `=` 为核心的调用或声明。
- **L753 EN**: Executes a call or declaration centered on `=`.
  **L753 CN**: 执行以 `=` 为核心的调用或声明。
- **L754 EN**: Executes a call or declaration centered on `=`.
  **L754 CN**: 执行以 `=` 为核心的调用或声明。
- **L755 EN**: Executes a call or declaration centered on `=`.
  **L755 CN**: 执行以 `=` 为核心的调用或声明。
- **L756 EN**: Executes a call or declaration centered on `vec_and`.
  **L756 CN**: 执行以 `vec_and` 为核心的调用或声明。
- **L757 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L758 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L758 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L759 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, (__v4sf)__c, __mask))`.
  **L759 CN**: 以 `((__m128)vec_sel((__v4sf)__A, (__v4sf)__c, __mask))` 从当前函数返回。
- **L760 EN**: Closes the current lexical scope or compound statement.
  **L760 CN**: 结束当前词法作用域或复合语句块。
- **L761 EN**: Blank line separating nearby declarations or logic blocks.
  **L761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L762 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L762 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L763 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L763 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L764 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpunord_ss(__m128 __A, __m128 __B) {`.
  **L764 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpunord_ss(__m128 __A, __m128 __B) {`。
- **L765 EN**: Adds a standalone statement or declaration: `__vector unsigned int __a, __b;`.
  **L765 CN**: 添加一条独立语句或声明：`__vector unsigned int __a, __b;`。
- **L766 EN**: Adds a standalone statement or declaration: `__vector unsigned int __c, __d;`.
  **L766 CN**: 添加一条独立语句或声明：`__vector unsigned int __c, __d;`。
- **L767 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned int __float_exp_mask = {`.
  **L767 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned int __float_exp_mask = {`。
- **L768 EN**: Adds a standalone statement or declaration: `0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};`.
  **L768 CN**: 添加一条独立语句或声明：`0x7f800000, 0x7f800000, 0x7f800000, 0x7f800000};`。

### Lines 769-792

````c
  static const __vector unsigned int __mask = {0xffffffff, 0, 0, 0};

  __a = (__vector unsigned int)vec_abs((__v4sf)__A);
  __b = (__vector unsigned int)vec_abs((__v4sf)__B);
  __c = (__vector unsigned int)vec_cmpgt(__a, __float_exp_mask);
  __d = (__vector unsigned int)vec_cmpgt(__b, __float_exp_mask);
  __c = vec_or(__c, __d);
  /* Then we merge the lower float result with the original upper
   * float elements from __A.  */
  return ((__m128)vec_sel((__v4sf)__A, (__v4sf)__c, __mask));
}

/* Compare the lower SPFP values of A and B and return 1 if true
   and 0 if false.  */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comieq_ss(__m128 __A, __m128 __B) {
  return (__A[0] == __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comilt_ss(__m128 __A, __m128 __B) {
  return (__A[0] < __B[0]);
````
- **L769 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L769 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L770 EN**: Blank line separating nearby declarations or logic blocks.
  **L770 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L771 EN**: Executes a call or declaration centered on `=`.
  **L771 CN**: 执行以 `=` 为核心的调用或声明。
- **L772 EN**: Executes a call or declaration centered on `=`.
  **L772 CN**: 执行以 `=` 为核心的调用或声明。
- **L773 EN**: Executes a call or declaration centered on `=`.
  **L773 CN**: 执行以 `=` 为核心的调用或声明。
- **L774 EN**: Executes a call or declaration centered on `=`.
  **L774 CN**: 执行以 `=` 为核心的调用或声明。
- **L775 EN**: Executes a call or declaration centered on `vec_or`.
  **L775 CN**: 执行以 `vec_or` 为核心的调用或声明。
- **L776 EN**: Comment explains nearby logic, constraints, or intent: `Then we merge the lower float result with the original upper`.
  **L776 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Then we merge the lower float result with the original upper`。
- **L777 EN**: Comment explains nearby logic, constraints, or intent: `float elements from __A.`.
  **L777 CN**: 注释解释附近代码的逻辑、约束或设计意图：`float elements from __A.`。
- **L778 EN**: Returns from the current function with `((__m128)vec_sel((__v4sf)__A, (__v4sf)__c, __mask))`.
  **L778 CN**: 以 `((__m128)vec_sel((__v4sf)__A, (__v4sf)__c, __mask))` 从当前函数返回。
- **L779 EN**: Closes the current lexical scope or compound statement.
  **L779 CN**: 结束当前词法作用域或复合语句块。
- **L780 EN**: Blank line separating nearby declarations or logic blocks.
  **L780 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L781 EN**: Comment explains nearby logic, constraints, or intent: `Compare the lower SPFP values of A and B and return 1 if true`.
  **L781 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare the lower SPFP values of A and B and return 1 if true`。
- **L782 EN**: Continues the surrounding expression or declaration: `and 0 if false.  */`.
  **L782 CN**: 继续构造周围的表达式或声明：`and 0 if false.  */`。
- **L783 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L783 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L784 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L784 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L785 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comieq_ss(__m128 __A, __m128 __B) {`.
  **L785 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comieq_ss(__m128 __A, __m128 __B) {`。
- **L786 EN**: Returns from the current function with `(__A[0] == __B[0])`.
  **L786 CN**: 以 `(__A[0] == __B[0])` 从当前函数返回。
- **L787 EN**: Closes the current lexical scope or compound statement.
  **L787 CN**: 结束当前词法作用域或复合语句块。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L789 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L789 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L790 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L790 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L791 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comilt_ss(__m128 __A, __m128 __B) {`.
  **L791 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comilt_ss(__m128 __A, __m128 __B) {`。
- **L792 EN**: Returns from the current function with `(__A[0] < __B[0])`.
  **L792 CN**: 以 `(__A[0] < __B[0])` 从当前函数返回。

### Lines 793-816

````c
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comile_ss(__m128 __A, __m128 __B) {
  return (__A[0] <= __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comigt_ss(__m128 __A, __m128 __B) {
  return (__A[0] > __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comige_ss(__m128 __A, __m128 __B) {
  return (__A[0] >= __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_comineq_ss(__m128 __A, __m128 __B) {
  return (__A[0] != __B[0]);
````
- **L793 EN**: Closes the current lexical scope or compound statement.
  **L793 CN**: 结束当前词法作用域或复合语句块。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L795 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L795 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L796 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L796 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L797 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comile_ss(__m128 __A, __m128 __B) {`.
  **L797 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comile_ss(__m128 __A, __m128 __B) {`。
- **L798 EN**: Returns from the current function with `(__A[0] <= __B[0])`.
  **L798 CN**: 以 `(__A[0] <= __B[0])` 从当前函数返回。
- **L799 EN**: Closes the current lexical scope or compound statement.
  **L799 CN**: 结束当前词法作用域或复合语句块。
- **L800 EN**: Blank line separating nearby declarations or logic blocks.
  **L800 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L801 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L801 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L802 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L802 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L803 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comigt_ss(__m128 __A, __m128 __B) {`.
  **L803 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comigt_ss(__m128 __A, __m128 __B) {`。
- **L804 EN**: Returns from the current function with `(__A[0] > __B[0])`.
  **L804 CN**: 以 `(__A[0] > __B[0])` 从当前函数返回。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L807 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L807 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L808 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L808 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L809 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comige_ss(__m128 __A, __m128 __B) {`.
  **L809 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comige_ss(__m128 __A, __m128 __B) {`。
- **L810 EN**: Returns from the current function with `(__A[0] >= __B[0])`.
  **L810 CN**: 以 `(__A[0] >= __B[0])` 从当前函数返回。
- **L811 EN**: Closes the current lexical scope or compound statement.
  **L811 CN**: 结束当前词法作用域或复合语句块。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L813 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L814 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L814 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L815 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_comineq_ss(__m128 __A, __m128 __B) {`.
  **L815 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_comineq_ss(__m128 __A, __m128 __B) {`。
- **L816 EN**: Returns from the current function with `(__A[0] != __B[0])`.
  **L816 CN**: 以 `(__A[0] != __B[0])` 从当前函数返回。

### Lines 817-840

````c
}

/* FIXME
 * The __mm_ucomi??_ss implementations below are exactly the same as
 * __mm_comi??_ss because GCC for PowerPC only generates unordered
 * compares (scalar and vector).
 * Technically __mm_comieq_ss et al should be using the ordered
 * compare and signal for QNaNs.
 * The __mm_ucomieq_sd et all should be OK, as is.
 */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomieq_ss(__m128 __A, __m128 __B) {
  return (__A[0] == __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomilt_ss(__m128 __A, __m128 __B) {
  return (__A[0] < __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L817 EN**: Closes the current lexical scope or compound statement.
  **L817 CN**: 结束当前词法作用域或复合语句块。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L819 EN**: Comment records a pending task or caution: `FIXME`.
  **L819 CN**: 注释记录待办事项或注意点：`FIXME`。
- **L820 EN**: Comment explains nearby logic, constraints, or intent: `The __mm_ucomi??_ss implementations below are exactly the same as`.
  **L820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The __mm_ucomi??_ss implementations below are exactly the same as`。
- **L821 EN**: Comment explains nearby logic, constraints, or intent: `__mm_comi??_ss because GCC for PowerPC only generates unordered`.
  **L821 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__mm_comi??_ss because GCC for PowerPC only generates unordered`。
- **L822 EN**: Comment explains nearby logic, constraints, or intent: `compares (scalar and vector).`.
  **L822 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compares (scalar and vector).`。
- **L823 EN**: Comment explains nearby logic, constraints, or intent: `Technically __mm_comieq_ss et al should be using the ordered`.
  **L823 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Technically __mm_comieq_ss et al should be using the ordered`。
- **L824 EN**: Comment explains nearby logic, constraints, or intent: `compare and signal for QNaNs.`.
  **L824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`compare and signal for QNaNs.`。
- **L825 EN**: Comment explains nearby logic, constraints, or intent: `The __mm_ucomieq_sd et all should be OK, as is.`.
  **L825 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The __mm_ucomieq_sd et all should be OK, as is.`。
- **L826 EN**: Separator comment used for visual grouping.
  **L826 CN**: 用于视觉分组的分隔注释。
- **L827 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L827 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L828 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L828 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L829 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomieq_ss(__m128 __A, __m128 __B) {`.
  **L829 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomieq_ss(__m128 __A, __m128 __B) {`。
- **L830 EN**: Returns from the current function with `(__A[0] == __B[0])`.
  **L830 CN**: 以 `(__A[0] == __B[0])` 从当前函数返回。
- **L831 EN**: Closes the current lexical scope or compound statement.
  **L831 CN**: 结束当前词法作用域或复合语句块。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L833 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L833 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L834 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L834 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L835 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomilt_ss(__m128 __A, __m128 __B) {`.
  **L835 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomilt_ss(__m128 __A, __m128 __B) {`。
- **L836 EN**: Returns from the current function with `(__A[0] < __B[0])`.
  **L836 CN**: 以 `(__A[0] < __B[0])` 从当前函数返回。
- **L837 EN**: Closes the current lexical scope or compound statement.
  **L837 CN**: 结束当前词法作用域或复合语句块。
- **L838 EN**: Blank line separating nearby declarations or logic blocks.
  **L838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L839 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L839 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L840 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L840 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 841-864

````c
    _mm_ucomile_ss(__m128 __A, __m128 __B) {
  return (__A[0] <= __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomigt_ss(__m128 __A, __m128 __B) {
  return (__A[0] > __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomige_ss(__m128 __A, __m128 __B) {
  return (__A[0] >= __B[0]);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_ucomineq_ss(__m128 __A, __m128 __B) {
  return (__A[0] != __B[0]);
}

extern __inline float
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L841 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomile_ss(__m128 __A, __m128 __B) {`.
  **L841 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomile_ss(__m128 __A, __m128 __B) {`。
- **L842 EN**: Returns from the current function with `(__A[0] <= __B[0])`.
  **L842 CN**: 以 `(__A[0] <= __B[0])` 从当前函数返回。
- **L843 EN**: Closes the current lexical scope or compound statement.
  **L843 CN**: 结束当前词法作用域或复合语句块。
- **L844 EN**: Blank line separating nearby declarations or logic blocks.
  **L844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L845 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L845 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L846 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L846 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L847 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomigt_ss(__m128 __A, __m128 __B) {`.
  **L847 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomigt_ss(__m128 __A, __m128 __B) {`。
- **L848 EN**: Returns from the current function with `(__A[0] > __B[0])`.
  **L848 CN**: 以 `(__A[0] > __B[0])` 从当前函数返回。
- **L849 EN**: Closes the current lexical scope or compound statement.
  **L849 CN**: 结束当前词法作用域或复合语句块。
- **L850 EN**: Blank line separating nearby declarations or logic blocks.
  **L850 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L851 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L851 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L852 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L852 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L853 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomige_ss(__m128 __A, __m128 __B) {`.
  **L853 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomige_ss(__m128 __A, __m128 __B) {`。
- **L854 EN**: Returns from the current function with `(__A[0] >= __B[0])`.
  **L854 CN**: 以 `(__A[0] >= __B[0])` 从当前函数返回。
- **L855 EN**: Closes the current lexical scope or compound statement.
  **L855 CN**: 结束当前词法作用域或复合语句块。
- **L856 EN**: Blank line separating nearby declarations or logic blocks.
  **L856 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L857 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L857 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L858 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L858 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L859 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_ucomineq_ss(__m128 __A, __m128 __B) {`.
  **L859 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_ucomineq_ss(__m128 __A, __m128 __B) {`。
- **L860 EN**: Returns from the current function with `(__A[0] != __B[0])`.
  **L860 CN**: 以 `(__A[0] != __B[0])` 从当前函数返回。
- **L861 EN**: Closes the current lexical scope or compound statement.
  **L861 CN**: 结束当前词法作用域或复合语句块。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L863 EN**: Continues the surrounding expression or declaration: `extern __inline float`.
  **L863 CN**: 继续构造周围的表达式或声明：`extern __inline float`。
- **L864 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L864 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 865-888

````c
    _mm_cvtss_f32(__m128 __A) {
  return ((__v4sf)__A)[0];
}

/* Convert the lower SPFP value to a 32-bit integer according to the current
   rounding mode.  */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtss_si32(__m128 __A) {
  int __res;
#ifdef _ARCH_PWR8
  double __dtmp;
  __asm__(
#ifdef __LITTLE_ENDIAN__
      "xxsldwi %x0,%x0,%x0,3;\n"
#endif
      "xscvspdp %x2,%x0;\n"
      "fctiw  %2,%2;\n"
      "mfvsrd  %1,%x2;\n"
      : "+wa"(__A), "=r"(__res), "=f"(__dtmp)
      :);
#else
  __res = __builtin_rint(__A[0]);
#endif
````
- **L865 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtss_f32(__m128 __A) {`.
  **L865 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtss_f32(__m128 __A) {`。
- **L866 EN**: Returns from the current function with `((__v4sf)__A)[0]`.
  **L866 CN**: 以 `((__v4sf)__A)[0]` 从当前函数返回。
- **L867 EN**: Closes the current lexical scope or compound statement.
  **L867 CN**: 结束当前词法作用域或复合语句块。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L869 EN**: Comment explains nearby logic, constraints, or intent: `Convert the lower SPFP value to a 32-bit integer according to the current`.
  **L869 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the lower SPFP value to a 32-bit integer according to the current`。
- **L870 EN**: Continues the surrounding expression or declaration: `rounding mode.  */`.
  **L870 CN**: 继续构造周围的表达式或声明：`rounding mode.  */`。
- **L871 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L871 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L872 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L872 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L873 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtss_si32(__m128 __A) {`.
  **L873 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtss_si32(__m128 __A) {`。
- **L874 EN**: Adds a standalone statement or declaration: `int __res;`.
  **L874 CN**: 添加一条独立语句或声明：`int __res;`。
- **L875 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L875 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L876 EN**: Adds a standalone statement or declaration: `double __dtmp;`.
  **L876 CN**: 添加一条独立语句或声明：`double __dtmp;`。
- **L877 EN**: Continues logic associated with callable symbol `__asm__`.
  **L877 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L878 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L878 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L879 EN**: Continues the surrounding expression or declaration: `"xxsldwi %x0,%x0,%x0,3;\n"`.
  **L879 CN**: 继续构造周围的表达式或声明：`"xxsldwi %x0,%x0,%x0,3;\n"`。
- **L880 EN**: Closes the current preprocessor conditional block.
  **L880 CN**: 结束当前预处理条件块。
- **L881 EN**: Continues the surrounding expression or declaration: `"xscvspdp %x2,%x0;\n"`.
  **L881 CN**: 继续构造周围的表达式或声明：`"xscvspdp %x2,%x0;\n"`。
- **L882 EN**: Continues the surrounding expression or declaration: `"fctiw  %2,%2;\n"`.
  **L882 CN**: 继续构造周围的表达式或声明：`"fctiw  %2,%2;\n"`。
- **L883 EN**: Continues the surrounding expression or declaration: `"mfvsrd  %1,%x2;\n"`.
  **L883 CN**: 继续构造周围的表达式或声明：`"mfvsrd  %1,%x2;\n"`。
- **L884 EN**: Continues the surrounding expression or declaration: `: "+wa"(__A), "=r"(__res), "=f"(__dtmp)`.
  **L884 CN**: 继续构造周围的表达式或声明：`: "+wa"(__A), "=r"(__res), "=f"(__dtmp)`。
- **L885 EN**: Adds a standalone statement or declaration: `:);`.
  **L885 CN**: 添加一条独立语句或声明：`:);`。
- **L886 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L886 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L887 EN**: Executes a call or declaration centered on `__builtin_rint`.
  **L887 CN**: 执行以 `__builtin_rint` 为核心的调用或声明。
- **L888 EN**: Closes the current preprocessor conditional block.
  **L888 CN**: 结束当前预处理条件块。

### Lines 889-912

````c
  return __res;
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvt_ss2si(__m128 __A) {
  return _mm_cvtss_si32(__A);
}

/* Convert the lower SPFP value to a 32-bit integer according to the
   current rounding mode.  */

/* Intel intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtss_si64(__m128 __A) {
  long long __res;
#if defined(_ARCH_PWR8) && defined(__powerpc64__)
  double __dtmp;
  __asm__(
#ifdef __LITTLE_ENDIAN__
      "xxsldwi %x0,%x0,%x0,3;\n"
#endif
      "xscvspdp %x2,%x0;\n"
````
- **L889 EN**: Returns from the current function with `__res`.
  **L889 CN**: 以 `__res` 从当前函数返回。
- **L890 EN**: Closes the current lexical scope or compound statement.
  **L890 CN**: 结束当前词法作用域或复合语句块。
- **L891 EN**: Blank line separating nearby declarations or logic blocks.
  **L891 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L892 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L892 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L893 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L893 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L894 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvt_ss2si(__m128 __A) {`.
  **L894 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvt_ss2si(__m128 __A) {`。
- **L895 EN**: Returns from the current function with `_mm_cvtss_si32(__A)`.
  **L895 CN**: 以 `_mm_cvtss_si32(__A)` 从当前函数返回。
- **L896 EN**: Closes the current lexical scope or compound statement.
  **L896 CN**: 结束当前词法作用域或复合语句块。
- **L897 EN**: Blank line separating nearby declarations or logic blocks.
  **L897 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L898 EN**: Comment explains nearby logic, constraints, or intent: `Convert the lower SPFP value to a 32-bit integer according to the`.
  **L898 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the lower SPFP value to a 32-bit integer according to the`。
- **L899 EN**: Continues the surrounding expression or declaration: `current rounding mode.  */`.
  **L899 CN**: 继续构造周围的表达式或声明：`current rounding mode.  */`。
- **L900 EN**: Blank line separating nearby declarations or logic blocks.
  **L900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L901 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L901 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L902 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L902 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L903 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L903 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L904 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtss_si64(__m128 __A) {`.
  **L904 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtss_si64(__m128 __A) {`。
- **L905 EN**: Adds a standalone statement or declaration: `long long __res;`.
  **L905 CN**: 添加一条独立语句或声明：`long long __res;`。
- **L906 EN**: Starts a preprocessor conditional block: `#if defined(_ARCH_PWR8) && defined(__powerpc64__)`.
  **L906 CN**: 开始一个预处理条件块：`#if defined(_ARCH_PWR8) && defined(__powerpc64__)`。
- **L907 EN**: Adds a standalone statement or declaration: `double __dtmp;`.
  **L907 CN**: 添加一条独立语句或声明：`double __dtmp;`。
- **L908 EN**: Continues logic associated with callable symbol `__asm__`.
  **L908 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L909 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L909 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L910 EN**: Continues the surrounding expression or declaration: `"xxsldwi %x0,%x0,%x0,3;\n"`.
  **L910 CN**: 继续构造周围的表达式或声明：`"xxsldwi %x0,%x0,%x0,3;\n"`。
- **L911 EN**: Closes the current preprocessor conditional block.
  **L911 CN**: 结束当前预处理条件块。
- **L912 EN**: Continues the surrounding expression or declaration: `"xscvspdp %x2,%x0;\n"`.
  **L912 CN**: 继续构造周围的表达式或声明：`"xscvspdp %x2,%x0;\n"`。

### Lines 913-936

````c
      "fctid  %2,%2;\n"
      "mfvsrd  %1,%x2;\n"
      : "+wa"(__A), "=r"(__res), "=f"(__dtmp)
      :);
#else
  __res = __builtin_llrint(__A[0]);
#endif
  return __res;
}

/* Microsoft intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtss_si64x(__m128 __A) {
  return _mm_cvtss_si64((__v4sf)__A);
}

/* Constants for use with _mm_prefetch.  */
enum _mm_hint {
  /* _MM_HINT_ET is _MM_HINT_T with set 3rd bit.  */
  _MM_HINT_ET0 = 7,
  _MM_HINT_ET1 = 6,
  _MM_HINT_T0 = 3,
  _MM_HINT_T1 = 2,
````
- **L913 EN**: Continues the surrounding expression or declaration: `"fctid  %2,%2;\n"`.
  **L913 CN**: 继续构造周围的表达式或声明：`"fctid  %2,%2;\n"`。
- **L914 EN**: Continues the surrounding expression or declaration: `"mfvsrd  %1,%x2;\n"`.
  **L914 CN**: 继续构造周围的表达式或声明：`"mfvsrd  %1,%x2;\n"`。
- **L915 EN**: Continues the surrounding expression or declaration: `: "+wa"(__A), "=r"(__res), "=f"(__dtmp)`.
  **L915 CN**: 继续构造周围的表达式或声明：`: "+wa"(__A), "=r"(__res), "=f"(__dtmp)`。
- **L916 EN**: Adds a standalone statement or declaration: `:);`.
  **L916 CN**: 添加一条独立语句或声明：`:);`。
- **L917 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L917 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L918 EN**: Executes a call or declaration centered on `__builtin_llrint`.
  **L918 CN**: 执行以 `__builtin_llrint` 为核心的调用或声明。
- **L919 EN**: Closes the current preprocessor conditional block.
  **L919 CN**: 结束当前预处理条件块。
- **L920 EN**: Returns from the current function with `__res`.
  **L920 CN**: 以 `__res` 从当前函数返回。
- **L921 EN**: Closes the current lexical scope or compound statement.
  **L921 CN**: 结束当前词法作用域或复合语句块。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L923 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L923 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L924 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L924 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L925 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L925 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L926 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtss_si64x(__m128 __A) {`.
  **L926 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtss_si64x(__m128 __A) {`。
- **L927 EN**: Returns from the current function with `_mm_cvtss_si64((__v4sf)__A)`.
  **L927 CN**: 以 `_mm_cvtss_si64((__v4sf)__A)` 从当前函数返回。
- **L928 EN**: Closes the current lexical scope or compound statement.
  **L928 CN**: 结束当前词法作用域或复合语句块。
- **L929 EN**: Blank line separating nearby declarations or logic blocks.
  **L929 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L930 EN**: Comment explains nearby logic, constraints, or intent: `Constants for use with _mm_prefetch.`.
  **L930 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constants for use with _mm_prefetch.`。
- **L931 EN**: Declares enum `_mm_hint`.
  **L931 CN**: 声明 enum `_mm_hint`。
- **L932 EN**: Comment explains nearby logic, constraints, or intent: `_MM_HINT_ET is _MM_HINT_T with set 3rd bit.`.
  **L932 CN**: 注释解释附近代码的逻辑、约束或设计意图：`_MM_HINT_ET is _MM_HINT_T with set 3rd bit.`。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_MM_HINT_ET0 = 7,`.
  **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`_MM_HINT_ET0 = 7,`。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_MM_HINT_ET1 = 6,`.
  **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`_MM_HINT_ET1 = 6,`。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_MM_HINT_T0 = 3,`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`_MM_HINT_T0 = 3,`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_MM_HINT_T1 = 2,`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`_MM_HINT_T1 = 2,`。

### Lines 937-960

````c
  _MM_HINT_T2 = 1,
  _MM_HINT_NTA = 0
};

/* Loads one cache line from address P to a location "closer" to the
   processor.  The selector I specifies the type of prefetch operation.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_prefetch(const void *__P, enum _mm_hint __I) {
  /* Current PowerPC will ignores the hint parameters.  */
  __builtin_prefetch(__P);
}

/* Convert the two lower SPFP values to 32-bit integers according to the
   current rounding mode.  Return the integers in packed form.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtps_pi32(__m128 __A) {
  /* Splat two lower SPFP values to both halves.  */
  __v4sf __temp, __rounded;
  __vector unsigned long long __result;

  /* Splat two lower SPFP values to both halves.  */
  __temp = (__v4sf)vec_splat((__vector long long)__A, 0);
````
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_MM_HINT_T2 = 1,`.
  **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`_MM_HINT_T2 = 1,`。
- **L938 EN**: Continues the surrounding expression or declaration: `_MM_HINT_NTA = 0`.
  **L938 CN**: 继续构造周围的表达式或声明：`_MM_HINT_NTA = 0`。
- **L939 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L939 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L940 EN**: Blank line separating nearby declarations or logic blocks.
  **L940 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L941 EN**: Comment explains nearby logic, constraints, or intent: `Loads one cache line from address P to a location "closer" to the`.
  **L941 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Loads one cache line from address P to a location "closer" to the`。
- **L942 EN**: Continues the surrounding expression or declaration: `processor.  The selector I specifies the type of prefetch operation.  */`.
  **L942 CN**: 继续构造周围的表达式或声明：`processor.  The selector I specifies the type of prefetch operation.  */`。
- **L943 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L943 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L944 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L944 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L945 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_prefetch(const void *__P, enum _mm_hint __I) {`.
  **L945 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_prefetch(const void *__P, enum _mm_hint __I) {`。
- **L946 EN**: Comment explains nearby logic, constraints, or intent: `Current PowerPC will ignores the hint parameters.`.
  **L946 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Current PowerPC will ignores the hint parameters.`。
- **L947 EN**: Executes a call or declaration centered on `__builtin_prefetch`.
  **L947 CN**: 执行以 `__builtin_prefetch` 为核心的调用或声明。
- **L948 EN**: Closes the current lexical scope or compound statement.
  **L948 CN**: 结束当前词法作用域或复合语句块。
- **L949 EN**: Blank line separating nearby declarations or logic blocks.
  **L949 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L950 EN**: Comment explains nearby logic, constraints, or intent: `Convert the two lower SPFP values to 32-bit integers according to the`.
  **L950 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the two lower SPFP values to 32-bit integers according to the`。
- **L951 EN**: Continues the surrounding expression or declaration: `current rounding mode.  Return the integers in packed form.  */`.
  **L951 CN**: 继续构造周围的表达式或声明：`current rounding mode.  Return the integers in packed form.  */`。
- **L952 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L952 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L953 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L953 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L954 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtps_pi32(__m128 __A) {`.
  **L954 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtps_pi32(__m128 __A) {`。
- **L955 EN**: Comment explains nearby logic, constraints, or intent: `Splat two lower SPFP values to both halves.`.
  **L955 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Splat two lower SPFP values to both halves.`。
- **L956 EN**: Adds a standalone statement or declaration: `__v4sf __temp, __rounded;`.
  **L956 CN**: 添加一条独立语句或声明：`__v4sf __temp, __rounded;`。
- **L957 EN**: Adds a standalone statement or declaration: `__vector unsigned long long __result;`.
  **L957 CN**: 添加一条独立语句或声明：`__vector unsigned long long __result;`。
- **L958 EN**: Blank line separating nearby declarations or logic blocks.
  **L958 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L959 EN**: Comment explains nearby logic, constraints, or intent: `Splat two lower SPFP values to both halves.`.
  **L959 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Splat two lower SPFP values to both halves.`。
- **L960 EN**: Executes a call or declaration centered on `=`.
  **L960 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 961-984

````c
  __rounded = vec_rint(__temp);
  __result = (__vector unsigned long long)vec_cts(__rounded, 0);

  return (__m64)((__vector long long)__result)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvt_ps2pi(__m128 __A) {
  return _mm_cvtps_pi32(__A);
}

/* Truncate the lower SPFP value to a 32-bit integer.  */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttss_si32(__m128 __A) {
  /* Extract the lower float element.  */
  float __temp = __A[0];
  /* truncate to 32-bit integer and return.  */
  return __temp;
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L961 EN**: Executes a call or declaration centered on `vec_rint`.
  **L961 CN**: 执行以 `vec_rint` 为核心的调用或声明。
- **L962 EN**: Executes a call or declaration centered on `=`.
  **L962 CN**: 执行以 `=` 为核心的调用或声明。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L964 EN**: Returns from the current function with `(__m64)((__vector long long)__result)[0]`.
  **L964 CN**: 以 `(__m64)((__vector long long)__result)[0]` 从当前函数返回。
- **L965 EN**: Closes the current lexical scope or compound statement.
  **L965 CN**: 结束当前词法作用域或复合语句块。
- **L966 EN**: Blank line separating nearby declarations or logic blocks.
  **L966 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L967 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L967 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L968 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L968 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L969 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvt_ps2pi(__m128 __A) {`.
  **L969 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvt_ps2pi(__m128 __A) {`。
- **L970 EN**: Returns from the current function with `_mm_cvtps_pi32(__A)`.
  **L970 CN**: 以 `_mm_cvtps_pi32(__A)` 从当前函数返回。
- **L971 EN**: Closes the current lexical scope or compound statement.
  **L971 CN**: 结束当前词法作用域或复合语句块。
- **L972 EN**: Blank line separating nearby declarations or logic blocks.
  **L972 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L973 EN**: Comment explains nearby logic, constraints, or intent: `Truncate the lower SPFP value to a 32-bit integer.`.
  **L973 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Truncate the lower SPFP value to a 32-bit integer.`。
- **L974 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L974 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L975 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L975 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L976 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttss_si32(__m128 __A) {`.
  **L976 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttss_si32(__m128 __A) {`。
- **L977 EN**: Comment explains nearby logic, constraints, or intent: `Extract the lower float element.`.
  **L977 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract the lower float element.`。
- **L978 EN**: Initializes variable `__temp` from the expression on the right-hand side.
  **L978 CN**: 使用右侧表达式初始化变量 `__temp`。
- **L979 EN**: Comment explains nearby logic, constraints, or intent: `truncate to 32-bit integer and return.`.
  **L979 CN**: 注释解释附近代码的逻辑、约束或设计意图：`truncate to 32-bit integer and return.`。
- **L980 EN**: Returns from the current function with `__temp`.
  **L980 CN**: 以 `__temp` 从当前函数返回。
- **L981 EN**: Closes the current lexical scope or compound statement.
  **L981 CN**: 结束当前词法作用域或复合语句块。
- **L982 EN**: Blank line separating nearby declarations or logic blocks.
  **L982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L983 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L983 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L984 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L984 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 985-1008

````c
    _mm_cvtt_ss2si(__m128 __A) {
  return _mm_cvttss_si32(__A);
}

/* Intel intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttss_si64(__m128 __A) {
  /* Extract the lower float element.  */
  float __temp = __A[0];
  /* truncate to 32-bit integer and return.  */
  return __temp;
}

/* Microsoft intrinsic.  */
extern __inline long long
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttss_si64x(__m128 __A) {
  /* Extract the lower float element.  */
  float __temp = __A[0];
  /* truncate to 32-bit integer and return.  */
  return __temp;
}

````
- **L985 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtt_ss2si(__m128 __A) {`.
  **L985 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtt_ss2si(__m128 __A) {`。
- **L986 EN**: Returns from the current function with `_mm_cvttss_si32(__A)`.
  **L986 CN**: 以 `_mm_cvttss_si32(__A)` 从当前函数返回。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Blank line separating nearby declarations or logic blocks.
  **L988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L989 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L989 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L990 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L990 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L991 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L991 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L992 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttss_si64(__m128 __A) {`.
  **L992 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttss_si64(__m128 __A) {`。
- **L993 EN**: Comment explains nearby logic, constraints, or intent: `Extract the lower float element.`.
  **L993 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract the lower float element.`。
- **L994 EN**: Initializes variable `__temp` from the expression on the right-hand side.
  **L994 CN**: 使用右侧表达式初始化变量 `__temp`。
- **L995 EN**: Comment explains nearby logic, constraints, or intent: `truncate to 32-bit integer and return.`.
  **L995 CN**: 注释解释附近代码的逻辑、约束或设计意图：`truncate to 32-bit integer and return.`。
- **L996 EN**: Returns from the current function with `__temp`.
  **L996 CN**: 以 `__temp` 从当前函数返回。
- **L997 EN**: Closes the current lexical scope or compound statement.
  **L997 CN**: 结束当前词法作用域或复合语句块。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L999 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L999 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L1000 EN**: Continues the surrounding expression or declaration: `extern __inline long long`.
  **L1000 CN**: 继续构造周围的表达式或声明：`extern __inline long long`。
- **L1001 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1001 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1002 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttss_si64x(__m128 __A) {`.
  **L1002 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttss_si64x(__m128 __A) {`。
- **L1003 EN**: Comment explains nearby logic, constraints, or intent: `Extract the lower float element.`.
  **L1003 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract the lower float element.`。
- **L1004 EN**: Initializes variable `__temp` from the expression on the right-hand side.
  **L1004 CN**: 使用右侧表达式初始化变量 `__temp`。
- **L1005 EN**: Comment explains nearby logic, constraints, or intent: `truncate to 32-bit integer and return.`.
  **L1005 CN**: 注释解释附近代码的逻辑、约束或设计意图：`truncate to 32-bit integer and return.`。
- **L1006 EN**: Returns from the current function with `__temp`.
  **L1006 CN**: 以 `__temp` 从当前函数返回。
- **L1007 EN**: Closes the current lexical scope or compound statement.
  **L1007 CN**: 结束当前词法作用域或复合语句块。
- **L1008 EN**: Blank line separating nearby declarations or logic blocks.
  **L1008 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1009-1032

````c
/* Truncate the two lower SPFP values to 32-bit integers.  Return the
   integers in packed form.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvttps_pi32(__m128 __A) {
  __v4sf __temp;
  __vector unsigned long long __result;

  /* Splat two lower SPFP values to both halves.  */
  __temp = (__v4sf)vec_splat((__vector long long)__A, 0);
  __result = (__vector unsigned long long)vec_cts(__temp, 0);

  return (__m64)((__vector long long)__result)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtt_ps2pi(__m128 __A) {
  return _mm_cvttps_pi32(__A);
}

/* Convert B to a SPFP value and insert it as element zero in A.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L1009 EN**: Comment explains nearby logic, constraints, or intent: `Truncate the two lower SPFP values to 32-bit integers. Return the`.
  **L1009 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Truncate the two lower SPFP values to 32-bit integers. Return the`。
- **L1010 EN**: Continues the surrounding expression or declaration: `integers in packed form.  */`.
  **L1010 CN**: 继续构造周围的表达式或声明：`integers in packed form.  */`。
- **L1011 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1011 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1012 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1012 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1013 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttps_pi32(__m128 __A) {`.
  **L1013 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttps_pi32(__m128 __A) {`。
- **L1014 EN**: Adds a standalone statement or declaration: `__v4sf __temp;`.
  **L1014 CN**: 添加一条独立语句或声明：`__v4sf __temp;`。
- **L1015 EN**: Adds a standalone statement or declaration: `__vector unsigned long long __result;`.
  **L1015 CN**: 添加一条独立语句或声明：`__vector unsigned long long __result;`。
- **L1016 EN**: Blank line separating nearby declarations or logic blocks.
  **L1016 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1017 EN**: Comment explains nearby logic, constraints, or intent: `Splat two lower SPFP values to both halves.`.
  **L1017 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Splat two lower SPFP values to both halves.`。
- **L1018 EN**: Executes a call or declaration centered on `=`.
  **L1018 CN**: 执行以 `=` 为核心的调用或声明。
- **L1019 EN**: Executes a call or declaration centered on `=`.
  **L1019 CN**: 执行以 `=` 为核心的调用或声明。
- **L1020 EN**: Blank line separating nearby declarations or logic blocks.
  **L1020 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1021 EN**: Returns from the current function with `(__m64)((__vector long long)__result)[0]`.
  **L1021 CN**: 以 `(__m64)((__vector long long)__result)[0]` 从当前函数返回。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Blank line separating nearby declarations or logic blocks.
  **L1023 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1024 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1024 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1025 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1025 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1026 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtt_ps2pi(__m128 __A) {`.
  **L1026 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtt_ps2pi(__m128 __A) {`。
- **L1027 EN**: Returns from the current function with `_mm_cvttps_pi32(__A)`.
  **L1027 CN**: 以 `_mm_cvttps_pi32(__A)` 从当前函数返回。
- **L1028 EN**: Closes the current lexical scope or compound statement.
  **L1028 CN**: 结束当前词法作用域或复合语句块。
- **L1029 EN**: Blank line separating nearby declarations or logic blocks.
  **L1029 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1030 EN**: Comment explains nearby logic, constraints, or intent: `Convert B to a SPFP value and insert it as element zero in A.`.
  **L1030 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert B to a SPFP value and insert it as element zero in A.`。
- **L1031 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1031 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1032 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1032 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 1033-1056

````c
    _mm_cvtsi32_ss(__m128 __A, int __B) {
  float __temp = __B;
  __A[0] = __temp;

  return __A;
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvt_si2ss(__m128 __A, int __B) {
  return _mm_cvtsi32_ss(__A, __B);
}

/* Convert B to a SPFP value and insert it as element zero in A.  */
/* Intel intrinsic.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64_ss(__m128 __A, long long __B) {
  float __temp = __B;
  __A[0] = __temp;

  return __A;
}

````
- **L1033 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi32_ss(__m128 __A, int __B) {`.
  **L1033 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi32_ss(__m128 __A, int __B) {`。
- **L1034 EN**: Initializes variable `__temp` from the expression on the right-hand side.
  **L1034 CN**: 使用右侧表达式初始化变量 `__temp`。
- **L1035 EN**: Adds a standalone statement or declaration: `__A[0] = __temp;`.
  **L1035 CN**: 添加一条独立语句或声明：`__A[0] = __temp;`。
- **L1036 EN**: Blank line separating nearby declarations or logic blocks.
  **L1036 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1037 EN**: Returns from the current function with `__A`.
  **L1037 CN**: 以 `__A` 从当前函数返回。
- **L1038 EN**: Closes the current lexical scope or compound statement.
  **L1038 CN**: 结束当前词法作用域或复合语句块。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1040 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1040 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1041 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1041 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1042 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvt_si2ss(__m128 __A, int __B) {`.
  **L1042 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvt_si2ss(__m128 __A, int __B) {`。
- **L1043 EN**: Returns from the current function with `_mm_cvtsi32_ss(__A, __B)`.
  **L1043 CN**: 以 `_mm_cvtsi32_ss(__A, __B)` 从当前函数返回。
- **L1044 EN**: Closes the current lexical scope or compound statement.
  **L1044 CN**: 结束当前词法作用域或复合语句块。
- **L1045 EN**: Blank line separating nearby declarations or logic blocks.
  **L1045 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1046 EN**: Comment explains nearby logic, constraints, or intent: `Convert B to a SPFP value and insert it as element zero in A.`.
  **L1046 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert B to a SPFP value and insert it as element zero in A.`。
- **L1047 EN**: Comment explains nearby logic, constraints, or intent: `Intel intrinsic.`.
  **L1047 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intel intrinsic.`。
- **L1048 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1048 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1049 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1049 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1050 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64_ss(__m128 __A, long long __B) {`.
  **L1050 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64_ss(__m128 __A, long long __B) {`。
- **L1051 EN**: Initializes variable `__temp` from the expression on the right-hand side.
  **L1051 CN**: 使用右侧表达式初始化变量 `__temp`。
- **L1052 EN**: Adds a standalone statement or declaration: `__A[0] = __temp;`.
  **L1052 CN**: 添加一条独立语句或声明：`__A[0] = __temp;`。
- **L1053 EN**: Blank line separating nearby declarations or logic blocks.
  **L1053 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1054 EN**: Returns from the current function with `__A`.
  **L1054 CN**: 以 `__A` 从当前函数返回。
- **L1055 EN**: Closes the current lexical scope or compound statement.
  **L1055 CN**: 结束当前词法作用域或复合语句块。
- **L1056 EN**: Blank line separating nearby declarations or logic blocks.
  **L1056 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1057-1080

````c
/* Microsoft intrinsic.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtsi64x_ss(__m128 __A, long long __B) {
  return _mm_cvtsi64_ss(__A, __B);
}

/* Convert the two 32-bit values in B to SPFP form and insert them
   as the two lower elements in A.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpi32_ps(__m128 __A, __m64 __B) {
  __vector signed int __vm1;
  __vector float __vf1;

  __vm1 = (__vector signed int)(__vector unsigned long long){__B, __B};
  __vf1 = (__vector float)vec_ctf(__vm1, 0);

  return ((__m128)(__vector unsigned long long){
      ((__vector unsigned long long)__vf1)[0],
      ((__vector unsigned long long)__A)[1]});
}

extern __inline __m128
````
- **L1057 EN**: Comment explains nearby logic, constraints, or intent: `Microsoft intrinsic.`.
  **L1057 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Microsoft intrinsic.`。
- **L1058 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1058 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1059 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1059 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1060 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsi64x_ss(__m128 __A, long long __B) {`.
  **L1060 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsi64x_ss(__m128 __A, long long __B) {`。
- **L1061 EN**: Returns from the current function with `_mm_cvtsi64_ss(__A, __B)`.
  **L1061 CN**: 以 `_mm_cvtsi64_ss(__A, __B)` 从当前函数返回。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1064 EN**: Comment explains nearby logic, constraints, or intent: `Convert the two 32-bit values in B to SPFP form and insert them`.
  **L1064 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the two 32-bit values in B to SPFP form and insert them`。
- **L1065 EN**: Continues the surrounding expression or declaration: `as the two lower elements in A.  */`.
  **L1065 CN**: 继续构造周围的表达式或声明：`as the two lower elements in A.  */`。
- **L1066 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1066 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1067 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1067 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1068 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpi32_ps(__m128 __A, __m64 __B) {`.
  **L1068 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpi32_ps(__m128 __A, __m64 __B) {`。
- **L1069 EN**: Adds a standalone statement or declaration: `__vector signed int __vm1;`.
  **L1069 CN**: 添加一条独立语句或声明：`__vector signed int __vm1;`。
- **L1070 EN**: Adds a standalone statement or declaration: `__vector float __vf1;`.
  **L1070 CN**: 添加一条独立语句或声明：`__vector float __vf1;`。
- **L1071 EN**: Blank line separating nearby declarations or logic blocks.
  **L1071 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1072 EN**: Executes a call or declaration centered on `=`.
  **L1072 CN**: 执行以 `=` 为核心的调用或声明。
- **L1073 EN**: Executes a call or declaration centered on `=`.
  **L1073 CN**: 执行以 `=` 为核心的调用或声明。
- **L1074 EN**: Blank line separating nearby declarations or logic blocks.
  **L1074 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1075 EN**: Returns from the current function with `((__m128)(__vector unsigned long long){`.
  **L1075 CN**: 以 `((__m128)(__vector unsigned long long){` 从当前函数返回。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__vector unsigned long long)__vf1)[0],`.
  **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__vector unsigned long long)__vf1)[0],`。
- **L1077 EN**: Executes a call or declaration centered on `statement`.
  **L1077 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1078 EN**: Closes the current lexical scope or compound statement.
  **L1078 CN**: 结束当前词法作用域或复合语句块。
- **L1079 EN**: Blank line separating nearby declarations or logic blocks.
  **L1079 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1080 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1080 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。

### Lines 1081-1104

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvt_pi2ps(__m128 __A, __m64 __B) {
  return _mm_cvtpi32_ps(__A, __B);
}

/* Convert the four signed 16-bit values in A to SPFP form.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpi16_ps(__m64 __A) {
  __vector signed short __vs8;
  __vector signed int __vi4;
  __vector float __vf1;

  __vs8 = (__vector signed short)(__vector unsigned long long){__A, __A};
  __vi4 = vec_vupklsh(__vs8);
  __vf1 = (__vector float)vec_ctf(__vi4, 0);

  return (__m128)__vf1;
}

/* Convert the four unsigned 16-bit values in A to SPFP form.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpu16_ps(__m64 __A) {
````
- **L1081 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1081 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1082 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvt_pi2ps(__m128 __A, __m64 __B) {`.
  **L1082 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvt_pi2ps(__m128 __A, __m64 __B) {`。
- **L1083 EN**: Returns from the current function with `_mm_cvtpi32_ps(__A, __B)`.
  **L1083 CN**: 以 `_mm_cvtpi32_ps(__A, __B)` 从当前函数返回。
- **L1084 EN**: Closes the current lexical scope or compound statement.
  **L1084 CN**: 结束当前词法作用域或复合语句块。
- **L1085 EN**: Blank line separating nearby declarations or logic blocks.
  **L1085 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1086 EN**: Comment explains nearby logic, constraints, or intent: `Convert the four signed 16-bit values in A to SPFP form.`.
  **L1086 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the four signed 16-bit values in A to SPFP form.`。
- **L1087 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1087 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1088 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1088 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1089 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpi16_ps(__m64 __A) {`.
  **L1089 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpi16_ps(__m64 __A) {`。
- **L1090 EN**: Adds a standalone statement or declaration: `__vector signed short __vs8;`.
  **L1090 CN**: 添加一条独立语句或声明：`__vector signed short __vs8;`。
- **L1091 EN**: Adds a standalone statement or declaration: `__vector signed int __vi4;`.
  **L1091 CN**: 添加一条独立语句或声明：`__vector signed int __vi4;`。
- **L1092 EN**: Adds a standalone statement or declaration: `__vector float __vf1;`.
  **L1092 CN**: 添加一条独立语句或声明：`__vector float __vf1;`。
- **L1093 EN**: Blank line separating nearby declarations or logic blocks.
  **L1093 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1094 EN**: Executes a call or declaration centered on `=`.
  **L1094 CN**: 执行以 `=` 为核心的调用或声明。
- **L1095 EN**: Executes a call or declaration centered on `vec_vupklsh`.
  **L1095 CN**: 执行以 `vec_vupklsh` 为核心的调用或声明。
- **L1096 EN**: Executes a call or declaration centered on `=`.
  **L1096 CN**: 执行以 `=` 为核心的调用或声明。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1098 EN**: Returns from the current function with `(__m128)__vf1`.
  **L1098 CN**: 以 `(__m128)__vf1` 从当前函数返回。
- **L1099 EN**: Closes the current lexical scope or compound statement.
  **L1099 CN**: 结束当前词法作用域或复合语句块。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1101 EN**: Comment explains nearby logic, constraints, or intent: `Convert the four unsigned 16-bit values in A to SPFP form.`.
  **L1101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the four unsigned 16-bit values in A to SPFP form.`。
- **L1102 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1102 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1103 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1103 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1104 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpu16_ps(__m64 __A) {`.
  **L1104 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpu16_ps(__m64 __A) {`。

### Lines 1105-1128

````c
  const __vector unsigned short __zero = {0, 0, 0, 0, 0, 0, 0, 0};
  __vector unsigned short __vs8;
  __vector unsigned int __vi4;
  __vector float __vf1;

  __vs8 = (__vector unsigned short)(__vector unsigned long long){__A, __A};
  __vi4 = (__vector unsigned int)vec_mergel
#ifdef __LITTLE_ENDIAN__
      (__vs8, __zero);
#else
      (__zero, __vs8);
#endif
  __vf1 = (__vector float)vec_ctf(__vi4, 0);

  return (__m128)__vf1;
}

/* Convert the low four signed 8-bit values in A to SPFP form.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpi8_ps(__m64 __A) {
  __vector signed char __vc16;
  __vector signed short __vs8;
  __vector signed int __vi4;
````
- **L1105 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L1105 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L1106 EN**: Adds a standalone statement or declaration: `__vector unsigned short __vs8;`.
  **L1106 CN**: 添加一条独立语句或声明：`__vector unsigned short __vs8;`。
- **L1107 EN**: Adds a standalone statement or declaration: `__vector unsigned int __vi4;`.
  **L1107 CN**: 添加一条独立语句或声明：`__vector unsigned int __vi4;`。
- **L1108 EN**: Adds a standalone statement or declaration: `__vector float __vf1;`.
  **L1108 CN**: 添加一条独立语句或声明：`__vector float __vf1;`。
- **L1109 EN**: Blank line separating nearby declarations or logic blocks.
  **L1109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1110 EN**: Executes a call or declaration centered on `=`.
  **L1110 CN**: 执行以 `=` 为核心的调用或声明。
- **L1111 EN**: Continues the surrounding expression or declaration: `__vi4 = (__vector unsigned int)vec_mergel`.
  **L1111 CN**: 继续构造周围的表达式或声明：`__vi4 = (__vector unsigned int)vec_mergel`。
- **L1112 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1112 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1113 EN**: Executes a call or declaration centered on `statement`.
  **L1113 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1114 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1114 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1115 EN**: Executes a call or declaration centered on `statement`.
  **L1115 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1116 EN**: Closes the current preprocessor conditional block.
  **L1116 CN**: 结束当前预处理条件块。
- **L1117 EN**: Executes a call or declaration centered on `=`.
  **L1117 CN**: 执行以 `=` 为核心的调用或声明。
- **L1118 EN**: Blank line separating nearby declarations or logic blocks.
  **L1118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1119 EN**: Returns from the current function with `(__m128)__vf1`.
  **L1119 CN**: 以 `(__m128)__vf1` 从当前函数返回。
- **L1120 EN**: Closes the current lexical scope or compound statement.
  **L1120 CN**: 结束当前词法作用域或复合语句块。
- **L1121 EN**: Blank line separating nearby declarations or logic blocks.
  **L1121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1122 EN**: Comment explains nearby logic, constraints, or intent: `Convert the low four signed 8-bit values in A to SPFP form.`.
  **L1122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the low four signed 8-bit values in A to SPFP form.`。
- **L1123 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1123 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1124 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1124 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpi8_ps(__m64 __A) {`.
  **L1125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpi8_ps(__m64 __A) {`。
- **L1126 EN**: Adds a standalone statement or declaration: `__vector signed char __vc16;`.
  **L1126 CN**: 添加一条独立语句或声明：`__vector signed char __vc16;`。
- **L1127 EN**: Adds a standalone statement or declaration: `__vector signed short __vs8;`.
  **L1127 CN**: 添加一条独立语句或声明：`__vector signed short __vs8;`。
- **L1128 EN**: Adds a standalone statement or declaration: `__vector signed int __vi4;`.
  **L1128 CN**: 添加一条独立语句或声明：`__vector signed int __vi4;`。

### Lines 1129-1152

````c
  __vector float __vf1;

  __vc16 = (__vector signed char)(__vector unsigned long long){__A, __A};
  __vs8 = vec_vupkhsb(__vc16);
  __vi4 = vec_vupkhsh(__vs8);
  __vf1 = (__vector float)vec_ctf(__vi4, 0);

  return (__m128)__vf1;
}

/* Convert the low four unsigned 8-bit values in A to SPFP form.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))

    _mm_cvtpu8_ps(__m64 __A) {
  const __vector unsigned char __zero = {0, 0, 0, 0, 0, 0, 0, 0};
  __vector unsigned char __vc16;
  __vector unsigned short __vs8;
  __vector unsigned int __vi4;
  __vector float __vf1;

  __vc16 = (__vector unsigned char)(__vector unsigned long long){__A, __A};
#ifdef __LITTLE_ENDIAN__
  __vs8 = (__vector unsigned short)vec_mergel(__vc16, __zero);
````
- **L1129 EN**: Adds a standalone statement or declaration: `__vector float __vf1;`.
  **L1129 CN**: 添加一条独立语句或声明：`__vector float __vf1;`。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1131 EN**: Executes a call or declaration centered on `=`.
  **L1131 CN**: 执行以 `=` 为核心的调用或声明。
- **L1132 EN**: Executes a call or declaration centered on `vec_vupkhsb`.
  **L1132 CN**: 执行以 `vec_vupkhsb` 为核心的调用或声明。
- **L1133 EN**: Executes a call or declaration centered on `vec_vupkhsh`.
  **L1133 CN**: 执行以 `vec_vupkhsh` 为核心的调用或声明。
- **L1134 EN**: Executes a call or declaration centered on `=`.
  **L1134 CN**: 执行以 `=` 为核心的调用或声明。
- **L1135 EN**: Blank line separating nearby declarations or logic blocks.
  **L1135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1136 EN**: Returns from the current function with `(__m128)__vf1`.
  **L1136 CN**: 以 `(__m128)__vf1` 从当前函数返回。
- **L1137 EN**: Closes the current lexical scope or compound statement.
  **L1137 CN**: 结束当前词法作用域或复合语句块。
- **L1138 EN**: Blank line separating nearby declarations or logic blocks.
  **L1138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1139 EN**: Comment explains nearby logic, constraints, or intent: `Convert the low four unsigned 8-bit values in A to SPFP form.`.
  **L1139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the low four unsigned 8-bit values in A to SPFP form.`。
- **L1140 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1140 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1141 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1141 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1142 EN**: Blank line separating nearby declarations or logic blocks.
  **L1142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1143 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpu8_ps(__m64 __A) {`.
  **L1143 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpu8_ps(__m64 __A) {`。
- **L1144 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L1144 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L1145 EN**: Adds a standalone statement or declaration: `__vector unsigned char __vc16;`.
  **L1145 CN**: 添加一条独立语句或声明：`__vector unsigned char __vc16;`。
- **L1146 EN**: Adds a standalone statement or declaration: `__vector unsigned short __vs8;`.
  **L1146 CN**: 添加一条独立语句或声明：`__vector unsigned short __vs8;`。
- **L1147 EN**: Adds a standalone statement or declaration: `__vector unsigned int __vi4;`.
  **L1147 CN**: 添加一条独立语句或声明：`__vector unsigned int __vi4;`。
- **L1148 EN**: Adds a standalone statement or declaration: `__vector float __vf1;`.
  **L1148 CN**: 添加一条独立语句或声明：`__vector float __vf1;`。
- **L1149 EN**: Blank line separating nearby declarations or logic blocks.
  **L1149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1150 EN**: Executes a call or declaration centered on `=`.
  **L1150 CN**: 执行以 `=` 为核心的调用或声明。
- **L1151 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1151 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1152 EN**: Executes a call or declaration centered on `=`.
  **L1152 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 1153-1176

````c
  __vi4 =
      (__vector unsigned int)vec_mergeh(__vs8, (__vector unsigned short)__zero);
#else
  __vs8 = (__vector unsigned short)vec_mergel(__zero, __vc16);
  __vi4 =
      (__vector unsigned int)vec_mergeh((__vector unsigned short)__zero, __vs8);
#endif
  __vf1 = (__vector float)vec_ctf(__vi4, 0);

  return (__m128)__vf1;
}

/* Convert the four signed 32-bit values in A and B to SPFP form.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtpi32x2_ps(__m64 __A, __m64 __B) {
  __vector signed int __vi4;
  __vector float __vf4;

  __vi4 = (__vector signed int)(__vector unsigned long long){__A, __B};
  __vf4 = (__vector float)vec_ctf(__vi4, 0);
  return (__m128)__vf4;
}

````
- **L1153 EN**: Continues the surrounding expression or declaration: `__vi4 =`.
  **L1153 CN**: 继续构造周围的表达式或声明：`__vi4 =`。
- **L1154 EN**: Executes a call or declaration centered on `statement`.
  **L1154 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1155 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1155 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1156 EN**: Executes a call or declaration centered on `=`.
  **L1156 CN**: 执行以 `=` 为核心的调用或声明。
- **L1157 EN**: Continues the surrounding expression or declaration: `__vi4 =`.
  **L1157 CN**: 继续构造周围的表达式或声明：`__vi4 =`。
- **L1158 EN**: Executes a call or declaration centered on `statement`.
  **L1158 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1159 EN**: Closes the current preprocessor conditional block.
  **L1159 CN**: 结束当前预处理条件块。
- **L1160 EN**: Executes a call or declaration centered on `=`.
  **L1160 CN**: 执行以 `=` 为核心的调用或声明。
- **L1161 EN**: Blank line separating nearby declarations or logic blocks.
  **L1161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1162 EN**: Returns from the current function with `(__m128)__vf1`.
  **L1162 CN**: 以 `(__m128)__vf1` 从当前函数返回。
- **L1163 EN**: Closes the current lexical scope or compound statement.
  **L1163 CN**: 结束当前词法作用域或复合语句块。
- **L1164 EN**: Blank line separating nearby declarations or logic blocks.
  **L1164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1165 EN**: Comment explains nearby logic, constraints, or intent: `Convert the four signed 32-bit values in A and B to SPFP form.`.
  **L1165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the four signed 32-bit values in A and B to SPFP form.`。
- **L1166 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1166 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1167 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1167 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1168 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpi32x2_ps(__m64 __A, __m64 __B) {`.
  **L1168 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpi32x2_ps(__m64 __A, __m64 __B) {`。
- **L1169 EN**: Adds a standalone statement or declaration: `__vector signed int __vi4;`.
  **L1169 CN**: 添加一条独立语句或声明：`__vector signed int __vi4;`。
- **L1170 EN**: Adds a standalone statement or declaration: `__vector float __vf4;`.
  **L1170 CN**: 添加一条独立语句或声明：`__vector float __vf4;`。
- **L1171 EN**: Blank line separating nearby declarations or logic blocks.
  **L1171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1172 EN**: Executes a call or declaration centered on `=`.
  **L1172 CN**: 执行以 `=` 为核心的调用或声明。
- **L1173 EN**: Executes a call or declaration centered on `=`.
  **L1173 CN**: 执行以 `=` 为核心的调用或声明。
- **L1174 EN**: Returns from the current function with `(__m128)__vf4`.
  **L1174 CN**: 以 `(__m128)__vf4` 从当前函数返回。
- **L1175 EN**: Closes the current lexical scope or compound statement.
  **L1175 CN**: 结束当前词法作用域或复合语句块。
- **L1176 EN**: Blank line separating nearby declarations or logic blocks.
  **L1176 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1177-1200

````c
/* Convert the four SPFP values in A to four signed 16-bit integers.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtps_pi16(__m128 __A) {
  __v4sf __rounded;
  __vector signed int __temp;
  __vector unsigned long long __result;

  __rounded = vec_rint(__A);
  __temp = vec_cts(__rounded, 0);
  __result = (__vector unsigned long long)vec_pack(__temp, __temp);

  return (__m64)((__vector long long)__result)[0];
}

/* Convert the four SPFP values in A to four signed 8-bit integers.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtps_pi8(__m128 __A) {
  __v4sf __rounded;
  __vector signed int __tmp_i;
  static const __vector signed int __zero = {0, 0, 0, 0};
  __vector signed short __tmp_s;
  __vector signed char __res_v;
````
- **L1177 EN**: Comment explains nearby logic, constraints, or intent: `Convert the four SPFP values in A to four signed 16-bit integers.`.
  **L1177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the four SPFP values in A to four signed 16-bit integers.`。
- **L1178 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1178 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1179 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1179 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1180 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtps_pi16(__m128 __A) {`.
  **L1180 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtps_pi16(__m128 __A) {`。
- **L1181 EN**: Adds a standalone statement or declaration: `__v4sf __rounded;`.
  **L1181 CN**: 添加一条独立语句或声明：`__v4sf __rounded;`。
- **L1182 EN**: Adds a standalone statement or declaration: `__vector signed int __temp;`.
  **L1182 CN**: 添加一条独立语句或声明：`__vector signed int __temp;`。
- **L1183 EN**: Adds a standalone statement or declaration: `__vector unsigned long long __result;`.
  **L1183 CN**: 添加一条独立语句或声明：`__vector unsigned long long __result;`。
- **L1184 EN**: Blank line separating nearby declarations or logic blocks.
  **L1184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1185 EN**: Executes a call or declaration centered on `vec_rint`.
  **L1185 CN**: 执行以 `vec_rint` 为核心的调用或声明。
- **L1186 EN**: Executes a call or declaration centered on `vec_cts`.
  **L1186 CN**: 执行以 `vec_cts` 为核心的调用或声明。
- **L1187 EN**: Executes a call or declaration centered on `=`.
  **L1187 CN**: 执行以 `=` 为核心的调用或声明。
- **L1188 EN**: Blank line separating nearby declarations or logic blocks.
  **L1188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1189 EN**: Returns from the current function with `(__m64)((__vector long long)__result)[0]`.
  **L1189 CN**: 以 `(__m64)((__vector long long)__result)[0]` 从当前函数返回。
- **L1190 EN**: Closes the current lexical scope or compound statement.
  **L1190 CN**: 结束当前词法作用域或复合语句块。
- **L1191 EN**: Blank line separating nearby declarations or logic blocks.
  **L1191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1192 EN**: Comment explains nearby logic, constraints, or intent: `Convert the four SPFP values in A to four signed 8-bit integers.`.
  **L1192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert the four SPFP values in A to four signed 8-bit integers.`。
- **L1193 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1193 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1194 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1194 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1195 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtps_pi8(__m128 __A) {`.
  **L1195 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtps_pi8(__m128 __A) {`。
- **L1196 EN**: Adds a standalone statement or declaration: `__v4sf __rounded;`.
  **L1196 CN**: 添加一条独立语句或声明：`__v4sf __rounded;`。
- **L1197 EN**: Adds a standalone statement or declaration: `__vector signed int __tmp_i;`.
  **L1197 CN**: 添加一条独立语句或声明：`__vector signed int __tmp_i;`。
- **L1198 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L1198 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L1199 EN**: Adds a standalone statement or declaration: `__vector signed short __tmp_s;`.
  **L1199 CN**: 添加一条独立语句或声明：`__vector signed short __tmp_s;`。
- **L1200 EN**: Adds a standalone statement or declaration: `__vector signed char __res_v;`.
  **L1200 CN**: 添加一条独立语句或声明：`__vector signed char __res_v;`。

### Lines 1201-1224

````c

  __rounded = vec_rint(__A);
  __tmp_i = vec_cts(__rounded, 0);
  __tmp_s = vec_pack(__tmp_i, __zero);
  __res_v = vec_pack(__tmp_s, __tmp_s);
  return (__m64)((__vector long long)__res_v)[0];
}

/* Selects four specific SPFP values from A and B based on MASK.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))

    _mm_shuffle_ps(__m128 __A, __m128 __B, int const __mask) {
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
````
- **L1201 EN**: Blank line separating nearby declarations or logic blocks.
  **L1201 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1202 EN**: Executes a call or declaration centered on `vec_rint`.
  **L1202 CN**: 执行以 `vec_rint` 为核心的调用或声明。
- **L1203 EN**: Executes a call or declaration centered on `vec_cts`.
  **L1203 CN**: 执行以 `vec_cts` 为核心的调用或声明。
- **L1204 EN**: Executes a call or declaration centered on `vec_pack`.
  **L1204 CN**: 执行以 `vec_pack` 为核心的调用或声明。
- **L1205 EN**: Executes a call or declaration centered on `vec_pack`.
  **L1205 CN**: 执行以 `vec_pack` 为核心的调用或声明。
- **L1206 EN**: Returns from the current function with `(__m64)((__vector long long)__res_v)[0]`.
  **L1206 CN**: 以 `(__m64)((__vector long long)__res_v)[0]` 从当前函数返回。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1209 EN**: Comment explains nearby logic, constraints, or intent: `Selects four specific SPFP values from A and B based on MASK.`.
  **L1209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Selects four specific SPFP values from A and B based on MASK.`。
- **L1210 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1210 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1211 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1211 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1212 EN**: Blank line separating nearby declarations or logic blocks.
  **L1212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1213 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_shuffle_ps(__m128 __A, __m128 __B, int const __mask) {`.
  **L1213 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_shuffle_ps(__m128 __A, __m128 __B, int const __mask) {`。
- **L1214 EN**: Initializes variable `__element_selector_10` from the expression on the right-hand side.
  **L1214 CN**: 使用右侧表达式初始化变量 `__element_selector_10`。
- **L1215 EN**: Initializes variable `__element_selector_32` from the expression on the right-hand side.
  **L1215 CN**: 使用右侧表达式初始化变量 `__element_selector_32`。
- **L1216 EN**: Initializes variable `__element_selector_54` from the expression on the right-hand side.
  **L1216 CN**: 使用右侧表达式初始化变量 `__element_selector_54`。
- **L1217 EN**: Initializes variable `__element_selector_76` from the expression on the right-hand side.
  **L1217 CN**: 使用右侧表达式初始化变量 `__element_selector_76`。
- **L1218 EN**: Continues the surrounding expression or declaration: `static const unsigned int __permute_selectors[4] = {`.
  **L1218 CN**: 继续构造周围的表达式或声明：`static const unsigned int __permute_selectors[4] = {`。
- **L1219 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1219 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1220 EN**: Continues the surrounding expression or declaration: `0x03020100, 0x07060504, 0x0B0A0908, 0x0F0E0D0C`.
  **L1220 CN**: 继续构造周围的表达式或声明：`0x03020100, 0x07060504, 0x0B0A0908, 0x0F0E0D0C`。
- **L1221 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1221 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1222 EN**: Continues the surrounding expression or declaration: `0x00010203, 0x04050607, 0x08090A0B, 0x0C0D0E0F`.
  **L1222 CN**: 继续构造周围的表达式或声明：`0x00010203, 0x04050607, 0x08090A0B, 0x0C0D0E0F`。
- **L1223 EN**: Closes the current preprocessor conditional block.
  **L1223 CN**: 结束当前预处理条件块。
- **L1224 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1224 CN**: 结束当前声明作用域，例如结构体、枚举或类。

### Lines 1225-1248

````c
  __vector unsigned int __t;

  __t[0] = __permute_selectors[__element_selector_10];
  __t[1] = __permute_selectors[__element_selector_32];
  __t[2] = __permute_selectors[__element_selector_54] + 0x10101010;
  __t[3] = __permute_selectors[__element_selector_76] + 0x10101010;
  return vec_perm((__v4sf)__A, (__v4sf)__B, (__vector unsigned char)__t);
}

/* Selects and interleaves the upper two SPFP values from A and B.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpackhi_ps(__m128 __A, __m128 __B) {
  return (__m128)vec_vmrglw((__v4sf)__A, (__v4sf)__B);
}

/* Selects and interleaves the lower two SPFP values from A and B.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_unpacklo_ps(__m128 __A, __m128 __B) {
  return (__m128)vec_vmrghw((__v4sf)__A, (__v4sf)__B);
}

/* Sets the upper two SPFP values with 64-bits of data loaded from P;
````
- **L1225 EN**: Adds a standalone statement or declaration: `__vector unsigned int __t;`.
  **L1225 CN**: 添加一条独立语句或声明：`__vector unsigned int __t;`。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1227 EN**: Adds a standalone statement or declaration: `__t[0] = __permute_selectors[__element_selector_10];`.
  **L1227 CN**: 添加一条独立语句或声明：`__t[0] = __permute_selectors[__element_selector_10];`。
- **L1228 EN**: Adds a standalone statement or declaration: `__t[1] = __permute_selectors[__element_selector_32];`.
  **L1228 CN**: 添加一条独立语句或声明：`__t[1] = __permute_selectors[__element_selector_32];`。
- **L1229 EN**: Adds a standalone statement or declaration: `__t[2] = __permute_selectors[__element_selector_54] + 0x10101010;`.
  **L1229 CN**: 添加一条独立语句或声明：`__t[2] = __permute_selectors[__element_selector_54] + 0x10101010;`。
- **L1230 EN**: Adds a standalone statement or declaration: `__t[3] = __permute_selectors[__element_selector_76] + 0x10101010;`.
  **L1230 CN**: 添加一条独立语句或声明：`__t[3] = __permute_selectors[__element_selector_76] + 0x10101010;`。
- **L1231 EN**: Returns from the current function with `vec_perm((__v4sf)__A, (__v4sf)__B, (__vector unsigned char)__t)`.
  **L1231 CN**: 以 `vec_perm((__v4sf)__A, (__v4sf)__B, (__vector unsigned char)__t)` 从当前函数返回。
- **L1232 EN**: Closes the current lexical scope or compound statement.
  **L1232 CN**: 结束当前词法作用域或复合语句块。
- **L1233 EN**: Blank line separating nearby declarations or logic blocks.
  **L1233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1234 EN**: Comment explains nearby logic, constraints, or intent: `Selects and interleaves the upper two SPFP values from A and B.`.
  **L1234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Selects and interleaves the upper two SPFP values from A and B.`。
- **L1235 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1235 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1236 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1236 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1237 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpackhi_ps(__m128 __A, __m128 __B) {`.
  **L1237 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpackhi_ps(__m128 __A, __m128 __B) {`。
- **L1238 EN**: Returns from the current function with `(__m128)vec_vmrglw((__v4sf)__A, (__v4sf)__B)`.
  **L1238 CN**: 以 `(__m128)vec_vmrglw((__v4sf)__A, (__v4sf)__B)` 从当前函数返回。
- **L1239 EN**: Closes the current lexical scope or compound statement.
  **L1239 CN**: 结束当前词法作用域或复合语句块。
- **L1240 EN**: Blank line separating nearby declarations or logic blocks.
  **L1240 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1241 EN**: Comment explains nearby logic, constraints, or intent: `Selects and interleaves the lower two SPFP values from A and B.`.
  **L1241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Selects and interleaves the lower two SPFP values from A and B.`。
- **L1242 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1242 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1243 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1243 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1244 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_unpacklo_ps(__m128 __A, __m128 __B) {`.
  **L1244 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_unpacklo_ps(__m128 __A, __m128 __B) {`。
- **L1245 EN**: Returns from the current function with `(__m128)vec_vmrghw((__v4sf)__A, (__v4sf)__B)`.
  **L1245 CN**: 以 `(__m128)vec_vmrghw((__v4sf)__A, (__v4sf)__B)` 从当前函数返回。
- **L1246 EN**: Closes the current lexical scope or compound statement.
  **L1246 CN**: 结束当前词法作用域或复合语句块。
- **L1247 EN**: Blank line separating nearby declarations or logic blocks.
  **L1247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1248 EN**: Comment explains nearby logic, constraints, or intent: `Sets the upper two SPFP values with 64-bits of data loaded from P;`.
  **L1248 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sets the upper two SPFP values with 64-bits of data loaded from P;`。

### Lines 1249-1272

````c
   the lower two values are passed through from A.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadh_pi(__m128 __A, __m64 const *__P) {
  __vector unsigned long long __a = (__vector unsigned long long)__A;
  __vector unsigned long long __p = vec_splats(*__P);
  __a[1] = __p[1];

  return (__m128)__a;
}

/* Stores the upper two SPFP values of A into P.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storeh_pi(__m64 *__P, __m128 __A) {
  __vector unsigned long long __a = (__vector unsigned long long)__A;

  *__P = __a[1];
}

/* Moves the upper two values of B into the lower two values of A.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_movehl_ps(__m128 __A, __m128 __B) {
````
- **L1249 EN**: Continues the surrounding expression or declaration: `the lower two values are passed through from A.  */`.
  **L1249 CN**: 继续构造周围的表达式或声明：`the lower two values are passed through from A.  */`。
- **L1250 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1250 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1251 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1251 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1252 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadh_pi(__m128 __A, __m64 const *__P) {`.
  **L1252 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadh_pi(__m128 __A, __m64 const *__P) {`。
- **L1253 EN**: Initializes variable `__a` from the expression on the right-hand side.
  **L1253 CN**: 使用右侧表达式初始化变量 `__a`。
- **L1254 EN**: Initializes variable `__p` from the expression on the right-hand side.
  **L1254 CN**: 使用右侧表达式初始化变量 `__p`。
- **L1255 EN**: Adds a standalone statement or declaration: `__a[1] = __p[1];`.
  **L1255 CN**: 添加一条独立语句或声明：`__a[1] = __p[1];`。
- **L1256 EN**: Blank line separating nearby declarations or logic blocks.
  **L1256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1257 EN**: Returns from the current function with `(__m128)__a`.
  **L1257 CN**: 以 `(__m128)__a` 从当前函数返回。
- **L1258 EN**: Closes the current lexical scope or compound statement.
  **L1258 CN**: 结束当前词法作用域或复合语句块。
- **L1259 EN**: Blank line separating nearby declarations or logic blocks.
  **L1259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1260 EN**: Comment explains nearby logic, constraints, or intent: `Stores the upper two SPFP values of A into P.`.
  **L1260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores the upper two SPFP values of A into P.`。
- **L1261 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L1261 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L1262 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1262 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1263 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storeh_pi(__m64 *__P, __m128 __A) {`.
  **L1263 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storeh_pi(__m64 *__P, __m128 __A) {`。
- **L1264 EN**: Initializes variable `__a` from the expression on the right-hand side.
  **L1264 CN**: 使用右侧表达式初始化变量 `__a`。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Comment explains nearby logic, constraints, or intent: `__P __a[1];`.
  **L1266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__P __a[1];`。
- **L1267 EN**: Closes the current lexical scope or compound statement.
  **L1267 CN**: 结束当前词法作用域或复合语句块。
- **L1268 EN**: Blank line separating nearby declarations or logic blocks.
  **L1268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1269 EN**: Comment explains nearby logic, constraints, or intent: `Moves the upper two values of B into the lower two values of A.`.
  **L1269 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Moves the upper two values of B into the lower two values of A.`。
- **L1270 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1270 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1271 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1271 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1272 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movehl_ps(__m128 __A, __m128 __B) {`.
  **L1272 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movehl_ps(__m128 __A, __m128 __B) {`。

### Lines 1273-1296

````c
  return (__m128)vec_mergel((__vector unsigned long long)__B,
                            (__vector unsigned long long)__A);
}

/* Moves the lower two values of B into the upper two values of A.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_movelh_ps(__m128 __A, __m128 __B) {
  return (__m128)vec_mergeh((__vector unsigned long long)__A,
                            (__vector unsigned long long)__B);
}

/* Sets the lower two SPFP values with 64-bits of data loaded from P;
   the upper two values are passed through from A.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_loadl_pi(__m128 __A, __m64 const *__P) {
  __vector unsigned long long __a = (__vector unsigned long long)__A;
  __vector unsigned long long __p = vec_splats(*__P);
  __a[0] = __p[0];

  return (__m128)__a;
}

````
- **L1273 EN**: Returns from the current function with `(__m128)vec_mergel((__vector unsigned long long)__B,`.
  **L1273 CN**: 以 `(__m128)vec_mergel((__vector unsigned long long)__B,` 从当前函数返回。
- **L1274 EN**: Executes a call or declaration centered on `statement`.
  **L1274 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1275 EN**: Closes the current lexical scope or compound statement.
  **L1275 CN**: 结束当前词法作用域或复合语句块。
- **L1276 EN**: Blank line separating nearby declarations or logic blocks.
  **L1276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1277 EN**: Comment explains nearby logic, constraints, or intent: `Moves the lower two values of B into the upper two values of A.`.
  **L1277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Moves the lower two values of B into the upper two values of A.`。
- **L1278 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1278 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1279 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1279 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1280 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movelh_ps(__m128 __A, __m128 __B) {`.
  **L1280 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movelh_ps(__m128 __A, __m128 __B) {`。
- **L1281 EN**: Returns from the current function with `(__m128)vec_mergeh((__vector unsigned long long)__A,`.
  **L1281 CN**: 以 `(__m128)vec_mergeh((__vector unsigned long long)__A,` 从当前函数返回。
- **L1282 EN**: Executes a call or declaration centered on `statement`.
  **L1282 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1283 EN**: Closes the current lexical scope or compound statement.
  **L1283 CN**: 结束当前词法作用域或复合语句块。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1285 EN**: Comment explains nearby logic, constraints, or intent: `Sets the lower two SPFP values with 64-bits of data loaded from P;`.
  **L1285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sets the lower two SPFP values with 64-bits of data loaded from P;`。
- **L1286 EN**: Continues the surrounding expression or declaration: `the upper two values are passed through from A.  */`.
  **L1286 CN**: 继续构造周围的表达式或声明：`the upper two values are passed through from A.  */`。
- **L1287 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1287 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1288 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1288 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1289 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadl_pi(__m128 __A, __m64 const *__P) {`.
  **L1289 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadl_pi(__m128 __A, __m64 const *__P) {`。
- **L1290 EN**: Initializes variable `__a` from the expression on the right-hand side.
  **L1290 CN**: 使用右侧表达式初始化变量 `__a`。
- **L1291 EN**: Initializes variable `__p` from the expression on the right-hand side.
  **L1291 CN**: 使用右侧表达式初始化变量 `__p`。
- **L1292 EN**: Adds a standalone statement or declaration: `__a[0] = __p[0];`.
  **L1292 CN**: 添加一条独立语句或声明：`__a[0] = __p[0];`。
- **L1293 EN**: Blank line separating nearby declarations or logic blocks.
  **L1293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1294 EN**: Returns from the current function with `(__m128)__a`.
  **L1294 CN**: 以 `(__m128)__a` 从当前函数返回。
- **L1295 EN**: Closes the current lexical scope or compound statement.
  **L1295 CN**: 结束当前词法作用域或复合语句块。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1297-1320

````c
/* Stores the lower two SPFP values of A into P.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_storel_pi(__m64 *__P, __m128 __A) {
  __vector unsigned long long __a = (__vector unsigned long long)__A;

  *__P = __a[0];
}

#ifdef _ARCH_PWR8
/* Intrinsic functions that require PowerISA 2.07 minimum.  */

/* Creates a 4-bit mask from the most significant bits of the SPFP values.  */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_movemask_ps(__m128 __A) {
#ifdef _ARCH_PWR10
  return vec_extractm((__vector unsigned int)__A);
#else
  __vector unsigned long long __result;
  static const __vector unsigned int __perm_mask = {
#ifdef __LITTLE_ENDIAN__
      0x00204060, 0x80808080, 0x80808080, 0x80808080
#else
````
- **L1297 EN**: Comment explains nearby logic, constraints, or intent: `Stores the lower two SPFP values of A into P.`.
  **L1297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores the lower two SPFP values of A into P.`。
- **L1298 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L1298 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L1299 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1299 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1300 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_storel_pi(__m64 *__P, __m128 __A) {`.
  **L1300 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_storel_pi(__m64 *__P, __m128 __A) {`。
- **L1301 EN**: Initializes variable `__a` from the expression on the right-hand side.
  **L1301 CN**: 使用右侧表达式初始化变量 `__a`。
- **L1302 EN**: Blank line separating nearby declarations or logic blocks.
  **L1302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1303 EN**: Comment explains nearby logic, constraints, or intent: `__P __a[0];`.
  **L1303 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__P __a[0];`。
- **L1304 EN**: Closes the current lexical scope or compound statement.
  **L1304 CN**: 结束当前词法作用域或复合语句块。
- **L1305 EN**: Blank line separating nearby declarations or logic blocks.
  **L1305 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1306 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L1306 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L1307 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsic functions that require PowerISA 2.07 minimum.`.
  **L1307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsic functions that require PowerISA 2.07 minimum.`。
- **L1308 EN**: Blank line separating nearby declarations or logic blocks.
  **L1308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1309 EN**: Comment explains nearby logic, constraints, or intent: `Creates a 4-bit mask from the most significant bits of the SPFP values.`.
  **L1309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Creates a 4-bit mask from the most significant bits of the SPFP values.`。
- **L1310 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1310 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1311 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1311 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1312 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movemask_ps(__m128 __A) {`.
  **L1312 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movemask_ps(__m128 __A) {`。
- **L1313 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR10`.
  **L1313 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR10`。
- **L1314 EN**: Returns from the current function with `vec_extractm((__vector unsigned int)__A)`.
  **L1314 CN**: 以 `vec_extractm((__vector unsigned int)__A)` 从当前函数返回。
- **L1315 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1315 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1316 EN**: Adds a standalone statement or declaration: `__vector unsigned long long __result;`.
  **L1316 CN**: 添加一条独立语句或声明：`__vector unsigned long long __result;`。
- **L1317 EN**: Continues the surrounding expression or declaration: `static const __vector unsigned int __perm_mask = {`.
  **L1317 CN**: 继续构造周围的表达式或声明：`static const __vector unsigned int __perm_mask = {`。
- **L1318 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1318 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1319 EN**: Continues the surrounding expression or declaration: `0x00204060, 0x80808080, 0x80808080, 0x80808080`.
  **L1319 CN**: 继续构造周围的表达式或声明：`0x00204060, 0x80808080, 0x80808080, 0x80808080`。
- **L1320 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1320 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 1321-1344

````c
      0x80808080, 0x80808080, 0x80808080, 0x00204060
#endif
  };

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

/* Create a vector with all four elements equal to *P.  */
extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load1_ps(float const *__P) {
  return _mm_set1_ps(*__P);
}

extern __inline __m128
````
- **L1321 EN**: Continues the surrounding expression or declaration: `0x80808080, 0x80808080, 0x80808080, 0x00204060`.
  **L1321 CN**: 继续构造周围的表达式或声明：`0x80808080, 0x80808080, 0x80808080, 0x00204060`。
- **L1322 EN**: Closes the current preprocessor conditional block.
  **L1322 CN**: 结束当前预处理条件块。
- **L1323 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1323 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1324 EN**: Blank line separating nearby declarations or logic blocks.
  **L1324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1325 EN**: Continues logic associated with callable symbol `vec_vbpermq`.
  **L1325 CN**: 继续与可调用符号 `vec_vbpermq` 相关的逻辑。
- **L1326 EN**: Executes a call or declaration centered on `statement`.
  **L1326 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1327 EN**: Blank line separating nearby declarations or logic blocks.
  **L1327 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1328 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1328 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1329 EN**: Returns from the current function with `__result[1]`.
  **L1329 CN**: 以 `__result[1]` 从当前函数返回。
- **L1330 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1330 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1331 EN**: Returns from the current function with `__result[0]`.
  **L1331 CN**: 以 `__result[0]` 从当前函数返回。
- **L1332 EN**: Closes the current preprocessor conditional block.
  **L1332 CN**: 结束当前预处理条件块。
- **L1333 EN**: Closes the current preprocessor conditional block.
  **L1333 CN**: 结束当前预处理条件块。
- **L1334 EN**: Closes the current lexical scope or compound statement.
  **L1334 CN**: 结束当前词法作用域或复合语句块。
- **L1335 EN**: Closes the current preprocessor conditional block.
  **L1335 CN**: 结束当前预处理条件块。
- **L1336 EN**: Blank line separating nearby declarations or logic blocks.
  **L1336 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1337 EN**: Comment explains nearby logic, constraints, or intent: `Create a vector with all four elements equal to *P.`.
  **L1337 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create a vector with all four elements equal to *P.`。
- **L1338 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1338 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L1339 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1339 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1340 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load1_ps(float const *__P) {`.
  **L1340 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load1_ps(float const *__P) {`。
- **L1341 EN**: Returns from the current function with `_mm_set1_ps(*__P)`.
  **L1341 CN**: 以 `_mm_set1_ps(*__P)` 从当前函数返回。
- **L1342 EN**: Closes the current lexical scope or compound statement.
  **L1342 CN**: 结束当前词法作用域或复合语句块。
- **L1343 EN**: Blank line separating nearby declarations or logic blocks.
  **L1343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1344 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L1344 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。

### Lines 1345-1368

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_load_ps1(float const *__P) {
  return _mm_load1_ps(__P);
}

/* Extracts one of the four words of A.  The selector N must be immediate.  */
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_extract_pi16(__m64 const __A, int const __N) {
  unsigned int __shiftr = __N & 3;
#ifdef __BIG_ENDIAN__
  __shiftr = 3 - __shiftr;
#endif

  return ((__A >> (__shiftr * 16)) & 0xffff);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pextrw(__m64 const __A, int const __N) {
  return _mm_extract_pi16(__A, __N);
}

/* Inserts word D into one of four words of A.  The selector N must be
````
- **L1345 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1345 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1346 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load_ps1(float const *__P) {`.
  **L1346 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load_ps1(float const *__P) {`。
- **L1347 EN**: Returns from the current function with `_mm_load1_ps(__P)`.
  **L1347 CN**: 以 `_mm_load1_ps(__P)` 从当前函数返回。
- **L1348 EN**: Closes the current lexical scope or compound statement.
  **L1348 CN**: 结束当前词法作用域或复合语句块。
- **L1349 EN**: Blank line separating nearby declarations or logic blocks.
  **L1349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1350 EN**: Comment explains nearby logic, constraints, or intent: `Extracts one of the four words of A. The selector N must be immediate.`.
  **L1350 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extracts one of the four words of A. The selector N must be immediate.`。
- **L1351 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1351 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1352 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1352 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1353 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_extract_pi16(__m64 const __A, int const __N) {`.
  **L1353 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_extract_pi16(__m64 const __A, int const __N) {`。
- **L1354 EN**: Initializes variable `__shiftr` from the expression on the right-hand side.
  **L1354 CN**: 使用右侧表达式初始化变量 `__shiftr`。
- **L1355 EN**: Starts a preprocessor conditional block: `#ifdef __BIG_ENDIAN__`.
  **L1355 CN**: 开始一个预处理条件块：`#ifdef __BIG_ENDIAN__`。
- **L1356 EN**: Adds a standalone statement or declaration: `__shiftr = 3 - __shiftr;`.
  **L1356 CN**: 添加一条独立语句或声明：`__shiftr = 3 - __shiftr;`。
- **L1357 EN**: Closes the current preprocessor conditional block.
  **L1357 CN**: 结束当前预处理条件块。
- **L1358 EN**: Blank line separating nearby declarations or logic blocks.
  **L1358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1359 EN**: Returns from the current function with `((__A >> (__shiftr * 16)) & 0xffff)`.
  **L1359 CN**: 以 `((__A >> (__shiftr * 16)) & 0xffff)` 从当前函数返回。
- **L1360 EN**: Closes the current lexical scope or compound statement.
  **L1360 CN**: 结束当前词法作用域或复合语句块。
- **L1361 EN**: Blank line separating nearby declarations or logic blocks.
  **L1361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1362 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1362 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1363 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1363 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1364 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pextrw(__m64 const __A, int const __N) {`.
  **L1364 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pextrw(__m64 const __A, int const __N) {`。
- **L1365 EN**: Returns from the current function with `_mm_extract_pi16(__A, __N)`.
  **L1365 CN**: 以 `_mm_extract_pi16(__A, __N)` 从当前函数返回。
- **L1366 EN**: Closes the current lexical scope or compound statement.
  **L1366 CN**: 结束当前词法作用域或复合语句块。
- **L1367 EN**: Blank line separating nearby declarations or logic blocks.
  **L1367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1368 EN**: Comment explains nearby logic, constraints, or intent: `Inserts word D into one of four words of A. The selector N must be`.
  **L1368 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Inserts word D into one of four words of A. The selector N must be`。

### Lines 1369-1392

````c
   immediate.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_insert_pi16(__m64 const __A, int const __D, int const __N) {
  const int __shiftl = (__N & 3) * 16;
  const __m64 __shiftD = (const __m64)__D << __shiftl;
  const __m64 __mask = 0xffffUL << __shiftl;
  __m64 __result = (__A & (~__mask)) | (__shiftD & __mask);

  return __result;
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pinsrw(__m64 const __A, int const __D, int const __N) {
  return _mm_insert_pi16(__A, __D, __N);
}

/* Compute the element-wise maximum of signed 16-bit values.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))

    _mm_max_pi16(__m64 __A, __m64 __B) {
#if _ARCH_PWR8
````
- **L1369 EN**: Continues the surrounding expression or declaration: `immediate.  */`.
  **L1369 CN**: 继续构造周围的表达式或声明：`immediate.  */`。
- **L1370 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1370 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1371 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1371 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1372 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_insert_pi16(__m64 const __A, int const __D, int const __N) {`.
  **L1372 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_insert_pi16(__m64 const __A, int const __D, int const __N) {`。
- **L1373 EN**: Initializes variable `__shiftl` from the expression on the right-hand side.
  **L1373 CN**: 使用右侧表达式初始化变量 `__shiftl`。
- **L1374 EN**: Initializes variable `__shiftD` from the expression on the right-hand side.
  **L1374 CN**: 使用右侧表达式初始化变量 `__shiftD`。
- **L1375 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L1375 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L1376 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1376 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1377 EN**: Blank line separating nearby declarations or logic blocks.
  **L1377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1378 EN**: Returns from the current function with `__result`.
  **L1378 CN**: 以 `__result` 从当前函数返回。
- **L1379 EN**: Closes the current lexical scope or compound statement.
  **L1379 CN**: 结束当前词法作用域或复合语句块。
- **L1380 EN**: Blank line separating nearby declarations or logic blocks.
  **L1380 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1381 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1381 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1382 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1382 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1383 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pinsrw(__m64 const __A, int const __D, int const __N) {`.
  **L1383 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pinsrw(__m64 const __A, int const __D, int const __N) {`。
- **L1384 EN**: Returns from the current function with `_mm_insert_pi16(__A, __D, __N)`.
  **L1384 CN**: 以 `_mm_insert_pi16(__A, __D, __N)` 从当前函数返回。
- **L1385 EN**: Closes the current lexical scope or compound statement.
  **L1385 CN**: 结束当前词法作用域或复合语句块。
- **L1386 EN**: Blank line separating nearby declarations or logic blocks.
  **L1386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1387 EN**: Comment explains nearby logic, constraints, or intent: `Compute the element-wise maximum of signed 16-bit values.`.
  **L1387 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compute the element-wise maximum of signed 16-bit values.`。
- **L1388 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1388 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1389 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1389 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1390 EN**: Blank line separating nearby declarations or logic blocks.
  **L1390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_pi16(__m64 __A, __m64 __B) {`.
  **L1391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_pi16(__m64 __A, __m64 __B) {`。
- **L1392 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L1392 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。

### Lines 1393-1416

````c
  __vector signed short __a, __b, __r;
  __vector __bool short __c;

  __a = (__vector signed short)vec_splats(__A);
  __b = (__vector signed short)vec_splats(__B);
  __c = (__vector __bool short)vec_cmpgt(__a, __b);
  __r = vec_sel(__b, __a, __c);
  return (__m64)((__vector long long)__r)[0];
#else
  __m64_union __m1, __m2, __res;

  __m1.as_m64 = __A;
  __m2.as_m64 = __B;

  __res.as_short[0] = (__m1.as_short[0] > __m2.as_short[0]) ? __m1.as_short[0]
                                                            : __m2.as_short[0];
  __res.as_short[1] = (__m1.as_short[1] > __m2.as_short[1]) ? __m1.as_short[1]
                                                            : __m2.as_short[1];
  __res.as_short[2] = (__m1.as_short[2] > __m2.as_short[2]) ? __m1.as_short[2]
                                                            : __m2.as_short[2];
  __res.as_short[3] = (__m1.as_short[3] > __m2.as_short[3]) ? __m1.as_short[3]
                                                            : __m2.as_short[3];

  return (__m64)__res.as_m64;
````
- **L1393 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __r;`.
  **L1393 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __r;`。
- **L1394 EN**: Adds a standalone statement or declaration: `__vector __bool short __c;`.
  **L1394 CN**: 添加一条独立语句或声明：`__vector __bool short __c;`。
- **L1395 EN**: Blank line separating nearby declarations or logic blocks.
  **L1395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1396 EN**: Executes a call or declaration centered on `=`.
  **L1396 CN**: 执行以 `=` 为核心的调用或声明。
- **L1397 EN**: Executes a call or declaration centered on `=`.
  **L1397 CN**: 执行以 `=` 为核心的调用或声明。
- **L1398 EN**: Executes a call or declaration centered on `=`.
  **L1398 CN**: 执行以 `=` 为核心的调用或声明。
- **L1399 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1399 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1400 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L1400 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。
- **L1401 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1401 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1402 EN**: Adds a standalone statement or declaration: `__m64_union __m1, __m2, __res;`.
  **L1402 CN**: 添加一条独立语句或声明：`__m64_union __m1, __m2, __res;`。
- **L1403 EN**: Blank line separating nearby declarations or logic blocks.
  **L1403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1404 EN**: Adds a standalone statement or declaration: `__m1.as_m64 = __A;`.
  **L1404 CN**: 添加一条独立语句或声明：`__m1.as_m64 = __A;`。
- **L1405 EN**: Adds a standalone statement or declaration: `__m2.as_m64 = __B;`.
  **L1405 CN**: 添加一条独立语句或声明：`__m2.as_m64 = __B;`。
- **L1406 EN**: Blank line separating nearby declarations or logic blocks.
  **L1406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1407 EN**: Continues the surrounding expression or declaration: `__res.as_short[0] = (__m1.as_short[0] > __m2.as_short[0]) ? __m1.as_short[0]`.
  **L1407 CN**: 继续构造周围的表达式或声明：`__res.as_short[0] = (__m1.as_short[0] > __m2.as_short[0]) ? __m1.as_short[0]`。
- **L1408 EN**: Adds a standalone statement or declaration: `: __m2.as_short[0];`.
  **L1408 CN**: 添加一条独立语句或声明：`: __m2.as_short[0];`。
- **L1409 EN**: Continues the surrounding expression or declaration: `__res.as_short[1] = (__m1.as_short[1] > __m2.as_short[1]) ? __m1.as_short[1]`.
  **L1409 CN**: 继续构造周围的表达式或声明：`__res.as_short[1] = (__m1.as_short[1] > __m2.as_short[1]) ? __m1.as_short[1]`。
- **L1410 EN**: Adds a standalone statement or declaration: `: __m2.as_short[1];`.
  **L1410 CN**: 添加一条独立语句或声明：`: __m2.as_short[1];`。
- **L1411 EN**: Continues the surrounding expression or declaration: `__res.as_short[2] = (__m1.as_short[2] > __m2.as_short[2]) ? __m1.as_short[2]`.
  **L1411 CN**: 继续构造周围的表达式或声明：`__res.as_short[2] = (__m1.as_short[2] > __m2.as_short[2]) ? __m1.as_short[2]`。
- **L1412 EN**: Adds a standalone statement or declaration: `: __m2.as_short[2];`.
  **L1412 CN**: 添加一条独立语句或声明：`: __m2.as_short[2];`。
- **L1413 EN**: Continues the surrounding expression or declaration: `__res.as_short[3] = (__m1.as_short[3] > __m2.as_short[3]) ? __m1.as_short[3]`.
  **L1413 CN**: 继续构造周围的表达式或声明：`__res.as_short[3] = (__m1.as_short[3] > __m2.as_short[3]) ? __m1.as_short[3]`。
- **L1414 EN**: Adds a standalone statement or declaration: `: __m2.as_short[3];`.
  **L1414 CN**: 添加一条独立语句或声明：`: __m2.as_short[3];`。
- **L1415 EN**: Blank line separating nearby declarations or logic blocks.
  **L1415 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1416 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L1416 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。

### Lines 1417-1440

````c
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pmaxsw(__m64 __A, __m64 __B) {
  return _mm_max_pi16(__A, __B);
}

/* Compute the element-wise maximum of unsigned 8-bit values.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_pu8(__m64 __A, __m64 __B) {
#if _ARCH_PWR8
  __vector unsigned char __a, __b, __r;
  __vector __bool char __c;

  __a = (__vector unsigned char)vec_splats(__A);
  __b = (__vector unsigned char)vec_splats(__B);
  __c = (__vector __bool char)vec_cmpgt(__a, __b);
  __r = vec_sel(__b, __a, __c);
  return (__m64)((__vector long long)__r)[0];
#else
  __m64_union __m1, __m2, __res;
````
- **L1417 EN**: Closes the current preprocessor conditional block.
  **L1417 CN**: 结束当前预处理条件块。
- **L1418 EN**: Closes the current lexical scope or compound statement.
  **L1418 CN**: 结束当前词法作用域或复合语句块。
- **L1419 EN**: Blank line separating nearby declarations or logic blocks.
  **L1419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1420 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1420 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1421 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1421 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1422 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pmaxsw(__m64 __A, __m64 __B) {`.
  **L1422 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pmaxsw(__m64 __A, __m64 __B) {`。
- **L1423 EN**: Returns from the current function with `_mm_max_pi16(__A, __B)`.
  **L1423 CN**: 以 `_mm_max_pi16(__A, __B)` 从当前函数返回。
- **L1424 EN**: Closes the current lexical scope or compound statement.
  **L1424 CN**: 结束当前词法作用域或复合语句块。
- **L1425 EN**: Blank line separating nearby declarations or logic blocks.
  **L1425 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1426 EN**: Comment explains nearby logic, constraints, or intent: `Compute the element-wise maximum of unsigned 8-bit values.`.
  **L1426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compute the element-wise maximum of unsigned 8-bit values.`。
- **L1427 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1427 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1428 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1428 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1429 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_pu8(__m64 __A, __m64 __B) {`.
  **L1429 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_pu8(__m64 __A, __m64 __B) {`。
- **L1430 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L1430 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L1431 EN**: Adds a standalone statement or declaration: `__vector unsigned char __a, __b, __r;`.
  **L1431 CN**: 添加一条独立语句或声明：`__vector unsigned char __a, __b, __r;`。
- **L1432 EN**: Adds a standalone statement or declaration: `__vector __bool char __c;`.
  **L1432 CN**: 添加一条独立语句或声明：`__vector __bool char __c;`。
- **L1433 EN**: Blank line separating nearby declarations or logic blocks.
  **L1433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1434 EN**: Executes a call or declaration centered on `=`.
  **L1434 CN**: 执行以 `=` 为核心的调用或声明。
- **L1435 EN**: Executes a call or declaration centered on `=`.
  **L1435 CN**: 执行以 `=` 为核心的调用或声明。
- **L1436 EN**: Executes a call or declaration centered on `=`.
  **L1436 CN**: 执行以 `=` 为核心的调用或声明。
- **L1437 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1437 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1438 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L1438 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。
- **L1439 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1439 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1440 EN**: Adds a standalone statement or declaration: `__m64_union __m1, __m2, __res;`.
  **L1440 CN**: 添加一条独立语句或声明：`__m64_union __m1, __m2, __res;`。

### Lines 1441-1464

````c
  long __i;

  __m1.as_m64 = __A;
  __m2.as_m64 = __B;

  for (__i = 0; __i < 8; __i++)
    __res.as_char[__i] =
        ((unsigned char)__m1.as_char[__i] > (unsigned char)__m2.as_char[__i])
            ? __m1.as_char[__i]
            : __m2.as_char[__i];

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pmaxub(__m64 __A, __m64 __B) {
  return _mm_max_pu8(__A, __B);
}

/* Compute the element-wise minimum of signed 16-bit values.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
````
- **L1441 EN**: Adds a standalone statement or declaration: `long __i;`.
  **L1441 CN**: 添加一条独立语句或声明：`long __i;`。
- **L1442 EN**: Blank line separating nearby declarations or logic blocks.
  **L1442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1443 EN**: Adds a standalone statement or declaration: `__m1.as_m64 = __A;`.
  **L1443 CN**: 添加一条独立语句或声明：`__m1.as_m64 = __A;`。
- **L1444 EN**: Adds a standalone statement or declaration: `__m2.as_m64 = __B;`.
  **L1444 CN**: 添加一条独立语句或声明：`__m2.as_m64 = __B;`。
- **L1445 EN**: Blank line separating nearby declarations or logic blocks.
  **L1445 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1446 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1446 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1447 EN**: Continues the surrounding expression or declaration: `__res.as_char[__i] =`.
  **L1447 CN**: 继续构造周围的表达式或声明：`__res.as_char[__i] =`。
- **L1448 EN**: Continues the surrounding expression or declaration: `((unsigned char)__m1.as_char[__i] > (unsigned char)__m2.as_char[__i])`.
  **L1448 CN**: 继续构造周围的表达式或声明：`((unsigned char)__m1.as_char[__i] > (unsigned char)__m2.as_char[__i])`。
- **L1449 EN**: Continues the surrounding expression or declaration: `? __m1.as_char[__i]`.
  **L1449 CN**: 继续构造周围的表达式或声明：`? __m1.as_char[__i]`。
- **L1450 EN**: Adds a standalone statement or declaration: `: __m2.as_char[__i];`.
  **L1450 CN**: 添加一条独立语句或声明：`: __m2.as_char[__i];`。
- **L1451 EN**: Blank line separating nearby declarations or logic blocks.
  **L1451 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1452 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L1452 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L1453 EN**: Closes the current preprocessor conditional block.
  **L1453 CN**: 结束当前预处理条件块。
- **L1454 EN**: Closes the current lexical scope or compound statement.
  **L1454 CN**: 结束当前词法作用域或复合语句块。
- **L1455 EN**: Blank line separating nearby declarations or logic blocks.
  **L1455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1456 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1456 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1457 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1457 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1458 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pmaxub(__m64 __A, __m64 __B) {`.
  **L1458 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pmaxub(__m64 __A, __m64 __B) {`。
- **L1459 EN**: Returns from the current function with `_mm_max_pu8(__A, __B)`.
  **L1459 CN**: 以 `_mm_max_pu8(__A, __B)` 从当前函数返回。
- **L1460 EN**: Closes the current lexical scope or compound statement.
  **L1460 CN**: 结束当前词法作用域或复合语句块。
- **L1461 EN**: Blank line separating nearby declarations or logic blocks.
  **L1461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1462 EN**: Comment explains nearby logic, constraints, or intent: `Compute the element-wise minimum of signed 16-bit values.`.
  **L1462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compute the element-wise minimum of signed 16-bit values.`。
- **L1463 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1463 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1464 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1464 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。

### Lines 1465-1488

````c
    _mm_min_pi16(__m64 __A, __m64 __B) {
#if _ARCH_PWR8
  __vector signed short __a, __b, __r;
  __vector __bool short __c;

  __a = (__vector signed short)vec_splats(__A);
  __b = (__vector signed short)vec_splats(__B);
  __c = (__vector __bool short)vec_cmplt(__a, __b);
  __r = vec_sel(__b, __a, __c);
  return (__m64)((__vector long long)__r)[0];
#else
  __m64_union __m1, __m2, __res;

  __m1.as_m64 = __A;
  __m2.as_m64 = __B;

  __res.as_short[0] = (__m1.as_short[0] < __m2.as_short[0]) ? __m1.as_short[0]
                                                            : __m2.as_short[0];
  __res.as_short[1] = (__m1.as_short[1] < __m2.as_short[1]) ? __m1.as_short[1]
                                                            : __m2.as_short[1];
  __res.as_short[2] = (__m1.as_short[2] < __m2.as_short[2]) ? __m1.as_short[2]
                                                            : __m2.as_short[2];
  __res.as_short[3] = (__m1.as_short[3] < __m2.as_short[3]) ? __m1.as_short[3]
                                                            : __m2.as_short[3];
````
- **L1465 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_pi16(__m64 __A, __m64 __B) {`.
  **L1465 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_pi16(__m64 __A, __m64 __B) {`。
- **L1466 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L1466 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L1467 EN**: Adds a standalone statement or declaration: `__vector signed short __a, __b, __r;`.
  **L1467 CN**: 添加一条独立语句或声明：`__vector signed short __a, __b, __r;`。
- **L1468 EN**: Adds a standalone statement or declaration: `__vector __bool short __c;`.
  **L1468 CN**: 添加一条独立语句或声明：`__vector __bool short __c;`。
- **L1469 EN**: Blank line separating nearby declarations or logic blocks.
  **L1469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1470 EN**: Executes a call or declaration centered on `=`.
  **L1470 CN**: 执行以 `=` 为核心的调用或声明。
- **L1471 EN**: Executes a call or declaration centered on `=`.
  **L1471 CN**: 执行以 `=` 为核心的调用或声明。
- **L1472 EN**: Executes a call or declaration centered on `=`.
  **L1472 CN**: 执行以 `=` 为核心的调用或声明。
- **L1473 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1473 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1474 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L1474 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。
- **L1475 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1475 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1476 EN**: Adds a standalone statement or declaration: `__m64_union __m1, __m2, __res;`.
  **L1476 CN**: 添加一条独立语句或声明：`__m64_union __m1, __m2, __res;`。
- **L1477 EN**: Blank line separating nearby declarations or logic blocks.
  **L1477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1478 EN**: Adds a standalone statement or declaration: `__m1.as_m64 = __A;`.
  **L1478 CN**: 添加一条独立语句或声明：`__m1.as_m64 = __A;`。
- **L1479 EN**: Adds a standalone statement or declaration: `__m2.as_m64 = __B;`.
  **L1479 CN**: 添加一条独立语句或声明：`__m2.as_m64 = __B;`。
- **L1480 EN**: Blank line separating nearby declarations or logic blocks.
  **L1480 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1481 EN**: Continues the surrounding expression or declaration: `__res.as_short[0] = (__m1.as_short[0] < __m2.as_short[0]) ? __m1.as_short[0]`.
  **L1481 CN**: 继续构造周围的表达式或声明：`__res.as_short[0] = (__m1.as_short[0] < __m2.as_short[0]) ? __m1.as_short[0]`。
- **L1482 EN**: Adds a standalone statement or declaration: `: __m2.as_short[0];`.
  **L1482 CN**: 添加一条独立语句或声明：`: __m2.as_short[0];`。
- **L1483 EN**: Continues the surrounding expression or declaration: `__res.as_short[1] = (__m1.as_short[1] < __m2.as_short[1]) ? __m1.as_short[1]`.
  **L1483 CN**: 继续构造周围的表达式或声明：`__res.as_short[1] = (__m1.as_short[1] < __m2.as_short[1]) ? __m1.as_short[1]`。
- **L1484 EN**: Adds a standalone statement or declaration: `: __m2.as_short[1];`.
  **L1484 CN**: 添加一条独立语句或声明：`: __m2.as_short[1];`。
- **L1485 EN**: Continues the surrounding expression or declaration: `__res.as_short[2] = (__m1.as_short[2] < __m2.as_short[2]) ? __m1.as_short[2]`.
  **L1485 CN**: 继续构造周围的表达式或声明：`__res.as_short[2] = (__m1.as_short[2] < __m2.as_short[2]) ? __m1.as_short[2]`。
- **L1486 EN**: Adds a standalone statement or declaration: `: __m2.as_short[2];`.
  **L1486 CN**: 添加一条独立语句或声明：`: __m2.as_short[2];`。
- **L1487 EN**: Continues the surrounding expression or declaration: `__res.as_short[3] = (__m1.as_short[3] < __m2.as_short[3]) ? __m1.as_short[3]`.
  **L1487 CN**: 继续构造周围的表达式或声明：`__res.as_short[3] = (__m1.as_short[3] < __m2.as_short[3]) ? __m1.as_short[3]`。
- **L1488 EN**: Adds a standalone statement or declaration: `: __m2.as_short[3];`.
  **L1488 CN**: 添加一条独立语句或声明：`: __m2.as_short[3];`。

### Lines 1489-1512

````c

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pminsw(__m64 __A, __m64 __B) {
  return _mm_min_pi16(__A, __B);
}

/* Compute the element-wise minimum of unsigned 8-bit values.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_pu8(__m64 __A, __m64 __B) {
#if _ARCH_PWR8
  __vector unsigned char __a, __b, __r;
  __vector __bool char __c;

  __a = (__vector unsigned char)vec_splats(__A);
  __b = (__vector unsigned char)vec_splats(__B);
  __c = (__vector __bool char)vec_cmplt(__a, __b);
  __r = vec_sel(__b, __a, __c);
  return (__m64)((__vector long long)__r)[0];
````
- **L1489 EN**: Blank line separating nearby declarations or logic blocks.
  **L1489 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1490 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L1490 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L1491 EN**: Closes the current preprocessor conditional block.
  **L1491 CN**: 结束当前预处理条件块。
- **L1492 EN**: Closes the current lexical scope or compound statement.
  **L1492 CN**: 结束当前词法作用域或复合语句块。
- **L1493 EN**: Blank line separating nearby declarations or logic blocks.
  **L1493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1494 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1494 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1495 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1495 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1496 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pminsw(__m64 __A, __m64 __B) {`.
  **L1496 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pminsw(__m64 __A, __m64 __B) {`。
- **L1497 EN**: Returns from the current function with `_mm_min_pi16(__A, __B)`.
  **L1497 CN**: 以 `_mm_min_pi16(__A, __B)` 从当前函数返回。
- **L1498 EN**: Closes the current lexical scope or compound statement.
  **L1498 CN**: 结束当前词法作用域或复合语句块。
- **L1499 EN**: Blank line separating nearby declarations or logic blocks.
  **L1499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1500 EN**: Comment explains nearby logic, constraints, or intent: `Compute the element-wise minimum of unsigned 8-bit values.`.
  **L1500 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compute the element-wise minimum of unsigned 8-bit values.`。
- **L1501 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1501 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1502 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1502 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1503 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_pu8(__m64 __A, __m64 __B) {`.
  **L1503 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_pu8(__m64 __A, __m64 __B) {`。
- **L1504 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L1504 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L1505 EN**: Adds a standalone statement or declaration: `__vector unsigned char __a, __b, __r;`.
  **L1505 CN**: 添加一条独立语句或声明：`__vector unsigned char __a, __b, __r;`。
- **L1506 EN**: Adds a standalone statement or declaration: `__vector __bool char __c;`.
  **L1506 CN**: 添加一条独立语句或声明：`__vector __bool char __c;`。
- **L1507 EN**: Blank line separating nearby declarations or logic blocks.
  **L1507 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1508 EN**: Executes a call or declaration centered on `=`.
  **L1508 CN**: 执行以 `=` 为核心的调用或声明。
- **L1509 EN**: Executes a call or declaration centered on `=`.
  **L1509 CN**: 执行以 `=` 为核心的调用或声明。
- **L1510 EN**: Executes a call or declaration centered on `=`.
  **L1510 CN**: 执行以 `=` 为核心的调用或声明。
- **L1511 EN**: Executes a call or declaration centered on `vec_sel`.
  **L1511 CN**: 执行以 `vec_sel` 为核心的调用或声明。
- **L1512 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L1512 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。

### Lines 1513-1536

````c
#else
  __m64_union __m1, __m2, __res;
  long __i;

  __m1.as_m64 = __A;
  __m2.as_m64 = __B;

  for (__i = 0; __i < 8; __i++)
    __res.as_char[__i] =
        ((unsigned char)__m1.as_char[__i] < (unsigned char)__m2.as_char[__i])
            ? __m1.as_char[__i]
            : __m2.as_char[__i];

  return (__m64)__res.as_m64;
#endif
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pminub(__m64 __A, __m64 __B) {
  return _mm_min_pu8(__A, __B);
}

/* Create an 8-bit mask of the signs of 8-bit values.  */
````
- **L1513 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1513 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1514 EN**: Adds a standalone statement or declaration: `__m64_union __m1, __m2, __res;`.
  **L1514 CN**: 添加一条独立语句或声明：`__m64_union __m1, __m2, __res;`。
- **L1515 EN**: Adds a standalone statement or declaration: `long __i;`.
  **L1515 CN**: 添加一条独立语句或声明：`long __i;`。
- **L1516 EN**: Blank line separating nearby declarations or logic blocks.
  **L1516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1517 EN**: Adds a standalone statement or declaration: `__m1.as_m64 = __A;`.
  **L1517 CN**: 添加一条独立语句或声明：`__m1.as_m64 = __A;`。
- **L1518 EN**: Adds a standalone statement or declaration: `__m2.as_m64 = __B;`.
  **L1518 CN**: 添加一条独立语句或声明：`__m2.as_m64 = __B;`。
- **L1519 EN**: Blank line separating nearby declarations or logic blocks.
  **L1519 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1520 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1520 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1521 EN**: Continues the surrounding expression or declaration: `__res.as_char[__i] =`.
  **L1521 CN**: 继续构造周围的表达式或声明：`__res.as_char[__i] =`。
- **L1522 EN**: Continues the surrounding expression or declaration: `((unsigned char)__m1.as_char[__i] < (unsigned char)__m2.as_char[__i])`.
  **L1522 CN**: 继续构造周围的表达式或声明：`((unsigned char)__m1.as_char[__i] < (unsigned char)__m2.as_char[__i])`。
- **L1523 EN**: Continues the surrounding expression or declaration: `? __m1.as_char[__i]`.
  **L1523 CN**: 继续构造周围的表达式或声明：`? __m1.as_char[__i]`。
- **L1524 EN**: Adds a standalone statement or declaration: `: __m2.as_char[__i];`.
  **L1524 CN**: 添加一条独立语句或声明：`: __m2.as_char[__i];`。
- **L1525 EN**: Blank line separating nearby declarations or logic blocks.
  **L1525 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1526 EN**: Returns from the current function with `(__m64)__res.as_m64`.
  **L1526 CN**: 以 `(__m64)__res.as_m64` 从当前函数返回。
- **L1527 EN**: Closes the current preprocessor conditional block.
  **L1527 CN**: 结束当前预处理条件块。
- **L1528 EN**: Closes the current lexical scope or compound statement.
  **L1528 CN**: 结束当前词法作用域或复合语句块。
- **L1529 EN**: Blank line separating nearby declarations or logic blocks.
  **L1529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1530 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1530 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1531 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1531 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1532 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pminub(__m64 __A, __m64 __B) {`.
  **L1532 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pminub(__m64 __A, __m64 __B) {`。
- **L1533 EN**: Returns from the current function with `_mm_min_pu8(__A, __B)`.
  **L1533 CN**: 以 `_mm_min_pu8(__A, __B)` 从当前函数返回。
- **L1534 EN**: Closes the current lexical scope or compound statement.
  **L1534 CN**: 结束当前词法作用域或复合语句块。
- **L1535 EN**: Blank line separating nearby declarations or logic blocks.
  **L1535 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1536 EN**: Comment explains nearby logic, constraints, or intent: `Create an 8-bit mask of the signs of 8-bit values.`.
  **L1536 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create an 8-bit mask of the signs of 8-bit values.`。

### Lines 1537-1560

````c
extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_movemask_pi8(__m64 __A) {
#ifdef __powerpc64__
  unsigned long long __p =
#ifdef __LITTLE_ENDIAN__
      0x0008101820283038UL; // permute control for sign bits
#else
      0x3830282018100800UL; // permute control for sign bits
#endif
  return __builtin_bpermd(__p, __A);
#else
#ifdef __LITTLE_ENDIAN__
  unsigned int __mask = 0x20283038UL;
  unsigned int __r1 = __builtin_bpermd(__mask, __A) & 0xf;
  unsigned int __r2 = __builtin_bpermd(__mask, __A >> 32) & 0xf;
#else
  unsigned int __mask = 0x38302820UL;
  unsigned int __r1 = __builtin_bpermd(__mask, __A >> 32) & 0xf;
  unsigned int __r2 = __builtin_bpermd(__mask, __A) & 0xf;
#endif
  return (__r2 << 4) | __r1;
#endif
}
````
- **L1537 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1537 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1538 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1538 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1539 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movemask_pi8(__m64 __A) {`.
  **L1539 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movemask_pi8(__m64 __A) {`。
- **L1540 EN**: Starts a preprocessor conditional block: `#ifdef __powerpc64__`.
  **L1540 CN**: 开始一个预处理条件块：`#ifdef __powerpc64__`。
- **L1541 EN**: Continues the surrounding expression or declaration: `unsigned long long __p =`.
  **L1541 CN**: 继续构造周围的表达式或声明：`unsigned long long __p =`。
- **L1542 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1542 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1543 EN**: Continues the surrounding expression or declaration: `0x0008101820283038UL; // permute control for sign bits`.
  **L1543 CN**: 继续构造周围的表达式或声明：`0x0008101820283038UL; // permute control for sign bits`。
- **L1544 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1544 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1545 EN**: Continues the surrounding expression or declaration: `0x3830282018100800UL; // permute control for sign bits`.
  **L1545 CN**: 继续构造周围的表达式或声明：`0x3830282018100800UL; // permute control for sign bits`。
- **L1546 EN**: Closes the current preprocessor conditional block.
  **L1546 CN**: 结束当前预处理条件块。
- **L1547 EN**: Returns from the current function with `__builtin_bpermd(__p, __A)`.
  **L1547 CN**: 以 `__builtin_bpermd(__p, __A)` 从当前函数返回。
- **L1548 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1548 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1549 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1549 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1550 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L1550 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L1551 EN**: Initializes variable `__r1` from the expression on the right-hand side.
  **L1551 CN**: 使用右侧表达式初始化变量 `__r1`。
- **L1552 EN**: Initializes variable `__r2` from the expression on the right-hand side.
  **L1552 CN**: 使用右侧表达式初始化变量 `__r2`。
- **L1553 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1553 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1554 EN**: Initializes variable `__mask` from the expression on the right-hand side.
  **L1554 CN**: 使用右侧表达式初始化变量 `__mask`。
- **L1555 EN**: Initializes variable `__r1` from the expression on the right-hand side.
  **L1555 CN**: 使用右侧表达式初始化变量 `__r1`。
- **L1556 EN**: Initializes variable `__r2` from the expression on the right-hand side.
  **L1556 CN**: 使用右侧表达式初始化变量 `__r2`。
- **L1557 EN**: Closes the current preprocessor conditional block.
  **L1557 CN**: 结束当前预处理条件块。
- **L1558 EN**: Returns from the current function with `(__r2 << 4) | __r1`.
  **L1558 CN**: 以 `(__r2 << 4) | __r1` 从当前函数返回。
- **L1559 EN**: Closes the current preprocessor conditional block.
  **L1559 CN**: 结束当前预处理条件块。
- **L1560 EN**: Closes the current lexical scope or compound statement.
  **L1560 CN**: 结束当前词法作用域或复合语句块。

### Lines 1561-1584

````c

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pmovmskb(__m64 __A) {
  return _mm_movemask_pi8(__A);
}

/* Multiply four unsigned 16-bit values in A by four unsigned 16-bit values
   in B and produce the high 16 bits of the 32-bit results.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mulhi_pu16(__m64 __A, __m64 __B) {
  __vector unsigned short __a, __b;
  __vector unsigned short __c;
  __vector unsigned int __w0, __w1;
  __vector unsigned char __xform1 = {
#ifdef __LITTLE_ENDIAN__
      0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,
      0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F
#else
      0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x00,
      0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15
#endif
  };
````
- **L1561 EN**: Blank line separating nearby declarations or logic blocks.
  **L1561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1562 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L1562 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L1563 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1563 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1564 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pmovmskb(__m64 __A) {`.
  **L1564 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pmovmskb(__m64 __A) {`。
- **L1565 EN**: Returns from the current function with `_mm_movemask_pi8(__A)`.
  **L1565 CN**: 以 `_mm_movemask_pi8(__A)` 从当前函数返回。
- **L1566 EN**: Closes the current lexical scope or compound statement.
  **L1566 CN**: 结束当前词法作用域或复合语句块。
- **L1567 EN**: Blank line separating nearby declarations or logic blocks.
  **L1567 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1568 EN**: Comment explains nearby logic, constraints, or intent: `Multiply four unsigned 16-bit values in A by four unsigned 16-bit values`.
  **L1568 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiply four unsigned 16-bit values in A by four unsigned 16-bit values`。
- **L1569 EN**: Continues the surrounding expression or declaration: `in B and produce the high 16 bits of the 32-bit results.  */`.
  **L1569 CN**: 继续构造周围的表达式或声明：`in B and produce the high 16 bits of the 32-bit results.  */`。
- **L1570 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1570 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1571 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1571 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1572 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mulhi_pu16(__m64 __A, __m64 __B) {`.
  **L1572 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mulhi_pu16(__m64 __A, __m64 __B) {`。
- **L1573 EN**: Adds a standalone statement or declaration: `__vector unsigned short __a, __b;`.
  **L1573 CN**: 添加一条独立语句或声明：`__vector unsigned short __a, __b;`。
- **L1574 EN**: Adds a standalone statement or declaration: `__vector unsigned short __c;`.
  **L1574 CN**: 添加一条独立语句或声明：`__vector unsigned short __c;`。
- **L1575 EN**: Adds a standalone statement or declaration: `__vector unsigned int __w0, __w1;`.
  **L1575 CN**: 添加一条独立语句或声明：`__vector unsigned int __w0, __w1;`。
- **L1576 EN**: Continues the surrounding expression or declaration: `__vector unsigned char __xform1 = {`.
  **L1576 CN**: 继续构造周围的表达式或声明：`__vector unsigned char __xform1 = {`。
- **L1577 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1577 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,`.
  **L1578 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x02, 0x03, 0x12, 0x13, 0x06, 0x07, 0x16, 0x17, 0x0A,`。
- **L1579 EN**: Continues the surrounding expression or declaration: `0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F`.
  **L1579 CN**: 继续构造周围的表达式或声明：`0x0B, 0x1A, 0x1B, 0x0E, 0x0F, 0x1E, 0x1F`。
- **L1580 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1580 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x00,`.
  **L1581 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x00, 0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15, 0x00,`。
- **L1582 EN**: Continues the surrounding expression or declaration: `0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15`.
  **L1582 CN**: 继续构造周围的表达式或声明：`0x01, 0x10, 0x11, 0x04, 0x05, 0x14, 0x15`。
- **L1583 EN**: Closes the current preprocessor conditional block.
  **L1583 CN**: 结束当前预处理条件块。
- **L1584 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1584 CN**: 结束当前声明作用域，例如结构体、枚举或类。

### Lines 1585-1608

````c

  __a = (__vector unsigned short)vec_splats(__A);
  __b = (__vector unsigned short)vec_splats(__B);

  __w0 = vec_vmuleuh(__a, __b);
  __w1 = vec_vmulouh(__a, __b);
  __c = (__vector unsigned short)vec_perm(__w0, __w1, __xform1);

  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pmulhuw(__m64 __A, __m64 __B) {
  return _mm_mulhi_pu16(__A, __B);
}

/* Return a combination of the four 16-bit values in A.  The selector
   must be an immediate.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_shuffle_pi16(__m64 __A, int const __N) {
  unsigned long __element_selector_10 = __N & 0x03;
  unsigned long __element_selector_32 = (__N >> 2) & 0x03;
````
- **L1585 EN**: Blank line separating nearby declarations or logic blocks.
  **L1585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1586 EN**: Executes a call or declaration centered on `=`.
  **L1586 CN**: 执行以 `=` 为核心的调用或声明。
- **L1587 EN**: Executes a call or declaration centered on `=`.
  **L1587 CN**: 执行以 `=` 为核心的调用或声明。
- **L1588 EN**: Blank line separating nearby declarations or logic blocks.
  **L1588 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1589 EN**: Executes a call or declaration centered on `vec_vmuleuh`.
  **L1589 CN**: 执行以 `vec_vmuleuh` 为核心的调用或声明。
- **L1590 EN**: Executes a call or declaration centered on `vec_vmulouh`.
  **L1590 CN**: 执行以 `vec_vmulouh` 为核心的调用或声明。
- **L1591 EN**: Executes a call or declaration centered on `=`.
  **L1591 CN**: 执行以 `=` 为核心的调用或声明。
- **L1592 EN**: Blank line separating nearby declarations or logic blocks.
  **L1592 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1593 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1593 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1594 EN**: Closes the current lexical scope or compound statement.
  **L1594 CN**: 结束当前词法作用域或复合语句块。
- **L1595 EN**: Blank line separating nearby declarations or logic blocks.
  **L1595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1596 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1596 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1597 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1597 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1598 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pmulhuw(__m64 __A, __m64 __B) {`.
  **L1598 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pmulhuw(__m64 __A, __m64 __B) {`。
- **L1599 EN**: Returns from the current function with `_mm_mulhi_pu16(__A, __B)`.
  **L1599 CN**: 以 `_mm_mulhi_pu16(__A, __B)` 从当前函数返回。
- **L1600 EN**: Closes the current lexical scope or compound statement.
  **L1600 CN**: 结束当前词法作用域或复合语句块。
- **L1601 EN**: Blank line separating nearby declarations or logic blocks.
  **L1601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1602 EN**: Comment explains nearby logic, constraints, or intent: `Return a combination of the four 16-bit values in A. The selector`.
  **L1602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Return a combination of the four 16-bit values in A. The selector`。
- **L1603 EN**: Continues the surrounding expression or declaration: `must be an immediate.  */`.
  **L1603 CN**: 继续构造周围的表达式或声明：`must be an immediate.  */`。
- **L1604 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1604 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1605 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1605 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1606 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_shuffle_pi16(__m64 __A, int const __N) {`.
  **L1606 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_shuffle_pi16(__m64 __A, int const __N) {`。
- **L1607 EN**: Initializes variable `__element_selector_10` from the expression on the right-hand side.
  **L1607 CN**: 使用右侧表达式初始化变量 `__element_selector_10`。
- **L1608 EN**: Initializes variable `__element_selector_32` from the expression on the right-hand side.
  **L1608 CN**: 使用右侧表达式初始化变量 `__element_selector_32`。

### Lines 1609-1632

````c
  unsigned long __element_selector_54 = (__N >> 4) & 0x03;
  unsigned long __element_selector_76 = (__N >> 6) & 0x03;
  static const unsigned short __permute_selectors[4] = {
#ifdef __LITTLE_ENDIAN__
      0x0908, 0x0B0A, 0x0D0C, 0x0F0E
#else
      0x0607, 0x0405, 0x0203, 0x0001
#endif
  };
  __m64_union __t;
  __vector unsigned long long __a, __p, __r;

#ifdef __LITTLE_ENDIAN__
  __t.as_short[0] = __permute_selectors[__element_selector_10];
  __t.as_short[1] = __permute_selectors[__element_selector_32];
  __t.as_short[2] = __permute_selectors[__element_selector_54];
  __t.as_short[3] = __permute_selectors[__element_selector_76];
#else
  __t.as_short[3] = __permute_selectors[__element_selector_10];
  __t.as_short[2] = __permute_selectors[__element_selector_32];
  __t.as_short[1] = __permute_selectors[__element_selector_54];
  __t.as_short[0] = __permute_selectors[__element_selector_76];
#endif
  __p = vec_splats(__t.as_m64);
````
- **L1609 EN**: Initializes variable `__element_selector_54` from the expression on the right-hand side.
  **L1609 CN**: 使用右侧表达式初始化变量 `__element_selector_54`。
- **L1610 EN**: Initializes variable `__element_selector_76` from the expression on the right-hand side.
  **L1610 CN**: 使用右侧表达式初始化变量 `__element_selector_76`。
- **L1611 EN**: Continues the surrounding expression or declaration: `static const unsigned short __permute_selectors[4] = {`.
  **L1611 CN**: 继续构造周围的表达式或声明：`static const unsigned short __permute_selectors[4] = {`。
- **L1612 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1612 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1613 EN**: Continues the surrounding expression or declaration: `0x0908, 0x0B0A, 0x0D0C, 0x0F0E`.
  **L1613 CN**: 继续构造周围的表达式或声明：`0x0908, 0x0B0A, 0x0D0C, 0x0F0E`。
- **L1614 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1614 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1615 EN**: Continues the surrounding expression or declaration: `0x0607, 0x0405, 0x0203, 0x0001`.
  **L1615 CN**: 继续构造周围的表达式或声明：`0x0607, 0x0405, 0x0203, 0x0001`。
- **L1616 EN**: Closes the current preprocessor conditional block.
  **L1616 CN**: 结束当前预处理条件块。
- **L1617 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1617 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1618 EN**: Adds a standalone statement or declaration: `__m64_union __t;`.
  **L1618 CN**: 添加一条独立语句或声明：`__m64_union __t;`。
- **L1619 EN**: Adds a standalone statement or declaration: `__vector unsigned long long __a, __p, __r;`.
  **L1619 CN**: 添加一条独立语句或声明：`__vector unsigned long long __a, __p, __r;`。
- **L1620 EN**: Blank line separating nearby declarations or logic blocks.
  **L1620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1621 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L1621 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L1622 EN**: Adds a standalone statement or declaration: `__t.as_short[0] = __permute_selectors[__element_selector_10];`.
  **L1622 CN**: 添加一条独立语句或声明：`__t.as_short[0] = __permute_selectors[__element_selector_10];`。
- **L1623 EN**: Adds a standalone statement or declaration: `__t.as_short[1] = __permute_selectors[__element_selector_32];`.
  **L1623 CN**: 添加一条独立语句或声明：`__t.as_short[1] = __permute_selectors[__element_selector_32];`。
- **L1624 EN**: Adds a standalone statement or declaration: `__t.as_short[2] = __permute_selectors[__element_selector_54];`.
  **L1624 CN**: 添加一条独立语句或声明：`__t.as_short[2] = __permute_selectors[__element_selector_54];`。
- **L1625 EN**: Adds a standalone statement or declaration: `__t.as_short[3] = __permute_selectors[__element_selector_76];`.
  **L1625 CN**: 添加一条独立语句或声明：`__t.as_short[3] = __permute_selectors[__element_selector_76];`。
- **L1626 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1626 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1627 EN**: Adds a standalone statement or declaration: `__t.as_short[3] = __permute_selectors[__element_selector_10];`.
  **L1627 CN**: 添加一条独立语句或声明：`__t.as_short[3] = __permute_selectors[__element_selector_10];`。
- **L1628 EN**: Adds a standalone statement or declaration: `__t.as_short[2] = __permute_selectors[__element_selector_32];`.
  **L1628 CN**: 添加一条独立语句或声明：`__t.as_short[2] = __permute_selectors[__element_selector_32];`。
- **L1629 EN**: Adds a standalone statement or declaration: `__t.as_short[1] = __permute_selectors[__element_selector_54];`.
  **L1629 CN**: 添加一条独立语句或声明：`__t.as_short[1] = __permute_selectors[__element_selector_54];`。
- **L1630 EN**: Adds a standalone statement or declaration: `__t.as_short[0] = __permute_selectors[__element_selector_76];`.
  **L1630 CN**: 添加一条独立语句或声明：`__t.as_short[0] = __permute_selectors[__element_selector_76];`。
- **L1631 EN**: Closes the current preprocessor conditional block.
  **L1631 CN**: 结束当前预处理条件块。
- **L1632 EN**: Executes a call or declaration centered on `vec_splats`.
  **L1632 CN**: 执行以 `vec_splats` 为核心的调用或声明。

### Lines 1633-1656

````c
  __a = vec_splats(__A);
  __r = vec_perm(__a, __a, (__vector unsigned char)__p);
  return (__m64)((__vector long long)__r)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pshufw(__m64 __A, int const __N) {
  return _mm_shuffle_pi16(__A, __N);
}

/* Conditionally store byte elements of A into P.  The high bit of each
   byte in the selector N determines whether the corresponding byte from
   A is stored.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_maskmove_si64(__m64 __A, __m64 __N, char *__P) {
  __m64 __hibit = 0x8080808080808080UL;
  __m64 __mask, __tmp;
  __m64 *__p = (__m64 *)__P;

  __tmp = *__p;
  __mask = _mm_cmpeq_pi8((__N & __hibit), __hibit);
  __tmp = (__tmp & (~__mask)) | (__A & __mask);
````
- **L1633 EN**: Executes a call or declaration centered on `vec_splats`.
  **L1633 CN**: 执行以 `vec_splats` 为核心的调用或声明。
- **L1634 EN**: Executes a call or declaration centered on `vec_perm`.
  **L1634 CN**: 执行以 `vec_perm` 为核心的调用或声明。
- **L1635 EN**: Returns from the current function with `(__m64)((__vector long long)__r)[0]`.
  **L1635 CN**: 以 `(__m64)((__vector long long)__r)[0]` 从当前函数返回。
- **L1636 EN**: Closes the current lexical scope or compound statement.
  **L1636 CN**: 结束当前词法作用域或复合语句块。
- **L1637 EN**: Blank line separating nearby declarations or logic blocks.
  **L1637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1638 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1638 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1639 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1639 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1640 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pshufw(__m64 __A, int const __N) {`.
  **L1640 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pshufw(__m64 __A, int const __N) {`。
- **L1641 EN**: Returns from the current function with `_mm_shuffle_pi16(__A, __N)`.
  **L1641 CN**: 以 `_mm_shuffle_pi16(__A, __N)` 从当前函数返回。
- **L1642 EN**: Closes the current lexical scope or compound statement.
  **L1642 CN**: 结束当前词法作用域或复合语句块。
- **L1643 EN**: Blank line separating nearby declarations or logic blocks.
  **L1643 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1644 EN**: Comment explains nearby logic, constraints, or intent: `Conditionally store byte elements of A into P. The high bit of each`.
  **L1644 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Conditionally store byte elements of A into P. The high bit of each`。
- **L1645 EN**: Continues the surrounding expression or declaration: `byte in the selector N determines whether the corresponding byte from`.
  **L1645 CN**: 继续构造周围的表达式或声明：`byte in the selector N determines whether the corresponding byte from`。
- **L1646 EN**: Continues the surrounding expression or declaration: `A is stored.  */`.
  **L1646 CN**: 继续构造周围的表达式或声明：`A is stored.  */`。
- **L1647 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L1647 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L1648 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1648 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1649 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskmove_si64(__m64 __A, __m64 __N, char *__P) {`.
  **L1649 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskmove_si64(__m64 __A, __m64 __N, char *__P) {`。
- **L1650 EN**: Initializes variable `__hibit` from the expression on the right-hand side.
  **L1650 CN**: 使用右侧表达式初始化变量 `__hibit`。
- **L1651 EN**: Adds a standalone statement or declaration: `__m64 __mask, __tmp;`.
  **L1651 CN**: 添加一条独立语句或声明：`__m64 __mask, __tmp;`。
- **L1652 EN**: Executes a call or declaration centered on `=`.
  **L1652 CN**: 执行以 `=` 为核心的调用或声明。
- **L1653 EN**: Blank line separating nearby declarations or logic blocks.
  **L1653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1654 EN**: Adds a standalone statement or declaration: `__tmp = *__p;`.
  **L1654 CN**: 添加一条独立语句或声明：`__tmp = *__p;`。
- **L1655 EN**: Executes a call or declaration centered on `_mm_cmpeq_pi8`.
  **L1655 CN**: 执行以 `_mm_cmpeq_pi8` 为核心的调用或声明。
- **L1656 EN**: Executes a call or declaration centered on `=`.
  **L1656 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 1657-1680

````c
  *__p = __tmp;
}

extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_maskmovq(__m64 __A, __m64 __N, char *__P) {
  _mm_maskmove_si64(__A, __N, __P);
}

/* Compute the rounded averages of the unsigned 8-bit values in A and B.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_avg_pu8(__m64 __A, __m64 __B) {
  __vector unsigned char __a, __b, __c;

  __a = (__vector unsigned char)vec_splats(__A);
  __b = (__vector unsigned char)vec_splats(__B);
  __c = vec_avg(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pavgb(__m64 __A, __m64 __B) {
````
- **L1657 EN**: Comment explains nearby logic, constraints, or intent: `__p __tmp;`.
  **L1657 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__p __tmp;`。
- **L1658 EN**: Closes the current lexical scope or compound statement.
  **L1658 CN**: 结束当前词法作用域或复合语句块。
- **L1659 EN**: Blank line separating nearby declarations or logic blocks.
  **L1659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1660 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L1660 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L1661 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1661 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1662 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_maskmovq(__m64 __A, __m64 __N, char *__P) {`.
  **L1662 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_maskmovq(__m64 __A, __m64 __N, char *__P) {`。
- **L1663 EN**: Executes a call or declaration centered on `_mm_maskmove_si64`.
  **L1663 CN**: 执行以 `_mm_maskmove_si64` 为核心的调用或声明。
- **L1664 EN**: Closes the current lexical scope or compound statement.
  **L1664 CN**: 结束当前词法作用域或复合语句块。
- **L1665 EN**: Blank line separating nearby declarations or logic blocks.
  **L1665 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1666 EN**: Comment explains nearby logic, constraints, or intent: `Compute the rounded averages of the unsigned 8-bit values in A and B.`.
  **L1666 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compute the rounded averages of the unsigned 8-bit values in A and B.`。
- **L1667 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1667 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1668 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1668 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1669 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_avg_pu8(__m64 __A, __m64 __B) {`.
  **L1669 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_avg_pu8(__m64 __A, __m64 __B) {`。
- **L1670 EN**: Adds a standalone statement or declaration: `__vector unsigned char __a, __b, __c;`.
  **L1670 CN**: 添加一条独立语句或声明：`__vector unsigned char __a, __b, __c;`。
- **L1671 EN**: Blank line separating nearby declarations or logic blocks.
  **L1671 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1672 EN**: Executes a call or declaration centered on `=`.
  **L1672 CN**: 执行以 `=` 为核心的调用或声明。
- **L1673 EN**: Executes a call or declaration centered on `=`.
  **L1673 CN**: 执行以 `=` 为核心的调用或声明。
- **L1674 EN**: Executes a call or declaration centered on `vec_avg`.
  **L1674 CN**: 执行以 `vec_avg` 为核心的调用或声明。
- **L1675 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1675 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1676 EN**: Closes the current lexical scope or compound statement.
  **L1676 CN**: 结束当前词法作用域或复合语句块。
- **L1677 EN**: Blank line separating nearby declarations or logic blocks.
  **L1677 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1678 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1678 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1679 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1679 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1680 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pavgb(__m64 __A, __m64 __B) {`.
  **L1680 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pavgb(__m64 __A, __m64 __B) {`。

### Lines 1681-1704

````c
  return _mm_avg_pu8(__A, __B);
}

/* Compute the rounded averages of the unsigned 16-bit values in A and B.  */
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_avg_pu16(__m64 __A, __m64 __B) {
  __vector unsigned short __a, __b, __c;

  __a = (__vector unsigned short)vec_splats(__A);
  __b = (__vector unsigned short)vec_splats(__B);
  __c = vec_avg(__a, __b);
  return (__m64)((__vector long long)__c)[0];
}

extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_pavgw(__m64 __A, __m64 __B) {
  return _mm_avg_pu16(__A, __B);
}

/* Compute the sum of the absolute differences of the unsigned 8-bit
   values in A and B.  Return the value in the lower 16-bit word; the
   upper words are cleared.  */
````
- **L1681 EN**: Returns from the current function with `_mm_avg_pu8(__A, __B)`.
  **L1681 CN**: 以 `_mm_avg_pu8(__A, __B)` 从当前函数返回。
- **L1682 EN**: Closes the current lexical scope or compound statement.
  **L1682 CN**: 结束当前词法作用域或复合语句块。
- **L1683 EN**: Blank line separating nearby declarations or logic blocks.
  **L1683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1684 EN**: Comment explains nearby logic, constraints, or intent: `Compute the rounded averages of the unsigned 16-bit values in A and B.`.
  **L1684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compute the rounded averages of the unsigned 16-bit values in A and B.`。
- **L1685 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1685 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1686 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1686 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1687 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_avg_pu16(__m64 __A, __m64 __B) {`.
  **L1687 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_avg_pu16(__m64 __A, __m64 __B) {`。
- **L1688 EN**: Adds a standalone statement or declaration: `__vector unsigned short __a, __b, __c;`.
  **L1688 CN**: 添加一条独立语句或声明：`__vector unsigned short __a, __b, __c;`。
- **L1689 EN**: Blank line separating nearby declarations or logic blocks.
  **L1689 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1690 EN**: Executes a call or declaration centered on `=`.
  **L1690 CN**: 执行以 `=` 为核心的调用或声明。
- **L1691 EN**: Executes a call or declaration centered on `=`.
  **L1691 CN**: 执行以 `=` 为核心的调用或声明。
- **L1692 EN**: Executes a call or declaration centered on `vec_avg`.
  **L1692 CN**: 执行以 `vec_avg` 为核心的调用或声明。
- **L1693 EN**: Returns from the current function with `(__m64)((__vector long long)__c)[0]`.
  **L1693 CN**: 以 `(__m64)((__vector long long)__c)[0]` 从当前函数返回。
- **L1694 EN**: Closes the current lexical scope or compound statement.
  **L1694 CN**: 结束当前词法作用域或复合语句块。
- **L1695 EN**: Blank line separating nearby declarations or logic blocks.
  **L1695 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1696 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1696 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1697 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1697 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1698 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_pavgw(__m64 __A, __m64 __B) {`.
  **L1698 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_pavgw(__m64 __A, __m64 __B) {`。
- **L1699 EN**: Returns from the current function with `_mm_avg_pu16(__A, __B)`.
  **L1699 CN**: 以 `_mm_avg_pu16(__A, __B)` 从当前函数返回。
- **L1700 EN**: Closes the current lexical scope or compound statement.
  **L1700 CN**: 结束当前词法作用域或复合语句块。
- **L1701 EN**: Blank line separating nearby declarations or logic blocks.
  **L1701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1702 EN**: Comment explains nearby logic, constraints, or intent: `Compute the sum of the absolute differences of the unsigned 8-bit`.
  **L1702 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compute the sum of the absolute differences of the unsigned 8-bit`。
- **L1703 EN**: Continues the surrounding expression or declaration: `values in A and B.  Return the value in the lower 16-bit word; the`.
  **L1703 CN**: 继续构造周围的表达式或声明：`values in A and B.  Return the value in the lower 16-bit word; the`。
- **L1704 EN**: Continues the surrounding expression or declaration: `upper words are cleared.  */`.
  **L1704 CN**: 继续构造周围的表达式或声明：`upper words are cleared.  */`。

### Lines 1705-1728

````c
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sad_pu8(__m64 __A, __m64 __B) {
  __vector unsigned char __a, __b;
  __vector unsigned char __vmin, __vmax, __vabsdiff;
  __vector signed int __vsum;
  const __vector unsigned int __zero = {0, 0, 0, 0};
  __m64_union __result = {0};

  __a = (__vector unsigned char)(__vector unsigned long long){0UL, __A};
  __b = (__vector unsigned char)(__vector unsigned long long){0UL, __B};
  __vmin = vec_min(__a, __b);
  __vmax = vec_max(__a, __b);
  __vabsdiff = vec_sub(__vmax, __vmin);
  /* Sum four groups of bytes into integers.  */
  __vsum = (__vector signed int)vec_sum4s(__vabsdiff, __zero);
  /* Sum across four integers with integer result.  */
  __vsum = vec_sums(__vsum, (__vector signed int)__zero);
  /* The sum is in the right most 32-bits of the vector result.
     Transfer to a GPR and truncate to 16 bits.  */
  __result.as_short[0] = __vsum[3];
  return __result.as_m64;
}

````
- **L1705 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1705 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1706 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1706 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1707 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sad_pu8(__m64 __A, __m64 __B) {`.
  **L1707 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sad_pu8(__m64 __A, __m64 __B) {`。
- **L1708 EN**: Adds a standalone statement or declaration: `__vector unsigned char __a, __b;`.
  **L1708 CN**: 添加一条独立语句或声明：`__vector unsigned char __a, __b;`。
- **L1709 EN**: Adds a standalone statement or declaration: `__vector unsigned char __vmin, __vmax, __vabsdiff;`.
  **L1709 CN**: 添加一条独立语句或声明：`__vector unsigned char __vmin, __vmax, __vabsdiff;`。
- **L1710 EN**: Adds a standalone statement or declaration: `__vector signed int __vsum;`.
  **L1710 CN**: 添加一条独立语句或声明：`__vector signed int __vsum;`。
- **L1711 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L1711 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L1712 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L1712 CN**: 使用右侧表达式初始化变量 `__result`。
- **L1713 EN**: Blank line separating nearby declarations or logic blocks.
  **L1713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1714 EN**: Executes a call or declaration centered on `=`.
  **L1714 CN**: 执行以 `=` 为核心的调用或声明。
- **L1715 EN**: Executes a call or declaration centered on `=`.
  **L1715 CN**: 执行以 `=` 为核心的调用或声明。
- **L1716 EN**: Executes a call or declaration centered on `vec_min`.
  **L1716 CN**: 执行以 `vec_min` 为核心的调用或声明。
- **L1717 EN**: Executes a call or declaration centered on `vec_max`.
  **L1717 CN**: 执行以 `vec_max` 为核心的调用或声明。
- **L1718 EN**: Executes a call or declaration centered on `vec_sub`.
  **L1718 CN**: 执行以 `vec_sub` 为核心的调用或声明。
- **L1719 EN**: Comment explains nearby logic, constraints, or intent: `Sum four groups of bytes into integers.`.
  **L1719 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sum four groups of bytes into integers.`。
- **L1720 EN**: Executes a call or declaration centered on `=`.
  **L1720 CN**: 执行以 `=` 为核心的调用或声明。
- **L1721 EN**: Comment explains nearby logic, constraints, or intent: `Sum across four integers with integer result.`.
  **L1721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sum across four integers with integer result.`。
- **L1722 EN**: Executes a call or declaration centered on `vec_sums`.
  **L1722 CN**: 执行以 `vec_sums` 为核心的调用或声明。
- **L1723 EN**: Comment explains nearby logic, constraints, or intent: `The sum is in the right most 32-bits of the vector result.`.
  **L1723 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The sum is in the right most 32-bits of the vector result.`。
- **L1724 EN**: Continues the surrounding expression or declaration: `Transfer to a GPR and truncate to 16 bits.  */`.
  **L1724 CN**: 继续构造周围的表达式或声明：`Transfer to a GPR and truncate to 16 bits.  */`。
- **L1725 EN**: Adds a standalone statement or declaration: `__result.as_short[0] = __vsum[3];`.
  **L1725 CN**: 添加一条独立语句或声明：`__result.as_short[0] = __vsum[3];`。
- **L1726 EN**: Returns from the current function with `__result.as_m64`.
  **L1726 CN**: 以 `__result.as_m64` 从当前函数返回。
- **L1727 EN**: Closes the current lexical scope or compound statement.
  **L1727 CN**: 结束当前词法作用域或复合语句块。
- **L1728 EN**: Blank line separating nearby declarations or logic blocks.
  **L1728 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1729-1752

````c
extern __inline __m64
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _m_psadbw(__m64 __A, __m64 __B) {
  return _mm_sad_pu8(__A, __B);
}

/* Stores the data in A to the address P without polluting the caches.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_stream_pi(__m64 *__P, __m64 __A) {
  /* Use the data cache block touch for store transient.  */
  __asm__("	dcbtstt	0,%0" : : "b"(__P) : "memory");
  *__P = __A;
}

/* Likewise.  The address must be 16-byte aligned.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_stream_ps(float *__P, __m128 __A) {
  /* Use the data cache block touch for store transient.  */
  __asm__("	dcbtstt	0,%0" : : "b"(__P) : "memory");
  _mm_store_ps(__P, __A);
}

````
- **L1729 EN**: Continues the surrounding expression or declaration: `extern __inline __m64`.
  **L1729 CN**: 继续构造周围的表达式或声明：`extern __inline __m64`。
- **L1730 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1730 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1731 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_m_psadbw(__m64 __A, __m64 __B) {`.
  **L1731 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_m_psadbw(__m64 __A, __m64 __B) {`。
- **L1732 EN**: Returns from the current function with `_mm_sad_pu8(__A, __B)`.
  **L1732 CN**: 以 `_mm_sad_pu8(__A, __B)` 从当前函数返回。
- **L1733 EN**: Closes the current lexical scope or compound statement.
  **L1733 CN**: 结束当前词法作用域或复合语句块。
- **L1734 EN**: Blank line separating nearby declarations or logic blocks.
  **L1734 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1735 EN**: Comment explains nearby logic, constraints, or intent: `Stores the data in A to the address P without polluting the caches.`.
  **L1735 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores the data in A to the address P without polluting the caches.`。
- **L1736 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L1736 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L1737 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1737 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1738 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_stream_pi(__m64 *__P, __m64 __A) {`.
  **L1738 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_stream_pi(__m64 *__P, __m64 __A) {`。
- **L1739 EN**: Comment explains nearby logic, constraints, or intent: `Use the data cache block touch for store transient.`.
  **L1739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use the data cache block touch for store transient.`。
- **L1740 EN**: Executes a call or declaration centered on `__asm__`.
  **L1740 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L1741 EN**: Comment explains nearby logic, constraints, or intent: `__P __A;`.
  **L1741 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__P __A;`。
- **L1742 EN**: Closes the current lexical scope or compound statement.
  **L1742 CN**: 结束当前词法作用域或复合语句块。
- **L1743 EN**: Blank line separating nearby declarations or logic blocks.
  **L1743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1744 EN**: Comment explains nearby logic, constraints, or intent: `Likewise. The address must be 16-byte aligned.`.
  **L1744 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Likewise. The address must be 16-byte aligned.`。
- **L1745 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L1745 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L1746 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1746 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1747 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_stream_ps(float *__P, __m128 __A) {`.
  **L1747 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_stream_ps(float *__P, __m128 __A) {`。
- **L1748 EN**: Comment explains nearby logic, constraints, or intent: `Use the data cache block touch for store transient.`.
  **L1748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Use the data cache block touch for store transient.`。
- **L1749 EN**: Executes a call or declaration centered on `__asm__`.
  **L1749 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L1750 EN**: Executes a call or declaration centered on `_mm_store_ps`.
  **L1750 CN**: 执行以 `_mm_store_ps` 为核心的调用或声明。
- **L1751 EN**: Closes the current lexical scope or compound statement.
  **L1751 CN**: 结束当前词法作用域或复合语句块。
- **L1752 EN**: Blank line separating nearby declarations or logic blocks.
  **L1752 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1753-1776

````c
/* Guarantees that every preceding store is globally visible before
   any subsequent store.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_sfence(void) {
  /* Generate a light weight sync.  */
  __atomic_thread_fence(__ATOMIC_RELEASE);
}

/* The execution of the next instruction is delayed by an implementation
   specific amount of time.  The instruction does not modify the
   architectural state.  This is after the pop_options pragma because
   it does not require SSE support in the processor--the encoding is a
   nop on processors that do not support it.  */
extern __inline void
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_pause(void) {
  /* There is no exact match with this construct, but the following is
     close to the desired effect.  */
#if _ARCH_PWR8
  /* On power8 and later processors we can depend on Program Priority
     (PRI) and associated "very low" PPI setting.  Since we don't know
     what PPI this thread is running at we: 1) save the current PRI
     from the PPR SPR into a local GRP, 2) set the PRI to "very low*
````
- **L1753 EN**: Comment explains nearby logic, constraints, or intent: `Guarantees that every preceding store is globally visible before`.
  **L1753 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Guarantees that every preceding store is globally visible before`。
- **L1754 EN**: Continues the surrounding expression or declaration: `any subsequent store.  */`.
  **L1754 CN**: 继续构造周围的表达式或声明：`any subsequent store.  */`。
- **L1755 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L1755 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L1756 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1756 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1757 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_sfence(void) {`.
  **L1757 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_sfence(void) {`。
- **L1758 EN**: Comment explains nearby logic, constraints, or intent: `Generate a light weight sync.`.
  **L1758 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generate a light weight sync.`。
- **L1759 EN**: Executes a call or declaration centered on `__atomic_thread_fence`.
  **L1759 CN**: 执行以 `__atomic_thread_fence` 为核心的调用或声明。
- **L1760 EN**: Closes the current lexical scope or compound statement.
  **L1760 CN**: 结束当前词法作用域或复合语句块。
- **L1761 EN**: Blank line separating nearby declarations or logic blocks.
  **L1761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1762 EN**: Comment explains nearby logic, constraints, or intent: `The execution of the next instruction is delayed by an implementation`.
  **L1762 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The execution of the next instruction is delayed by an implementation`。
- **L1763 EN**: Continues the surrounding expression or declaration: `specific amount of time.  The instruction does not modify the`.
  **L1763 CN**: 继续构造周围的表达式或声明：`specific amount of time.  The instruction does not modify the`。
- **L1764 EN**: Continues the surrounding expression or declaration: `architectural state.  This is after the pop_options pragma because`.
  **L1764 CN**: 继续构造周围的表达式或声明：`architectural state.  This is after the pop_options pragma because`。
- **L1765 EN**: Continues the surrounding expression or declaration: `it does not require SSE support in the processor--the encoding is a`.
  **L1765 CN**: 继续构造周围的表达式或声明：`it does not require SSE support in the processor--the encoding is a`。
- **L1766 EN**: Continues the surrounding expression or declaration: `nop on processors that do not support it.  */`.
  **L1766 CN**: 继续构造周围的表达式或声明：`nop on processors that do not support it.  */`。
- **L1767 EN**: Continues the surrounding expression or declaration: `extern __inline void`.
  **L1767 CN**: 继续构造周围的表达式或声明：`extern __inline void`。
- **L1768 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L1768 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L1769 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_pause(void) {`.
  **L1769 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_pause(void) {`。
- **L1770 EN**: Comment explains nearby logic, constraints, or intent: `There is no exact match with this construct, but the following is`.
  **L1770 CN**: 注释解释附近代码的逻辑、约束或设计意图：`There is no exact match with this construct, but the following is`。
- **L1771 EN**: Continues the surrounding expression or declaration: `close to the desired effect.  */`.
  **L1771 CN**: 继续构造周围的表达式或声明：`close to the desired effect.  */`。
- **L1772 EN**: Starts a preprocessor conditional block: `#if _ARCH_PWR8`.
  **L1772 CN**: 开始一个预处理条件块：`#if _ARCH_PWR8`。
- **L1773 EN**: Comment explains nearby logic, constraints, or intent: `On power8 and later processors we can depend on Program Priority`.
  **L1773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`On power8 and later processors we can depend on Program Priority`。
- **L1774 EN**: Continues the surrounding expression or declaration: `(PRI) and associated "very low" PPI setting.  Since we don't know`.
  **L1774 CN**: 继续构造周围的表达式或声明：`(PRI) and associated "very low" PPI setting.  Since we don't know`。
- **L1775 EN**: Continues the surrounding expression or declaration: `what PPI this thread is running at we: 1) save the current PRI`.
  **L1775 CN**: 继续构造周围的表达式或声明：`what PPI this thread is running at we: 1) save the current PRI`。
- **L1776 EN**: Continues the surrounding expression or declaration: `from the PPR SPR into a local GRP, 2) set the PRI to "very low*`.
  **L1776 CN**: 继续构造周围的表达式或声明：`from the PPR SPR into a local GRP, 2) set the PRI to "very low*`。

### Lines 1777-1800

````c
     via the special or 31,31,31 encoding. 3) issue an "isync" to
     insure the PRI change takes effect before we execute any more
     instructions.
     Now we can execute a lwsync (release barrier) while we execute
     this thread at "very low" PRI.  Finally we restore the original
     PRI and continue execution.  */
  unsigned long __PPR;

  __asm__ volatile("	mfppr	%0;"
                   "   or 31,31,31;"
                   "   isync;"
                   "   lwsync;"
                   "   isync;"
                   "   mtppr	%0;"
                   : "=r"(__PPR)
                   :
                   : "memory");
#else
  /* For older processor where we may not even have Program Priority
     controls we can only depend on Heavy Weight Sync.  */
  __atomic_thread_fence(__ATOMIC_SEQ_CST);
#endif
}

````
- **L1777 EN**: Continues the surrounding expression or declaration: `via the special or 31,31,31 encoding. 3) issue an "isync" to`.
  **L1777 CN**: 继续构造周围的表达式或声明：`via the special or 31,31,31 encoding. 3) issue an "isync" to`。
- **L1778 EN**: Continues the surrounding expression or declaration: `insure the PRI change takes effect before we execute any more`.
  **L1778 CN**: 继续构造周围的表达式或声明：`insure the PRI change takes effect before we execute any more`。
- **L1779 EN**: Continues the surrounding expression or declaration: `instructions.`.
  **L1779 CN**: 继续构造周围的表达式或声明：`instructions.`。
- **L1780 EN**: Continues logic associated with callable symbol `lwsync`.
  **L1780 CN**: 继续与可调用符号 `lwsync` 相关的逻辑。
- **L1781 EN**: Continues the surrounding expression or declaration: `this thread at "very low" PRI.  Finally we restore the original`.
  **L1781 CN**: 继续构造周围的表达式或声明：`this thread at "very low" PRI.  Finally we restore the original`。
- **L1782 EN**: Continues the surrounding expression or declaration: `PRI and continue execution.  */`.
  **L1782 CN**: 继续构造周围的表达式或声明：`PRI and continue execution.  */`。
- **L1783 EN**: Adds a standalone statement or declaration: `unsigned long __PPR;`.
  **L1783 CN**: 添加一条独立语句或声明：`unsigned long __PPR;`。
- **L1784 EN**: Blank line separating nearby declarations or logic blocks.
  **L1784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1785 EN**: Continues logic associated with callable symbol `volatile`.
  **L1785 CN**: 继续与可调用符号 `volatile` 相关的逻辑。
- **L1786 EN**: Continues the surrounding expression or declaration: `"   or 31,31,31;"`.
  **L1786 CN**: 继续构造周围的表达式或声明：`"   or 31,31,31;"`。
- **L1787 EN**: Continues the surrounding expression or declaration: `"   isync;"`.
  **L1787 CN**: 继续构造周围的表达式或声明：`"   isync;"`。
- **L1788 EN**: Continues the surrounding expression or declaration: `"   lwsync;"`.
  **L1788 CN**: 继续构造周围的表达式或声明：`"   lwsync;"`。
- **L1789 EN**: Continues the surrounding expression or declaration: `"   isync;"`.
  **L1789 CN**: 继续构造周围的表达式或声明：`"   isync;"`。
- **L1790 EN**: Continues the surrounding expression or declaration: `"   mtppr	%0;"`.
  **L1790 CN**: 继续构造周围的表达式或声明：`"   mtppr	%0;"`。
- **L1791 EN**: Continues the surrounding expression or declaration: `: "=r"(__PPR)`.
  **L1791 CN**: 继续构造周围的表达式或声明：`: "=r"(__PPR)`。
- **L1792 EN**: Continues the surrounding expression or declaration: `:`.
  **L1792 CN**: 继续构造周围的表达式或声明：`:`。
- **L1793 EN**: Adds a standalone statement or declaration: `: "memory");`.
  **L1793 CN**: 添加一条独立语句或声明：`: "memory");`。
- **L1794 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1794 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1795 EN**: Comment explains nearby logic, constraints, or intent: `For older processor where we may not even have Program Priority`.
  **L1795 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For older processor where we may not even have Program Priority`。
- **L1796 EN**: Continues the surrounding expression or declaration: `controls we can only depend on Heavy Weight Sync.  */`.
  **L1796 CN**: 继续构造周围的表达式或声明：`controls we can only depend on Heavy Weight Sync.  */`。
- **L1797 EN**: Executes a call or declaration centered on `__atomic_thread_fence`.
  **L1797 CN**: 执行以 `__atomic_thread_fence` 为核心的调用或声明。
- **L1798 EN**: Closes the current preprocessor conditional block.
  **L1798 CN**: 结束当前预处理条件块。
- **L1799 EN**: Closes the current lexical scope or compound statement.
  **L1799 CN**: 结束当前词法作用域或复合语句块。
- **L1800 EN**: Blank line separating nearby declarations or logic blocks.
  **L1800 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1801-1824

````c
/* Transpose the 4x4 matrix composed of row[0-3].  */
#define _MM_TRANSPOSE4_PS(row0, row1, row2, row3)                              \
  do {                                                                         \
    __v4sf __r0 = (row0), __r1 = (row1), __r2 = (row2), __r3 = (row3);         \
    __v4sf __t0 = vec_vmrghw(__r0, __r1);                                      \
    __v4sf __t1 = vec_vmrghw(__r2, __r3);                                      \
    __v4sf __t2 = vec_vmrglw(__r0, __r1);                                      \
    __v4sf __t3 = vec_vmrglw(__r2, __r3);                                      \
    (row0) = (__v4sf)vec_mergeh((__vector long long)__t0,                      \
                                (__vector long long)__t1);                     \
    (row1) = (__v4sf)vec_mergel((__vector long long)__t0,                      \
                                (__vector long long)__t1);                     \
    (row2) = (__v4sf)vec_mergeh((__vector long long)__t2,                      \
                                (__vector long long)__t3);                     \
    (row3) = (__v4sf)vec_mergel((__vector long long)__t2,                      \
                                (__vector long long)__t3);                     \
  } while (0)

/* For backward source compatibility.  */
//# include <emmintrin.h>

#else
#include_next <xmmintrin.h>
#endif /* defined(__powerpc64__) &&                                            \
````
- **L1801 EN**: Comment explains nearby logic, constraints, or intent: `Transpose the 4x4 matrix composed of row[0-3].`.
  **L1801 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Transpose the 4x4 matrix composed of row[0-3].`。
- **L1802 EN**: Defines macro `_MM_TRANSPOSE4_PS(row0, row1, row2, row3)` for conditional compilation, shorthand, or API generation.
  **L1802 CN**: 定义宏 `_MM_TRANSPOSE4_PS(row0, row1, row2, row3)`，用于条件编译、简写或 API 生成。
- **L1803 EN**: Continues the surrounding expression or declaration: `do {                                                                         \`.
  **L1803 CN**: 继续构造周围的表达式或声明：`do {                                                                         \`。
- **L1804 EN**: Continues the surrounding expression or declaration: `__v4sf __r0 = (row0), __r1 = (row1), __r2 = (row2), __r3 = (row3);         \`.
  **L1804 CN**: 继续构造周围的表达式或声明：`__v4sf __r0 = (row0), __r1 = (row1), __r2 = (row2), __r3 = (row3);         \`。
- **L1805 EN**: Continues logic associated with callable symbol `vec_vmrghw`.
  **L1805 CN**: 继续与可调用符号 `vec_vmrghw` 相关的逻辑。
- **L1806 EN**: Continues logic associated with callable symbol `vec_vmrghw`.
  **L1806 CN**: 继续与可调用符号 `vec_vmrghw` 相关的逻辑。
- **L1807 EN**: Continues logic associated with callable symbol `vec_vmrglw`.
  **L1807 CN**: 继续与可调用符号 `vec_vmrglw` 相关的逻辑。
- **L1808 EN**: Continues logic associated with callable symbol `vec_vmrglw`.
  **L1808 CN**: 继续与可调用符号 `vec_vmrglw` 相关的逻辑。
- **L1809 EN**: Continues logic associated with callable symbol `vec_mergeh`.
  **L1809 CN**: 继续与可调用符号 `vec_mergeh` 相关的逻辑。
- **L1810 EN**: Continues the surrounding expression or declaration: `(__vector long long)__t1);                     \`.
  **L1810 CN**: 继续构造周围的表达式或声明：`(__vector long long)__t1);                     \`。
- **L1811 EN**: Continues logic associated with callable symbol `vec_mergel`.
  **L1811 CN**: 继续与可调用符号 `vec_mergel` 相关的逻辑。
- **L1812 EN**: Continues the surrounding expression or declaration: `(__vector long long)__t1);                     \`.
  **L1812 CN**: 继续构造周围的表达式或声明：`(__vector long long)__t1);                     \`。
- **L1813 EN**: Continues logic associated with callable symbol `vec_mergeh`.
  **L1813 CN**: 继续与可调用符号 `vec_mergeh` 相关的逻辑。
- **L1814 EN**: Continues the surrounding expression or declaration: `(__vector long long)__t3);                     \`.
  **L1814 CN**: 继续构造周围的表达式或声明：`(__vector long long)__t3);                     \`。
- **L1815 EN**: Continues logic associated with callable symbol `vec_mergel`.
  **L1815 CN**: 继续与可调用符号 `vec_mergel` 相关的逻辑。
- **L1816 EN**: Continues the surrounding expression or declaration: `(__vector long long)__t3);                     \`.
  **L1816 CN**: 继续构造周围的表达式或声明：`(__vector long long)__t3);                     \`。
- **L1817 EN**: Continues the surrounding expression or declaration: `} while (0)`.
  **L1817 CN**: 继续构造周围的表达式或声明：`} while (0)`。
- **L1818 EN**: Blank line separating nearby declarations or logic blocks.
  **L1818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1819 EN**: Comment explains nearby logic, constraints, or intent: `For backward source compatibility.`.
  **L1819 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For backward source compatibility.`。
- **L1820 EN**: Comment explains nearby logic, constraints, or intent: `# include <emmintrin.h>`.
  **L1820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`# include <emmintrin.h>`。
- **L1821 EN**: Blank line separating nearby declarations or logic blocks.
  **L1821 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1822 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1822 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1823 EN**: Includes <xmmintrin.h> to access related header declarations.
  **L1823 CN**: 引入 <xmmintrin.h> 以使用相关头文件声明。
- **L1824 EN**: Closes the current preprocessor conditional block.
  **L1824 CN**: 结束当前预处理条件块。

### Lines 1825-1827

````c
        *   (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX)) */

#endif /* XMMINTRIN_H_ */
````
- **L1825 EN**: Comment explains nearby logic, constraints, or intent: `(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`.
  **L1825 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`。
- **L1826 EN**: Blank line separating nearby declarations or logic blocks.
  **L1826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1827 EN**: Closes the current preprocessor conditional block.
  **L1827 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **PowerPC vector or system interfaces / PowerPC 向量或系统接口**
- **wrapper forwarding layers / 包装转发层**
- **Header composition through includes / 通过 include 组合头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Enumerated constants / 枚举常量**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `altivec.h`: Provides related header declarations. / 提供相关头文件声明。
  - `mmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
  - `mm_malloc.h`: Provides related header declarations. / 提供相关头文件声明。
  - `xmmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `NO_WARN_X86_INTRINSICS`, `XMMINTRIN_H_`, `__powerpc64__`, `__linux__`, `__FreeBSD__`, `_AIX`, `__STRICT_ANSI__`, `__cplusplus`, `__STDC_VERSION__`, `_ARCH_PWR7`, `_ARCH_PWR8`, `__LITTLE_ENDIAN__`, `_ARCH_PWR10`, `__BIG_ENDIAN__`
- **External builtins / 外部 builtin**: `__builtin_ia32_andps`, `__builtin_rint`, `__builtin_llrint`, `__builtin_prefetch`, `__builtin_bpermd`
