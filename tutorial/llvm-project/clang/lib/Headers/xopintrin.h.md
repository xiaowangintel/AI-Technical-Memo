# xopintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/xopintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: XOP intrinsics.
- **Purpose (CN)**: 提供 XOP intrinsic 接口。
- **Line Count / 行数**: 780

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*===---- xopintrin.h - XOP intrinsics -------------------------------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __X86INTRIN_H
#error "Never use <xopintrin.h> directly; include <x86intrin.h> instead."
#endif

#ifndef __XOPINTRIN_H
#define __XOPINTRIN_H

#include <fma4intrin.h>

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS __attribute__((__always_inline__, __nodebug__, __target__("xop"), __min_vector_width__(128)))
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
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __X86INTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __X86INTRIN_H`。
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <xopintrin.h> directly; include <x86intrin.h> instead."`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error "Never use <xopintrin.h> directly; include <x86intrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifndef __XOPINTRIN_H`.
  **L14 CN**: 开始一个预处理条件块：`#ifndef __XOPINTRIN_H`。
- **L15 EN**: Defines macro `__XOPINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `__XOPINTRIN_H`，用于条件编译、简写或 API 生成。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Includes <fma4intrin.h> to access related header declarations.
  **L17 CN**: 引入 <fma4intrin.h> 以使用相关头文件声明。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L20 EN**: Defines macro `__DEFAULT_FN_ATTRS` for conditional compilation, shorthand, or API generation.
  **L20 CN**: 定义宏 `__DEFAULT_FN_ATTRS`，用于条件编译、简写或 API 生成。

### Lines 21-40

````c
#define __DEFAULT_FN_ATTRS256 __attribute__((__always_inline__, __nodebug__, __target__("xop"), __min_vector_width__(256)))

#if defined(__cplusplus) && (__cplusplus >= 201103L)
#define __DEFAULT_FN_ATTRS_CONSTEXPR __DEFAULT_FN_ATTRS constexpr
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256 constexpr
#else
#define __DEFAULT_FN_ATTRS_CONSTEXPR __DEFAULT_FN_ATTRS
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256
#endif

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_maccs_epi16(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacssww((__v8hi)__A, (__v8hi)__B, (__v8hi)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_macc_epi16(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacsww((__v8hi)__A, (__v8hi)__B, (__v8hi)__C);
````
- **L21 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L21 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L23 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L24 EN**: Defines macro `__DEFAULT_FN_ATTRS_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L24 CN**: 定义宏 `__DEFAULT_FN_ATTRS_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L25 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L25 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L26 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L26 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L27 EN**: Defines macro `__DEFAULT_FN_ATTRS_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L27 CN**: 定义宏 `__DEFAULT_FN_ATTRS_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L28 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L28 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L29 EN**: Closes the current preprocessor conditional block.
  **L29 CN**: 结束当前预处理条件块。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L31 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L32 EN**: Continues logic associated with callable symbol `_mm_maccs_epi16`.
  **L32 CN**: 继续与可调用符号 `_mm_maccs_epi16` 相关的逻辑。
- **L33 EN**: Opens a new lexical scope or compound statement.
  **L33 CN**: 打开一个新的词法作用域或复合语句块。
- **L34 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacssww((__v8hi)__A, (__v8hi)__B, (__v8hi)__C)`.
  **L34 CN**: 以 `(__m128i)__builtin_ia32_vpmacssww((__v8hi)__A, (__v8hi)__B, (__v8hi)__C)` 从当前函数返回。
- **L35 EN**: Closes the current lexical scope or compound statement.
  **L35 CN**: 结束当前词法作用域或复合语句块。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L37 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L37 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L38 EN**: Continues logic associated with callable symbol `_mm_macc_epi16`.
  **L38 CN**: 继续与可调用符号 `_mm_macc_epi16` 相关的逻辑。
- **L39 EN**: Opens a new lexical scope or compound statement.
  **L39 CN**: 打开一个新的词法作用域或复合语句块。
- **L40 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacsww((__v8hi)__A, (__v8hi)__B, (__v8hi)__C)`.
  **L40 CN**: 以 `(__m128i)__builtin_ia32_vpmacsww((__v8hi)__A, (__v8hi)__B, (__v8hi)__C)` 从当前函数返回。

### Lines 41-60

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_maccsd_epi16(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacsswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_maccd_epi16(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_maccs_epi32(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacssdd((__v4si)__A, (__v4si)__B, (__v4si)__C);
}

````
- **L41 EN**: Closes the current lexical scope or compound statement.
  **L41 CN**: 结束当前词法作用域或复合语句块。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L43 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L43 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L44 EN**: Continues logic associated with callable symbol `_mm_maccsd_epi16`.
  **L44 CN**: 继续与可调用符号 `_mm_maccsd_epi16` 相关的逻辑。
- **L45 EN**: Opens a new lexical scope or compound statement.
  **L45 CN**: 打开一个新的词法作用域或复合语句块。
- **L46 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacsswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C)`.
  **L46 CN**: 以 `(__m128i)__builtin_ia32_vpmacsswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C)` 从当前函数返回。
- **L47 EN**: Closes the current lexical scope or compound statement.
  **L47 CN**: 结束当前词法作用域或复合语句块。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L49 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L49 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L50 EN**: Continues logic associated with callable symbol `_mm_maccd_epi16`.
  **L50 CN**: 继续与可调用符号 `_mm_maccd_epi16` 相关的逻辑。
- **L51 EN**: Opens a new lexical scope or compound statement.
  **L51 CN**: 打开一个新的词法作用域或复合语句块。
