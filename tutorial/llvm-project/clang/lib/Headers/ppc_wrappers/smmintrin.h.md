# smmintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/ppc_wrappers/smmintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implementation of SSE4 intrinsics on PowerPC.
- **Purpose (CN)**: 该头文件主要作用是：Implementation of SSE4 intrinsics on PowerPC。
- **Line Count / 行数**: 683

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*===---- smmintrin.h - Implementation of SSE4 intrinsics on PowerPC -------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

/* Implemented from the specification included in the Intel C++ Compiler
   User Guide and Reference, version 9.0.

   NOTE: This is NOT a complete implementation of the SSE4 intrinsics!  */

#ifndef NO_WARN_X86_INTRINSICS
/* This header is distributed to simplify porting x86_64 code that
   makes explicit use of Intel intrinsics to powerpc64/powerpc64le.

   It is the user's responsibility to determine if the results are
   acceptable and make additional changes as necessary.
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
- **L11 EN**: Continues the surrounding expression or declaration: `User Guide and Reference, version 9.0.`.
  **L11 CN**: 继续构造周围的表达式或声明：`User Guide and Reference, version 9.0.`。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Continues the surrounding expression or declaration: `NOTE: This is NOT a complete implementation of the SSE4 intrinsics!  */`.
  **L13 CN**: 继续构造周围的表达式或声明：`NOTE: This is NOT a complete implementation of the SSE4 intrinsics!  */`。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Starts a preprocessor conditional block: `#ifndef NO_WARN_X86_INTRINSICS`.
  **L15 CN**: 开始一个预处理条件块：`#ifndef NO_WARN_X86_INTRINSICS`。
- **L16 EN**: Comment explains nearby logic, constraints, or intent: `This header is distributed to simplify porting x86_64 code that`.
  **L16 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This header is distributed to simplify porting x86_64 code that`。
- **L17 EN**: Continues the surrounding expression or declaration: `makes explicit use of Intel intrinsics to powerpc64/powerpc64le.`.
  **L17 CN**: 继续构造周围的表达式或声明：`makes explicit use of Intel intrinsics to powerpc64/powerpc64le.`。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Continues the surrounding expression or declaration: `It is the user's responsibility to determine if the results are`.
  **L19 CN**: 继续构造周围的表达式或声明：`It is the user's responsibility to determine if the results are`。
- **L20 EN**: Continues the surrounding expression or declaration: `acceptable and make additional changes as necessary.`.
  **L20 CN**: 继续构造周围的表达式或声明：`acceptable and make additional changes as necessary.`。

### Lines 21-40

````c

   Note that much code that uses Intel intrinsics can be rewritten in
   standard C or GNU C extensions, which are more portable and better
   optimized across multiple targets.  */
#error                                                                         \
    "Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."
#endif

#ifndef SMMINTRIN_H_
#define SMMINTRIN_H_

#if defined(__powerpc64__) &&                                                  \
    (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))

#include <altivec.h>
#include <tmmintrin.h>

/* Rounding mode macros. */
#define _MM_FROUND_TO_NEAREST_INT 0x00
#define _MM_FROUND_TO_ZERO 0x01
````
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L22 EN**: Continues the surrounding expression or declaration: `Note that much code that uses Intel intrinsics can be rewritten in`.
  **L22 CN**: 继续构造周围的表达式或声明：`Note that much code that uses Intel intrinsics can be rewritten in`。
- **L23 EN**: Continues the surrounding expression or declaration: `standard C or GNU C extensions, which are more portable and better`.
  **L23 CN**: 继续构造周围的表达式或声明：`standard C or GNU C extensions, which are more portable and better`。
- **L24 EN**: Continues the surrounding expression or declaration: `optimized across multiple targets.  */`.
  **L24 CN**: 继续构造周围的表达式或声明：`optimized across multiple targets.  */`。
- **L25 EN**: Emits a compilation error for an unsupported configuration: `#error                                                                         \`.
  **L25 CN**: 为不受支持的配置触发编译错误：`#error                                                                         \`。
- **L26 EN**: Continues the surrounding expression or declaration: `"Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."`.
  **L26 CN**: 继续构造周围的表达式或声明：`"Please read comment above.  Use -DNO_WARN_X86_INTRINSICS to disable this error."`。
- **L27 EN**: Closes the current preprocessor conditional block.
  **L27 CN**: 结束当前预处理条件块。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L29 EN**: Starts a preprocessor conditional block: `#ifndef SMMINTRIN_H_`.
  **L29 CN**: 开始一个预处理条件块：`#ifndef SMMINTRIN_H_`。
- **L30 EN**: Defines macro `SMMINTRIN_H_` for conditional compilation, shorthand, or API generation.
  **L30 CN**: 定义宏 `SMMINTRIN_H_`，用于条件编译、简写或 API 生成。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L32 EN**: Starts a preprocessor conditional block: `#if defined(__powerpc64__) &&                                                  \`.
  **L32 CN**: 开始一个预处理条件块：`#if defined(__powerpc64__) &&                                                  \`。
- **L33 EN**: Continues logic associated with callable symbol `defined`.
  **L33 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Includes <altivec.h> to access related header declarations.
  **L35 CN**: 引入 <altivec.h> 以使用相关头文件声明。
- **L36 EN**: Includes <tmmintrin.h> to access related header declarations.
  **L36 CN**: 引入 <tmmintrin.h> 以使用相关头文件声明。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Comment explains nearby logic, constraints, or intent: `Rounding mode macros.`.
  **L38 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rounding mode macros.`。
- **L39 EN**: Defines macro `_MM_FROUND_TO_NEAREST_INT` for conditional compilation, shorthand, or API generation.
  **L39 CN**: 定义宏 `_MM_FROUND_TO_NEAREST_INT`，用于条件编译、简写或 API 生成。
- **L40 EN**: Defines macro `_MM_FROUND_TO_ZERO` for conditional compilation, shorthand, or API generation.
  **L40 CN**: 定义宏 `_MM_FROUND_TO_ZERO`，用于条件编译、简写或 API 生成。

### Lines 41-60

````c
#define _MM_FROUND_TO_POS_INF 0x02
#define _MM_FROUND_TO_NEG_INF 0x03
#define _MM_FROUND_CUR_DIRECTION 0x04

#define _MM_FROUND_NINT (_MM_FROUND_TO_NEAREST_INT | _MM_FROUND_RAISE_EXC)
#define _MM_FROUND_FLOOR (_MM_FROUND_TO_NEG_INF | _MM_FROUND_RAISE_EXC)
#define _MM_FROUND_CEIL (_MM_FROUND_TO_POS_INF | _MM_FROUND_RAISE_EXC)
#define _MM_FROUND_TRUNC (_MM_FROUND_TO_ZERO | _MM_FROUND_RAISE_EXC)
#define _MM_FROUND_RINT (_MM_FROUND_CUR_DIRECTION | _MM_FROUND_RAISE_EXC)
#define _MM_FROUND_NEARBYINT (_MM_FROUND_CUR_DIRECTION | _MM_FROUND_NO_EXC)

#define _MM_FROUND_RAISE_EXC 0x00
#define _MM_FROUND_NO_EXC 0x08

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_round_pd(__m128d __A, int __rounding) {
  __v2df __r;
  union {
    double __fr;
````
- **L41 EN**: Defines macro `_MM_FROUND_TO_POS_INF` for conditional compilation, shorthand, or API generation.
  **L41 CN**: 定义宏 `_MM_FROUND_TO_POS_INF`，用于条件编译、简写或 API 生成。
- **L42 EN**: Defines macro `_MM_FROUND_TO_NEG_INF` for conditional compilation, shorthand, or API generation.
  **L42 CN**: 定义宏 `_MM_FROUND_TO_NEG_INF`，用于条件编译、简写或 API 生成。
- **L43 EN**: Defines macro `_MM_FROUND_CUR_DIRECTION` for conditional compilation, shorthand, or API generation.
  **L43 CN**: 定义宏 `_MM_FROUND_CUR_DIRECTION`，用于条件编译、简写或 API 生成。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L45 EN**: Defines macro `_MM_FROUND_NINT` for conditional compilation, shorthand, or API generation.
  **L45 CN**: 定义宏 `_MM_FROUND_NINT`，用于条件编译、简写或 API 生成。
- **L46 EN**: Defines macro `_MM_FROUND_FLOOR` for conditional compilation, shorthand, or API generation.
  **L46 CN**: 定义宏 `_MM_FROUND_FLOOR`，用于条件编译、简写或 API 生成。
- **L47 EN**: Defines macro `_MM_FROUND_CEIL` for conditional compilation, shorthand, or API generation.
  **L47 CN**: 定义宏 `_MM_FROUND_CEIL`，用于条件编译、简写或 API 生成。
- **L48 EN**: Defines macro `_MM_FROUND_TRUNC` for conditional compilation, shorthand, or API generation.
  **L48 CN**: 定义宏 `_MM_FROUND_TRUNC`，用于条件编译、简写或 API 生成。
- **L49 EN**: Defines macro `_MM_FROUND_RINT` for conditional compilation, shorthand, or API generation.
  **L49 CN**: 定义宏 `_MM_FROUND_RINT`，用于条件编译、简写或 API 生成。
- **L50 EN**: Defines macro `_MM_FROUND_NEARBYINT` for conditional compilation, shorthand, or API generation.
  **L50 CN**: 定义宏 `_MM_FROUND_NEARBYINT`，用于条件编译、简写或 API 生成。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Defines macro `_MM_FROUND_RAISE_EXC` for conditional compilation, shorthand, or API generation.
  **L52 CN**: 定义宏 `_MM_FROUND_RAISE_EXC`，用于条件编译、简写或 API 生成。
- **L53 EN**: Defines macro `_MM_FROUND_NO_EXC` for conditional compilation, shorthand, or API generation.
  **L53 CN**: 定义宏 `_MM_FROUND_NO_EXC`，用于条件编译、简写或 API 生成。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L55 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L55 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L56 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L56 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L57 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_round_pd(__m128d __A, int __rounding) {`.
  **L57 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_round_pd(__m128d __A, int __rounding) {`。
- **L58 EN**: Adds a standalone statement or declaration: `__v2df __r;`.
  **L58 CN**: 添加一条独立语句或声明：`__v2df __r;`。
- **L59 EN**: Declares union `union`.
  **L59 CN**: 声明 union `union`。
- **L60 EN**: Adds a standalone statement or declaration: `double __fr;`.
  **L60 CN**: 添加一条独立语句或声明：`double __fr;`。

### Lines 61-80

````c
    long long __fpscr;
  } __enables_save, __fpscr_save;

  if (__rounding & _MM_FROUND_NO_EXC) {
    /* Save enabled exceptions, disable all exceptions,
       and preserve the rounding mode.  */
#ifdef _ARCH_PWR9
    __asm__("mffsce %0" : "=f"(__fpscr_save.__fr));
    __enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;
#else
    __fpscr_save.__fr = __builtin_ppc_mffs();
    __enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;
    __fpscr_save.__fpscr &= ~0xf8;
    __builtin_ppc_mtfsf(0b00000011, __fpscr_save.__fr);
#endif
    /* Insert an artificial "read/write" reference to the variable
       read below, to ensure the compiler does not schedule
       a read/use of the variable before the FPSCR is modified, above.
       This can be removed if and when GCC PR102783 is fixed.
     */
````
- **L61 EN**: Adds a standalone statement or declaration: `long long __fpscr;`.
  **L61 CN**: 添加一条独立语句或声明：`long long __fpscr;`。
- **L62 EN**: Adds a standalone statement or declaration: `} __enables_save, __fpscr_save;`.
  **L62 CN**: 添加一条独立语句或声明：`} __enables_save, __fpscr_save;`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L64 CN**: 开始 `if` 控制流语句并计算其条件。
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `Save enabled exceptions, disable all exceptions,`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Save enabled exceptions, disable all exceptions,`。
- **L66 EN**: Continues the surrounding expression or declaration: `and preserve the rounding mode.  */`.
  **L66 CN**: 继续构造周围的表达式或声明：`and preserve the rounding mode.  */`。
- **L67 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR9`.
  **L67 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR9`。
- **L68 EN**: Executes a call or declaration centered on `__asm__`.
  **L68 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L69 EN**: Adds a standalone statement or declaration: `__enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;`.
  **L69 CN**: 添加一条独立语句或声明：`__enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;`。
- **L70 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L70 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L71 EN**: Executes a call or declaration centered on `__builtin_ppc_mffs`.
  **L71 CN**: 执行以 `__builtin_ppc_mffs` 为核心的调用或声明。
- **L72 EN**: Adds a standalone statement or declaration: `__enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;`.
  **L72 CN**: 添加一条独立语句或声明：`__enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;`。
- **L73 EN**: Adds a standalone statement or declaration: `__fpscr_save.__fpscr &= ~0xf8;`.
  **L73 CN**: 添加一条独立语句或声明：`__fpscr_save.__fpscr &= ~0xf8;`。