- **L52 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C)`.
  **L52 CN**: 以 `(__m128i)__builtin_ia32_vpmacswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C)` 从当前函数返回。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L55 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L55 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L56 EN**: Continues logic associated with callable symbol `_mm_maccs_epi32`.
  **L56 CN**: 继续与可调用符号 `_mm_maccs_epi32` 相关的逻辑。
- **L57 EN**: Opens a new lexical scope or compound statement.
  **L57 CN**: 打开一个新的词法作用域或复合语句块。
- **L58 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacssdd((__v4si)__A, (__v4si)__B, (__v4si)__C)`.
  **L58 CN**: 以 `(__m128i)__builtin_ia32_vpmacssdd((__v4si)__A, (__v4si)__B, (__v4si)__C)` 从当前函数返回。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 61-80

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_macc_epi32(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacsdd((__v4si)__A, (__v4si)__B, (__v4si)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_maccslo_epi32(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacssdql((__v4si)__A, (__v4si)__B, (__v2di)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_macclo_epi32(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacsdql((__v4si)__A, (__v4si)__B, (__v2di)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_maccshi_epi32(__m128i __A, __m128i __B, __m128i __C)
````
- **L61 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L61 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L62 EN**: Continues logic associated with callable symbol `_mm_macc_epi32`.
  **L62 CN**: 继续与可调用符号 `_mm_macc_epi32` 相关的逻辑。
- **L63 EN**: Opens a new lexical scope or compound statement.
  **L63 CN**: 打开一个新的词法作用域或复合语句块。
- **L64 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacsdd((__v4si)__A, (__v4si)__B, (__v4si)__C)`.
  **L64 CN**: 以 `(__m128i)__builtin_ia32_vpmacsdd((__v4si)__A, (__v4si)__B, (__v4si)__C)` 从当前函数返回。
- **L65 EN**: Closes the current lexical scope or compound statement.
  **L65 CN**: 结束当前词法作用域或复合语句块。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L67 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L67 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L68 EN**: Continues logic associated with callable symbol `_mm_maccslo_epi32`.
  **L68 CN**: 继续与可调用符号 `_mm_maccslo_epi32` 相关的逻辑。
- **L69 EN**: Opens a new lexical scope or compound statement.
  **L69 CN**: 打开一个新的词法作用域或复合语句块。
- **L70 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacssdql((__v4si)__A, (__v4si)__B, (__v2di)__C)`.
  **L70 CN**: 以 `(__m128i)__builtin_ia32_vpmacssdql((__v4si)__A, (__v4si)__B, (__v2di)__C)` 从当前函数返回。
- **L71 EN**: Closes the current lexical scope or compound statement.
  **L71 CN**: 结束当前词法作用域或复合语句块。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L73 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L73 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L74 EN**: Continues logic associated with callable symbol `_mm_macclo_epi32`.
  **L74 CN**: 继续与可调用符号 `_mm_macclo_epi32` 相关的逻辑。
- **L75 EN**: Opens a new lexical scope or compound statement.
  **L75 CN**: 打开一个新的词法作用域或复合语句块。
- **L76 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacsdql((__v4si)__A, (__v4si)__B, (__v2di)__C)`.
  **L76 CN**: 以 `(__m128i)__builtin_ia32_vpmacsdql((__v4si)__A, (__v4si)__B, (__v2di)__C)` 从当前函数返回。
- **L77 EN**: Closes the current lexical scope or compound statement.
  **L77 CN**: 结束当前词法作用域或复合语句块。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L79 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L79 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L80 EN**: Continues logic associated with callable symbol `_mm_maccshi_epi32`.
  **L80 CN**: 继续与可调用符号 `_mm_maccshi_epi32` 相关的逻辑。

### Lines 81-100

````c
{
  return (__m128i)__builtin_ia32_vpmacssdqh((__v4si)__A, (__v4si)__B, (__v2di)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_macchi_epi32(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmacsdqh((__v4si)__A, (__v4si)__B, (__v2di)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_maddsd_epi16(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmadcsswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_maddd_epi16(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpmadcswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C);
````
- **L81 EN**: Opens a new lexical scope or compound statement.
  **L81 CN**: 打开一个新的词法作用域或复合语句块。
- **L82 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacssdqh((__v4si)__A, (__v4si)__B, (__v2di)__C)`.
  **L82 CN**: 以 `(__m128i)__builtin_ia32_vpmacssdqh((__v4si)__A, (__v4si)__B, (__v2di)__C)` 从当前函数返回。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L85 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L86 EN**: Continues logic associated with callable symbol `_mm_macchi_epi32`.
  **L86 CN**: 继续与可调用符号 `_mm_macchi_epi32` 相关的逻辑。
- **L87 EN**: Opens a new lexical scope or compound statement.
  **L87 CN**: 打开一个新的词法作用域或复合语句块。
- **L88 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmacsdqh((__v4si)__A, (__v4si)__B, (__v2di)__C)`.
  **L88 CN**: 以 `(__m128i)__builtin_ia32_vpmacsdqh((__v4si)__A, (__v4si)__B, (__v2di)__C)` 从当前函数返回。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L91 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L91 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L92 EN**: Continues logic associated with callable symbol `_mm_maddsd_epi16`.
  **L92 CN**: 继续与可调用符号 `_mm_maddsd_epi16` 相关的逻辑。
- **L93 EN**: Opens a new lexical scope or compound statement.
  **L93 CN**: 打开一个新的词法作用域或复合语句块。
- **L94 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmadcsswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C)`.
  **L94 CN**: 以 `(__m128i)__builtin_ia32_vpmadcsswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C)` 从当前函数返回。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L97 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L97 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L98 EN**: Continues logic associated with callable symbol `_mm_maddd_epi16`.
  **L98 CN**: 继续与可调用符号 `_mm_maddd_epi16` 相关的逻辑。
- **L99 EN**: Opens a new lexical scope or compound statement.
  **L99 CN**: 打开一个新的词法作用域或复合语句块。
- **L100 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpmadcswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C)`.
  **L100 CN**: 以 `(__m128i)__builtin_ia32_vpmadcswd((__v8hi)__A, (__v8hi)__B, (__v4si)__C)` 从当前函数返回。

### Lines 101-120

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddw_epi8(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphaddbw((__v16qi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddd_epi8(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphaddbd((__v16qi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddq_epi8(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphaddbq((__v16qi)__A);
}

````
- **L101 EN**: Closes the current lexical scope or compound statement.
  **L101 CN**: 结束当前词法作用域或复合语句块。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L103 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L103 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L104 EN**: Continues logic associated with callable symbol `_mm_haddw_epi8`.
  **L104 CN**: 继续与可调用符号 `_mm_haddw_epi8` 相关的逻辑。
- **L105 EN**: Opens a new lexical scope or compound statement.
  **L105 CN**: 打开一个新的词法作用域或复合语句块。
- **L106 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddbw((__v16qi)__A)`.
  **L106 CN**: 以 `(__m128i)__builtin_ia32_vphaddbw((__v16qi)__A)` 从当前函数返回。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L109 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L109 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L110 EN**: Continues logic associated with callable symbol `_mm_haddd_epi8`.
  **L110 CN**: 继续与可调用符号 `_mm_haddd_epi8` 相关的逻辑。
- **L111 EN**: Opens a new lexical scope or compound statement.
  **L111 CN**: 打开一个新的词法作用域或复合语句块。
- **L112 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddbd((__v16qi)__A)`.
  **L112 CN**: 以 `(__m128i)__builtin_ia32_vphaddbd((__v16qi)__A)` 从当前函数返回。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L115 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L116 EN**: Continues logic associated with callable symbol `_mm_haddq_epi8`.
  **L116 CN**: 继续与可调用符号 `_mm_haddq_epi8` 相关的逻辑。
- **L117 EN**: Opens a new lexical scope or compound statement.
  **L117 CN**: 打开一个新的词法作用域或复合语句块。
- **L118 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddbq((__v16qi)__A)`.
  **L118 CN**: 以 `(__m128i)__builtin_ia32_vphaddbq((__v16qi)__A)` 从当前函数返回。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 121-140

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddd_epi16(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphaddwd((__v8hi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddq_epi16(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphaddwq((__v8hi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddq_epi32(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphadddq((__v4si)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddw_epu8(__m128i __A)
````
- **L121 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L121 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L122 EN**: Continues logic associated with callable symbol `_mm_haddd_epi16`.
  **L122 CN**: 继续与可调用符号 `_mm_haddd_epi16` 相关的逻辑。
- **L123 EN**: Opens a new lexical scope or compound statement.
  **L123 CN**: 打开一个新的词法作用域或复合语句块。
- **L124 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddwd((__v8hi)__A)`.
  **L124 CN**: 以 `(__m128i)__builtin_ia32_vphaddwd((__v8hi)__A)` 从当前函数返回。
- **L125 EN**: Closes the current lexical scope or compound statement.
  **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L127 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L127 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L128 EN**: Continues logic associated with callable symbol `_mm_haddq_epi16`.
  **L128 CN**: 继续与可调用符号 `_mm_haddq_epi16` 相关的逻辑。
- **L129 EN**: Opens a new lexical scope or compound statement.
  **L129 CN**: 打开一个新的词法作用域或复合语句块。
- **L130 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddwq((__v8hi)__A)`.
  **L130 CN**: 以 `(__m128i)__builtin_ia32_vphaddwq((__v8hi)__A)` 从当前函数返回。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L133 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L133 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L134 EN**: Continues logic associated with callable symbol `_mm_haddq_epi32`.
  **L134 CN**: 继续与可调用符号 `_mm_haddq_epi32` 相关的逻辑。
- **L135 EN**: Opens a new lexical scope or compound statement.
  **L135 CN**: 打开一个新的词法作用域或复合语句块。
- **L136 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphadddq((__v4si)__A)`.
  **L136 CN**: 以 `(__m128i)__builtin_ia32_vphadddq((__v4si)__A)` 从当前函数返回。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L139 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L139 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L140 EN**: Continues logic associated with callable symbol `_mm_haddw_epu8`.
  **L140 CN**: 继续与可调用符号 `_mm_haddw_epu8` 相关的逻辑。

### Lines 141-160

````c
{
  return (__m128i)__builtin_ia32_vphaddubw((__v16qi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddd_epu8(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphaddubd((__v16qi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddq_epu8(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphaddubq((__v16qi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddd_epu16(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphadduwd((__v8hi)__A);
````
- **L141 EN**: Opens a new lexical scope or compound statement.
  **L141 CN**: 打开一个新的词法作用域或复合语句块。
- **L142 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddubw((__v16qi)__A)`.
  **L142 CN**: 以 `(__m128i)__builtin_ia32_vphaddubw((__v16qi)__A)` 从当前函数返回。
- **L143 EN**: Closes the current lexical scope or compound statement.
  **L143 CN**: 结束当前词法作用域或复合语句块。
- **L144 EN**: Blank line separating nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L145 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L145 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L146 EN**: Continues logic associated with callable symbol `_mm_haddd_epu8`.
  **L146 CN**: 继续与可调用符号 `_mm_haddd_epu8` 相关的逻辑。
- **L147 EN**: Opens a new lexical scope or compound statement.
  **L147 CN**: 打开一个新的词法作用域或复合语句块。
- **L148 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddubd((__v16qi)__A)`.
  **L148 CN**: 以 `(__m128i)__builtin_ia32_vphaddubd((__v16qi)__A)` 从当前函数返回。
- **L149 EN**: Closes the current lexical scope or compound statement.
  **L149 CN**: 结束当前词法作用域或复合语句块。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L151 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L151 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L152 EN**: Continues logic associated with callable symbol `_mm_haddq_epu8`.
  **L152 CN**: 继续与可调用符号 `_mm_haddq_epu8` 相关的逻辑。
- **L153 EN**: Opens a new lexical scope or compound statement.
  **L153 CN**: 打开一个新的词法作用域或复合语句块。
- **L154 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddubq((__v16qi)__A)`.
  **L154 CN**: 以 `(__m128i)__builtin_ia32_vphaddubq((__v16qi)__A)` 从当前函数返回。
- **L155 EN**: Closes the current lexical scope or compound statement.
  **L155 CN**: 结束当前词法作用域或复合语句块。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L157 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L157 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L158 EN**: Continues logic associated with callable symbol `_mm_haddd_epu16`.
  **L158 CN**: 继续与可调用符号 `_mm_haddd_epu16` 相关的逻辑。
- **L159 EN**: Opens a new lexical scope or compound statement.
  **L159 CN**: 打开一个新的词法作用域或复合语句块。
- **L160 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphadduwd((__v8hi)__A)`.
  **L160 CN**: 以 `(__m128i)__builtin_ia32_vphadduwd((__v8hi)__A)` 从当前函数返回。

### Lines 161-180

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddq_epu16(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphadduwq((__v8hi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_haddq_epu32(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphaddudq((__v4si)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_hsubw_epi8(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphsubbw((__v16qi)__A);
}

````
- **L161 EN**: Closes the current lexical scope or compound statement.
  **L161 CN**: 结束当前词法作用域或复合语句块。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L163 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L163 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L164 EN**: Continues logic associated with callable symbol `_mm_haddq_epu16`.
  **L164 CN**: 继续与可调用符号 `_mm_haddq_epu16` 相关的逻辑。
- **L165 EN**: Opens a new lexical scope or compound statement.
  **L165 CN**: 打开一个新的词法作用域或复合语句块。
- **L166 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphadduwq((__v8hi)__A)`.
  **L166 CN**: 以 `(__m128i)__builtin_ia32_vphadduwq((__v8hi)__A)` 从当前函数返回。
- **L167 EN**: Closes the current lexical scope or compound statement.
  **L167 CN**: 结束当前词法作用域或复合语句块。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L169 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L169 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L170 EN**: Continues logic associated with callable symbol `_mm_haddq_epu32`.
  **L170 CN**: 继续与可调用符号 `_mm_haddq_epu32` 相关的逻辑。
- **L171 EN**: Opens a new lexical scope or compound statement.
  **L171 CN**: 打开一个新的词法作用域或复合语句块。
- **L172 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphaddudq((__v4si)__A)`.
  **L172 CN**: 以 `(__m128i)__builtin_ia32_vphaddudq((__v4si)__A)` 从当前函数返回。
- **L173 EN**: Closes the current lexical scope or compound statement.
  **L173 CN**: 结束当前词法作用域或复合语句块。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L175 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L175 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L176 EN**: Continues logic associated with callable symbol `_mm_hsubw_epi8`.
  **L176 CN**: 继续与可调用符号 `_mm_hsubw_epi8` 相关的逻辑。
- **L177 EN**: Opens a new lexical scope or compound statement.
  **L177 CN**: 打开一个新的词法作用域或复合语句块。
- **L178 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphsubbw((__v16qi)__A)`.
  **L178 CN**: 以 `(__m128i)__builtin_ia32_vphsubbw((__v16qi)__A)` 从当前函数返回。
- **L179 EN**: Closes the current lexical scope or compound statement.
  **L179 CN**: 结束当前词法作用域或复合语句块。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 181-200

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_hsubd_epi16(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphsubwd((__v8hi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_hsubq_epi32(__m128i __A)
{
  return (__m128i)__builtin_ia32_vphsubdq((__v4si)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_cmov_si128(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)(((__v2du)__A & (__v2du)__C) | ((__v2du)__B & ~(__v2du)__C));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cmov_si256(__m256i __A, __m256i __B, __m256i __C)
````
- **L181 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L181 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L182 EN**: Continues logic associated with callable symbol `_mm_hsubd_epi16`.
  **L182 CN**: 继续与可调用符号 `_mm_hsubd_epi16` 相关的逻辑。
- **L183 EN**: Opens a new lexical scope or compound statement.
  **L183 CN**: 打开一个新的词法作用域或复合语句块。
- **L184 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphsubwd((__v8hi)__A)`.
  **L184 CN**: 以 `(__m128i)__builtin_ia32_vphsubwd((__v8hi)__A)` 从当前函数返回。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Blank line separating nearby declarations or logic blocks.
  **L186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L187 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L187 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L188 EN**: Continues logic associated with callable symbol `_mm_hsubq_epi32`.
  **L188 CN**: 继续与可调用符号 `_mm_hsubq_epi32` 相关的逻辑。
- **L189 EN**: Opens a new lexical scope or compound statement.
  **L189 CN**: 打开一个新的词法作用域或复合语句块。
- **L190 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vphsubdq((__v4si)__A)`.
  **L190 CN**: 以 `(__m128i)__builtin_ia32_vphsubdq((__v4si)__A)` 从当前函数返回。
- **L191 EN**: Closes the current lexical scope or compound statement.
  **L191 CN**: 结束当前词法作用域或复合语句块。
- **L192 EN**: Blank line separating nearby declarations or logic blocks.
  **L192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L193 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L193 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L194 EN**: Continues logic associated with callable symbol `_mm_cmov_si128`.
  **L194 CN**: 继续与可调用符号 `_mm_cmov_si128` 相关的逻辑。
- **L195 EN**: Opens a new lexical scope or compound statement.
  **L195 CN**: 打开一个新的词法作用域或复合语句块。
- **L196 EN**: Returns from the current function with `(__m128i)(((__v2du)__A & (__v2du)__C) | ((__v2du)__B & ~(__v2du)__C))`.
  **L196 CN**: 以 `(__m128i)(((__v2du)__A & (__v2du)__C) | ((__v2du)__B & ~(__v2du)__C))` 从当前函数返回。
- **L197 EN**: Closes the current lexical scope or compound statement.
  **L197 CN**: 结束当前词法作用域或复合语句块。
- **L198 EN**: Blank line separating nearby declarations or logic blocks.
  **L198 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L199 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L199 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L200 EN**: Continues logic associated with callable symbol `_mm256_cmov_si256`.
  **L200 CN**: 继续与可调用符号 `_mm256_cmov_si256` 相关的逻辑。

### Lines 201-220

````c
{
  return (__m256i)(((__v4du)__A & (__v4du)__C) | ((__v4du)__B & ~(__v4du)__C));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_perm_epi8(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_vpperm((__v16qi)__A, (__v16qi)__B, (__v16qi)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_rot_epi8(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_elementwise_fshl((__v16qu)__A, (__v16qu)__A, (__v16qu)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_rot_epi16(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_elementwise_fshl((__v8hu)__A, (__v8hu)__A, (__v8hu)__B);
````
- **L201 EN**: Opens a new lexical scope or compound statement.
  **L201 CN**: 打开一个新的词法作用域或复合语句块。
- **L202 EN**: Returns from the current function with `(__m256i)(((__v4du)__A & (__v4du)__C) | ((__v4du)__B & ~(__v4du)__C))`.
  **L202 CN**: 以 `(__m256i)(((__v4du)__A & (__v4du)__C) | ((__v4du)__B & ~(__v4du)__C))` 从当前函数返回。
- **L203 EN**: Closes the current lexical scope or compound statement.
  **L203 CN**: 结束当前词法作用域或复合语句块。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L205 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L205 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L206 EN**: Continues logic associated with callable symbol `_mm_perm_epi8`.
  **L206 CN**: 继续与可调用符号 `_mm_perm_epi8` 相关的逻辑。
- **L207 EN**: Opens a new lexical scope or compound statement.
  **L207 CN**: 打开一个新的词法作用域或复合语句块。
- **L208 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpperm((__v16qi)__A, (__v16qi)__B, (__v16qi)__C)`.
  **L208 CN**: 以 `(__m128i)__builtin_ia32_vpperm((__v16qi)__A, (__v16qi)__B, (__v16qi)__C)` 从当前函数返回。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L211 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L211 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L212 EN**: Continues logic associated with callable symbol `_mm_rot_epi8`.
  **L212 CN**: 继续与可调用符号 `_mm_rot_epi8` 相关的逻辑。
- **L213 EN**: Opens a new lexical scope or compound statement.
  **L213 CN**: 打开一个新的词法作用域或复合语句块。
- **L214 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshl((__v16qu)__A, (__v16qu)__A, (__v16qu)__B)`.
  **L214 CN**: 以 `(__m128i)__builtin_elementwise_fshl((__v16qu)__A, (__v16qu)__A, (__v16qu)__B)` 从当前函数返回。
- **L215 EN**: Closes the current lexical scope or compound statement.
  **L215 CN**: 结束当前词法作用域或复合语句块。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L217 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L217 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L218 EN**: Continues logic associated with callable symbol `_mm_rot_epi16`.
  **L218 CN**: 继续与可调用符号 `_mm_rot_epi16` 相关的逻辑。
- **L219 EN**: Opens a new lexical scope or compound statement.
  **L219 CN**: 打开一个新的词法作用域或复合语句块。
- **L220 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshl((__v8hu)__A, (__v8hu)__A, (__v8hu)__B)`.
  **L220 CN**: 以 `(__m128i)__builtin_elementwise_fshl((__v8hu)__A, (__v8hu)__A, (__v8hu)__B)` 从当前函数返回。

### Lines 221-240

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_rot_epi32(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_elementwise_fshl((__v4su)__A, (__v4su)__A, (__v4su)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR
_mm_rot_epi64(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_elementwise_fshl((__v2du)__A, (__v2du)__A, (__v2du)__B);
}

#define _mm_roti_epi8(A, N) \
  ((__m128i)__builtin_ia32_vprotbi((__v16qi)(__m128i)(A), (N)))

#define _mm_roti_epi16(A, N) \
  ((__m128i)__builtin_ia32_vprotwi((__v8hi)(__m128i)(A), (N)))

````
- **L221 EN**: Closes the current lexical scope or compound statement.
  **L221 CN**: 结束当前词法作用域或复合语句块。
- **L222 EN**: Blank line separating nearby declarations or logic blocks.
  **L222 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L223 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L223 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L224 EN**: Continues logic associated with callable symbol `_mm_rot_epi32`.
  **L224 CN**: 继续与可调用符号 `_mm_rot_epi32` 相关的逻辑。
- **L225 EN**: Opens a new lexical scope or compound statement.
  **L225 CN**: 打开一个新的词法作用域或复合语句块。
- **L226 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshl((__v4su)__A, (__v4su)__A, (__v4su)__B)`.
  **L226 CN**: 以 `(__m128i)__builtin_elementwise_fshl((__v4su)__A, (__v4su)__A, (__v4su)__B)` 从当前函数返回。
- **L227 EN**: Closes the current lexical scope or compound statement.
  **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L229 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L229 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L230 EN**: Continues logic associated with callable symbol `_mm_rot_epi64`.
  **L230 CN**: 继续与可调用符号 `_mm_rot_epi64` 相关的逻辑。
- **L231 EN**: Opens a new lexical scope or compound statement.
  **L231 CN**: 打开一个新的词法作用域或复合语句块。
- **L232 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshl((__v2du)__A, (__v2du)__A, (__v2du)__B)`.
  **L232 CN**: 以 `(__m128i)__builtin_elementwise_fshl((__v2du)__A, (__v2du)__A, (__v2du)__B)` 从当前函数返回。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L235 EN**: Defines macro `_mm_roti_epi8(A, N)` for conditional compilation, shorthand, or API generation.
  **L235 CN**: 定义宏 `_mm_roti_epi8(A, N)`，用于条件编译、简写或 API 生成。
- **L236 EN**: Continues logic associated with callable symbol `__builtin_ia32_vprotbi`.
  **L236 CN**: 继续与可调用符号 `__builtin_ia32_vprotbi` 相关的逻辑。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L238 EN**: Defines macro `_mm_roti_epi16(A, N)` for conditional compilation, shorthand, or API generation.
  **L238 CN**: 定义宏 `_mm_roti_epi16(A, N)`，用于条件编译、简写或 API 生成。
- **L239 EN**: Continues logic associated with callable symbol `__builtin_ia32_vprotwi`.
  **L239 CN**: 继续与可调用符号 `__builtin_ia32_vprotwi` 相关的逻辑。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 241-260

````c
#define _mm_roti_epi32(A, N) \
  ((__m128i)__builtin_ia32_vprotdi((__v4si)(__m128i)(A), (N)))

#define _mm_roti_epi64(A, N) \
  ((__m128i)__builtin_ia32_vprotqi((__v2di)(__m128i)(A), (N)))

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_shl_epi8(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_vpshlb((__v16qi)__A, (__v16qi)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_shl_epi16(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_vpshlw((__v8hi)__A, (__v8hi)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_shl_epi32(__m128i __A, __m128i __B)
````
- **L241 EN**: Defines macro `_mm_roti_epi32(A, N)` for conditional compilation, shorthand, or API generation.
  **L241 CN**: 定义宏 `_mm_roti_epi32(A, N)`，用于条件编译、简写或 API 生成。
- **L242 EN**: Continues logic associated with callable symbol `__builtin_ia32_vprotdi`.
  **L242 CN**: 继续与可调用符号 `__builtin_ia32_vprotdi` 相关的逻辑。
- **L243 EN**: Blank line separating nearby declarations or logic blocks.
  **L243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L244 EN**: Defines macro `_mm_roti_epi64(A, N)` for conditional compilation, shorthand, or API generation.
  **L244 CN**: 定义宏 `_mm_roti_epi64(A, N)`，用于条件编译、简写或 API 生成。
- **L245 EN**: Continues logic associated with callable symbol `__builtin_ia32_vprotqi`.
  **L245 CN**: 继续与可调用符号 `__builtin_ia32_vprotqi` 相关的逻辑。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L247 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L247 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L248 EN**: Continues logic associated with callable symbol `_mm_shl_epi8`.
  **L248 CN**: 继续与可调用符号 `_mm_shl_epi8` 相关的逻辑。
- **L249 EN**: Opens a new lexical scope or compound statement.
  **L249 CN**: 打开一个新的词法作用域或复合语句块。
- **L250 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpshlb((__v16qi)__A, (__v16qi)__B)`.
  **L250 CN**: 以 `(__m128i)__builtin_ia32_vpshlb((__v16qi)__A, (__v16qi)__B)` 从当前函数返回。
- **L251 EN**: Closes the current lexical scope or compound statement.
  **L251 CN**: 结束当前词法作用域或复合语句块。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L253 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L253 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L254 EN**: Continues logic associated with callable symbol `_mm_shl_epi16`.
  **L254 CN**: 继续与可调用符号 `_mm_shl_epi16` 相关的逻辑。
- **L255 EN**: Opens a new lexical scope or compound statement.
  **L255 CN**: 打开一个新的词法作用域或复合语句块。
- **L256 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpshlw((__v8hi)__A, (__v8hi)__B)`.
  **L256 CN**: 以 `(__m128i)__builtin_ia32_vpshlw((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L257 EN**: Closes the current lexical scope or compound statement.
  **L257 CN**: 结束当前词法作用域或复合语句块。
- **L258 EN**: Blank line separating nearby declarations or logic blocks.
  **L258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L259 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L259 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L260 EN**: Continues logic associated with callable symbol `_mm_shl_epi32`.
  **L260 CN**: 继续与可调用符号 `_mm_shl_epi32` 相关的逻辑。

### Lines 261-280

````c
{
  return (__m128i)__builtin_ia32_vpshld((__v4si)__A, (__v4si)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_shl_epi64(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_vpshlq((__v2di)__A, (__v2di)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_sha_epi8(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_vpshab((__v16qi)__A, (__v16qi)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_sha_epi16(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_vpshaw((__v8hi)__A, (__v8hi)__B);
````
- **L261 EN**: Opens a new lexical scope or compound statement.
  **L261 CN**: 打开一个新的词法作用域或复合语句块。
- **L262 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpshld((__v4si)__A, (__v4si)__B)`.
  **L262 CN**: 以 `(__m128i)__builtin_ia32_vpshld((__v4si)__A, (__v4si)__B)` 从当前函数返回。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L265 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L265 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L266 EN**: Continues logic associated with callable symbol `_mm_shl_epi64`.
  **L266 CN**: 继续与可调用符号 `_mm_shl_epi64` 相关的逻辑。
- **L267 EN**: Opens a new lexical scope or compound statement.
  **L267 CN**: 打开一个新的词法作用域或复合语句块。
- **L268 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpshlq((__v2di)__A, (__v2di)__B)`.
  **L268 CN**: 以 `(__m128i)__builtin_ia32_vpshlq((__v2di)__A, (__v2di)__B)` 从当前函数返回。
- **L269 EN**: Closes the current lexical scope or compound statement.
  **L269 CN**: 结束当前词法作用域或复合语句块。
- **L270 EN**: Blank line separating nearby declarations or logic blocks.
  **L270 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L271 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L271 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L272 EN**: Continues logic associated with callable symbol `_mm_sha_epi8`.
  **L272 CN**: 继续与可调用符号 `_mm_sha_epi8` 相关的逻辑。
- **L273 EN**: Opens a new lexical scope or compound statement.
  **L273 CN**: 打开一个新的词法作用域或复合语句块。
- **L274 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpshab((__v16qi)__A, (__v16qi)__B)`.
  **L274 CN**: 以 `(__m128i)__builtin_ia32_vpshab((__v16qi)__A, (__v16qi)__B)` 从当前函数返回。
- **L275 EN**: Closes the current lexical scope or compound statement.
  **L275 CN**: 结束当前词法作用域或复合语句块。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L277 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L277 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L278 EN**: Continues logic associated with callable symbol `_mm_sha_epi16`.
  **L278 CN**: 继续与可调用符号 `_mm_sha_epi16` 相关的逻辑。
- **L279 EN**: Opens a new lexical scope or compound statement.
  **L279 CN**: 打开一个新的词法作用域或复合语句块。
- **L280 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpshaw((__v8hi)__A, (__v8hi)__B)`.
  **L280 CN**: 以 `(__m128i)__builtin_ia32_vpshaw((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。

### Lines 281-300

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_sha_epi32(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_vpshad((__v4si)__A, (__v4si)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_sha_epi64(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_vpshaq((__v2di)__A, (__v2di)__B);
}

#define _mm_com_epu8(A, B, N) \
  ((__m128i)__builtin_ia32_vpcomub((__v16qi)(__m128i)(A), \
                                   (__v16qi)(__m128i)(B), (N)))

#define _mm_com_epu16(A, B, N) \
  ((__m128i)__builtin_ia32_vpcomuw((__v8hi)(__m128i)(A), \
````
- **L281 EN**: Closes the current lexical scope or compound statement.
  **L281 CN**: 结束当前词法作用域或复合语句块。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L283 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L283 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L284 EN**: Continues logic associated with callable symbol `_mm_sha_epi32`.
  **L284 CN**: 继续与可调用符号 `_mm_sha_epi32` 相关的逻辑。
- **L285 EN**: Opens a new lexical scope or compound statement.
  **L285 CN**: 打开一个新的词法作用域或复合语句块。
- **L286 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpshad((__v4si)__A, (__v4si)__B)`.
  **L286 CN**: 以 `(__m128i)__builtin_ia32_vpshad((__v4si)__A, (__v4si)__B)` 从当前函数返回。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L289 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L289 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L290 EN**: Continues logic associated with callable symbol `_mm_sha_epi64`.
  **L290 CN**: 继续与可调用符号 `_mm_sha_epi64` 相关的逻辑。
- **L291 EN**: Opens a new lexical scope or compound statement.
  **L291 CN**: 打开一个新的词法作用域或复合语句块。
- **L292 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpshaq((__v2di)__A, (__v2di)__B)`.
  **L292 CN**: 以 `(__m128i)__builtin_ia32_vpshaq((__v2di)__A, (__v2di)__B)` 从当前函数返回。
- **L293 EN**: Closes the current lexical scope or compound statement.
  **L293 CN**: 结束当前词法作用域或复合语句块。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L295 EN**: Defines macro `_mm_com_epu8(A, B, N)` for conditional compilation, shorthand, or API generation.
  **L295 CN**: 定义宏 `_mm_com_epu8(A, B, N)`，用于条件编译、简写或 API 生成。
- **L296 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpcomub`.
  **L296 CN**: 继续与可调用符号 `__builtin_ia32_vpcomub` 相关的逻辑。
- **L297 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (N)))`.
  **L297 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (N)))`。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L299 EN**: Defines macro `_mm_com_epu16(A, B, N)` for conditional compilation, shorthand, or API generation.
  **L299 CN**: 定义宏 `_mm_com_epu16(A, B, N)`，用于条件编译、简写或 API 生成。
- **L300 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpcomuw`.
  **L300 CN**: 继续与可调用符号 `__builtin_ia32_vpcomuw` 相关的逻辑。

### Lines 301-320

````c
                                   (__v8hi)(__m128i)(B), (N)))

#define _mm_com_epu32(A, B, N) \
  ((__m128i)__builtin_ia32_vpcomud((__v4si)(__m128i)(A), \
                                   (__v4si)(__m128i)(B), (N)))

#define _mm_com_epu64(A, B, N) \
  ((__m128i)__builtin_ia32_vpcomuq((__v2di)(__m128i)(A), \
                                   (__v2di)(__m128i)(B), (N)))

#define _mm_com_epi8(A, B, N) \
  ((__m128i)__builtin_ia32_vpcomb((__v16qi)(__m128i)(A), \
                                  (__v16qi)(__m128i)(B), (N)))

#define _mm_com_epi16(A, B, N) \
  ((__m128i)__builtin_ia32_vpcomw((__v8hi)(__m128i)(A), \
                                  (__v8hi)(__m128i)(B), (N)))

#define _mm_com_epi32(A, B, N) \
  ((__m128i)__builtin_ia32_vpcomd((__v4si)(__m128i)(A), \
````
- **L301 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(B), (N)))`.
  **L301 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(B), (N)))`。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L303 EN**: Defines macro `_mm_com_epu32(A, B, N)` for conditional compilation, shorthand, or API generation.
  **L303 CN**: 定义宏 `_mm_com_epu32(A, B, N)`，用于条件编译、简写或 API 生成。
- **L304 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpcomud`.
  **L304 CN**: 继续与可调用符号 `__builtin_ia32_vpcomud` 相关的逻辑。
- **L305 EN**: Continues the surrounding expression or declaration: `(__v4si)(__m128i)(B), (N)))`.
  **L305 CN**: 继续构造周围的表达式或声明：`(__v4si)(__m128i)(B), (N)))`。
- **L306 EN**: Blank line separating nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L307 EN**: Defines macro `_mm_com_epu64(A, B, N)` for conditional compilation, shorthand, or API generation.
  **L307 CN**: 定义宏 `_mm_com_epu64(A, B, N)`，用于条件编译、简写或 API 生成。
- **L308 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpcomuq`.
  **L308 CN**: 继续与可调用符号 `__builtin_ia32_vpcomuq` 相关的逻辑。
- **L309 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(B), (N)))`.
  **L309 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(B), (N)))`。
- **L310 EN**: Blank line separating nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L311 EN**: Defines macro `_mm_com_epi8(A, B, N)` for conditional compilation, shorthand, or API generation.
  **L311 CN**: 定义宏 `_mm_com_epi8(A, B, N)`，用于条件编译、简写或 API 生成。
- **L312 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpcomb`.
  **L312 CN**: 继续与可调用符号 `__builtin_ia32_vpcomb` 相关的逻辑。
- **L313 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (N)))`.
  **L313 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (N)))`。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Defines macro `_mm_com_epi16(A, B, N)` for conditional compilation, shorthand, or API generation.
  **L315 CN**: 定义宏 `_mm_com_epi16(A, B, N)`，用于条件编译、简写或 API 生成。
- **L316 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpcomw`.
  **L316 CN**: 继续与可调用符号 `__builtin_ia32_vpcomw` 相关的逻辑。
- **L317 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(B), (N)))`.
  **L317 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(B), (N)))`。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L319 EN**: Defines macro `_mm_com_epi32(A, B, N)` for conditional compilation, shorthand, or API generation.
  **L319 CN**: 定义宏 `_mm_com_epi32(A, B, N)`，用于条件编译、简写或 API 生成。
- **L320 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpcomd`.
  **L320 CN**: 继续与可调用符号 `__builtin_ia32_vpcomd` 相关的逻辑。

### Lines 321-340

````c
                                  (__v4si)(__m128i)(B), (N)))

#define _mm_com_epi64(A, B, N) \
  ((__m128i)__builtin_ia32_vpcomq((__v2di)(__m128i)(A), \
                                  (__v2di)(__m128i)(B), (N)))

#define _MM_PCOMCTRL_LT    0
#define _MM_PCOMCTRL_LE    1
#define _MM_PCOMCTRL_GT    2
#define _MM_PCOMCTRL_GE    3
#define _MM_PCOMCTRL_EQ    4
#define _MM_PCOMCTRL_NEQ   5
#define _MM_PCOMCTRL_FALSE 6
#define _MM_PCOMCTRL_TRUE  7

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comlt_epu8(__m128i __A, __m128i __B)
{
  return _mm_com_epu8(__A, __B, _MM_PCOMCTRL_LT);
}
````
- **L321 EN**: Continues the surrounding expression or declaration: `(__v4si)(__m128i)(B), (N)))`.
  **L321 CN**: 继续构造周围的表达式或声明：`(__v4si)(__m128i)(B), (N)))`。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Defines macro `_mm_com_epi64(A, B, N)` for conditional compilation, shorthand, or API generation.
  **L323 CN**: 定义宏 `_mm_com_epi64(A, B, N)`，用于条件编译、简写或 API 生成。
- **L324 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpcomq`.
  **L324 CN**: 继续与可调用符号 `__builtin_ia32_vpcomq` 相关的逻辑。
- **L325 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(B), (N)))`.
  **L325 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(B), (N)))`。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Defines macro `_MM_PCOMCTRL_LT` for conditional compilation, shorthand, or API generation.
  **L327 CN**: 定义宏 `_MM_PCOMCTRL_LT`，用于条件编译、简写或 API 生成。
- **L328 EN**: Defines macro `_MM_PCOMCTRL_LE` for conditional compilation, shorthand, or API generation.
  **L328 CN**: 定义宏 `_MM_PCOMCTRL_LE`，用于条件编译、简写或 API 生成。
- **L329 EN**: Defines macro `_MM_PCOMCTRL_GT` for conditional compilation, shorthand, or API generation.
  **L329 CN**: 定义宏 `_MM_PCOMCTRL_GT`，用于条件编译、简写或 API 生成。
- **L330 EN**: Defines macro `_MM_PCOMCTRL_GE` for conditional compilation, shorthand, or API generation.
  **L330 CN**: 定义宏 `_MM_PCOMCTRL_GE`，用于条件编译、简写或 API 生成。
- **L331 EN**: Defines macro `_MM_PCOMCTRL_EQ` for conditional compilation, shorthand, or API generation.
  **L331 CN**: 定义宏 `_MM_PCOMCTRL_EQ`，用于条件编译、简写或 API 生成。
- **L332 EN**: Defines macro `_MM_PCOMCTRL_NEQ` for conditional compilation, shorthand, or API generation.
  **L332 CN**: 定义宏 `_MM_PCOMCTRL_NEQ`，用于条件编译、简写或 API 生成。
- **L333 EN**: Defines macro `_MM_PCOMCTRL_FALSE` for conditional compilation, shorthand, or API generation.
  **L333 CN**: 定义宏 `_MM_PCOMCTRL_FALSE`，用于条件编译、简写或 API 生成。
- **L334 EN**: Defines macro `_MM_PCOMCTRL_TRUE` for conditional compilation, shorthand, or API generation.
  **L334 CN**: 定义宏 `_MM_PCOMCTRL_TRUE`，用于条件编译、简写或 API 生成。
- **L335 EN**: Blank line separating nearby declarations or logic blocks.
  **L335 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L336 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L336 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L337 EN**: Continues logic associated with callable symbol `_mm_comlt_epu8`.
  **L337 CN**: 继续与可调用符号 `_mm_comlt_epu8` 相关的逻辑。
- **L338 EN**: Opens a new lexical scope or compound statement.
  **L338 CN**: 打开一个新的词法作用域或复合语句块。
- **L339 EN**: Returns from the current function with `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_LT)`.
  **L339 CN**: 以 `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_LT)` 从当前函数返回。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。

### Lines 341-360

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comle_epu8(__m128i __A, __m128i __B)
{
  return _mm_com_epu8(__A, __B, _MM_PCOMCTRL_LE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comgt_epu8(__m128i __A, __m128i __B)
{
  return _mm_com_epu8(__A, __B, _MM_PCOMCTRL_GT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comge_epu8(__m128i __A, __m128i __B)
{
  return _mm_com_epu8(__A, __B, _MM_PCOMCTRL_GE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
````
- **L341 EN**: Blank line separating nearby declarations or logic blocks.
  **L341 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L342 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L342 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L343 EN**: Continues logic associated with callable symbol `_mm_comle_epu8`.
  **L343 CN**: 继续与可调用符号 `_mm_comle_epu8` 相关的逻辑。
- **L344 EN**: Opens a new lexical scope or compound statement.
  **L344 CN**: 打开一个新的词法作用域或复合语句块。
- **L345 EN**: Returns from the current function with `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_LE)`.
  **L345 CN**: 以 `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_LE)` 从当前函数返回。
- **L346 EN**: Closes the current lexical scope or compound statement.
  **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Blank line separating nearby declarations or logic blocks.
  **L347 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L348 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L348 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L349 EN**: Continues logic associated with callable symbol `_mm_comgt_epu8`.
  **L349 CN**: 继续与可调用符号 `_mm_comgt_epu8` 相关的逻辑。
- **L350 EN**: Opens a new lexical scope or compound statement.
  **L350 CN**: 打开一个新的词法作用域或复合语句块。
- **L351 EN**: Returns from the current function with `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_GT)`.
  **L351 CN**: 以 `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_GT)` 从当前函数返回。
- **L352 EN**: Closes the current lexical scope or compound statement.
  **L352 CN**: 结束当前词法作用域或复合语句块。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L354 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L354 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L355 EN**: Continues logic associated with callable symbol `_mm_comge_epu8`.
  **L355 CN**: 继续与可调用符号 `_mm_comge_epu8` 相关的逻辑。
- **L356 EN**: Opens a new lexical scope or compound statement.
  **L356 CN**: 打开一个新的词法作用域或复合语句块。
- **L357 EN**: Returns from the current function with `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_GE)`.
  **L357 CN**: 以 `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_GE)` 从当前函数返回。
- **L358 EN**: Closes the current lexical scope or compound statement.
  **L358 CN**: 结束当前词法作用域或复合语句块。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L360 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L360 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。

### Lines 361-380

````c
_mm_comeq_epu8(__m128i __A, __m128i __B)
{
  return _mm_com_epu8(__A, __B, _MM_PCOMCTRL_EQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comneq_epu8(__m128i __A, __m128i __B)
{
  return _mm_com_epu8(__A, __B, _MM_PCOMCTRL_NEQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comfalse_epu8(__m128i __A, __m128i __B)
{
  return _mm_com_epu8(__A, __B, _MM_PCOMCTRL_FALSE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comtrue_epu8(__m128i __A, __m128i __B)
{
````
- **L361 EN**: Continues logic associated with callable symbol `_mm_comeq_epu8`.
  **L361 CN**: 继续与可调用符号 `_mm_comeq_epu8` 相关的逻辑。
- **L362 EN**: Opens a new lexical scope or compound statement.
  **L362 CN**: 打开一个新的词法作用域或复合语句块。
- **L363 EN**: Returns from the current function with `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_EQ)`.
  **L363 CN**: 以 `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_EQ)` 从当前函数返回。
- **L364 EN**: Closes the current lexical scope or compound statement.
  **L364 CN**: 结束当前词法作用域或复合语句块。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L366 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L366 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L367 EN**: Continues logic associated with callable symbol `_mm_comneq_epu8`.
  **L367 CN**: 继续与可调用符号 `_mm_comneq_epu8` 相关的逻辑。
- **L368 EN**: Opens a new lexical scope or compound statement.
  **L368 CN**: 打开一个新的词法作用域或复合语句块。
- **L369 EN**: Returns from the current function with `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_NEQ)`.
  **L369 CN**: 以 `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_NEQ)` 从当前函数返回。
- **L370 EN**: Closes the current lexical scope or compound statement.
  **L370 CN**: 结束当前词法作用域或复合语句块。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L372 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L372 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L373 EN**: Continues logic associated with callable symbol `_mm_comfalse_epu8`.
  **L373 CN**: 继续与可调用符号 `_mm_comfalse_epu8` 相关的逻辑。
- **L374 EN**: Opens a new lexical scope or compound statement.
  **L374 CN**: 打开一个新的词法作用域或复合语句块。
- **L375 EN**: Returns from the current function with `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_FALSE)`.
  **L375 CN**: 以 `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_FALSE)` 从当前函数返回。
- **L376 EN**: Closes the current lexical scope or compound statement.
  **L376 CN**: 结束当前词法作用域或复合语句块。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L378 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L378 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L379 EN**: Continues logic associated with callable symbol `_mm_comtrue_epu8`.
  **L379 CN**: 继续与可调用符号 `_mm_comtrue_epu8` 相关的逻辑。
- **L380 EN**: Opens a new lexical scope or compound statement.
  **L380 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 381-400

````c
  return _mm_com_epu8(__A, __B, _MM_PCOMCTRL_TRUE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comlt_epu16(__m128i __A, __m128i __B)
{
  return _mm_com_epu16(__A, __B, _MM_PCOMCTRL_LT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comle_epu16(__m128i __A, __m128i __B)
{
  return _mm_com_epu16(__A, __B, _MM_PCOMCTRL_LE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comgt_epu16(__m128i __A, __m128i __B)
{
  return _mm_com_epu16(__A, __B, _MM_PCOMCTRL_GT);
}
````
- **L381 EN**: Returns from the current function with `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_TRUE)`.
  **L381 CN**: 以 `_mm_com_epu8(__A, __B, _MM_PCOMCTRL_TRUE)` 从当前函数返回。
- **L382 EN**: Closes the current lexical scope or compound statement.
  **L382 CN**: 结束当前词法作用域或复合语句块。
- **L383 EN**: Blank line separating nearby declarations or logic blocks.
  **L383 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L384 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L384 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L385 EN**: Continues logic associated with callable symbol `_mm_comlt_epu16`.
  **L385 CN**: 继续与可调用符号 `_mm_comlt_epu16` 相关的逻辑。
- **L386 EN**: Opens a new lexical scope or compound statement.
  **L386 CN**: 打开一个新的词法作用域或复合语句块。
- **L387 EN**: Returns from the current function with `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_LT)`.
  **L387 CN**: 以 `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_LT)` 从当前函数返回。
- **L388 EN**: Closes the current lexical scope or compound statement.
  **L388 CN**: 结束当前词法作用域或复合语句块。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L390 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L390 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L391 EN**: Continues logic associated with callable symbol `_mm_comle_epu16`.
  **L391 CN**: 继续与可调用符号 `_mm_comle_epu16` 相关的逻辑。
- **L392 EN**: Opens a new lexical scope or compound statement.
  **L392 CN**: 打开一个新的词法作用域或复合语句块。
- **L393 EN**: Returns from the current function with `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_LE)`.
  **L393 CN**: 以 `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_LE)` 从当前函数返回。
- **L394 EN**: Closes the current lexical scope or compound statement.
  **L394 CN**: 结束当前词法作用域或复合语句块。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L396 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L397 EN**: Continues logic associated with callable symbol `_mm_comgt_epu16`.
  **L397 CN**: 继续与可调用符号 `_mm_comgt_epu16` 相关的逻辑。
- **L398 EN**: Opens a new lexical scope or compound statement.
  **L398 CN**: 打开一个新的词法作用域或复合语句块。
- **L399 EN**: Returns from the current function with `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_GT)`.
  **L399 CN**: 以 `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_GT)` 从当前函数返回。
- **L400 EN**: Closes the current lexical scope or compound statement.
  **L400 CN**: 结束当前词法作用域或复合语句块。

### Lines 401-420

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comge_epu16(__m128i __A, __m128i __B)
{
  return _mm_com_epu16(__A, __B, _MM_PCOMCTRL_GE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comeq_epu16(__m128i __A, __m128i __B)
{
  return _mm_com_epu16(__A, __B, _MM_PCOMCTRL_EQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comneq_epu16(__m128i __A, __m128i __B)
{
  return _mm_com_epu16(__A, __B, _MM_PCOMCTRL_NEQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
````
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L402 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L402 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L403 EN**: Continues logic associated with callable symbol `_mm_comge_epu16`.
  **L403 CN**: 继续与可调用符号 `_mm_comge_epu16` 相关的逻辑。
- **L404 EN**: Opens a new lexical scope or compound statement.
  **L404 CN**: 打开一个新的词法作用域或复合语句块。
- **L405 EN**: Returns from the current function with `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_GE)`.
  **L405 CN**: 以 `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_GE)` 从当前函数返回。
- **L406 EN**: Closes the current lexical scope or compound statement.
  **L406 CN**: 结束当前词法作用域或复合语句块。
- **L407 EN**: Blank line separating nearby declarations or logic blocks.
  **L407 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L408 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L408 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L409 EN**: Continues logic associated with callable symbol `_mm_comeq_epu16`.
  **L409 CN**: 继续与可调用符号 `_mm_comeq_epu16` 相关的逻辑。
- **L410 EN**: Opens a new lexical scope or compound statement.
  **L410 CN**: 打开一个新的词法作用域或复合语句块。
- **L411 EN**: Returns from the current function with `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_EQ)`.
  **L411 CN**: 以 `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_EQ)` 从当前函数返回。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L414 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L415 EN**: Continues logic associated with callable symbol `_mm_comneq_epu16`.
  **L415 CN**: 继续与可调用符号 `_mm_comneq_epu16` 相关的逻辑。
- **L416 EN**: Opens a new lexical scope or compound statement.
  **L416 CN**: 打开一个新的词法作用域或复合语句块。
- **L417 EN**: Returns from the current function with `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_NEQ)`.
  **L417 CN**: 以 `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_NEQ)` 从当前函数返回。
- **L418 EN**: Closes the current lexical scope or compound statement.
  **L418 CN**: 结束当前词法作用域或复合语句块。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L420 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L420 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。

### Lines 421-440

````c
_mm_comfalse_epu16(__m128i __A, __m128i __B)
{
  return _mm_com_epu16(__A, __B, _MM_PCOMCTRL_FALSE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comtrue_epu16(__m128i __A, __m128i __B)
{
  return _mm_com_epu16(__A, __B, _MM_PCOMCTRL_TRUE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comlt_epu32(__m128i __A, __m128i __B)
{
  return _mm_com_epu32(__A, __B, _MM_PCOMCTRL_LT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comle_epu32(__m128i __A, __m128i __B)
{
````
- **L421 EN**: Continues logic associated with callable symbol `_mm_comfalse_epu16`.
  **L421 CN**: 继续与可调用符号 `_mm_comfalse_epu16` 相关的逻辑。
- **L422 EN**: Opens a new lexical scope or compound statement.
  **L422 CN**: 打开一个新的词法作用域或复合语句块。
- **L423 EN**: Returns from the current function with `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_FALSE)`.
  **L423 CN**: 以 `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_FALSE)` 从当前函数返回。
- **L424 EN**: Closes the current lexical scope or compound statement.
  **L424 CN**: 结束当前词法作用域或复合语句块。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L426 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L426 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L427 EN**: Continues logic associated with callable symbol `_mm_comtrue_epu16`.
  **L427 CN**: 继续与可调用符号 `_mm_comtrue_epu16` 相关的逻辑。
- **L428 EN**: Opens a new lexical scope or compound statement.
  **L428 CN**: 打开一个新的词法作用域或复合语句块。
- **L429 EN**: Returns from the current function with `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_TRUE)`.
  **L429 CN**: 以 `_mm_com_epu16(__A, __B, _MM_PCOMCTRL_TRUE)` 从当前函数返回。
- **L430 EN**: Closes the current lexical scope or compound statement.
  **L430 CN**: 结束当前词法作用域或复合语句块。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L432 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L432 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L433 EN**: Continues logic associated with callable symbol `_mm_comlt_epu32`.
  **L433 CN**: 继续与可调用符号 `_mm_comlt_epu32` 相关的逻辑。
- **L434 EN**: Opens a new lexical scope or compound statement.
  **L434 CN**: 打开一个新的词法作用域或复合语句块。
- **L435 EN**: Returns from the current function with `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_LT)`.
  **L435 CN**: 以 `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_LT)` 从当前函数返回。
- **L436 EN**: Closes the current lexical scope or compound statement.
  **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L438 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L439 EN**: Continues logic associated with callable symbol `_mm_comle_epu32`.
  **L439 CN**: 继续与可调用符号 `_mm_comle_epu32` 相关的逻辑。
- **L440 EN**: Opens a new lexical scope or compound statement.
  **L440 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 441-460

````c
  return _mm_com_epu32(__A, __B, _MM_PCOMCTRL_LE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comgt_epu32(__m128i __A, __m128i __B)
{
  return _mm_com_epu32(__A, __B, _MM_PCOMCTRL_GT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comge_epu32(__m128i __A, __m128i __B)
{
  return _mm_com_epu32(__A, __B, _MM_PCOMCTRL_GE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comeq_epu32(__m128i __A, __m128i __B)
{
  return _mm_com_epu32(__A, __B, _MM_PCOMCTRL_EQ);
}
````
- **L441 EN**: Returns from the current function with `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_LE)`.
  **L441 CN**: 以 `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_LE)` 从当前函数返回。
- **L442 EN**: Closes the current lexical scope or compound statement.
  **L442 CN**: 结束当前词法作用域或复合语句块。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L444 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L444 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L445 EN**: Continues logic associated with callable symbol `_mm_comgt_epu32`.
  **L445 CN**: 继续与可调用符号 `_mm_comgt_epu32` 相关的逻辑。
- **L446 EN**: Opens a new lexical scope or compound statement.
  **L446 CN**: 打开一个新的词法作用域或复合语句块。
- **L447 EN**: Returns from the current function with `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_GT)`.
  **L447 CN**: 以 `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_GT)` 从当前函数返回。
- **L448 EN**: Closes the current lexical scope or compound statement.
  **L448 CN**: 结束当前词法作用域或复合语句块。
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L450 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L450 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L451 EN**: Continues logic associated with callable symbol `_mm_comge_epu32`.
  **L451 CN**: 继续与可调用符号 `_mm_comge_epu32` 相关的逻辑。
- **L452 EN**: Opens a new lexical scope or compound statement.
  **L452 CN**: 打开一个新的词法作用域或复合语句块。
- **L453 EN**: Returns from the current function with `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_GE)`.
  **L453 CN**: 以 `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_GE)` 从当前函数返回。
- **L454 EN**: Closes the current lexical scope or compound statement.
  **L454 CN**: 结束当前词法作用域或复合语句块。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L456 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L456 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L457 EN**: Continues logic associated with callable symbol `_mm_comeq_epu32`.
  **L457 CN**: 继续与可调用符号 `_mm_comeq_epu32` 相关的逻辑。
- **L458 EN**: Opens a new lexical scope or compound statement.
  **L458 CN**: 打开一个新的词法作用域或复合语句块。
- **L459 EN**: Returns from the current function with `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_EQ)`.
  **L459 CN**: 以 `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_EQ)` 从当前函数返回。
- **L460 EN**: Closes the current lexical scope or compound statement.
  **L460 CN**: 结束当前词法作用域或复合语句块。

### Lines 461-480

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comneq_epu32(__m128i __A, __m128i __B)
{
  return _mm_com_epu32(__A, __B, _MM_PCOMCTRL_NEQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comfalse_epu32(__m128i __A, __m128i __B)
{
  return _mm_com_epu32(__A, __B, _MM_PCOMCTRL_FALSE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comtrue_epu32(__m128i __A, __m128i __B)
{
  return _mm_com_epu32(__A, __B, _MM_PCOMCTRL_TRUE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
````
- **L461 EN**: Blank line separating nearby declarations or logic blocks.
  **L461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L462 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L462 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L463 EN**: Continues logic associated with callable symbol `_mm_comneq_epu32`.
  **L463 CN**: 继续与可调用符号 `_mm_comneq_epu32` 相关的逻辑。
- **L464 EN**: Opens a new lexical scope or compound statement.
  **L464 CN**: 打开一个新的词法作用域或复合语句块。
- **L465 EN**: Returns from the current function with `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_NEQ)`.
  **L465 CN**: 以 `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_NEQ)` 从当前函数返回。
- **L466 EN**: Closes the current lexical scope or compound statement.
  **L466 CN**: 结束当前词法作用域或复合语句块。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L468 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L469 EN**: Continues logic associated with callable symbol `_mm_comfalse_epu32`.
  **L469 CN**: 继续与可调用符号 `_mm_comfalse_epu32` 相关的逻辑。
- **L470 EN**: Opens a new lexical scope or compound statement.
  **L470 CN**: 打开一个新的词法作用域或复合语句块。
- **L471 EN**: Returns from the current function with `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_FALSE)`.
  **L471 CN**: 以 `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_FALSE)` 从当前函数返回。
- **L472 EN**: Closes the current lexical scope or compound statement.
  **L472 CN**: 结束当前词法作用域或复合语句块。
- **L473 EN**: Blank line separating nearby declarations or logic blocks.
  **L473 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L474 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L474 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L475 EN**: Continues logic associated with callable symbol `_mm_comtrue_epu32`.
  **L475 CN**: 继续与可调用符号 `_mm_comtrue_epu32` 相关的逻辑。
- **L476 EN**: Opens a new lexical scope or compound statement.
  **L476 CN**: 打开一个新的词法作用域或复合语句块。
- **L477 EN**: Returns from the current function with `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_TRUE)`.
  **L477 CN**: 以 `_mm_com_epu32(__A, __B, _MM_PCOMCTRL_TRUE)` 从当前函数返回。
- **L478 EN**: Closes the current lexical scope or compound statement.
  **L478 CN**: 结束当前词法作用域或复合语句块。
- **L479 EN**: Blank line separating nearby declarations or logic blocks.
  **L479 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L480 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L480 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。

### Lines 481-500

````c
_mm_comlt_epu64(__m128i __A, __m128i __B)
{
  return _mm_com_epu64(__A, __B, _MM_PCOMCTRL_LT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comle_epu64(__m128i __A, __m128i __B)
{
  return _mm_com_epu64(__A, __B, _MM_PCOMCTRL_LE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comgt_epu64(__m128i __A, __m128i __B)
{
  return _mm_com_epu64(__A, __B, _MM_PCOMCTRL_GT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comge_epu64(__m128i __A, __m128i __B)
{
````
- **L481 EN**: Continues logic associated with callable symbol `_mm_comlt_epu64`.
  **L481 CN**: 继续与可调用符号 `_mm_comlt_epu64` 相关的逻辑。
- **L482 EN**: Opens a new lexical scope or compound statement.
  **L482 CN**: 打开一个新的词法作用域或复合语句块。
- **L483 EN**: Returns from the current function with `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_LT)`.
  **L483 CN**: 以 `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_LT)` 从当前函数返回。
- **L484 EN**: Closes the current lexical scope or compound statement.
  **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Blank line separating nearby declarations or logic blocks.
  **L485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L486 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L486 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L487 EN**: Continues logic associated with callable symbol `_mm_comle_epu64`.
  **L487 CN**: 继续与可调用符号 `_mm_comle_epu64` 相关的逻辑。
- **L488 EN**: Opens a new lexical scope or compound statement.
  **L488 CN**: 打开一个新的词法作用域或复合语句块。
- **L489 EN**: Returns from the current function with `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_LE)`.
  **L489 CN**: 以 `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_LE)` 从当前函数返回。
- **L490 EN**: Closes the current lexical scope or compound statement.
  **L490 CN**: 结束当前词法作用域或复合语句块。
- **L491 EN**: Blank line separating nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L492 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L492 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L493 EN**: Continues logic associated with callable symbol `_mm_comgt_epu64`.
  **L493 CN**: 继续与可调用符号 `_mm_comgt_epu64` 相关的逻辑。
- **L494 EN**: Opens a new lexical scope or compound statement.
  **L494 CN**: 打开一个新的词法作用域或复合语句块。
- **L495 EN**: Returns from the current function with `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_GT)`.
  **L495 CN**: 以 `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_GT)` 从当前函数返回。
- **L496 EN**: Closes the current lexical scope or compound statement.
  **L496 CN**: 结束当前词法作用域或复合语句块。
- **L497 EN**: Blank line separating nearby declarations or logic blocks.
  **L497 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L498 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L498 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L499 EN**: Continues logic associated with callable symbol `_mm_comge_epu64`.
  **L499 CN**: 继续与可调用符号 `_mm_comge_epu64` 相关的逻辑。
- **L500 EN**: Opens a new lexical scope or compound statement.
  **L500 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 501-520

````c
  return _mm_com_epu64(__A, __B, _MM_PCOMCTRL_GE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comeq_epu64(__m128i __A, __m128i __B)
{
  return _mm_com_epu64(__A, __B, _MM_PCOMCTRL_EQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comneq_epu64(__m128i __A, __m128i __B)
{
  return _mm_com_epu64(__A, __B, _MM_PCOMCTRL_NEQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comfalse_epu64(__m128i __A, __m128i __B)
{
  return _mm_com_epu64(__A, __B, _MM_PCOMCTRL_FALSE);
}
````
- **L501 EN**: Returns from the current function with `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_GE)`.
  **L501 CN**: 以 `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_GE)` 从当前函数返回。
- **L502 EN**: Closes the current lexical scope or compound statement.
  **L502 CN**: 结束当前词法作用域或复合语句块。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L504 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L504 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L505 EN**: Continues logic associated with callable symbol `_mm_comeq_epu64`.
  **L505 CN**: 继续与可调用符号 `_mm_comeq_epu64` 相关的逻辑。
- **L506 EN**: Opens a new lexical scope or compound statement.
  **L506 CN**: 打开一个新的词法作用域或复合语句块。
- **L507 EN**: Returns from the current function with `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_EQ)`.
  **L507 CN**: 以 `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_EQ)` 从当前函数返回。
- **L508 EN**: Closes the current lexical scope or compound statement.
  **L508 CN**: 结束当前词法作用域或复合语句块。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L510 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L510 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L511 EN**: Continues logic associated with callable symbol `_mm_comneq_epu64`.
  **L511 CN**: 继续与可调用符号 `_mm_comneq_epu64` 相关的逻辑。
- **L512 EN**: Opens a new lexical scope or compound statement.
  **L512 CN**: 打开一个新的词法作用域或复合语句块。
- **L513 EN**: Returns from the current function with `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_NEQ)`.
  **L513 CN**: 以 `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_NEQ)` 从当前函数返回。
- **L514 EN**: Closes the current lexical scope or compound statement.
  **L514 CN**: 结束当前词法作用域或复合语句块。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L516 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L516 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L517 EN**: Continues logic associated with callable symbol `_mm_comfalse_epu64`.
  **L517 CN**: 继续与可调用符号 `_mm_comfalse_epu64` 相关的逻辑。
- **L518 EN**: Opens a new lexical scope or compound statement.
  **L518 CN**: 打开一个新的词法作用域或复合语句块。
- **L519 EN**: Returns from the current function with `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_FALSE)`.
  **L519 CN**: 以 `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_FALSE)` 从当前函数返回。
- **L520 EN**: Closes the current lexical scope or compound statement.
  **L520 CN**: 结束当前词法作用域或复合语句块。

### Lines 521-540

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comtrue_epu64(__m128i __A, __m128i __B)
{
  return _mm_com_epu64(__A, __B, _MM_PCOMCTRL_TRUE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comlt_epi8(__m128i __A, __m128i __B)
{
  return _mm_com_epi8(__A, __B, _MM_PCOMCTRL_LT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comle_epi8(__m128i __A, __m128i __B)
{
  return _mm_com_epi8(__A, __B, _MM_PCOMCTRL_LE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
````
- **L521 EN**: Blank line separating nearby declarations or logic blocks.
  **L521 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L522 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L522 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L523 EN**: Continues logic associated with callable symbol `_mm_comtrue_epu64`.
  **L523 CN**: 继续与可调用符号 `_mm_comtrue_epu64` 相关的逻辑。
- **L524 EN**: Opens a new lexical scope or compound statement.
  **L524 CN**: 打开一个新的词法作用域或复合语句块。
- **L525 EN**: Returns from the current function with `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_TRUE)`.
  **L525 CN**: 以 `_mm_com_epu64(__A, __B, _MM_PCOMCTRL_TRUE)` 从当前函数返回。
- **L526 EN**: Closes the current lexical scope or compound statement.
  **L526 CN**: 结束当前词法作用域或复合语句块。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L528 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L528 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L529 EN**: Continues logic associated with callable symbol `_mm_comlt_epi8`.
  **L529 CN**: 继续与可调用符号 `_mm_comlt_epi8` 相关的逻辑。
- **L530 EN**: Opens a new lexical scope or compound statement.
  **L530 CN**: 打开一个新的词法作用域或复合语句块。
- **L531 EN**: Returns from the current function with `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_LT)`.
  **L531 CN**: 以 `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_LT)` 从当前函数返回。
- **L532 EN**: Closes the current lexical scope or compound statement.
  **L532 CN**: 结束当前词法作用域或复合语句块。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L534 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L534 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L535 EN**: Continues logic associated with callable symbol `_mm_comle_epi8`.
  **L535 CN**: 继续与可调用符号 `_mm_comle_epi8` 相关的逻辑。
- **L536 EN**: Opens a new lexical scope or compound statement.
  **L536 CN**: 打开一个新的词法作用域或复合语句块。
- **L537 EN**: Returns from the current function with `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_LE)`.
  **L537 CN**: 以 `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_LE)` 从当前函数返回。
- **L538 EN**: Closes the current lexical scope or compound statement.
  **L538 CN**: 结束当前词法作用域或复合语句块。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L540 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L540 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。

### Lines 541-560

````c
_mm_comgt_epi8(__m128i __A, __m128i __B)
{
  return _mm_com_epi8(__A, __B, _MM_PCOMCTRL_GT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comge_epi8(__m128i __A, __m128i __B)
{
  return _mm_com_epi8(__A, __B, _MM_PCOMCTRL_GE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comeq_epi8(__m128i __A, __m128i __B)
{
  return _mm_com_epi8(__A, __B, _MM_PCOMCTRL_EQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comneq_epi8(__m128i __A, __m128i __B)
{
````
- **L541 EN**: Continues logic associated with callable symbol `_mm_comgt_epi8`.
  **L541 CN**: 继续与可调用符号 `_mm_comgt_epi8` 相关的逻辑。
- **L542 EN**: Opens a new lexical scope or compound statement.
  **L542 CN**: 打开一个新的词法作用域或复合语句块。
- **L543 EN**: Returns from the current function with `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_GT)`.
  **L543 CN**: 以 `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_GT)` 从当前函数返回。
- **L544 EN**: Closes the current lexical scope or compound statement.
  **L544 CN**: 结束当前词法作用域或复合语句块。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L546 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L546 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L547 EN**: Continues logic associated with callable symbol `_mm_comge_epi8`.
  **L547 CN**: 继续与可调用符号 `_mm_comge_epi8` 相关的逻辑。
- **L548 EN**: Opens a new lexical scope or compound statement.
  **L548 CN**: 打开一个新的词法作用域或复合语句块。
- **L549 EN**: Returns from the current function with `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_GE)`.
  **L549 CN**: 以 `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_GE)` 从当前函数返回。
- **L550 EN**: Closes the current lexical scope or compound statement.
  **L550 CN**: 结束当前词法作用域或复合语句块。
- **L551 EN**: Blank line separating nearby declarations or logic blocks.
  **L551 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L552 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L552 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L553 EN**: Continues logic associated with callable symbol `_mm_comeq_epi8`.
  **L553 CN**: 继续与可调用符号 `_mm_comeq_epi8` 相关的逻辑。
- **L554 EN**: Opens a new lexical scope or compound statement.
  **L554 CN**: 打开一个新的词法作用域或复合语句块。
- **L555 EN**: Returns from the current function with `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_EQ)`.
  **L555 CN**: 以 `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_EQ)` 从当前函数返回。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L558 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L558 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L559 EN**: Continues logic associated with callable symbol `_mm_comneq_epi8`.
  **L559 CN**: 继续与可调用符号 `_mm_comneq_epi8` 相关的逻辑。
- **L560 EN**: Opens a new lexical scope or compound statement.
  **L560 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 561-580

````c
  return _mm_com_epi8(__A, __B, _MM_PCOMCTRL_NEQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comfalse_epi8(__m128i __A, __m128i __B)
{
  return _mm_com_epi8(__A, __B, _MM_PCOMCTRL_FALSE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comtrue_epi8(__m128i __A, __m128i __B)
{
  return _mm_com_epi8(__A, __B, _MM_PCOMCTRL_TRUE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comlt_epi16(__m128i __A, __m128i __B)
{
  return _mm_com_epi16(__A, __B, _MM_PCOMCTRL_LT);
}
````
- **L561 EN**: Returns from the current function with `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_NEQ)`.
  **L561 CN**: 以 `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_NEQ)` 从当前函数返回。
- **L562 EN**: Closes the current lexical scope or compound statement.
  **L562 CN**: 结束当前词法作用域或复合语句块。
- **L563 EN**: Blank line separating nearby declarations or logic blocks.
  **L563 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L564 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L564 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L565 EN**: Continues logic associated with callable symbol `_mm_comfalse_epi8`.
  **L565 CN**: 继续与可调用符号 `_mm_comfalse_epi8` 相关的逻辑。
- **L566 EN**: Opens a new lexical scope or compound statement.
  **L566 CN**: 打开一个新的词法作用域或复合语句块。
- **L567 EN**: Returns from the current function with `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_FALSE)`.
  **L567 CN**: 以 `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_FALSE)` 从当前函数返回。
- **L568 EN**: Closes the current lexical scope or compound statement.
  **L568 CN**: 结束当前词法作用域或复合语句块。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L570 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L570 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L571 EN**: Continues logic associated with callable symbol `_mm_comtrue_epi8`.
  **L571 CN**: 继续与可调用符号 `_mm_comtrue_epi8` 相关的逻辑。
- **L572 EN**: Opens a new lexical scope or compound statement.
  **L572 CN**: 打开一个新的词法作用域或复合语句块。
- **L573 EN**: Returns from the current function with `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_TRUE)`.
  **L573 CN**: 以 `_mm_com_epi8(__A, __B, _MM_PCOMCTRL_TRUE)` 从当前函数返回。
- **L574 EN**: Closes the current lexical scope or compound statement.
  **L574 CN**: 结束当前词法作用域或复合语句块。
- **L575 EN**: Blank line separating nearby declarations or logic blocks.
  **L575 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L576 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L576 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L577 EN**: Continues logic associated with callable symbol `_mm_comlt_epi16`.
  **L577 CN**: 继续与可调用符号 `_mm_comlt_epi16` 相关的逻辑。
- **L578 EN**: Opens a new lexical scope or compound statement.
  **L578 CN**: 打开一个新的词法作用域或复合语句块。
- **L579 EN**: Returns from the current function with `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_LT)`.
  **L579 CN**: 以 `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_LT)` 从当前函数返回。
- **L580 EN**: Closes the current lexical scope or compound statement.
  **L580 CN**: 结束当前词法作用域或复合语句块。

### Lines 581-600

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comle_epi16(__m128i __A, __m128i __B)
{
  return _mm_com_epi16(__A, __B, _MM_PCOMCTRL_LE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comgt_epi16(__m128i __A, __m128i __B)
{
  return _mm_com_epi16(__A, __B, _MM_PCOMCTRL_GT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comge_epi16(__m128i __A, __m128i __B)
{
  return _mm_com_epi16(__A, __B, _MM_PCOMCTRL_GE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
````
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L582 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L583 EN**: Continues logic associated with callable symbol `_mm_comle_epi16`.
  **L583 CN**: 继续与可调用符号 `_mm_comle_epi16` 相关的逻辑。
- **L584 EN**: Opens a new lexical scope or compound statement.
  **L584 CN**: 打开一个新的词法作用域或复合语句块。
- **L585 EN**: Returns from the current function with `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_LE)`.
  **L585 CN**: 以 `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_LE)` 从当前函数返回。
- **L586 EN**: Closes the current lexical scope or compound statement.
  **L586 CN**: 结束当前词法作用域或复合语句块。
- **L587 EN**: Blank line separating nearby declarations or logic blocks.
  **L587 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L588 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L588 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L589 EN**: Continues logic associated with callable symbol `_mm_comgt_epi16`.
  **L589 CN**: 继续与可调用符号 `_mm_comgt_epi16` 相关的逻辑。
- **L590 EN**: Opens a new lexical scope or compound statement.
  **L590 CN**: 打开一个新的词法作用域或复合语句块。
- **L591 EN**: Returns from the current function with `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_GT)`.
  **L591 CN**: 以 `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_GT)` 从当前函数返回。
- **L592 EN**: Closes the current lexical scope or compound statement.
  **L592 CN**: 结束当前词法作用域或复合语句块。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L594 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L595 EN**: Continues logic associated with callable symbol `_mm_comge_epi16`.
  **L595 CN**: 继续与可调用符号 `_mm_comge_epi16` 相关的逻辑。
- **L596 EN**: Opens a new lexical scope or compound statement.
  **L596 CN**: 打开一个新的词法作用域或复合语句块。
- **L597 EN**: Returns from the current function with `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_GE)`.
  **L597 CN**: 以 `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_GE)` 从当前函数返回。
- **L598 EN**: Closes the current lexical scope or compound statement.
  **L598 CN**: 结束当前词法作用域或复合语句块。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L600 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L600 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。

### Lines 601-620

````c
_mm_comeq_epi16(__m128i __A, __m128i __B)
{
  return _mm_com_epi16(__A, __B, _MM_PCOMCTRL_EQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comneq_epi16(__m128i __A, __m128i __B)
{
  return _mm_com_epi16(__A, __B, _MM_PCOMCTRL_NEQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comfalse_epi16(__m128i __A, __m128i __B)
{
  return _mm_com_epi16(__A, __B, _MM_PCOMCTRL_FALSE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comtrue_epi16(__m128i __A, __m128i __B)
{
````
- **L601 EN**: Continues logic associated with callable symbol `_mm_comeq_epi16`.
  **L601 CN**: 继续与可调用符号 `_mm_comeq_epi16` 相关的逻辑。
- **L602 EN**: Opens a new lexical scope or compound statement.
  **L602 CN**: 打开一个新的词法作用域或复合语句块。
- **L603 EN**: Returns from the current function with `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_EQ)`.
  **L603 CN**: 以 `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_EQ)` 从当前函数返回。
- **L604 EN**: Closes the current lexical scope or compound statement.
  **L604 CN**: 结束当前词法作用域或复合语句块。
- **L605 EN**: Blank line separating nearby declarations or logic blocks.
  **L605 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L606 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L606 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L607 EN**: Continues logic associated with callable symbol `_mm_comneq_epi16`.
  **L607 CN**: 继续与可调用符号 `_mm_comneq_epi16` 相关的逻辑。
- **L608 EN**: Opens a new lexical scope or compound statement.
  **L608 CN**: 打开一个新的词法作用域或复合语句块。
- **L609 EN**: Returns from the current function with `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_NEQ)`.
  **L609 CN**: 以 `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_NEQ)` 从当前函数返回。
- **L610 EN**: Closes the current lexical scope or compound statement.
  **L610 CN**: 结束当前词法作用域或复合语句块。
- **L611 EN**: Blank line separating nearby declarations or logic blocks.
  **L611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L612 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L612 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L613 EN**: Continues logic associated with callable symbol `_mm_comfalse_epi16`.
  **L613 CN**: 继续与可调用符号 `_mm_comfalse_epi16` 相关的逻辑。
- **L614 EN**: Opens a new lexical scope or compound statement.
  **L614 CN**: 打开一个新的词法作用域或复合语句块。
- **L615 EN**: Returns from the current function with `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_FALSE)`.
  **L615 CN**: 以 `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_FALSE)` 从当前函数返回。
- **L616 EN**: Closes the current lexical scope or compound statement.
  **L616 CN**: 结束当前词法作用域或复合语句块。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L618 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L618 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L619 EN**: Continues logic associated with callable symbol `_mm_comtrue_epi16`.
  **L619 CN**: 继续与可调用符号 `_mm_comtrue_epi16` 相关的逻辑。
- **L620 EN**: Opens a new lexical scope or compound statement.
  **L620 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 621-640

````c
  return _mm_com_epi16(__A, __B, _MM_PCOMCTRL_TRUE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comlt_epi32(__m128i __A, __m128i __B)
{
  return _mm_com_epi32(__A, __B, _MM_PCOMCTRL_LT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comle_epi32(__m128i __A, __m128i __B)
{
  return _mm_com_epi32(__A, __B, _MM_PCOMCTRL_LE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comgt_epi32(__m128i __A, __m128i __B)
{
  return _mm_com_epi32(__A, __B, _MM_PCOMCTRL_GT);
}
````
- **L621 EN**: Returns from the current function with `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_TRUE)`.
  **L621 CN**: 以 `_mm_com_epi16(__A, __B, _MM_PCOMCTRL_TRUE)` 从当前函数返回。
- **L622 EN**: Closes the current lexical scope or compound statement.
  **L622 CN**: 结束当前词法作用域或复合语句块。
- **L623 EN**: Blank line separating nearby declarations or logic blocks.
  **L623 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L624 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L624 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L625 EN**: Continues logic associated with callable symbol `_mm_comlt_epi32`.
  **L625 CN**: 继续与可调用符号 `_mm_comlt_epi32` 相关的逻辑。
- **L626 EN**: Opens a new lexical scope or compound statement.
  **L626 CN**: 打开一个新的词法作用域或复合语句块。
- **L627 EN**: Returns from the current function with `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_LT)`.
  **L627 CN**: 以 `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_LT)` 从当前函数返回。
- **L628 EN**: Closes the current lexical scope or compound statement.
  **L628 CN**: 结束当前词法作用域或复合语句块。
- **L629 EN**: Blank line separating nearby declarations or logic blocks.
  **L629 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L630 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L630 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L631 EN**: Continues logic associated with callable symbol `_mm_comle_epi32`.
  **L631 CN**: 继续与可调用符号 `_mm_comle_epi32` 相关的逻辑。
- **L632 EN**: Opens a new lexical scope or compound statement.
  **L632 CN**: 打开一个新的词法作用域或复合语句块。
- **L633 EN**: Returns from the current function with `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_LE)`.
  **L633 CN**: 以 `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_LE)` 从当前函数返回。
- **L634 EN**: Closes the current lexical scope or compound statement.
  **L634 CN**: 结束当前词法作用域或复合语句块。
- **L635 EN**: Blank line separating nearby declarations or logic blocks.
  **L635 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L636 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L636 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L637 EN**: Continues logic associated with callable symbol `_mm_comgt_epi32`.
  **L637 CN**: 继续与可调用符号 `_mm_comgt_epi32` 相关的逻辑。
- **L638 EN**: Opens a new lexical scope or compound statement.
  **L638 CN**: 打开一个新的词法作用域或复合语句块。
- **L639 EN**: Returns from the current function with `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_GT)`.
  **L639 CN**: 以 `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_GT)` 从当前函数返回。
- **L640 EN**: Closes the current lexical scope or compound statement.
  **L640 CN**: 结束当前词法作用域或复合语句块。

### Lines 641-660

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comge_epi32(__m128i __A, __m128i __B)
{
  return _mm_com_epi32(__A, __B, _MM_PCOMCTRL_GE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comeq_epi32(__m128i __A, __m128i __B)
{
  return _mm_com_epi32(__A, __B, _MM_PCOMCTRL_EQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comneq_epi32(__m128i __A, __m128i __B)
{
  return _mm_com_epi32(__A, __B, _MM_PCOMCTRL_NEQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
````
- **L641 EN**: Blank line separating nearby declarations or logic blocks.
  **L641 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L642 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L642 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L643 EN**: Continues logic associated with callable symbol `_mm_comge_epi32`.
  **L643 CN**: 继续与可调用符号 `_mm_comge_epi32` 相关的逻辑。
- **L644 EN**: Opens a new lexical scope or compound statement.
  **L644 CN**: 打开一个新的词法作用域或复合语句块。
- **L645 EN**: Returns from the current function with `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_GE)`.
  **L645 CN**: 以 `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_GE)` 从当前函数返回。
- **L646 EN**: Closes the current lexical scope or compound statement.
  **L646 CN**: 结束当前词法作用域或复合语句块。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L648 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L648 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L649 EN**: Continues logic associated with callable symbol `_mm_comeq_epi32`.
  **L649 CN**: 继续与可调用符号 `_mm_comeq_epi32` 相关的逻辑。
- **L650 EN**: Opens a new lexical scope or compound statement.
  **L650 CN**: 打开一个新的词法作用域或复合语句块。
- **L651 EN**: Returns from the current function with `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_EQ)`.
  **L651 CN**: 以 `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_EQ)` 从当前函数返回。
- **L652 EN**: Closes the current lexical scope or compound statement.
  **L652 CN**: 结束当前词法作用域或复合语句块。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L654 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L655 EN**: Continues logic associated with callable symbol `_mm_comneq_epi32`.
  **L655 CN**: 继续与可调用符号 `_mm_comneq_epi32` 相关的逻辑。
- **L656 EN**: Opens a new lexical scope or compound statement.
  **L656 CN**: 打开一个新的词法作用域或复合语句块。
- **L657 EN**: Returns from the current function with `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_NEQ)`.
  **L657 CN**: 以 `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_NEQ)` 从当前函数返回。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L660 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L660 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。

### Lines 661-680

````c
_mm_comfalse_epi32(__m128i __A, __m128i __B)
{
  return _mm_com_epi32(__A, __B, _MM_PCOMCTRL_FALSE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comtrue_epi32(__m128i __A, __m128i __B)
{
  return _mm_com_epi32(__A, __B, _MM_PCOMCTRL_TRUE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comlt_epi64(__m128i __A, __m128i __B)
{
  return _mm_com_epi64(__A, __B, _MM_PCOMCTRL_LT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comle_epi64(__m128i __A, __m128i __B)
{
````
- **L661 EN**: Continues logic associated with callable symbol `_mm_comfalse_epi32`.
  **L661 CN**: 继续与可调用符号 `_mm_comfalse_epi32` 相关的逻辑。
- **L662 EN**: Opens a new lexical scope or compound statement.
  **L662 CN**: 打开一个新的词法作用域或复合语句块。
- **L663 EN**: Returns from the current function with `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_FALSE)`.
  **L663 CN**: 以 `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_FALSE)` 从当前函数返回。
- **L664 EN**: Closes the current lexical scope or compound statement.
  **L664 CN**: 结束当前词法作用域或复合语句块。
- **L665 EN**: Blank line separating nearby declarations or logic blocks.
  **L665 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L666 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L666 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L667 EN**: Continues logic associated with callable symbol `_mm_comtrue_epi32`.
  **L667 CN**: 继续与可调用符号 `_mm_comtrue_epi32` 相关的逻辑。
- **L668 EN**: Opens a new lexical scope or compound statement.
  **L668 CN**: 打开一个新的词法作用域或复合语句块。
- **L669 EN**: Returns from the current function with `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_TRUE)`.
  **L669 CN**: 以 `_mm_com_epi32(__A, __B, _MM_PCOMCTRL_TRUE)` 从当前函数返回。
- **L670 EN**: Closes the current lexical scope or compound statement.
  **L670 CN**: 结束当前词法作用域或复合语句块。
- **L671 EN**: Blank line separating nearby declarations or logic blocks.
  **L671 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L672 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L672 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L673 EN**: Continues logic associated with callable symbol `_mm_comlt_epi64`.
  **L673 CN**: 继续与可调用符号 `_mm_comlt_epi64` 相关的逻辑。
- **L674 EN**: Opens a new lexical scope or compound statement.
  **L674 CN**: 打开一个新的词法作用域或复合语句块。
- **L675 EN**: Returns from the current function with `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_LT)`.
  **L675 CN**: 以 `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_LT)` 从当前函数返回。
- **L676 EN**: Closes the current lexical scope or compound statement.
  **L676 CN**: 结束当前词法作用域或复合语句块。
- **L677 EN**: Blank line separating nearby declarations or logic blocks.
  **L677 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L678 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L678 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L679 EN**: Continues logic associated with callable symbol `_mm_comle_epi64`.
  **L679 CN**: 继续与可调用符号 `_mm_comle_epi64` 相关的逻辑。
- **L680 EN**: Opens a new lexical scope or compound statement.
  **L680 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 681-700

````c
  return _mm_com_epi64(__A, __B, _MM_PCOMCTRL_LE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comgt_epi64(__m128i __A, __m128i __B)
{
  return _mm_com_epi64(__A, __B, _MM_PCOMCTRL_GT);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comge_epi64(__m128i __A, __m128i __B)
{
  return _mm_com_epi64(__A, __B, _MM_PCOMCTRL_GE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comeq_epi64(__m128i __A, __m128i __B)
{
  return _mm_com_epi64(__A, __B, _MM_PCOMCTRL_EQ);
}
````
- **L681 EN**: Returns from the current function with `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_LE)`.
  **L681 CN**: 以 `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_LE)` 从当前函数返回。
- **L682 EN**: Closes the current lexical scope or compound statement.
  **L682 CN**: 结束当前词法作用域或复合语句块。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L684 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L684 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L685 EN**: Continues logic associated with callable symbol `_mm_comgt_epi64`.
  **L685 CN**: 继续与可调用符号 `_mm_comgt_epi64` 相关的逻辑。
- **L686 EN**: Opens a new lexical scope or compound statement.
  **L686 CN**: 打开一个新的词法作用域或复合语句块。
- **L687 EN**: Returns from the current function with `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_GT)`.
  **L687 CN**: 以 `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_GT)` 从当前函数返回。
- **L688 EN**: Closes the current lexical scope or compound statement.
  **L688 CN**: 结束当前词法作用域或复合语句块。
- **L689 EN**: Blank line separating nearby declarations or logic blocks.
  **L689 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L690 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L690 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L691 EN**: Continues logic associated with callable symbol `_mm_comge_epi64`.
  **L691 CN**: 继续与可调用符号 `_mm_comge_epi64` 相关的逻辑。
- **L692 EN**: Opens a new lexical scope or compound statement.
  **L692 CN**: 打开一个新的词法作用域或复合语句块。
- **L693 EN**: Returns from the current function with `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_GE)`.
  **L693 CN**: 以 `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_GE)` 从当前函数返回。
- **L694 EN**: Closes the current lexical scope or compound statement.
  **L694 CN**: 结束当前词法作用域或复合语句块。
- **L695 EN**: Blank line separating nearby declarations or logic blocks.
  **L695 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L696 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L696 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L697 EN**: Continues logic associated with callable symbol `_mm_comeq_epi64`.
  **L697 CN**: 继续与可调用符号 `_mm_comeq_epi64` 相关的逻辑。
- **L698 EN**: Opens a new lexical scope or compound statement.
  **L698 CN**: 打开一个新的词法作用域或复合语句块。
- **L699 EN**: Returns from the current function with `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_EQ)`.
  **L699 CN**: 以 `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_EQ)` 从当前函数返回。
- **L700 EN**: Closes the current lexical scope or compound statement.
  **L700 CN**: 结束当前词法作用域或复合语句块。

### Lines 701-720

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comneq_epi64(__m128i __A, __m128i __B)
{
  return _mm_com_epi64(__A, __B, _MM_PCOMCTRL_NEQ);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comfalse_epi64(__m128i __A, __m128i __B)
{
  return _mm_com_epi64(__A, __B, _MM_PCOMCTRL_FALSE);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS
_mm_comtrue_epi64(__m128i __A, __m128i __B)
{
  return _mm_com_epi64(__A, __B, _MM_PCOMCTRL_TRUE);
}

#define _mm_permute2_pd(X, Y, C, I) \
````
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L702 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L703 EN**: Continues logic associated with callable symbol `_mm_comneq_epi64`.
  **L703 CN**: 继续与可调用符号 `_mm_comneq_epi64` 相关的逻辑。
- **L704 EN**: Opens a new lexical scope or compound statement.
  **L704 CN**: 打开一个新的词法作用域或复合语句块。
- **L705 EN**: Returns from the current function with `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_NEQ)`.
  **L705 CN**: 以 `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_NEQ)` 从当前函数返回。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L708 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L708 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L709 EN**: Continues logic associated with callable symbol `_mm_comfalse_epi64`.
  **L709 CN**: 继续与可调用符号 `_mm_comfalse_epi64` 相关的逻辑。
- **L710 EN**: Opens a new lexical scope or compound statement.
  **L710 CN**: 打开一个新的词法作用域或复合语句块。
- **L711 EN**: Returns from the current function with `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_FALSE)`.
  **L711 CN**: 以 `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_FALSE)` 从当前函数返回。
- **L712 EN**: Closes the current lexical scope or compound statement.
  **L712 CN**: 结束当前词法作用域或复合语句块。
- **L713 EN**: Blank line separating nearby declarations or logic blocks.
  **L713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L714 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS`.
  **L714 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS`。
- **L715 EN**: Continues logic associated with callable symbol `_mm_comtrue_epi64`.
  **L715 CN**: 继续与可调用符号 `_mm_comtrue_epi64` 相关的逻辑。
- **L716 EN**: Opens a new lexical scope or compound statement.
  **L716 CN**: 打开一个新的词法作用域或复合语句块。
- **L717 EN**: Returns from the current function with `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_TRUE)`.
  **L717 CN**: 以 `_mm_com_epi64(__A, __B, _MM_PCOMCTRL_TRUE)` 从当前函数返回。
- **L718 EN**: Closes the current lexical scope or compound statement.
  **L718 CN**: 结束当前词法作用域或复合语句块。
- **L719 EN**: Blank line separating nearby declarations or logic blocks.
  **L719 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L720 EN**: Defines macro `_mm_permute2_pd(X, Y, C, I)` for conditional compilation, shorthand, or API generation.
  **L720 CN**: 定义宏 `_mm_permute2_pd(X, Y, C, I)`，用于条件编译、简写或 API 生成。

### Lines 721-740

````c
  ((__m128d)__builtin_ia32_vpermil2pd((__v2df)(__m128d)(X), \
                                      (__v2df)(__m128d)(Y), \
                                      (__v2di)(__m128i)(C), (I)))

#define _mm256_permute2_pd(X, Y, C, I) \
  ((__m256d)__builtin_ia32_vpermil2pd256((__v4df)(__m256d)(X), \
                                         (__v4df)(__m256d)(Y), \
                                         (__v4di)(__m256i)(C), (I)))

#define _mm_permute2_ps(X, Y, C, I) \
  ((__m128)__builtin_ia32_vpermil2ps((__v4sf)(__m128)(X), (__v4sf)(__m128)(Y), \
                                     (__v4si)(__m128i)(C), (I)))

#define _mm256_permute2_ps(X, Y, C, I) \
  ((__m256)__builtin_ia32_vpermil2ps256((__v8sf)(__m256)(X), \
                                        (__v8sf)(__m256)(Y), \
                                        (__v8si)(__m256i)(C), (I)))

static __inline__ __m128 __DEFAULT_FN_ATTRS
_mm_frcz_ss(__m128 __A)
````
- **L721 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpermil2pd`.
  **L721 CN**: 继续与可调用符号 `__builtin_ia32_vpermil2pd` 相关的逻辑。