- **L74 EN**: Executes a call or declaration centered on `__builtin_ppc_mtfsf`.
  **L74 CN**: 执行以 `__builtin_ppc_mtfsf` 为核心的调用或声明。
- **L75 EN**: Closes the current preprocessor conditional block.
  **L75 CN**: 结束当前预处理条件块。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `Insert an artificial "read/write" reference to the variable`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert an artificial "read/write" reference to the variable`。
- **L77 EN**: Continues the surrounding expression or declaration: `read below, to ensure the compiler does not schedule`.
  **L77 CN**: 继续构造周围的表达式或声明：`read below, to ensure the compiler does not schedule`。
- **L78 EN**: Continues the surrounding expression or declaration: `a read/use of the variable before the FPSCR is modified, above.`.
  **L78 CN**: 继续构造周围的表达式或声明：`a read/use of the variable before the FPSCR is modified, above.`。
- **L79 EN**: Continues the surrounding expression or declaration: `This can be removed if and when GCC PR102783 is fixed.`.
  **L79 CN**: 继续构造周围的表达式或声明：`This can be removed if and when GCC PR102783 is fixed.`。
- **L80 EN**: Separator comment used for visual grouping.
  **L80 CN**: 用于视觉分组的分隔注释。

### Lines 81-100

````c
    __asm__("" : "+wa"(__A));
  }

  switch (__rounding) {
  case _MM_FROUND_TO_NEAREST_INT:
#ifdef _ARCH_PWR9
    __fpscr_save.__fr = __builtin_ppc_mffsl();
#else
    __fpscr_save.__fr = __builtin_ppc_mffs();
    __fpscr_save.__fpscr &= 0x70007f0ffL;
#endif
    __attribute__((fallthrough));
  case _MM_FROUND_TO_NEAREST_INT | _MM_FROUND_NO_EXC:
    __builtin_ppc_set_fpscr_rn(0b00);
    /* Insert an artificial "read/write" reference to the variable
       read below, to ensure the compiler does not schedule
       a read/use of the variable before the FPSCR is modified, above.
       This can be removed if and when GCC PR102783 is fixed.
     */
    __asm__("" : "+wa"(__A));
````
- **L81 EN**: Executes a call or declaration centered on `__asm__`.
  **L81 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L82 EN**: Closes the current lexical scope or compound statement.
  **L82 CN**: 结束当前词法作用域或复合语句块。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L84 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L84 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L85 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_NEAREST_INT:`.
  **L85 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_NEAREST_INT:`。
- **L86 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR9`.
  **L86 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR9`。
- **L87 EN**: Executes a call or declaration centered on `__builtin_ppc_mffsl`.
  **L87 CN**: 执行以 `__builtin_ppc_mffsl` 为核心的调用或声明。
- **L88 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L88 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L89 EN**: Executes a call or declaration centered on `__builtin_ppc_mffs`.
  **L89 CN**: 执行以 `__builtin_ppc_mffs` 为核心的调用或声明。
- **L90 EN**: Adds a standalone statement or declaration: `__fpscr_save.__fpscr &= 0x70007f0ffL;`.
  **L90 CN**: 添加一条独立语句或声明：`__fpscr_save.__fpscr &= 0x70007f0ffL;`。
- **L91 EN**: Closes the current preprocessor conditional block.
  **L91 CN**: 结束当前预处理条件块。
- **L92 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((fallthrough));`.
  **L92 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((fallthrough));`。
- **L93 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_NEAREST_INT | _MM_FROUND_NO_EXC:`.
  **L93 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_NEAREST_INT | _MM_FROUND_NO_EXC:`。
- **L94 EN**: Executes a call or declaration centered on `__builtin_ppc_set_fpscr_rn`.
  **L94 CN**: 执行以 `__builtin_ppc_set_fpscr_rn` 为核心的调用或声明。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `Insert an artificial "read/write" reference to the variable`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert an artificial "read/write" reference to the variable`。
- **L96 EN**: Continues the surrounding expression or declaration: `read below, to ensure the compiler does not schedule`.
  **L96 CN**: 继续构造周围的表达式或声明：`read below, to ensure the compiler does not schedule`。
- **L97 EN**: Continues the surrounding expression or declaration: `a read/use of the variable before the FPSCR is modified, above.`.
  **L97 CN**: 继续构造周围的表达式或声明：`a read/use of the variable before the FPSCR is modified, above.`。
- **L98 EN**: Continues the surrounding expression or declaration: `This can be removed if and when GCC PR102783 is fixed.`.
  **L98 CN**: 继续构造周围的表达式或声明：`This can be removed if and when GCC PR102783 is fixed.`。
- **L99 EN**: Separator comment used for visual grouping.
  **L99 CN**: 用于视觉分组的分隔注释。
- **L100 EN**: Executes a call or declaration centered on `__asm__`.
  **L100 CN**: 执行以 `__asm__` 为核心的调用或声明。

### Lines 101-120

````c

    __r = vec_rint((__v2df)__A);

    /* Insert an artificial "read" reference to the variable written
       above, to ensure the compiler does not schedule the computation
       of the value after the manipulation of the FPSCR, below.
       This can be removed if and when GCC PR102783 is fixed.
     */
    __asm__("" : : "wa"(__r));
    __builtin_ppc_set_fpscr_rn(__fpscr_save.__fpscr);
    break;
  case _MM_FROUND_TO_NEG_INF:
  case _MM_FROUND_TO_NEG_INF | _MM_FROUND_NO_EXC:
    __r = vec_floor((__v2df)__A);
    break;
  case _MM_FROUND_TO_POS_INF:
  case _MM_FROUND_TO_POS_INF | _MM_FROUND_NO_EXC:
    __r = vec_ceil((__v2df)__A);
    break;
  case _MM_FROUND_TO_ZERO:
````
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L102 EN**: Executes a call or declaration centered on `vec_rint`.
  **L102 CN**: 执行以 `vec_rint` 为核心的调用或声明。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L104 EN**: Comment explains nearby logic, constraints, or intent: `Insert an artificial "read" reference to the variable written`.
  **L104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert an artificial "read" reference to the variable written`。
- **L105 EN**: Continues the surrounding expression or declaration: `above, to ensure the compiler does not schedule the computation`.
  **L105 CN**: 继续构造周围的表达式或声明：`above, to ensure the compiler does not schedule the computation`。
- **L106 EN**: Continues the surrounding expression or declaration: `of the value after the manipulation of the FPSCR, below.`.
  **L106 CN**: 继续构造周围的表达式或声明：`of the value after the manipulation of the FPSCR, below.`。
- **L107 EN**: Continues the surrounding expression or declaration: `This can be removed if and when GCC PR102783 is fixed.`.
  **L107 CN**: 继续构造周围的表达式或声明：`This can be removed if and when GCC PR102783 is fixed.`。
- **L108 EN**: Separator comment used for visual grouping.
  **L108 CN**: 用于视觉分组的分隔注释。
- **L109 EN**: Executes a call or declaration centered on `__asm__`.
  **L109 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L110 EN**: Executes a call or declaration centered on `__builtin_ppc_set_fpscr_rn`.
  **L110 CN**: 执行以 `__builtin_ppc_set_fpscr_rn` 为核心的调用或声明。
- **L111 EN**: Exits the nearest loop or switch statement.
  **L111 CN**: 退出最近的循环或 `switch` 语句。
- **L112 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_NEG_INF:`.
  **L112 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_NEG_INF:`。
- **L113 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_NEG_INF | _MM_FROUND_NO_EXC:`.
  **L113 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_NEG_INF | _MM_FROUND_NO_EXC:`。
- **L114 EN**: Executes a call or declaration centered on `vec_floor`.
  **L114 CN**: 执行以 `vec_floor` 为核心的调用或声明。
- **L115 EN**: Exits the nearest loop or switch statement.
  **L115 CN**: 退出最近的循环或 `switch` 语句。
- **L116 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_POS_INF:`.
  **L116 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_POS_INF:`。
- **L117 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_POS_INF | _MM_FROUND_NO_EXC:`.
  **L117 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_POS_INF | _MM_FROUND_NO_EXC:`。
- **L118 EN**: Executes a call or declaration centered on `vec_ceil`.
  **L118 CN**: 执行以 `vec_ceil` 为核心的调用或声明。
- **L119 EN**: Exits the nearest loop or switch statement.
  **L119 CN**: 退出最近的循环或 `switch` 语句。
- **L120 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_ZERO:`.
  **L120 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_ZERO:`。

### Lines 121-140

````c
  case _MM_FROUND_TO_ZERO | _MM_FROUND_NO_EXC:
    __r = vec_trunc((__v2df)__A);
    break;
  case _MM_FROUND_CUR_DIRECTION:
    __r = vec_rint((__v2df)__A);
    break;
  }
  if (__rounding & _MM_FROUND_NO_EXC) {
    /* Insert an artificial "read" reference to the variable written
       above, to ensure the compiler does not schedule the computation
       of the value after the manipulation of the FPSCR, below.
       This can be removed if and when GCC PR102783 is fixed.
     */
    __asm__("" : : "wa"(__r));
    /* Restore enabled exceptions.  */
#ifdef _ARCH_PWR9
    __fpscr_save.__fr = __builtin_ppc_mffsl();
#else
    __fpscr_save.__fr = __builtin_ppc_mffs();
    __fpscr_save.__fpscr &= 0x70007f0ffL;
````
- **L121 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_ZERO | _MM_FROUND_NO_EXC:`.
  **L121 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_ZERO | _MM_FROUND_NO_EXC:`。
- **L122 EN**: Executes a call or declaration centered on `vec_trunc`.
  **L122 CN**: 执行以 `vec_trunc` 为核心的调用或声明。
- **L123 EN**: Exits the nearest loop or switch statement.
  **L123 CN**: 退出最近的循环或 `switch` 语句。
- **L124 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_CUR_DIRECTION:`.
  **L124 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_CUR_DIRECTION:`。
- **L125 EN**: Executes a call or declaration centered on `vec_rint`.
  **L125 CN**: 执行以 `vec_rint` 为核心的调用或声明。
- **L126 EN**: Exits the nearest loop or switch statement.
  **L126 CN**: 退出最近的循环或 `switch` 语句。
- **L127 EN**: Closes the current lexical scope or compound statement.
  **L127 CN**: 结束当前词法作用域或复合语句块。
- **L128 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L128 CN**: 开始 `if` 控制流语句并计算其条件。
- **L129 EN**: Comment explains nearby logic, constraints, or intent: `Insert an artificial "read" reference to the variable written`.
  **L129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert an artificial "read" reference to the variable written`。
- **L130 EN**: Continues the surrounding expression or declaration: `above, to ensure the compiler does not schedule the computation`.
  **L130 CN**: 继续构造周围的表达式或声明：`above, to ensure the compiler does not schedule the computation`。
- **L131 EN**: Continues the surrounding expression or declaration: `of the value after the manipulation of the FPSCR, below.`.
  **L131 CN**: 继续构造周围的表达式或声明：`of the value after the manipulation of the FPSCR, below.`。
- **L132 EN**: Continues the surrounding expression or declaration: `This can be removed if and when GCC PR102783 is fixed.`.
  **L132 CN**: 继续构造周围的表达式或声明：`This can be removed if and when GCC PR102783 is fixed.`。
- **L133 EN**: Separator comment used for visual grouping.
  **L133 CN**: 用于视觉分组的分隔注释。
- **L134 EN**: Executes a call or declaration centered on `__asm__`.
  **L134 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L135 EN**: Comment explains nearby logic, constraints, or intent: `Restore enabled exceptions.`.
  **L135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Restore enabled exceptions.`。
- **L136 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR9`.
  **L136 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR9`。
- **L137 EN**: Executes a call or declaration centered on `__builtin_ppc_mffsl`.
  **L137 CN**: 执行以 `__builtin_ppc_mffsl` 为核心的调用或声明。
- **L138 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L138 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L139 EN**: Executes a call or declaration centered on `__builtin_ppc_mffs`.
  **L139 CN**: 执行以 `__builtin_ppc_mffs` 为核心的调用或声明。
- **L140 EN**: Adds a standalone statement or declaration: `__fpscr_save.__fpscr &= 0x70007f0ffL;`.
  **L140 CN**: 添加一条独立语句或声明：`__fpscr_save.__fpscr &= 0x70007f0ffL;`。

### Lines 141-160

````c
#endif
    __fpscr_save.__fpscr |= __enables_save.__fpscr;
    __builtin_ppc_mtfsf(0b00000011, __fpscr_save.__fr);
  }
  return (__m128d)__r;
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_round_sd(__m128d __A, __m128d __B, int __rounding) {
  __B = _mm_round_pd(__B, __rounding);
  __v2df __r = {((__v2df)__B)[0], ((__v2df)__A)[1]};
  return (__m128d)__r;
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_round_ps(__m128 __A, int __rounding) {
  __v4sf __r;
  union {
````
- **L141 EN**: Closes the current preprocessor conditional block.
  **L141 CN**: 结束当前预处理条件块。
- **L142 EN**: Adds a standalone statement or declaration: `__fpscr_save.__fpscr |= __enables_save.__fpscr;`.
  **L142 CN**: 添加一条独立语句或声明：`__fpscr_save.__fpscr |= __enables_save.__fpscr;`。
- **L143 EN**: Executes a call or declaration centered on `__builtin_ppc_mtfsf`.
  **L143 CN**: 执行以 `__builtin_ppc_mtfsf` 为核心的调用或声明。
- **L144 EN**: Closes the current lexical scope or compound statement.
  **L144 CN**: 结束当前词法作用域或复合语句块。
- **L145 EN**: Returns from the current function with `(__m128d)__r`.
  **L145 CN**: 以 `(__m128d)__r` 从当前函数返回。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L148 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L149 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L149 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L150 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_round_sd(__m128d __A, __m128d __B, int __rounding) {`.
  **L150 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_round_sd(__m128d __A, __m128d __B, int __rounding) {`。
- **L151 EN**: Executes a call or declaration centered on `_mm_round_pd`.
  **L151 CN**: 执行以 `_mm_round_pd` 为核心的调用或声明。
- **L152 EN**: Initializes variable `__r` from the expression on the right-hand side.
  **L152 CN**: 使用右侧表达式初始化变量 `__r`。
- **L153 EN**: Returns from the current function with `(__m128d)__r`.
  **L153 CN**: 以 `(__m128d)__r` 从当前函数返回。
- **L154 EN**: Closes the current lexical scope or compound statement.
  **L154 CN**: 结束当前词法作用域或复合语句块。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L156 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L156 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L157 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L157 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L158 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_round_ps(__m128 __A, int __rounding) {`.
  **L158 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_round_ps(__m128 __A, int __rounding) {`。
- **L159 EN**: Adds a standalone statement or declaration: `__v4sf __r;`.
  **L159 CN**: 添加一条独立语句或声明：`__v4sf __r;`。
- **L160 EN**: Declares union `union`.
  **L160 CN**: 声明 union `union`。

### Lines 161-180

````c
    double __fr;
    long long __fpscr;
  } __enables_save, __fpscr_save;

  if (__rounding & _MM_FROUND_NO_EXC) {
    /* Save enabled exceptions, disable all exceptions,
       and preserve the rounding mode.  */
#ifdef _ARCH_PWR9
    __asm__("mffsce %0" : "=f"(__fpscr_save.__fr));
    __enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;
#else
    __fpscr_save.__fr = __builtin_ppc_mffs();
    __enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;
    __fpscr_save.__fpscr &= ~0xf8;
    __builtin_ppc_mtfsf(0b00000011, __fpscr_save.__fr);
#endif
    /* Insert an artificial "read/write" reference to the variable
       read below, to ensure the compiler does not schedule
       a read/use of the variable before the FPSCR is modified, above.
       This can be removed if and when GCC PR102783 is fixed.
````
- **L161 EN**: Adds a standalone statement or declaration: `double __fr;`.
  **L161 CN**: 添加一条独立语句或声明：`double __fr;`。
- **L162 EN**: Adds a standalone statement or declaration: `long long __fpscr;`.
  **L162 CN**: 添加一条独立语句或声明：`long long __fpscr;`。
- **L163 EN**: Adds a standalone statement or declaration: `} __enables_save, __fpscr_save;`.
  **L163 CN**: 添加一条独立语句或声明：`} __enables_save, __fpscr_save;`。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L165 CN**: 开始 `if` 控制流语句并计算其条件。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `Save enabled exceptions, disable all exceptions,`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Save enabled exceptions, disable all exceptions,`。
- **L167 EN**: Continues the surrounding expression or declaration: `and preserve the rounding mode.  */`.
  **L167 CN**: 继续构造周围的表达式或声明：`and preserve the rounding mode.  */`。
- **L168 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR9`.
  **L168 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR9`。
- **L169 EN**: Executes a call or declaration centered on `__asm__`.
  **L169 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L170 EN**: Adds a standalone statement or declaration: `__enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;`.
  **L170 CN**: 添加一条独立语句或声明：`__enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;`。
- **L171 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L171 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L172 EN**: Executes a call or declaration centered on `__builtin_ppc_mffs`.
  **L172 CN**: 执行以 `__builtin_ppc_mffs` 为核心的调用或声明。
- **L173 EN**: Adds a standalone statement or declaration: `__enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;`.
  **L173 CN**: 添加一条独立语句或声明：`__enables_save.__fpscr = __fpscr_save.__fpscr & 0xf8;`。
- **L174 EN**: Adds a standalone statement or declaration: `__fpscr_save.__fpscr &= ~0xf8;`.
  **L174 CN**: 添加一条独立语句或声明：`__fpscr_save.__fpscr &= ~0xf8;`。
- **L175 EN**: Executes a call or declaration centered on `__builtin_ppc_mtfsf`.
  **L175 CN**: 执行以 `__builtin_ppc_mtfsf` 为核心的调用或声明。
- **L176 EN**: Closes the current preprocessor conditional block.
  **L176 CN**: 结束当前预处理条件块。
- **L177 EN**: Comment explains nearby logic, constraints, or intent: `Insert an artificial "read/write" reference to the variable`.
  **L177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert an artificial "read/write" reference to the variable`。
- **L178 EN**: Continues the surrounding expression or declaration: `read below, to ensure the compiler does not schedule`.
  **L178 CN**: 继续构造周围的表达式或声明：`read below, to ensure the compiler does not schedule`。
- **L179 EN**: Continues the surrounding expression or declaration: `a read/use of the variable before the FPSCR is modified, above.`.
  **L179 CN**: 继续构造周围的表达式或声明：`a read/use of the variable before the FPSCR is modified, above.`。
- **L180 EN**: Continues the surrounding expression or declaration: `This can be removed if and when GCC PR102783 is fixed.`.
  **L180 CN**: 继续构造周围的表达式或声明：`This can be removed if and when GCC PR102783 is fixed.`。

### Lines 181-200

````c
     */
    __asm__("" : "+wa"(__A));
  }

  switch (__rounding) {
  case _MM_FROUND_TO_NEAREST_INT:
#ifdef _ARCH_PWR9
    __fpscr_save.__fr = __builtin_ppc_mffsl();
#else
    __fpscr_save.__fr = __builtin_ppc_mffs();
    __fpscr_save.__fpscr &= 0x70007f0ffL;
#endif
    __attribute__((fallthrough));
  case _MM_FROUND_TO_NEAREST_INT | _MM_FROUND_NO_EXC:
    __builtin_ppc_set_fpscr_rn(0b00);
    /* Insert an artificial "read/write" reference to the variable
       read below, to ensure the compiler does not schedule
       a read/use of the variable before the FPSCR is modified, above.
       This can be removed if and when GCC PR102783 is fixed.
     */
````
- **L181 EN**: Separator comment used for visual grouping.
  **L181 CN**: 用于视觉分组的分隔注释。
- **L182 EN**: Executes a call or declaration centered on `__asm__`.
  **L182 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L183 EN**: Closes the current lexical scope or compound statement.
  **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L185 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L185 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L186 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_NEAREST_INT:`.
  **L186 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_NEAREST_INT:`。
- **L187 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR9`.
  **L187 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR9`。
- **L188 EN**: Executes a call or declaration centered on `__builtin_ppc_mffsl`.
  **L188 CN**: 执行以 `__builtin_ppc_mffsl` 为核心的调用或声明。
- **L189 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L189 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L190 EN**: Executes a call or declaration centered on `__builtin_ppc_mffs`.
  **L190 CN**: 执行以 `__builtin_ppc_mffs` 为核心的调用或声明。
- **L191 EN**: Adds a standalone statement or declaration: `__fpscr_save.__fpscr &= 0x70007f0ffL;`.
  **L191 CN**: 添加一条独立语句或声明：`__fpscr_save.__fpscr &= 0x70007f0ffL;`。
- **L192 EN**: Closes the current preprocessor conditional block.
  **L192 CN**: 结束当前预处理条件块。
- **L193 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((fallthrough));`.
  **L193 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((fallthrough));`。
- **L194 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_NEAREST_INT | _MM_FROUND_NO_EXC:`.
  **L194 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_NEAREST_INT | _MM_FROUND_NO_EXC:`。
- **L195 EN**: Executes a call or declaration centered on `__builtin_ppc_set_fpscr_rn`.
  **L195 CN**: 执行以 `__builtin_ppc_set_fpscr_rn` 为核心的调用或声明。
- **L196 EN**: Comment explains nearby logic, constraints, or intent: `Insert an artificial "read/write" reference to the variable`.
  **L196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert an artificial "read/write" reference to the variable`。
- **L197 EN**: Continues the surrounding expression or declaration: `read below, to ensure the compiler does not schedule`.
  **L197 CN**: 继续构造周围的表达式或声明：`read below, to ensure the compiler does not schedule`。
- **L198 EN**: Continues the surrounding expression or declaration: `a read/use of the variable before the FPSCR is modified, above.`.
  **L198 CN**: 继续构造周围的表达式或声明：`a read/use of the variable before the FPSCR is modified, above.`。
- **L199 EN**: Continues the surrounding expression or declaration: `This can be removed if and when GCC PR102783 is fixed.`.
  **L199 CN**: 继续构造周围的表达式或声明：`This can be removed if and when GCC PR102783 is fixed.`。
- **L200 EN**: Separator comment used for visual grouping.
  **L200 CN**: 用于视觉分组的分隔注释。

### Lines 201-220

````c
    __asm__("" : "+wa"(__A));

    __r = vec_rint((__v4sf)__A);

    /* Insert an artificial "read" reference to the variable written
       above, to ensure the compiler does not schedule the computation
       of the value after the manipulation of the FPSCR, below.
       This can be removed if and when GCC PR102783 is fixed.
     */
    __asm__("" : : "wa"(__r));
    __builtin_ppc_set_fpscr_rn(__fpscr_save.__fpscr);
    break;
  case _MM_FROUND_TO_NEG_INF:
  case _MM_FROUND_TO_NEG_INF | _MM_FROUND_NO_EXC:
    __r = vec_floor((__v4sf)__A);
    break;
  case _MM_FROUND_TO_POS_INF:
  case _MM_FROUND_TO_POS_INF | _MM_FROUND_NO_EXC:
    __r = vec_ceil((__v4sf)__A);
    break;