- **L722 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(Y), \`.
  **L722 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(Y), \`。
- **L723 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(C), (I)))`.
  **L723 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(C), (I)))`。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L725 EN**: Defines macro `_mm256_permute2_pd(X, Y, C, I)` for conditional compilation, shorthand, or API generation.
  **L725 CN**: 定义宏 `_mm256_permute2_pd(X, Y, C, I)`，用于条件编译、简写或 API 生成。
- **L726 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpermil2pd256`.
  **L726 CN**: 继续与可调用符号 `__builtin_ia32_vpermil2pd256` 相关的逻辑。
- **L727 EN**: Continues the surrounding expression or declaration: `(__v4df)(__m256d)(Y), \`.
  **L727 CN**: 继续构造周围的表达式或声明：`(__v4df)(__m256d)(Y), \`。
- **L728 EN**: Continues the surrounding expression or declaration: `(__v4di)(__m256i)(C), (I)))`.
  **L728 CN**: 继续构造周围的表达式或声明：`(__v4di)(__m256i)(C), (I)))`。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L730 EN**: Defines macro `_mm_permute2_ps(X, Y, C, I)` for conditional compilation, shorthand, or API generation.
  **L730 CN**: 定义宏 `_mm_permute2_ps(X, Y, C, I)`，用于条件编译、简写或 API 生成。
- **L731 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpermil2ps`.
  **L731 CN**: 继续与可调用符号 `__builtin_ia32_vpermil2ps` 相关的逻辑。