````
- **L201 EN**: Executes a call or declaration centered on `__asm__`.
  **L201 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L203 EN**: Executes a call or declaration centered on `vec_rint`.
  **L203 CN**: 执行以 `vec_rint` 为核心的调用或声明。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L205 EN**: Comment explains nearby logic, constraints, or intent: `Insert an artificial "read" reference to the variable written`.
  **L205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert an artificial "read" reference to the variable written`。
- **L206 EN**: Continues the surrounding expression or declaration: `above, to ensure the compiler does not schedule the computation`.
  **L206 CN**: 继续构造周围的表达式或声明：`above, to ensure the compiler does not schedule the computation`。
- **L207 EN**: Continues the surrounding expression or declaration: `of the value after the manipulation of the FPSCR, below.`.
  **L207 CN**: 继续构造周围的表达式或声明：`of the value after the manipulation of the FPSCR, below.`。
- **L208 EN**: Continues the surrounding expression or declaration: `This can be removed if and when GCC PR102783 is fixed.`.
  **L208 CN**: 继续构造周围的表达式或声明：`This can be removed if and when GCC PR102783 is fixed.`。
- **L209 EN**: Separator comment used for visual grouping.
  **L209 CN**: 用于视觉分组的分隔注释。
- **L210 EN**: Executes a call or declaration centered on `__asm__`.
  **L210 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L211 EN**: Executes a call or declaration centered on `__builtin_ppc_set_fpscr_rn`.
  **L211 CN**: 执行以 `__builtin_ppc_set_fpscr_rn` 为核心的调用或声明。
- **L212 EN**: Exits the nearest loop or switch statement.
  **L212 CN**: 退出最近的循环或 `switch` 语句。
- **L213 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_NEG_INF:`.
  **L213 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_NEG_INF:`。
- **L214 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_NEG_INF | _MM_FROUND_NO_EXC:`.
  **L214 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_NEG_INF | _MM_FROUND_NO_EXC:`。
- **L215 EN**: Executes a call or declaration centered on `vec_floor`.
  **L215 CN**: 执行以 `vec_floor` 为核心的调用或声明。
- **L216 EN**: Exits the nearest loop or switch statement.
  **L216 CN**: 退出最近的循环或 `switch` 语句。
- **L217 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_POS_INF:`.
  **L217 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_POS_INF:`。
- **L218 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_POS_INF | _MM_FROUND_NO_EXC:`.
  **L218 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_POS_INF | _MM_FROUND_NO_EXC:`。
- **L219 EN**: Executes a call or declaration centered on `vec_ceil`.
  **L219 CN**: 执行以 `vec_ceil` 为核心的调用或声明。
- **L220 EN**: Exits the nearest loop or switch statement.
  **L220 CN**: 退出最近的循环或 `switch` 语句。

### Lines 221-240

````c
  case _MM_FROUND_TO_ZERO:
  case _MM_FROUND_TO_ZERO | _MM_FROUND_NO_EXC:
    __r = vec_trunc((__v4sf)__A);
    break;
  case _MM_FROUND_CUR_DIRECTION:
    __r = vec_rint((__v4sf)__A);
    break;
  }
  if (__rounding & _MM_FROUND_NO_EXC) {
    /* Insert an artificial "read" reference to the variable written
       above, to ensure the compiler does not schedule the computation
       of the value after the manipulation of the FPSCR, below.
       This can be removed if and when GCC PR102783 is fixed.
     */
    __asm__("" : : "wa"(__r));
    /* Restore enabled exceptions.  */
#ifdef _ARCH_PWR9
    __fpscr_save.__fr = __builtin_ppc_mffsl();
#else
    __fpscr_save.__fr = __builtin_ppc_mffs();
````
- **L221 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_ZERO:`.
  **L221 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_ZERO:`。
- **L222 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_TO_ZERO | _MM_FROUND_NO_EXC:`.
  **L222 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_TO_ZERO | _MM_FROUND_NO_EXC:`。
- **L223 EN**: Executes a call or declaration centered on `vec_trunc`.
  **L223 CN**: 执行以 `vec_trunc` 为核心的调用或声明。
- **L224 EN**: Exits the nearest loop or switch statement.
  **L224 CN**: 退出最近的循环或 `switch` 语句。
- **L225 EN**: Introduces a `switch` dispatch label: `case _MM_FROUND_CUR_DIRECTION:`.
  **L225 CN**: 引入一个 `switch` 分发标签：`case _MM_FROUND_CUR_DIRECTION:`。
- **L226 EN**: Executes a call or declaration centered on `vec_rint`.
  **L226 CN**: 执行以 `vec_rint` 为核心的调用或声明。
- **L227 EN**: Exits the nearest loop or switch statement.
  **L227 CN**: 退出最近的循环或 `switch` 语句。
- **L228 EN**: Closes the current lexical scope or compound statement.
  **L228 CN**: 结束当前词法作用域或复合语句块。
- **L229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L230 EN**: Comment explains nearby logic, constraints, or intent: `Insert an artificial "read" reference to the variable written`.
  **L230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Insert an artificial "read" reference to the variable written`。
- **L231 EN**: Continues the surrounding expression or declaration: `above, to ensure the compiler does not schedule the computation`.
  **L231 CN**: 继续构造周围的表达式或声明：`above, to ensure the compiler does not schedule the computation`。
- **L232 EN**: Continues the surrounding expression or declaration: `of the value after the manipulation of the FPSCR, below.`.
  **L232 CN**: 继续构造周围的表达式或声明：`of the value after the manipulation of the FPSCR, below.`。
- **L233 EN**: Continues the surrounding expression or declaration: `This can be removed if and when GCC PR102783 is fixed.`.
  **L233 CN**: 继续构造周围的表达式或声明：`This can be removed if and when GCC PR102783 is fixed.`。
- **L234 EN**: Separator comment used for visual grouping.
  **L234 CN**: 用于视觉分组的分隔注释。
- **L235 EN**: Executes a call or declaration centered on `__asm__`.
  **L235 CN**: 执行以 `__asm__` 为核心的调用或声明。