- **L732 EN**: Continues the surrounding expression or declaration: `(__v4si)(__m128i)(C), (I)))`.
  **L732 CN**: 继续构造周围的表达式或声明：`(__v4si)(__m128i)(C), (I)))`。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L734 EN**: Defines macro `_mm256_permute2_ps(X, Y, C, I)` for conditional compilation, shorthand, or API generation.
  **L734 CN**: 定义宏 `_mm256_permute2_ps(X, Y, C, I)`，用于条件编译、简写或 API 生成。
- **L735 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpermil2ps256`.
  **L735 CN**: 继续与可调用符号 `__builtin_ia32_vpermil2ps256` 相关的逻辑。
- **L736 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(Y), \`.
  **L736 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(Y), \`。
- **L737 EN**: Continues the surrounding expression or declaration: `(__v8si)(__m256i)(C), (I)))`.
  **L737 CN**: 继续构造周围的表达式或声明：`(__v8si)(__m256i)(C), (I)))`。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L739 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS`.
  **L739 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS`。
- **L740 EN**: Continues logic associated with callable symbol `_mm_frcz_ss`.
  **L740 CN**: 继续与可调用符号 `_mm_frcz_ss` 相关的逻辑。

### Lines 741-760

````c
{
  return (__m128)__builtin_ia32_vfrczss((__v4sf)__A);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS
_mm_frcz_sd(__m128d __A)
{
  return (__m128d)__builtin_ia32_vfrczsd((__v2df)__A);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS
_mm_frcz_ps(__m128 __A)
{
  return (__m128)__builtin_ia32_vfrczps((__v4sf)__A);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS
_mm_frcz_pd(__m128d __A)
{
  return (__m128d)__builtin_ia32_vfrczpd((__v2df)__A);
````
- **L741 EN**: Opens a new lexical scope or compound statement.
  **L741 CN**: 打开一个新的词法作用域或复合语句块。
- **L742 EN**: Returns from the current function with `(__m128)__builtin_ia32_vfrczss((__v4sf)__A)`.
  **L742 CN**: 以 `(__m128)__builtin_ia32_vfrczss((__v4sf)__A)` 从当前函数返回。
- **L743 EN**: Closes the current lexical scope or compound statement.
  **L743 CN**: 结束当前词法作用域或复合语句块。
- **L744 EN**: Blank line separating nearby declarations or logic blocks.
  **L744 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L745 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS`.
  **L745 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS`。
- **L746 EN**: Continues logic associated with callable symbol `_mm_frcz_sd`.
  **L746 CN**: 继续与可调用符号 `_mm_frcz_sd` 相关的逻辑。
- **L747 EN**: Opens a new lexical scope or compound statement.
  **L747 CN**: 打开一个新的词法作用域或复合语句块。
- **L748 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vfrczsd((__v2df)__A)`.
  **L748 CN**: 以 `(__m128d)__builtin_ia32_vfrczsd((__v2df)__A)` 从当前函数返回。