- **L236 EN**: Comment explains nearby logic, constraints, or intent: `Restore enabled exceptions.`.
  **L236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Restore enabled exceptions.`。
- **L237 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR9`.
  **L237 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR9`。
- **L238 EN**: Executes a call or declaration centered on `__builtin_ppc_mffsl`.
  **L238 CN**: 执行以 `__builtin_ppc_mffsl` 为核心的调用或声明。
- **L239 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L239 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L240 EN**: Executes a call or declaration centered on `__builtin_ppc_mffs`.
  **L240 CN**: 执行以 `__builtin_ppc_mffs` 为核心的调用或声明。

### Lines 241-260

````c
    __fpscr_save.__fpscr &= 0x70007f0ffL;
#endif
    __fpscr_save.__fpscr |= __enables_save.__fpscr;
    __builtin_ppc_mtfsf(0b00000011, __fpscr_save.__fr);
  }
  return (__m128)__r;
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_round_ss(__m128 __A, __m128 __B, int __rounding) {
  __B = _mm_round_ps(__B, __rounding);
  __v4sf __r = (__v4sf)__A;
  __r[0] = ((__v4sf)__B)[0];
  return (__m128)__r;
}

#define _mm_ceil_pd(V) _mm_round_pd((V), _MM_FROUND_CEIL)
#define _mm_ceil_sd(D, V) _mm_round_sd((D), (V), _MM_FROUND_CEIL)

````
- **L241 EN**: Adds a standalone statement or declaration: `__fpscr_save.__fpscr &= 0x70007f0ffL;`.
  **L241 CN**: 添加一条独立语句或声明：`__fpscr_save.__fpscr &= 0x70007f0ffL;`。
- **L242 EN**: Closes the current preprocessor conditional block.
  **L242 CN**: 结束当前预处理条件块。
- **L243 EN**: Adds a standalone statement or declaration: `__fpscr_save.__fpscr |= __enables_save.__fpscr;`.
  **L243 CN**: 添加一条独立语句或声明：`__fpscr_save.__fpscr |= __enables_save.__fpscr;`。
- **L244 EN**: Executes a call or declaration centered on `__builtin_ppc_mtfsf`.
  **L244 CN**: 执行以 `__builtin_ppc_mtfsf` 为核心的调用或声明。
- **L245 EN**: Closes the current lexical scope or compound statement.
  **L245 CN**: 结束当前词法作用域或复合语句块。
- **L246 EN**: Returns from the current function with `(__m128)__r`.
  **L246 CN**: 以 `(__m128)__r` 从当前函数返回。
- **L247 EN**: Closes the current lexical scope or compound statement.
  **L247 CN**: 结束当前词法作用域或复合语句块。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L249 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L249 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L250 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L250 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L251 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_round_ss(__m128 __A, __m128 __B, int __rounding) {`.
  **L251 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_round_ss(__m128 __A, __m128 __B, int __rounding) {`。
- **L252 EN**: Executes a call or declaration centered on `_mm_round_ps`.
  **L252 CN**: 执行以 `_mm_round_ps` 为核心的调用或声明。
- **L253 EN**: Initializes variable `__r` from the expression on the right-hand side.
  **L253 CN**: 使用右侧表达式初始化变量 `__r`。
- **L254 EN**: Executes a call or declaration centered on `=`.
  **L254 CN**: 执行以 `=` 为核心的调用或声明。
- **L255 EN**: Returns from the current function with `(__m128)__r`.
  **L255 CN**: 以 `(__m128)__r` 从当前函数返回。
- **L256 EN**: Closes the current lexical scope or compound statement.
  **L256 CN**: 结束当前词法作用域或复合语句块。
- **L257 EN**: Blank line separating nearby declarations or logic blocks.
  **L257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L258 EN**: Defines macro `_mm_ceil_pd(V)` for conditional compilation, shorthand, or API generation.
  **L258 CN**: 定义宏 `_mm_ceil_pd(V)`，用于条件编译、简写或 API 生成。
- **L259 EN**: Defines macro `_mm_ceil_sd(D, V)` for conditional compilation, shorthand, or API generation.
  **L259 CN**: 定义宏 `_mm_ceil_sd(D, V)`，用于条件编译、简写或 API 生成。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 261-280

````c
#define _mm_floor_pd(V) _mm_round_pd((V), _MM_FROUND_FLOOR)
#define _mm_floor_sd(D, V) _mm_round_sd((D), (V), _MM_FROUND_FLOOR)

#define _mm_ceil_ps(V) _mm_round_ps((V), _MM_FROUND_CEIL)
#define _mm_ceil_ss(D, V) _mm_round_ss((D), (V), _MM_FROUND_CEIL)

#define _mm_floor_ps(V) _mm_round_ps((V), _MM_FROUND_FLOOR)
#define _mm_floor_ss(D, V) _mm_round_ss((D), (V), _MM_FROUND_FLOOR)

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_insert_epi8(__m128i const __A, int const __D, int const __N) {
  __v16qi __result = (__v16qi)__A;

  __result[__N & 0xf] = __D;

  return (__m128i)__result;
}

extern __inline __m128i
````
- **L261 EN**: Defines macro `_mm_floor_pd(V)` for conditional compilation, shorthand, or API generation.
  **L261 CN**: 定义宏 `_mm_floor_pd(V)`，用于条件编译、简写或 API 生成。
- **L262 EN**: Defines macro `_mm_floor_sd(D, V)` for conditional compilation, shorthand, or API generation.
  **L262 CN**: 定义宏 `_mm_floor_sd(D, V)`，用于条件编译、简写或 API 生成。
- **L263 EN**: Blank line separating nearby declarations or logic blocks.
  **L263 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L264 EN**: Defines macro `_mm_ceil_ps(V)` for conditional compilation, shorthand, or API generation.
  **L264 CN**: 定义宏 `_mm_ceil_ps(V)`，用于条件编译、简写或 API 生成。
- **L265 EN**: Defines macro `_mm_ceil_ss(D, V)` for conditional compilation, shorthand, or API generation.
  **L265 CN**: 定义宏 `_mm_ceil_ss(D, V)`，用于条件编译、简写或 API 生成。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L267 EN**: Defines macro `_mm_floor_ps(V)` for conditional compilation, shorthand, or API generation.
  **L267 CN**: 定义宏 `_mm_floor_ps(V)`，用于条件编译、简写或 API 生成。
- **L268 EN**: Defines macro `_mm_floor_ss(D, V)` for conditional compilation, shorthand, or API generation.
  **L268 CN**: 定义宏 `_mm_floor_ss(D, V)`，用于条件编译、简写或 API 生成。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L270 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L270 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L271 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L271 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L272 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_insert_epi8(__m128i const __A, int const __D, int const __N) {`.
  **L272 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_insert_epi8(__m128i const __A, int const __D, int const __N) {`。
- **L273 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L273 CN**: 使用右侧表达式初始化变量 `__result`。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L275 EN**: Adds a standalone statement or declaration: `__result[__N & 0xf] = __D;`.
  **L275 CN**: 添加一条独立语句或声明：`__result[__N & 0xf] = __D;`。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L277 EN**: Returns from the current function with `(__m128i)__result`.
  **L277 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L278 EN**: Closes the current lexical scope or compound statement.
  **L278 CN**: 结束当前词法作用域或复合语句块。
- **L279 EN**: Blank line separating nearby declarations or logic blocks.
  **L279 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L280 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L280 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。

### Lines 281-300

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_insert_epi32(__m128i const __A, int const __D, int const __N) {
  __v4si __result = (__v4si)__A;

  __result[__N & 3] = __D;

  return (__m128i)__result;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_insert_epi64(__m128i const __A, long long const __D, int const __N) {
  __v2di __result = (__v2di)__A;

  __result[__N & 1] = __D;

  return (__m128i)__result;
}

extern __inline int
````
- **L281 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L281 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L282 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_insert_epi32(__m128i const __A, int const __D, int const __N) {`.
  **L282 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_insert_epi32(__m128i const __A, int const __D, int const __N) {`。
- **L283 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L283 CN**: 使用右侧表达式初始化变量 `__result`。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L285 EN**: Adds a standalone statement or declaration: `__result[__N & 3] = __D;`.
  **L285 CN**: 添加一条独立语句或声明：`__result[__N & 3] = __D;`。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L287 EN**: Returns from the current function with `(__m128i)__result`.
  **L287 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L288 EN**: Closes the current lexical scope or compound statement.
  **L288 CN**: 结束当前词法作用域或复合语句块。
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L290 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L290 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L291 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L291 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L292 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_insert_epi64(__m128i const __A, long long const __D, int const __N) {`.
  **L292 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_insert_epi64(__m128i const __A, long long const __D, int const __N) {`。
- **L293 EN**: Initializes variable `__result` from the expression on the right-hand side.
  **L293 CN**: 使用右侧表达式初始化变量 `__result`。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L295 EN**: Adds a standalone statement or declaration: `__result[__N & 1] = __D;`.
  **L295 CN**: 添加一条独立语句或声明：`__result[__N & 1] = __D;`。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Returns from the current function with `(__m128i)__result`.
  **L297 CN**: 以 `(__m128i)__result` 从当前函数返回。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L300 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L300 CN**: 继续构造周围的表达式或声明：`extern __inline int`。

### Lines 301-320

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_extract_epi8(__m128i __X, const int __N) {
  return (unsigned char)((__v16qi)__X)[__N & 15];
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_extract_epi32(__m128i __X, const int __N) {
  return ((__v4si)__X)[__N & 3];
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_extract_epi64(__m128i __X, const int __N) {
  return ((__v2di)__X)[__N & 1];
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_extract_ps(__m128 __X, const int __N) {
````
- **L301 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L301 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L302 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_extract_epi8(__m128i __X, const int __N) {`.
  **L302 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_extract_epi8(__m128i __X, const int __N) {`。
- **L303 EN**: Returns from the current function with `(unsigned char)((__v16qi)__X)[__N & 15]`.
  **L303 CN**: 以 `(unsigned char)((__v16qi)__X)[__N & 15]` 从当前函数返回。
- **L304 EN**: Closes the current lexical scope or compound statement.
  **L304 CN**: 结束当前词法作用域或复合语句块。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L306 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L306 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L307 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L307 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L308 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_extract_epi32(__m128i __X, const int __N) {`.
  **L308 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_extract_epi32(__m128i __X, const int __N) {`。
- **L309 EN**: Returns from the current function with `((__v4si)__X)[__N & 3]`.
  **L309 CN**: 以 `((__v4si)__X)[__N & 3]` 从当前函数返回。
- **L310 EN**: Closes the current lexical scope or compound statement.
  **L310 CN**: 结束当前词法作用域或复合语句块。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L312 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L312 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L313 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L313 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L314 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_extract_epi64(__m128i __X, const int __N) {`.
  **L314 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_extract_epi64(__m128i __X, const int __N) {`。
- **L315 EN**: Returns from the current function with `((__v2di)__X)[__N & 1]`.
  **L315 CN**: 以 `((__v2di)__X)[__N & 1]` 从当前函数返回。
- **L316 EN**: Closes the current lexical scope or compound statement.
  **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Blank line separating nearby declarations or logic blocks.
  **L317 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L318 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L318 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L319 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L319 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L320 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_extract_ps(__m128 __X, const int __N) {`.
  **L320 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_extract_ps(__m128 __X, const int __N) {`。

### Lines 321-340

````c
  return ((__v4si)__X)[__N & 3];
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_blend_epi16(__m128i __A, __m128i __B, const int __imm8) {
  __v16qu __charmask = vec_splats((unsigned char)__imm8);
  __charmask = vec_gb(__charmask);
  __v8hu __shortmask = (__v8hu)vec_unpackh((__v16qi)__charmask);
#ifdef __BIG_ENDIAN__
  __shortmask = vec_reve(__shortmask);
#endif
  return (__m128i)vec_sel((__v8hu)__A, (__v8hu)__B, __shortmask);
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_blendv_epi8(__m128i __A, __m128i __B, __m128i __mask) {
````
- **L321 EN**: Returns from the current function with `((__v4si)__X)[__N & 3]`.
  **L321 CN**: 以 `((__v4si)__X)[__N & 3]` 从当前函数返回。
- **L322 EN**: Closes the current lexical scope or compound statement.
  **L322 CN**: 结束当前词法作用域或复合语句块。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L324 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L324 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L325 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L325 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L326 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L326 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L327 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blend_epi16(__m128i __A, __m128i __B, const int __imm8) {`.
  **L327 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blend_epi16(__m128i __A, __m128i __B, const int __imm8) {`。
- **L328 EN**: Initializes variable `__charmask` from the expression on the right-hand side.
  **L328 CN**: 使用右侧表达式初始化变量 `__charmask`。
- **L329 EN**: Executes a call or declaration centered on `vec_gb`.
  **L329 CN**: 执行以 `vec_gb` 为核心的调用或声明。
- **L330 EN**: Initializes variable `__shortmask` from the expression on the right-hand side.
  **L330 CN**: 使用右侧表达式初始化变量 `__shortmask`。
- **L331 EN**: Starts a preprocessor conditional block: `#ifdef __BIG_ENDIAN__`.
  **L331 CN**: 开始一个预处理条件块：`#ifdef __BIG_ENDIAN__`。
- **L332 EN**: Executes a call or declaration centered on `vec_reve`.
  **L332 CN**: 执行以 `vec_reve` 为核心的调用或声明。
- **L333 EN**: Closes the current preprocessor conditional block.
  **L333 CN**: 结束当前预处理条件块。
- **L334 EN**: Returns from the current function with `(__m128i)vec_sel((__v8hu)__A, (__v8hu)__B, __shortmask)`.
  **L334 CN**: 以 `(__m128i)vec_sel((__v8hu)__A, (__v8hu)__B, __shortmask)` 从当前函数返回。
- **L335 EN**: Closes the current lexical scope or compound statement.
  **L335 CN**: 结束当前词法作用域或复合语句块。
- **L336 EN**: Closes the current preprocessor conditional block.
  **L336 CN**: 结束当前预处理条件块。
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L338 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L338 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L339 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L339 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L340 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blendv_epi8(__m128i __A, __m128i __B, __m128i __mask) {`.
  **L340 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blendv_epi8(__m128i __A, __m128i __B, __m128i __mask) {`。

### Lines 341-360

````c
#ifdef _ARCH_PWR10
  return (__m128i)vec_blendv((__v16qi)__A, (__v16qi)__B, (__v16qu)__mask);
#else
  const __v16qu __seven = vec_splats((unsigned char)0x07);
  __v16qu __lmask = vec_sra((__v16qu)__mask, __seven);
  return (__m128i)vec_sel((__v16qi)__A, (__v16qi)__B, __lmask);
#endif
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_blend_ps(__m128 __A, __m128 __B, const int __imm8) {
  __v16qu __pcv[] = {
      {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},
      {16, 17, 18, 19, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},
      {0, 1, 2, 3, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},
      {16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},
      {0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 12, 13, 14, 15},
      {16, 17, 18, 19, 4, 5, 6, 7, 24, 25, 26, 27, 12, 13, 14, 15},
      {0, 1, 2, 3, 20, 21, 22, 23, 24, 25, 26, 27, 12, 13, 14, 15},
````
- **L341 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR10`.
  **L341 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR10`。
- **L342 EN**: Returns from the current function with `(__m128i)vec_blendv((__v16qi)__A, (__v16qi)__B, (__v16qu)__mask)`.
  **L342 CN**: 以 `(__m128i)vec_blendv((__v16qi)__A, (__v16qi)__B, (__v16qu)__mask)` 从当前函数返回。
- **L343 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L343 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L344 EN**: Initializes variable `__seven` from the expression on the right-hand side.
  **L344 CN**: 使用右侧表达式初始化变量 `__seven`。
- **L345 EN**: Initializes variable `__lmask` from the expression on the right-hand side.
  **L345 CN**: 使用右侧表达式初始化变量 `__lmask`。
- **L346 EN**: Returns from the current function with `(__m128i)vec_sel((__v16qi)__A, (__v16qi)__B, __lmask)`.
  **L346 CN**: 以 `(__m128i)vec_sel((__v16qi)__A, (__v16qi)__B, __lmask)` 从当前函数返回。
- **L347 EN**: Closes the current preprocessor conditional block.
  **L347 CN**: 结束当前预处理条件块。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L350 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L350 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L351 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L351 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L352 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blend_ps(__m128 __A, __m128 __B, const int __imm8) {`.
  **L352 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blend_ps(__m128 __A, __m128 __B, const int __imm8) {`。
- **L353 EN**: Continues the surrounding expression or declaration: `__v16qu __pcv[] = {`.
  **L353 CN**: 继续构造周围的表达式或声明：`__v16qu __pcv[] = {`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},`。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 12, 13, 14, 15},`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 12, 13, 14, 15},`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 4, 5, 6, 7, 24, 25, 26, 27, 12, 13, 14, 15},`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 4, 5, 6, 7, 24, 25, 26, 27, 12, 13, 14, 15},`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 20, 21, 22, 23, 24, 25, 26, 27, 12, 13, 14, 15},`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 20, 21, 22, 23, 24, 25, 26, 27, 12, 13, 14, 15},`。

### Lines 361-380

````c
      {16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 12, 13, 14, 15},
      {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 28, 29, 30, 31},
      {16, 17, 18, 19, 4, 5, 6, 7, 8, 9, 10, 11, 28, 29, 30, 31},
      {0, 1, 2, 3, 20, 21, 22, 23, 8, 9, 10, 11, 28, 29, 30, 31},
      {16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 28, 29, 30, 31},
      {0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},
      {16, 17, 18, 19, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},
      {0, 1, 2, 3, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31},
      {16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31},
  };
  __v16qu __r = vec_perm((__v16qu)__A, (__v16qu)__B, __pcv[__imm8]);
  return (__m128)__r;
}

extern __inline __m128
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_blendv_ps(__m128 __A, __m128 __B, __m128 __mask) {
#ifdef _ARCH_PWR10
  return (__m128)vec_blendv((__v4sf)__A, (__v4sf)__B, (__v4su)__mask);
#else
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 12, 13, 14, 15},`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 12, 13, 14, 15},`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 28, 29, 30, 31},`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 28, 29, 30, 31},`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 4, 5, 6, 7, 8, 9, 10, 11, 28, 29, 30, 31},`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 4, 5, 6, 7, 8, 9, 10, 11, 28, 29, 30, 31},`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 20, 21, 22, 23, 8, 9, 10, 11, 28, 29, 30, 31},`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 20, 21, 22, 23, 8, 9, 10, 11, 28, 29, 30, 31},`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 28, 29, 30, 31},`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 28, 29, 30, 31},`。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31},`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31},`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31},`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31},`。
- **L370 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L370 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L371 EN**: Initializes variable `__r` from the expression on the right-hand side.
  **L371 CN**: 使用右侧表达式初始化变量 `__r`。
- **L372 EN**: Returns from the current function with `(__m128)__r`.
  **L372 CN**: 以 `(__m128)__r` 从当前函数返回。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L375 EN**: Continues the surrounding expression or declaration: `extern __inline __m128`.
  **L375 CN**: 继续构造周围的表达式或声明：`extern __inline __m128`。
- **L376 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L376 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L377 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blendv_ps(__m128 __A, __m128 __B, __m128 __mask) {`.
  **L377 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blendv_ps(__m128 __A, __m128 __B, __m128 __mask) {`。
- **L378 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR10`.
  **L378 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR10`。
- **L379 EN**: Returns from the current function with `(__m128)vec_blendv((__v4sf)__A, (__v4sf)__B, (__v4su)__mask)`.
  **L379 CN**: 以 `(__m128)vec_blendv((__v4sf)__A, (__v4sf)__B, (__v4su)__mask)` 从当前函数返回。
- **L380 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L380 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 381-400

````c
  const __v4si __zero = {0};
  const __vector __bool int __boolmask = vec_cmplt((__v4si)__mask, __zero);
  return (__m128)vec_sel((__v4su)__A, (__v4su)__B, (__v4su)__boolmask);
#endif
}

extern __inline __m128d
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_blend_pd(__m128d __A, __m128d __B, const int __imm8) {
  __v16qu __pcv[] = {
      {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},
      {16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},
      {0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},
      {16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31}};
  __v16qu __r = vec_perm((__v16qu)__A, (__v16qu)__B, __pcv[__imm8]);
  return (__m128d)__r;
}

#ifdef _ARCH_PWR8
extern __inline __m128d
````
- **L381 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L381 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L382 EN**: Initializes variable `__boolmask` from the expression on the right-hand side.
  **L382 CN**: 使用右侧表达式初始化变量 `__boolmask`。
- **L383 EN**: Returns from the current function with `(__m128)vec_sel((__v4su)__A, (__v4su)__B, (__v4su)__boolmask)`.
  **L383 CN**: 以 `(__m128)vec_sel((__v4su)__A, (__v4su)__B, (__v4su)__boolmask)` 从当前函数返回。
- **L384 EN**: Closes the current preprocessor conditional block.
  **L384 CN**: 结束当前预处理条件块。
- **L385 EN**: Closes the current lexical scope or compound statement.
  **L385 CN**: 结束当前词法作用域或复合语句块。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L387 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L387 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。
- **L388 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L388 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L389 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blend_pd(__m128d __A, __m128d __B, const int __imm8) {`.
  **L389 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blend_pd(__m128d __A, __m128d __B, const int __imm8) {`。
- **L390 EN**: Continues the surrounding expression or declaration: `__v16qu __pcv[] = {`.
  **L390 CN**: 继续构造周围的表达式或声明：`__v16qu __pcv[] = {`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15},`。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},`.
  **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`{16, 17, 18, 19, 20, 21, 22, 23, 8, 9, 10, 11, 12, 13, 14, 15},`。
- **L393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},`.
  **L393 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31},`。
- **L394 EN**: Adds a standalone statement or declaration: `{16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31}};`.
  **L394 CN**: 添加一条独立语句或声明：`{16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31}};`。
- **L395 EN**: Initializes variable `__r` from the expression on the right-hand side.
  **L395 CN**: 使用右侧表达式初始化变量 `__r`。
- **L396 EN**: Returns from the current function with `(__m128d)__r`.
  **L396 CN**: 以 `(__m128d)__r` 从当前函数返回。
- **L397 EN**: Closes the current lexical scope or compound statement.
  **L397 CN**: 结束当前词法作用域或复合语句块。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L399 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L399 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L400 EN**: Continues the surrounding expression or declaration: `extern __inline __m128d`.
  **L400 CN**: 继续构造周围的表达式或声明：`extern __inline __m128d`。

### Lines 401-420

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_blendv_pd(__m128d __A, __m128d __B, __m128d __mask) {
#ifdef _ARCH_PWR10
  return (__m128d)vec_blendv((__v2df)__A, (__v2df)__B, (__v2du)__mask);
#else
  const __v2di __zero = {0};
  const __vector __bool long long __boolmask =
      vec_cmplt((__v2di)__mask, __zero);
  return (__m128d)vec_sel((__v2du)__A, (__v2du)__B, (__v2du)__boolmask);
#endif
}
#endif

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_testz_si128(__m128i __A, __m128i __B) {
  /* Note: This implementation does NOT set "zero" or "carry" flags.  */
  const __v16qu __zero = {0};
  return vec_all_eq(vec_and((__v16qu)__A, (__v16qu)__B), __zero);
}
````
- **L401 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L401 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L402 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_blendv_pd(__m128d __A, __m128d __B, __m128d __mask) {`.
  **L402 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_blendv_pd(__m128d __A, __m128d __B, __m128d __mask) {`。
- **L403 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR10`.
  **L403 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR10`。
- **L404 EN**: Returns from the current function with `(__m128d)vec_blendv((__v2df)__A, (__v2df)__B, (__v2du)__mask)`.
  **L404 CN**: 以 `(__m128d)vec_blendv((__v2df)__A, (__v2df)__B, (__v2du)__mask)` 从当前函数返回。
- **L405 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L405 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L406 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L406 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L407 EN**: Continues the surrounding expression or declaration: `const __vector __bool long long __boolmask =`.
  **L407 CN**: 继续构造周围的表达式或声明：`const __vector __bool long long __boolmask =`。
- **L408 EN**: Executes a call or declaration centered on `vec_cmplt`.
  **L408 CN**: 执行以 `vec_cmplt` 为核心的调用或声明。
- **L409 EN**: Returns from the current function with `(__m128d)vec_sel((__v2du)__A, (__v2du)__B, (__v2du)__boolmask)`.
  **L409 CN**: 以 `(__m128d)vec_sel((__v2du)__A, (__v2du)__B, (__v2du)__boolmask)` 从当前函数返回。
- **L410 EN**: Closes the current preprocessor conditional block.
  **L410 CN**: 结束当前预处理条件块。
- **L411 EN**: Closes the current lexical scope or compound statement.
  **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Closes the current preprocessor conditional block.
  **L412 CN**: 结束当前预处理条件块。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L414 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L415 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L415 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L416 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_testz_si128(__m128i __A, __m128i __B) {`.
  **L416 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_testz_si128(__m128i __A, __m128i __B) {`。
- **L417 EN**: Comment highlights an implementation note: `Note: This implementation does NOT set "zero" or "carry" flags.`.
  **L417 CN**: 注释强调一条实现说明：`Note: This implementation does NOT set "zero" or "carry" flags.`。
- **L418 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L418 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L419 EN**: Returns from the current function with `vec_all_eq(vec_and((__v16qu)__A, (__v16qu)__B), __zero)`.
  **L419 CN**: 以 `vec_all_eq(vec_and((__v16qu)__A, (__v16qu)__B), __zero)` 从当前函数返回。
- **L420 EN**: Closes the current lexical scope or compound statement.
  **L420 CN**: 结束当前词法作用域或复合语句块。

### Lines 421-440

````c

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_testc_si128(__m128i __A, __m128i __B) {
  /* Note: This implementation does NOT set "zero" or "carry" flags.  */
  const __v16qu __zero = {0};
  const __v16qu __notA = vec_nor((__v16qu)__A, (__v16qu)__A);
  return vec_all_eq(vec_and((__v16qu)__notA, (__v16qu)__B), __zero);
}

extern __inline int
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_testnzc_si128(__m128i __A, __m128i __B) {
  /* Note: This implementation does NOT set "zero" or "carry" flags.  */
  return _mm_testz_si128(__A, __B) == 0 && _mm_testc_si128(__A, __B) == 0;
}

#define _mm_test_all_zeros(M, V) _mm_testz_si128((M), (V))

#define _mm_test_all_ones(V) _mm_testc_si128((V), _mm_cmpeq_epi32((V), (V)))
````
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L422 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L422 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L423 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L423 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L424 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_testc_si128(__m128i __A, __m128i __B) {`.
  **L424 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_testc_si128(__m128i __A, __m128i __B) {`。
- **L425 EN**: Comment highlights an implementation note: `Note: This implementation does NOT set "zero" or "carry" flags.`.
  **L425 CN**: 注释强调一条实现说明：`Note: This implementation does NOT set "zero" or "carry" flags.`。
- **L426 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L426 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L427 EN**: Initializes variable `__notA` from the expression on the right-hand side.
  **L427 CN**: 使用右侧表达式初始化变量 `__notA`。
- **L428 EN**: Returns from the current function with `vec_all_eq(vec_and((__v16qu)__notA, (__v16qu)__B), __zero)`.
  **L428 CN**: 以 `vec_all_eq(vec_and((__v16qu)__notA, (__v16qu)__B), __zero)` 从当前函数返回。
- **L429 EN**: Closes the current lexical scope or compound statement.
  **L429 CN**: 结束当前词法作用域或复合语句块。
- **L430 EN**: Blank line separating nearby declarations or logic blocks.
  **L430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L431 EN**: Continues the surrounding expression or declaration: `extern __inline int`.
  **L431 CN**: 继续构造周围的表达式或声明：`extern __inline int`。
- **L432 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L432 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L433 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_testnzc_si128(__m128i __A, __m128i __B) {`.
  **L433 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_testnzc_si128(__m128i __A, __m128i __B) {`。
- **L434 EN**: Comment highlights an implementation note: `Note: This implementation does NOT set "zero" or "carry" flags.`.
  **L434 CN**: 注释强调一条实现说明：`Note: This implementation does NOT set "zero" or "carry" flags.`。
- **L435 EN**: Returns from the current function with `_mm_testz_si128(__A, __B) == 0 && _mm_testc_si128(__A, __B) == 0`.
  **L435 CN**: 以 `_mm_testz_si128(__A, __B) == 0 && _mm_testc_si128(__A, __B) == 0` 从当前函数返回。
- **L436 EN**: Closes the current lexical scope or compound statement.
  **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Defines macro `_mm_test_all_zeros(M, V)` for conditional compilation, shorthand, or API generation.
  **L438 CN**: 定义宏 `_mm_test_all_zeros(M, V)`，用于条件编译、简写或 API 生成。
- **L439 EN**: Blank line separating nearby declarations or logic blocks.
  **L439 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L440 EN**: Defines macro `_mm_test_all_ones(V)` for conditional compilation, shorthand, or API generation.
  **L440 CN**: 定义宏 `_mm_test_all_ones(V)`，用于条件编译、简写或 API 生成。

### Lines 441-460

````c

#define _mm_test_mix_ones_zeros(M, V) _mm_testnzc_si128((M), (V))

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpeq_epi64(__m128i __X, __m128i __Y) {
  return (__m128i)vec_cmpeq((__v2di)__X, (__v2di)__Y);
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_epi8(__m128i __X, __m128i __Y) {
  return (__m128i)vec_min((__v16qi)__X, (__v16qi)__Y);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_epu16(__m128i __X, __m128i __Y) {
````
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L442 EN**: Defines macro `_mm_test_mix_ones_zeros(M, V)` for conditional compilation, shorthand, or API generation.
  **L442 CN**: 定义宏 `_mm_test_mix_ones_zeros(M, V)`，用于条件编译、简写或 API 生成。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L444 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L444 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L445 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L445 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L446 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L446 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L447 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpeq_epi64(__m128i __X, __m128i __Y) {`.
  **L447 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpeq_epi64(__m128i __X, __m128i __Y) {`。
- **L448 EN**: Returns from the current function with `(__m128i)vec_cmpeq((__v2di)__X, (__v2di)__Y)`.
  **L448 CN**: 以 `(__m128i)vec_cmpeq((__v2di)__X, (__v2di)__Y)` 从当前函数返回。
- **L449 EN**: Closes the current lexical scope or compound statement.
  **L449 CN**: 结束当前词法作用域或复合语句块。
- **L450 EN**: Closes the current preprocessor conditional block.
  **L450 CN**: 结束当前预处理条件块。
- **L451 EN**: Blank line separating nearby declarations or logic blocks.
  **L451 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L452 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L452 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L453 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L453 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L454 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epi8(__m128i __X, __m128i __Y) {`.
  **L454 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epi8(__m128i __X, __m128i __Y) {`。
- **L455 EN**: Returns from the current function with `(__m128i)vec_min((__v16qi)__X, (__v16qi)__Y)`.
  **L455 CN**: 以 `(__m128i)vec_min((__v16qi)__X, (__v16qi)__Y)` 从当前函数返回。
- **L456 EN**: Closes the current lexical scope or compound statement.
  **L456 CN**: 结束当前词法作用域或复合语句块。
- **L457 EN**: Blank line separating nearby declarations or logic blocks.
  **L457 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L458 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L458 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L459 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L459 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L460 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epu16(__m128i __X, __m128i __Y) {`.
  **L460 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epu16(__m128i __X, __m128i __Y) {`。

### Lines 461-480

````c
  return (__m128i)vec_min((__v8hu)__X, (__v8hu)__Y);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_epi32(__m128i __X, __m128i __Y) {
  return (__m128i)vec_min((__v4si)__X, (__v4si)__Y);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_min_epu32(__m128i __X, __m128i __Y) {
  return (__m128i)vec_min((__v4su)__X, (__v4su)__Y);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_epi8(__m128i __X, __m128i __Y) {
  return (__m128i)vec_max((__v16qi)__X, (__v16qi)__Y);
}
````
- **L461 EN**: Returns from the current function with `(__m128i)vec_min((__v8hu)__X, (__v8hu)__Y)`.
  **L461 CN**: 以 `(__m128i)vec_min((__v8hu)__X, (__v8hu)__Y)` 从当前函数返回。
- **L462 EN**: Closes the current lexical scope or compound statement.
  **L462 CN**: 结束当前词法作用域或复合语句块。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L464 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L464 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L465 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L465 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L466 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epi32(__m128i __X, __m128i __Y) {`.
  **L466 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epi32(__m128i __X, __m128i __Y) {`。
- **L467 EN**: Returns from the current function with `(__m128i)vec_min((__v4si)__X, (__v4si)__Y)`.
  **L467 CN**: 以 `(__m128i)vec_min((__v4si)__X, (__v4si)__Y)` 从当前函数返回。
- **L468 EN**: Closes the current lexical scope or compound statement.
  **L468 CN**: 结束当前词法作用域或复合语句块。
- **L469 EN**: Blank line separating nearby declarations or logic blocks.
  **L469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L470 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L470 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L471 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L471 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L472 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_min_epu32(__m128i __X, __m128i __Y) {`.
  **L472 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_min_epu32(__m128i __X, __m128i __Y) {`。
- **L473 EN**: Returns from the current function with `(__m128i)vec_min((__v4su)__X, (__v4su)__Y)`.
  **L473 CN**: 以 `(__m128i)vec_min((__v4su)__X, (__v4su)__Y)` 从当前函数返回。
- **L474 EN**: Closes the current lexical scope or compound statement.
  **L474 CN**: 结束当前词法作用域或复合语句块。
- **L475 EN**: Blank line separating nearby declarations or logic blocks.
  **L475 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L476 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L476 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L477 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L477 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L478 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epi8(__m128i __X, __m128i __Y) {`.
  **L478 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epi8(__m128i __X, __m128i __Y) {`。
- **L479 EN**: Returns from the current function with `(__m128i)vec_max((__v16qi)__X, (__v16qi)__Y)`.
  **L479 CN**: 以 `(__m128i)vec_max((__v16qi)__X, (__v16qi)__Y)` 从当前函数返回。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-500

````c

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_epu16(__m128i __X, __m128i __Y) {
  return (__m128i)vec_max((__v8hu)__X, (__v8hu)__Y);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_epi32(__m128i __X, __m128i __Y) {
  return (__m128i)vec_max((__v4si)__X, (__v4si)__Y);
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_max_epu32(__m128i __X, __m128i __Y) {
  return (__m128i)vec_max((__v4su)__X, (__v4su)__Y);
}

extern __inline __m128i
````
- **L481 EN**: Blank line separating nearby declarations or logic blocks.
  **L481 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L482 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L482 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L483 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L483 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L484 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epu16(__m128i __X, __m128i __Y) {`.
  **L484 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epu16(__m128i __X, __m128i __Y) {`。
- **L485 EN**: Returns from the current function with `(__m128i)vec_max((__v8hu)__X, (__v8hu)__Y)`.
  **L485 CN**: 以 `(__m128i)vec_max((__v8hu)__X, (__v8hu)__Y)` 从当前函数返回。
- **L486 EN**: Closes the current lexical scope or compound statement.
  **L486 CN**: 结束当前词法作用域或复合语句块。
- **L487 EN**: Blank line separating nearby declarations or logic blocks.
  **L487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L488 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L488 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L489 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L489 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L490 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epi32(__m128i __X, __m128i __Y) {`.
  **L490 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epi32(__m128i __X, __m128i __Y) {`。
- **L491 EN**: Returns from the current function with `(__m128i)vec_max((__v4si)__X, (__v4si)__Y)`.
  **L491 CN**: 以 `(__m128i)vec_max((__v4si)__X, (__v4si)__Y)` 从当前函数返回。
- **L492 EN**: Closes the current lexical scope or compound statement.
  **L492 CN**: 结束当前词法作用域或复合语句块。
- **L493 EN**: Blank line separating nearby declarations or logic blocks.
  **L493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L494 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L494 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L495 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L495 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L496 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_max_epu32(__m128i __X, __m128i __Y) {`.
  **L496 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_max_epu32(__m128i __X, __m128i __Y) {`。
- **L497 EN**: Returns from the current function with `(__m128i)vec_max((__v4su)__X, (__v4su)__Y)`.
  **L497 CN**: 以 `(__m128i)vec_max((__v4su)__X, (__v4su)__Y)` 从当前函数返回。
- **L498 EN**: Closes the current lexical scope or compound statement.
  **L498 CN**: 结束当前词法作用域或复合语句块。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L500 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L500 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。

### Lines 501-520

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mullo_epi32(__m128i __X, __m128i __Y) {
  return (__m128i)vec_mul((__v4su)__X, (__v4su)__Y);
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_mul_epi32(__m128i __X, __m128i __Y) {
  return (__m128i)vec_mule((__v4si)__X, (__v4si)__Y);
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepi8_epi16(__m128i __A) {
  return (__m128i)vec_unpackh((__v16qi)__A);
}

extern __inline __m128i
````
- **L501 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L501 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L502 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mullo_epi32(__m128i __X, __m128i __Y) {`.
  **L502 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mullo_epi32(__m128i __X, __m128i __Y) {`。
- **L503 EN**: Returns from the current function with `(__m128i)vec_mul((__v4su)__X, (__v4su)__Y)`.
  **L503 CN**: 以 `(__m128i)vec_mul((__v4su)__X, (__v4su)__Y)` 从当前函数返回。
- **L504 EN**: Closes the current lexical scope or compound statement.
  **L504 CN**: 结束当前词法作用域或复合语句块。
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L506 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L506 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L507 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L507 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L508 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L508 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L509 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mul_epi32(__m128i __X, __m128i __Y) {`.
  **L509 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mul_epi32(__m128i __X, __m128i __Y) {`。
- **L510 EN**: Returns from the current function with `(__m128i)vec_mule((__v4si)__X, (__v4si)__Y)`.
  **L510 CN**: 以 `(__m128i)vec_mule((__v4si)__X, (__v4si)__Y)` 从当前函数返回。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Closes the current preprocessor conditional block.
  **L512 CN**: 结束当前预处理条件块。
- **L513 EN**: Blank line separating nearby declarations or logic blocks.
  **L513 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L514 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L514 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L515 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L515 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L516 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi8_epi16(__m128i __A) {`.
  **L516 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi8_epi16(__m128i __A) {`。
- **L517 EN**: Returns from the current function with `(__m128i)vec_unpackh((__v16qi)__A)`.
  **L517 CN**: 以 `(__m128i)vec_unpackh((__v16qi)__A)` 从当前函数返回。
- **L518 EN**: Closes the current lexical scope or compound statement.
  **L518 CN**: 结束当前词法作用域或复合语句块。
- **L519 EN**: Blank line separating nearby declarations or logic blocks.
  **L519 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L520 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L520 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。

### Lines 521-540

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepi8_epi32(__m128i __A) {
  __A = (__m128i)vec_unpackh((__v16qi)__A);
  return (__m128i)vec_unpackh((__v8hi)__A);
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepi8_epi64(__m128i __A) {
  __A = (__m128i)vec_unpackh((__v16qi)__A);
  __A = (__m128i)vec_unpackh((__v8hi)__A);
  return (__m128i)vec_unpackh((__v4si)__A);
}
#endif

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepi16_epi32(__m128i __A) {
  return (__m128i)vec_unpackh((__v8hi)__A);
````
- **L521 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L521 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L522 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi8_epi32(__m128i __A) {`.
  **L522 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi8_epi32(__m128i __A) {`。
- **L523 EN**: Executes a call or declaration centered on `=`.
  **L523 CN**: 执行以 `=` 为核心的调用或声明。
- **L524 EN**: Returns from the current function with `(__m128i)vec_unpackh((__v8hi)__A)`.
  **L524 CN**: 以 `(__m128i)vec_unpackh((__v8hi)__A)` 从当前函数返回。
- **L525 EN**: Closes the current lexical scope or compound statement.
  **L525 CN**: 结束当前词法作用域或复合语句块。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L527 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L527 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L528 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L528 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L529 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L529 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L530 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi8_epi64(__m128i __A) {`.
  **L530 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi8_epi64(__m128i __A) {`。
- **L531 EN**: Executes a call or declaration centered on `=`.
  **L531 CN**: 执行以 `=` 为核心的调用或声明。
- **L532 EN**: Executes a call or declaration centered on `=`.
  **L532 CN**: 执行以 `=` 为核心的调用或声明。
- **L533 EN**: Returns from the current function with `(__m128i)vec_unpackh((__v4si)__A)`.
  **L533 CN**: 以 `(__m128i)vec_unpackh((__v4si)__A)` 从当前函数返回。
- **L534 EN**: Closes the current lexical scope or compound statement.
  **L534 CN**: 结束当前词法作用域或复合语句块。
- **L535 EN**: Closes the current preprocessor conditional block.
  **L535 CN**: 结束当前预处理条件块。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L537 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L538 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L538 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L539 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi16_epi32(__m128i __A) {`.
  **L539 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi16_epi32(__m128i __A) {`。
- **L540 EN**: Returns from the current function with `(__m128i)vec_unpackh((__v8hi)__A)`.
  **L540 CN**: 以 `(__m128i)vec_unpackh((__v8hi)__A)` 从当前函数返回。

### Lines 541-560

````c
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepi16_epi64(__m128i __A) {
  __A = (__m128i)vec_unpackh((__v8hi)__A);
  return (__m128i)vec_unpackh((__v4si)__A);
}
#endif

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepi32_epi64(__m128i __A) {
  return (__m128i)vec_unpackh((__v4si)__A);
}
#endif

extern __inline __m128i
````
- **L541 EN**: Closes the current lexical scope or compound statement.
  **L541 CN**: 结束当前词法作用域或复合语句块。
- **L542 EN**: Blank line separating nearby declarations or logic blocks.
  **L542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L543 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L543 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L544 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L544 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L545 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L545 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L546 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi16_epi64(__m128i __A) {`.
  **L546 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi16_epi64(__m128i __A) {`。
- **L547 EN**: Executes a call or declaration centered on `=`.
  **L547 CN**: 执行以 `=` 为核心的调用或声明。
- **L548 EN**: Returns from the current function with `(__m128i)vec_unpackh((__v4si)__A)`.
  **L548 CN**: 以 `(__m128i)vec_unpackh((__v4si)__A)` 从当前函数返回。
- **L549 EN**: Closes the current lexical scope or compound statement.
  **L549 CN**: 结束当前词法作用域或复合语句块。
- **L550 EN**: Closes the current preprocessor conditional block.
  **L550 CN**: 结束当前预处理条件块。
- **L551 EN**: Blank line separating nearby declarations or logic blocks.
  **L551 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L552 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L552 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L553 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L553 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L554 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L554 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L555 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi32_epi64(__m128i __A) {`.
  **L555 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi32_epi64(__m128i __A) {`。
- **L556 EN**: Returns from the current function with `(__m128i)vec_unpackh((__v4si)__A)`.
  **L556 CN**: 以 `(__m128i)vec_unpackh((__v4si)__A)` 从当前函数返回。
- **L557 EN**: Closes the current lexical scope or compound statement.
  **L557 CN**: 结束当前词法作用域或复合语句块。
- **L558 EN**: Closes the current preprocessor conditional block.
  **L558 CN**: 结束当前预处理条件块。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L560 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L560 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。

### Lines 561-580

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepu8_epi16(__m128i __A) {
  const __v16qu __zero = {0};
#ifdef __LITTLE_ENDIAN__
  __A = (__m128i)vec_mergeh((__v16qu)__A, __zero);
#else  /* __BIG_ENDIAN__.  */
  __A = (__m128i)vec_mergeh(__zero, (__v16qu)__A);
#endif /* __BIG_ENDIAN__.  */
  return __A;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepu8_epi32(__m128i __A) {
  const __v16qu __zero = {0};
#ifdef __LITTLE_ENDIAN__
  __A = (__m128i)vec_mergeh((__v16qu)__A, __zero);
  __A = (__m128i)vec_mergeh((__v8hu)__A, (__v8hu)__zero);
#else  /* __BIG_ENDIAN__.  */
  __A = (__m128i)vec_mergeh(__zero, (__v16qu)__A);
````
- **L561 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L561 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L562 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu8_epi16(__m128i __A) {`.
  **L562 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu8_epi16(__m128i __A) {`。
- **L563 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L563 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L564 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L564 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L565 EN**: Executes a call or declaration centered on `=`.
  **L565 CN**: 执行以 `=` 为核心的调用或声明。
- **L566 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L566 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L567 EN**: Executes a call or declaration centered on `=`.
  **L567 CN**: 执行以 `=` 为核心的调用或声明。
- **L568 EN**: Closes the current preprocessor conditional block.
  **L568 CN**: 结束当前预处理条件块。
- **L569 EN**: Returns from the current function with `__A`.
  **L569 CN**: 以 `__A` 从当前函数返回。
- **L570 EN**: Closes the current lexical scope or compound statement.
  **L570 CN**: 结束当前词法作用域或复合语句块。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L572 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L573 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L573 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L574 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu8_epi32(__m128i __A) {`.
  **L574 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu8_epi32(__m128i __A) {`。
- **L575 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L575 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L576 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L576 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L577 EN**: Executes a call or declaration centered on `=`.
  **L577 CN**: 执行以 `=` 为核心的调用或声明。
- **L578 EN**: Executes a call or declaration centered on `=`.
  **L578 CN**: 执行以 `=` 为核心的调用或声明。
- **L579 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L579 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L580 EN**: Executes a call or declaration centered on `=`.
  **L580 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 581-600

````c
  __A = (__m128i)vec_mergeh((__v8hu)__zero, (__v8hu)__A);
#endif /* __BIG_ENDIAN__.  */
  return __A;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepu8_epi64(__m128i __A) {
  const __v16qu __zero = {0};
#ifdef __LITTLE_ENDIAN__
  __A = (__m128i)vec_mergeh((__v16qu)__A, __zero);
  __A = (__m128i)vec_mergeh((__v8hu)__A, (__v8hu)__zero);
  __A = (__m128i)vec_mergeh((__v4su)__A, (__v4su)__zero);
#else  /* __BIG_ENDIAN__.  */
  __A = (__m128i)vec_mergeh(__zero, (__v16qu)__A);
  __A = (__m128i)vec_mergeh((__v8hu)__zero, (__v8hu)__A);
  __A = (__m128i)vec_mergeh((__v4su)__zero, (__v4su)__A);
#endif /* __BIG_ENDIAN__.  */
  return __A;
}
````
- **L581 EN**: Executes a call or declaration centered on `=`.
  **L581 CN**: 执行以 `=` 为核心的调用或声明。
- **L582 EN**: Closes the current preprocessor conditional block.
  **L582 CN**: 结束当前预处理条件块。
- **L583 EN**: Returns from the current function with `__A`.
  **L583 CN**: 以 `__A` 从当前函数返回。
- **L584 EN**: Closes the current lexical scope or compound statement.
  **L584 CN**: 结束当前词法作用域或复合语句块。
- **L585 EN**: Blank line separating nearby declarations or logic blocks.
  **L585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L586 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L586 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L587 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L587 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L588 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu8_epi64(__m128i __A) {`.
  **L588 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu8_epi64(__m128i __A) {`。
- **L589 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L589 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L590 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L590 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L591 EN**: Executes a call or declaration centered on `=`.
  **L591 CN**: 执行以 `=` 为核心的调用或声明。
- **L592 EN**: Executes a call or declaration centered on `=`.
  **L592 CN**: 执行以 `=` 为核心的调用或声明。
- **L593 EN**: Executes a call or declaration centered on `=`.
  **L593 CN**: 执行以 `=` 为核心的调用或声明。
- **L594 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L594 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L595 EN**: Executes a call or declaration centered on `=`.
  **L595 CN**: 执行以 `=` 为核心的调用或声明。
- **L596 EN**: Executes a call or declaration centered on `=`.
  **L596 CN**: 执行以 `=` 为核心的调用或声明。
- **L597 EN**: Executes a call or declaration centered on `=`.
  **L597 CN**: 执行以 `=` 为核心的调用或声明。
- **L598 EN**: Closes the current preprocessor conditional block.
  **L598 CN**: 结束当前预处理条件块。
- **L599 EN**: Returns from the current function with `__A`.
  **L599 CN**: 以 `__A` 从当前函数返回。
- **L600 EN**: Closes the current lexical scope or compound statement.
  **L600 CN**: 结束当前词法作用域或复合语句块。

### Lines 601-620

````c

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepu16_epi32(__m128i __A) {
  const __v8hu __zero = {0};
#ifdef __LITTLE_ENDIAN__
  __A = (__m128i)vec_mergeh((__v8hu)__A, __zero);
#else  /* __BIG_ENDIAN__.  */
  __A = (__m128i)vec_mergeh(__zero, (__v8hu)__A);
#endif /* __BIG_ENDIAN__.  */
  return __A;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepu16_epi64(__m128i __A) {
  const __v8hu __zero = {0};
#ifdef __LITTLE_ENDIAN__
  __A = (__m128i)vec_mergeh((__v8hu)__A, __zero);
  __A = (__m128i)vec_mergeh((__v4su)__A, (__v4su)__zero);
````
- **L601 EN**: Blank line separating nearby declarations or logic blocks.
  **L601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L602 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L602 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L603 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L603 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L604 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu16_epi32(__m128i __A) {`.
  **L604 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu16_epi32(__m128i __A) {`。
- **L605 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L605 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L606 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L606 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L607 EN**: Executes a call or declaration centered on `=`.
  **L607 CN**: 执行以 `=` 为核心的调用或声明。
- **L608 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L608 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L609 EN**: Executes a call or declaration centered on `=`.
  **L609 CN**: 执行以 `=` 为核心的调用或声明。
- **L610 EN**: Closes the current preprocessor conditional block.
  **L610 CN**: 结束当前预处理条件块。
- **L611 EN**: Returns from the current function with `__A`.
  **L611 CN**: 以 `__A` 从当前函数返回。
- **L612 EN**: Closes the current lexical scope or compound statement.
  **L612 CN**: 结束当前词法作用域或复合语句块。
- **L613 EN**: Blank line separating nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L614 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L614 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L615 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L615 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L616 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu16_epi64(__m128i __A) {`.
  **L616 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu16_epi64(__m128i __A) {`。
- **L617 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L617 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L618 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L618 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L619 EN**: Executes a call or declaration centered on `=`.
  **L619 CN**: 执行以 `=` 为核心的调用或声明。
- **L620 EN**: Executes a call or declaration centered on `=`.
  **L620 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 621-640

````c
#else  /* __BIG_ENDIAN__.  */
  __A = (__m128i)vec_mergeh(__zero, (__v8hu)__A);
  __A = (__m128i)vec_mergeh((__v4su)__zero, (__v4su)__A);
#endif /* __BIG_ENDIAN__.  */
  return __A;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cvtepu32_epi64(__m128i __A) {
  const __v4su __zero = {0};
#ifdef __LITTLE_ENDIAN__
  __A = (__m128i)vec_mergeh((__v4su)__A, __zero);
#else  /* __BIG_ENDIAN__.  */
  __A = (__m128i)vec_mergeh(__zero, (__v4su)__A);
#endif /* __BIG_ENDIAN__.  */
  return __A;
}

/* Return horizontal packed word minimum and its index in bits [15:0]
````
- **L621 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L621 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L622 EN**: Executes a call or declaration centered on `=`.
  **L622 CN**: 执行以 `=` 为核心的调用或声明。
- **L623 EN**: Executes a call or declaration centered on `=`.
  **L623 CN**: 执行以 `=` 为核心的调用或声明。
- **L624 EN**: Closes the current preprocessor conditional block.
  **L624 CN**: 结束当前预处理条件块。
- **L625 EN**: Returns from the current function with `__A`.
  **L625 CN**: 以 `__A` 从当前函数返回。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Blank line separating nearby declarations or logic blocks.
  **L627 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L628 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L628 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L629 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L629 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L630 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu32_epi64(__m128i __A) {`.
  **L630 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu32_epi64(__m128i __A) {`。
- **L631 EN**: Initializes variable `__zero` from the expression on the right-hand side.
  **L631 CN**: 使用右侧表达式初始化变量 `__zero`。
- **L632 EN**: Starts a preprocessor conditional block: `#ifdef __LITTLE_ENDIAN__`.
  **L632 CN**: 开始一个预处理条件块：`#ifdef __LITTLE_ENDIAN__`。
- **L633 EN**: Executes a call or declaration centered on `=`.
  **L633 CN**: 执行以 `=` 为核心的调用或声明。
- **L634 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L634 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L635 EN**: Executes a call or declaration centered on `=`.
  **L635 CN**: 执行以 `=` 为核心的调用或声明。
- **L636 EN**: Closes the current preprocessor conditional block.
  **L636 CN**: 结束当前预处理条件块。
- **L637 EN**: Returns from the current function with `__A`.
  **L637 CN**: 以 `__A` 从当前函数返回。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Blank line separating nearby declarations or logic blocks.
  **L639 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L640 EN**: Comment explains nearby logic, constraints, or intent: `Return horizontal packed word minimum and its index in bits [15:0]`.
  **L640 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Return horizontal packed word minimum and its index in bits [15:0]`。

### Lines 641-660

````c
   and bits [18:16] respectively.  */
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_minpos_epu16(__m128i __A) {
  union __u {
    __m128i __m;
    __v8hu __uh;
  };
  union __u __u = {.__m = __A}, __r = {.__m = {0}};
  unsigned short __ridx = 0;
  unsigned short __rmin = __u.__uh[__ridx];
  unsigned long __i;
  for (__i = 1; __i < 8; __i++) {
    if (__u.__uh[__i] < __rmin) {
      __rmin = __u.__uh[__i];
      __ridx = __i;
    }
  }
  __r.__uh[0] = __rmin;
  __r.__uh[1] = __ridx;
````
- **L641 EN**: Continues the surrounding expression or declaration: `and bits [18:16] respectively.  */`.
  **L641 CN**: 继续构造周围的表达式或声明：`and bits [18:16] respectively.  */`。
- **L642 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L642 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L643 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L643 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L644 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_minpos_epu16(__m128i __A) {`.
  **L644 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_minpos_epu16(__m128i __A) {`。
- **L645 EN**: Declares union `__u`.
  **L645 CN**: 声明 union `__u`。
- **L646 EN**: Adds a standalone statement or declaration: `__m128i __m;`.
  **L646 CN**: 添加一条独立语句或声明：`__m128i __m;`。
- **L647 EN**: Adds a standalone statement or declaration: `__v8hu __uh;`.
  **L647 CN**: 添加一条独立语句或声明：`__v8hu __uh;`。
- **L648 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L648 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L649 EN**: Declares union `__u`.
  **L649 CN**: 声明 union `__u`。
- **L650 EN**: Initializes variable `__ridx` from the expression on the right-hand side.
  **L650 CN**: 使用右侧表达式初始化变量 `__ridx`。
- **L651 EN**: Initializes variable `__rmin` from the expression on the right-hand side.
  **L651 CN**: 使用右侧表达式初始化变量 `__rmin`。
- **L652 EN**: Adds a standalone statement or declaration: `unsigned long __i;`.
  **L652 CN**: 添加一条独立语句或声明：`unsigned long __i;`。
- **L653 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L653 CN**: 开始 `for` 控制流语句并计算其条件。
- **L654 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L654 CN**: 开始 `if` 控制流语句并计算其条件。
- **L655 EN**: Adds a standalone statement or declaration: `__rmin = __u.__uh[__i];`.
  **L655 CN**: 添加一条独立语句或声明：`__rmin = __u.__uh[__i];`。
- **L656 EN**: Adds a standalone statement or declaration: `__ridx = __i;`.
  **L656 CN**: 添加一条独立语句或声明：`__ridx = __i;`。
- **L657 EN**: Closes the current lexical scope or compound statement.
  **L657 CN**: 结束当前词法作用域或复合语句块。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Adds a standalone statement or declaration: `__r.__uh[0] = __rmin;`.
  **L659 CN**: 添加一条独立语句或声明：`__r.__uh[0] = __rmin;`。
- **L660 EN**: Adds a standalone statement or declaration: `__r.__uh[1] = __ridx;`.
  **L660 CN**: 添加一条独立语句或声明：`__r.__uh[1] = __ridx;`。

### Lines 661-680

````c
  return __r.__m;
}

extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_packus_epi32(__m128i __X, __m128i __Y) {
  return (__m128i)vec_packsu((__v4si)__X, (__v4si)__Y);
}

#ifdef _ARCH_PWR8
extern __inline __m128i
    __attribute__((__gnu_inline__, __always_inline__, __artificial__))
    _mm_cmpgt_epi64(__m128i __X, __m128i __Y) {
  return (__m128i)vec_cmpgt((__v2di)__X, (__v2di)__Y);
}
#endif

#else
#include_next <smmintrin.h>
#endif /* defined(__powerpc64__) &&                                            \
````
- **L661 EN**: Returns from the current function with `__r.__m`.
  **L661 CN**: 以 `__r.__m` 从当前函数返回。
- **L662 EN**: Closes the current lexical scope or compound statement.
  **L662 CN**: 结束当前词法作用域或复合语句块。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L664 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L664 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L665 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L665 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L666 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_packus_epi32(__m128i __X, __m128i __Y) {`.
  **L666 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_packus_epi32(__m128i __X, __m128i __Y) {`。
- **L667 EN**: Returns from the current function with `(__m128i)vec_packsu((__v4si)__X, (__v4si)__Y)`.
  **L667 CN**: 以 `(__m128i)vec_packsu((__v4si)__X, (__v4si)__Y)` 从当前函数返回。
- **L668 EN**: Closes the current lexical scope or compound statement.
  **L668 CN**: 结束当前词法作用域或复合语句块。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L670 EN**: Starts a preprocessor conditional block: `#ifdef _ARCH_PWR8`.
  **L670 CN**: 开始一个预处理条件块：`#ifdef _ARCH_PWR8`。
- **L671 EN**: Continues the surrounding expression or declaration: `extern __inline __m128i`.
  **L671 CN**: 继续构造周围的表达式或声明：`extern __inline __m128i`。
- **L672 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__))`.
  **L672 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__))`。
- **L673 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cmpgt_epi64(__m128i __X, __m128i __Y) {`.
  **L673 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cmpgt_epi64(__m128i __X, __m128i __Y) {`。
- **L674 EN**: Returns from the current function with `(__m128i)vec_cmpgt((__v2di)__X, (__v2di)__Y)`.
  **L674 CN**: 以 `(__m128i)vec_cmpgt((__v2di)__X, (__v2di)__Y)` 从当前函数返回。
- **L675 EN**: Closes the current lexical scope or compound statement.
  **L675 CN**: 结束当前词法作用域或复合语句块。
- **L676 EN**: Closes the current preprocessor conditional block.
  **L676 CN**: 结束当前预处理条件块。
- **L677 EN**: Blank line separating nearby declarations or logic blocks.
  **L677 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L678 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L678 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L679 EN**: Includes <smmintrin.h> to access related header declarations.
  **L679 CN**: 引入 <smmintrin.h> 以使用相关头文件声明。
- **L680 EN**: Closes the current preprocessor conditional block.
  **L680 CN**: 结束当前预处理条件块。

### Lines 681-683

````c
        *   (defined(__linux__) || defined(__FreeBSD__) || defined(_AIX)) */

#endif /* SMMINTRIN_H_ */
````
- **L681 EN**: Comment explains nearby logic, constraints, or intent: `(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`.
  **L681 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(defined(__linux__) || defined(__FreeBSD__) || defined(_AIX))`。
- **L682 EN**: Blank line separating nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L683 EN**: Closes the current preprocessor conditional block.
  **L683 CN**: 结束当前预处理条件块。

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
  - `tmmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
  - `smmintrin.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `NO_WARN_X86_INTRINSICS`, `SMMINTRIN_H_`, `__powerpc64__`, `__linux__`, `__FreeBSD__`, `_AIX`, `_ARCH_PWR9`, `_ARCH_PWR8`, `__BIG_ENDIAN__`, `_ARCH_PWR10`, `__LITTLE_ENDIAN__`
- **External builtins / 外部 builtin**: `__builtin_ppc_mffs`, `__builtin_ppc_mtfsf`, `__builtin_ppc_mffsl`, `__builtin_ppc_set_fpscr_rn`