- **L749 EN**: Closes the current lexical scope or compound statement.
  **L749 CN**: 结束当前词法作用域或复合语句块。
- **L750 EN**: Blank line separating nearby declarations or logic blocks.
  **L750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L751 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS`.
  **L751 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS`。
- **L752 EN**: Continues logic associated with callable symbol `_mm_frcz_ps`.
  **L752 CN**: 继续与可调用符号 `_mm_frcz_ps` 相关的逻辑。
- **L753 EN**: Opens a new lexical scope or compound statement.
  **L753 CN**: 打开一个新的词法作用域或复合语句块。
- **L754 EN**: Returns from the current function with `(__m128)__builtin_ia32_vfrczps((__v4sf)__A)`.
  **L754 CN**: 以 `(__m128)__builtin_ia32_vfrczps((__v4sf)__A)` 从当前函数返回。
- **L755 EN**: Closes the current lexical scope or compound statement.
  **L755 CN**: 结束当前词法作用域或复合语句块。
- **L756 EN**: Blank line separating nearby declarations or logic blocks.
  **L756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L757 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS`.
  **L757 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS`。
- **L758 EN**: Continues logic associated with callable symbol `_mm_frcz_pd`.
  **L758 CN**: 继续与可调用符号 `_mm_frcz_pd` 相关的逻辑。
- **L759 EN**: Opens a new lexical scope or compound statement.
  **L759 CN**: 打开一个新的词法作用域或复合语句块。
- **L760 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vfrczpd((__v2df)__A)`.
  **L760 CN**: 以 `(__m128d)__builtin_ia32_vfrczpd((__v2df)__A)` 从当前函数返回。

### Lines 761-780

````c
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_frcz_ps(__m256 __A)
{
  return (__m256)__builtin_ia32_vfrczps256((__v8sf)__A);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_frcz_pd(__m256d __A)
{
  return (__m256d)__builtin_ia32_vfrczpd256((__v4df)__A);
}

#undef __DEFAULT_FN_ATTRS
#undef __DEFAULT_FN_ATTRS256
#undef __DEFAULT_FN_ATTRS_CONSTEXPR
#undef __DEFAULT_FN_ATTRS256_CONSTEXPR

#endif /* __XOPINTRIN_H */
````
- **L761 EN**: Closes the current lexical scope or compound statement.
  **L761 CN**: 结束当前词法作用域或复合语句块。
- **L762 EN**: Blank line separating nearby declarations or logic blocks.
  **L762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L763 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L763 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L764 EN**: Continues logic associated with callable symbol `_mm256_frcz_ps`.
  **L764 CN**: 继续与可调用符号 `_mm256_frcz_ps` 相关的逻辑。
- **L765 EN**: Opens a new lexical scope or compound statement.
  **L765 CN**: 打开一个新的词法作用域或复合语句块。
- **L766 EN**: Returns from the current function with `(__m256)__builtin_ia32_vfrczps256((__v8sf)__A)`.
  **L766 CN**: 以 `(__m256)__builtin_ia32_vfrczps256((__v8sf)__A)` 从当前函数返回。
- **L767 EN**: Closes the current lexical scope or compound statement.
  **L767 CN**: 结束当前词法作用域或复合语句块。
- **L768 EN**: Blank line separating nearby declarations or logic blocks.
  **L768 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L769 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L769 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L770 EN**: Continues logic associated with callable symbol `_mm256_frcz_pd`.
  **L770 CN**: 继续与可调用符号 `_mm256_frcz_pd` 相关的逻辑。
- **L771 EN**: Opens a new lexical scope or compound statement.
  **L771 CN**: 打开一个新的词法作用域或复合语句块。
- **L772 EN**: Returns from the current function with `(__m256d)__builtin_ia32_vfrczpd256((__v4df)__A)`.
  **L772 CN**: 以 `(__m256d)__builtin_ia32_vfrczpd256((__v4df)__A)` 从当前函数返回。
- **L773 EN**: Closes the current lexical scope or compound statement.
  **L773 CN**: 结束当前词法作用域或复合语句块。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L775 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS`.
  **L775 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS`。
- **L776 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L776 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。
- **L777 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS_CONSTEXPR`.
  **L777 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS_CONSTEXPR`。
- **L778 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L778 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L780 EN**: Closes the current preprocessor conditional block.
  **L780 CN**: 结束当前预处理条件块。

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
  - `fma4intrin.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `__X86INTRIN_H`, `__XOPINTRIN_H`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_ia32_vpmacssww`, `__builtin_ia32_vpmacsww`, `__builtin_ia32_vpmacsswd`, `__builtin_ia32_vpmacswd`, `__builtin_ia32_vpmacssdd`, `__builtin_ia32_vpmacsdd`, `__builtin_ia32_vpmacssdql`, `__builtin_ia32_vpmacsdql`, `__builtin_ia32_vpmacssdqh`, `__builtin_ia32_vpmacsdqh`, `__builtin_ia32_vpmadcsswd`, `__builtin_ia32_vpmadcswd`
