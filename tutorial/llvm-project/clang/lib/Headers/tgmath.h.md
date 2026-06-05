# tgmath.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/tgmath.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Standard header for type generic math.
- **Purpose (CN)**: 提供 Standard header for type generic math 对应的头文件接口。
- **Line Count / 行数**: 1368

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- tgmath.h - Standard header for type generic math ----------------===*\
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
\*===----------------------------------------------------------------------===*/

#ifndef __CLANG_TGMATH_H
#define __CLANG_TGMATH_H

/* C99 7.22 Type-generic math <tgmath.h>. */
#include <math.h>

/*
 * Allow additional definitions and implementation-defined values on Apple
 * platforms. This is done after #include <math.h> to avoid depcycle conflicts
 * between libcxx and darwin in C++ modules builds.
 */
#if defined(__APPLE__) && __STDC_HOSTED__ && __has_include_next(<tgmath.h>)
#  include_next <tgmath.h>
#else

/* C++ handles type genericity with overloading in math.h. */
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
- **L7 EN**: Continues the surrounding expression or declaration: `\*===----------------------------------------------------------------------===*/`.
  **L7 CN**: 继续构造周围的表达式或声明：`\*===----------------------------------------------------------------------===*/`。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L9 EN**: Starts a preprocessor conditional block: `#ifndef __CLANG_TGMATH_H`.
  **L9 CN**: 开始一个预处理条件块：`#ifndef __CLANG_TGMATH_H`。
- **L10 EN**: Defines macro `__CLANG_TGMATH_H` for conditional compilation, shorthand, or API generation.
  **L10 CN**: 定义宏 `__CLANG_TGMATH_H`，用于条件编译、简写或 API 生成。
- **L11 EN**: Blank line separating nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L12 EN**: Comment explains nearby logic, constraints, or intent: `C99 7.22 Type-generic math <tgmath.h>.`.
  **L12 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C99 7.22 Type-generic math <tgmath.h>.`。
- **L13 EN**: Includes <math.h> to access related header declarations.
  **L13 CN**: 引入 <math.h> 以使用相关头文件声明。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Separator comment used for visual grouping.
  **L15 CN**: 用于视觉分组的分隔注释。
- **L16 EN**: Comment explains nearby logic, constraints, or intent: `Allow additional definitions and implementation-defined values on Apple`.
  **L16 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Allow additional definitions and implementation-defined values on Apple`。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `platforms. This is done after #include <math.h> to avoid depcycle conflicts`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`platforms. This is done after #include <math.h> to avoid depcycle conflicts`。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `between libcxx and darwin in C++ modules builds.`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`between libcxx and darwin in C++ modules builds.`。
- **L19 EN**: Separator comment used for visual grouping.
  **L19 CN**: 用于视觉分组的分隔注释。
- **L20 EN**: Starts a preprocessor conditional block: `#if defined(__APPLE__) && __STDC_HOSTED__ && __has_include_next(<tgmath.h>)`.
  **L20 CN**: 开始一个预处理条件块：`#if defined(__APPLE__) && __STDC_HOSTED__ && __has_include_next(<tgmath.h>)`。
- **L21 EN**: Continues the surrounding expression or declaration: `#  include_next <tgmath.h>`.
  **L21 CN**: 继续构造周围的表达式或声明：`#  include_next <tgmath.h>`。
- **L22 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L22 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Comment explains nearby logic, constraints, or intent: `C++ handles type genericity with overloading in math.h.`.
  **L24 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++ handles type genericity with overloading in math.h.`。

### Lines 25-48

````c
#ifndef __cplusplus
#include <complex.h>

#define _TG_ATTRSp __attribute__((__overloadable__))
#define _TG_ATTRS __attribute__((__overloadable__, __always_inline__))

// promotion

typedef void _Argument_type_is_not_arithmetic;
static _Argument_type_is_not_arithmetic __tg_promote(...)
  __attribute__((__unavailable__,__overloadable__));
static double               _TG_ATTRSp __tg_promote(int);
static double               _TG_ATTRSp __tg_promote(unsigned int);
static double               _TG_ATTRSp __tg_promote(long);
static double               _TG_ATTRSp __tg_promote(unsigned long);
static double               _TG_ATTRSp __tg_promote(long long);
static double               _TG_ATTRSp __tg_promote(unsigned long long);
static float                _TG_ATTRSp __tg_promote(float);
static double               _TG_ATTRSp __tg_promote(double);
static long double          _TG_ATTRSp __tg_promote(long double);
static float _Complex       _TG_ATTRSp __tg_promote(float _Complex);
static double _Complex      _TG_ATTRSp __tg_promote(double _Complex);
static long double _Complex _TG_ATTRSp __tg_promote(long double _Complex);

````
- **L25 EN**: Starts a preprocessor conditional block: `#ifndef __cplusplus`.
  **L25 CN**: 开始一个预处理条件块：`#ifndef __cplusplus`。
- **L26 EN**: Includes <complex.h> to access related header declarations.
  **L26 CN**: 引入 <complex.h> 以使用相关头文件声明。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Defines macro `_TG_ATTRSp` for conditional compilation, shorthand, or API generation.
  **L28 CN**: 定义宏 `_TG_ATTRSp`，用于条件编译、简写或 API 生成。
- **L29 EN**: Defines macro `_TG_ATTRS` for conditional compilation, shorthand, or API generation.
  **L29 CN**: 定义宏 `_TG_ATTRS`，用于条件编译、简写或 API 生成。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `promotion`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`promotion`。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L33 EN**: Introduces an alias or helper declaration: `typedef void _Argument_type_is_not_arithmetic;`.
  **L33 CN**: 引入一条别名或辅助声明：`typedef void _Argument_type_is_not_arithmetic;`。
- **L34 EN**: Continues logic associated with callable symbol `__tg_promote`.
  **L34 CN**: 继续与可调用符号 `__tg_promote` 相关的逻辑。
- **L35 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__unavailable__,__overloadable__));`.
  **L35 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__unavailable__,__overloadable__));`。
- **L36 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L36 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L37 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L37 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L38 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L38 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L39 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L39 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L40 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L40 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L41 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L41 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L42 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L42 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L43 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L43 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L44 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L44 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L45 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L45 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L46 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L46 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L47 EN**: Executes a call or declaration centered on `__tg_promote`.
  **L47 CN**: 执行以 `__tg_promote` 为核心的调用或声明。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 49-72

````c
#define __tg_promote1(__x)           (__typeof__(__tg_promote(__x)))
#define __tg_promote2(__x, __y)      (__typeof__(__tg_promote(__x) + \
                                                 __tg_promote(__y)))
#define __tg_promote3(__x, __y, __z) (__typeof__(__tg_promote(__x) + \
                                                 __tg_promote(__y) + \
                                                 __tg_promote(__z)))

// acos

static float
    _TG_ATTRS
    __tg_acos(float __x) {return acosf(__x);}

static double
    _TG_ATTRS
    __tg_acos(double __x) {return acos(__x);}

static long double
    _TG_ATTRS
    __tg_acos(long double __x) {return acosl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_acos(float _Complex __x) {return cacosf(__x);}
````
- **L49 EN**: Defines macro `__tg_promote1(__x)` for conditional compilation, shorthand, or API generation.
  **L49 CN**: 定义宏 `__tg_promote1(__x)`，用于条件编译、简写或 API 生成。
- **L50 EN**: Defines macro `__tg_promote2(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L50 CN**: 定义宏 `__tg_promote2(__x, __y)`，用于条件编译、简写或 API 生成。
- **L51 EN**: Continues logic associated with callable symbol `__tg_promote`.
  **L51 CN**: 继续与可调用符号 `__tg_promote` 相关的逻辑。
- **L52 EN**: Defines macro `__tg_promote3(__x, __y, __z)` for conditional compilation, shorthand, or API generation.
  **L52 CN**: 定义宏 `__tg_promote3(__x, __y, __z)`，用于条件编译、简写或 API 生成。
- **L53 EN**: Continues logic associated with callable symbol `__tg_promote`.
  **L53 CN**: 继续与可调用符号 `__tg_promote` 相关的逻辑。
- **L54 EN**: Continues logic associated with callable symbol `__tg_promote`.
  **L54 CN**: 继续与可调用符号 `__tg_promote` 相关的逻辑。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L56 EN**: Comment explains nearby logic, constraints, or intent: `acos`.
  **L56 CN**: 注释解释附近代码的逻辑、约束或设计意图：`acos`。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L58 EN**: Continues the surrounding expression or declaration: `static float`.
  **L58 CN**: 继续构造周围的表达式或声明：`static float`。
- **L59 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L59 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L60 EN**: Continues logic associated with callable symbol `__tg_acos`.
  **L60 CN**: 继续与可调用符号 `__tg_acos` 相关的逻辑。
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L62 EN**: Continues the surrounding expression or declaration: `static double`.
  **L62 CN**: 继续构造周围的表达式或声明：`static double`。
- **L63 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L63 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L64 EN**: Continues logic associated with callable symbol `__tg_acos`.
  **L64 CN**: 继续与可调用符号 `__tg_acos` 相关的逻辑。
- **L65 EN**: Blank line separating nearby declarations or logic blocks.
  **L65 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L66 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L66 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L67 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L67 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L68 EN**: Continues logic associated with callable symbol `__tg_acos`.
  **L68 CN**: 继续与可调用符号 `__tg_acos` 相关的逻辑。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L70 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L71 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L71 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L72 EN**: Continues logic associated with callable symbol `__tg_acos`.
  **L72 CN**: 继续与可调用符号 `__tg_acos` 相关的逻辑。

### Lines 73-96

````c

static double _Complex
    _TG_ATTRS
    __tg_acos(double _Complex __x) {return cacos(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_acos(long double _Complex __x) {return cacosl(__x);}

#undef acos
#define acos(__x) __tg_acos(__tg_promote1((__x))(__x))

// asin

static float
    _TG_ATTRS
    __tg_asin(float __x) {return asinf(__x);}

static double
    _TG_ATTRS
    __tg_asin(double __x) {return asin(__x);}

static long double
    _TG_ATTRS
````
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L74 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L74 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L75 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L75 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L76 EN**: Continues logic associated with callable symbol `__tg_acos`.
  **L76 CN**: 继续与可调用符号 `__tg_acos` 相关的逻辑。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L78 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L78 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L79 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L79 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L80 EN**: Continues logic associated with callable symbol `__tg_acos`.
  **L80 CN**: 继续与可调用符号 `__tg_acos` 相关的逻辑。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L82 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef acos`.
  **L82 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef acos`。
- **L83 EN**: Defines macro `acos(__x)` for conditional compilation, shorthand, or API generation.
  **L83 CN**: 定义宏 `acos(__x)`，用于条件编译、简写或 API 生成。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Comment explains nearby logic, constraints, or intent: `asin`.
  **L85 CN**: 注释解释附近代码的逻辑、约束或设计意图：`asin`。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L87 EN**: Continues the surrounding expression or declaration: `static float`.
  **L87 CN**: 继续构造周围的表达式或声明：`static float`。
- **L88 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L88 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L89 EN**: Continues logic associated with callable symbol `__tg_asin`.
  **L89 CN**: 继续与可调用符号 `__tg_asin` 相关的逻辑。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L91 EN**: Continues the surrounding expression or declaration: `static double`.
  **L91 CN**: 继续构造周围的表达式或声明：`static double`。
- **L92 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L92 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L93 EN**: Continues logic associated with callable symbol `__tg_asin`.
  **L93 CN**: 继续与可调用符号 `__tg_asin` 相关的逻辑。
- **L94 EN**: Blank line separating nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L95 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L95 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L96 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L96 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 97-120

````c
    __tg_asin(long double __x) {return asinl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_asin(float _Complex __x) {return casinf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_asin(double _Complex __x) {return casin(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_asin(long double _Complex __x) {return casinl(__x);}

#undef asin
#define asin(__x) __tg_asin(__tg_promote1((__x))(__x))

// atan

static float
    _TG_ATTRS
    __tg_atan(float __x) {return atanf(__x);}

static double
````
- **L97 EN**: Continues logic associated with callable symbol `__tg_asin`.
  **L97 CN**: 继续与可调用符号 `__tg_asin` 相关的逻辑。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L99 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L99 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L100 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L100 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L101 EN**: Continues logic associated with callable symbol `__tg_asin`.
  **L101 CN**: 继续与可调用符号 `__tg_asin` 相关的逻辑。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L103 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L103 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L104 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L104 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L105 EN**: Continues logic associated with callable symbol `__tg_asin`.
  **L105 CN**: 继续与可调用符号 `__tg_asin` 相关的逻辑。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L107 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L107 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L108 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L108 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L109 EN**: Continues logic associated with callable symbol `__tg_asin`.
  **L109 CN**: 继续与可调用符号 `__tg_asin` 相关的逻辑。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L111 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef asin`.
  **L111 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef asin`。
- **L112 EN**: Defines macro `asin(__x)` for conditional compilation, shorthand, or API generation.
  **L112 CN**: 定义宏 `asin(__x)`，用于条件编译、简写或 API 生成。
- **L113 EN**: Blank line separating nearby declarations or logic blocks.
  **L113 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L114 EN**: Comment explains nearby logic, constraints, or intent: `atan`.
  **L114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`atan`。
- **L115 EN**: Blank line separating nearby declarations or logic blocks.
  **L115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L116 EN**: Continues the surrounding expression or declaration: `static float`.
  **L116 CN**: 继续构造周围的表达式或声明：`static float`。
- **L117 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L117 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L118 EN**: Continues logic associated with callable symbol `__tg_atan`.
  **L118 CN**: 继续与可调用符号 `__tg_atan` 相关的逻辑。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L120 EN**: Continues the surrounding expression or declaration: `static double`.
  **L120 CN**: 继续构造周围的表达式或声明：`static double`。

### Lines 121-144

````c
    _TG_ATTRS
    __tg_atan(double __x) {return atan(__x);}

static long double
    _TG_ATTRS
    __tg_atan(long double __x) {return atanl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_atan(float _Complex __x) {return catanf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_atan(double _Complex __x) {return catan(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_atan(long double _Complex __x) {return catanl(__x);}

#undef atan
#define atan(__x) __tg_atan(__tg_promote1((__x))(__x))

// acosh

````
- **L121 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L121 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L122 EN**: Continues logic associated with callable symbol `__tg_atan`.
  **L122 CN**: 继续与可调用符号 `__tg_atan` 相关的逻辑。
- **L123 EN**: Blank line separating nearby declarations or logic blocks.
  **L123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L124 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L124 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L125 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L125 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L126 EN**: Continues logic associated with callable symbol `__tg_atan`.
  **L126 CN**: 继续与可调用符号 `__tg_atan` 相关的逻辑。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L128 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L128 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L129 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L129 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L130 EN**: Continues logic associated with callable symbol `__tg_atan`.
  **L130 CN**: 继续与可调用符号 `__tg_atan` 相关的逻辑。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L132 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L132 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L133 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L133 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L134 EN**: Continues logic associated with callable symbol `__tg_atan`.
  **L134 CN**: 继续与可调用符号 `__tg_atan` 相关的逻辑。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L136 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L136 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L137 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L137 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L138 EN**: Continues logic associated with callable symbol `__tg_atan`.
  **L138 CN**: 继续与可调用符号 `__tg_atan` 相关的逻辑。
- **L139 EN**: Blank line separating nearby declarations or logic blocks.
  **L139 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L140 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef atan`.
  **L140 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef atan`。
- **L141 EN**: Defines macro `atan(__x)` for conditional compilation, shorthand, or API generation.
  **L141 CN**: 定义宏 `atan(__x)`，用于条件编译、简写或 API 生成。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `acosh`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`acosh`。
- **L144 EN**: Blank line separating nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 145-168

````c
static float
    _TG_ATTRS
    __tg_acosh(float __x) {return acoshf(__x);}

static double
    _TG_ATTRS
    __tg_acosh(double __x) {return acosh(__x);}

static long double
    _TG_ATTRS
    __tg_acosh(long double __x) {return acoshl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_acosh(float _Complex __x) {return cacoshf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_acosh(double _Complex __x) {return cacosh(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_acosh(long double _Complex __x) {return cacoshl(__x);}

````
- **L145 EN**: Continues the surrounding expression or declaration: `static float`.
  **L145 CN**: 继续构造周围的表达式或声明：`static float`。
- **L146 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L146 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L147 EN**: Continues logic associated with callable symbol `__tg_acosh`.
  **L147 CN**: 继续与可调用符号 `__tg_acosh` 相关的逻辑。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L149 EN**: Continues the surrounding expression or declaration: `static double`.
  **L149 CN**: 继续构造周围的表达式或声明：`static double`。
- **L150 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L150 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L151 EN**: Continues logic associated with callable symbol `__tg_acosh`.
  **L151 CN**: 继续与可调用符号 `__tg_acosh` 相关的逻辑。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L153 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L153 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L154 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L154 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L155 EN**: Continues logic associated with callable symbol `__tg_acosh`.
  **L155 CN**: 继续与可调用符号 `__tg_acosh` 相关的逻辑。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L157 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L157 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L158 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L158 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L159 EN**: Continues logic associated with callable symbol `__tg_acosh`.
  **L159 CN**: 继续与可调用符号 `__tg_acosh` 相关的逻辑。
- **L160 EN**: Blank line separating nearby declarations or logic blocks.
  **L160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L161 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L161 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L162 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L162 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L163 EN**: Continues logic associated with callable symbol `__tg_acosh`.
  **L163 CN**: 继续与可调用符号 `__tg_acosh` 相关的逻辑。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L165 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L166 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L166 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L167 EN**: Continues logic associated with callable symbol `__tg_acosh`.
  **L167 CN**: 继续与可调用符号 `__tg_acosh` 相关的逻辑。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 169-192

````c
#undef acosh
#define acosh(__x) __tg_acosh(__tg_promote1((__x))(__x))

// asinh

static float
    _TG_ATTRS
    __tg_asinh(float __x) {return asinhf(__x);}

static double
    _TG_ATTRS
    __tg_asinh(double __x) {return asinh(__x);}

static long double
    _TG_ATTRS
    __tg_asinh(long double __x) {return asinhl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_asinh(float _Complex __x) {return casinhf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_asinh(double _Complex __x) {return casinh(__x);}
````
- **L169 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef acosh`.
  **L169 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef acosh`。
- **L170 EN**: Defines macro `acosh(__x)` for conditional compilation, shorthand, or API generation.
  **L170 CN**: 定义宏 `acosh(__x)`，用于条件编译、简写或 API 生成。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L172 EN**: Comment explains nearby logic, constraints, or intent: `asinh`.
  **L172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`asinh`。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L174 EN**: Continues the surrounding expression or declaration: `static float`.
  **L174 CN**: 继续构造周围的表达式或声明：`static float`。
- **L175 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L175 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L176 EN**: Continues logic associated with callable symbol `__tg_asinh`.
  **L176 CN**: 继续与可调用符号 `__tg_asinh` 相关的逻辑。
- **L177 EN**: Blank line separating nearby declarations or logic blocks.
  **L177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L178 EN**: Continues the surrounding expression or declaration: `static double`.
  **L178 CN**: 继续构造周围的表达式或声明：`static double`。
- **L179 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L179 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L180 EN**: Continues logic associated with callable symbol `__tg_asinh`.
  **L180 CN**: 继续与可调用符号 `__tg_asinh` 相关的逻辑。
- **L181 EN**: Blank line separating nearby declarations or logic blocks.
  **L181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L182 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L182 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L183 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L183 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L184 EN**: Continues logic associated with callable symbol `__tg_asinh`.
  **L184 CN**: 继续与可调用符号 `__tg_asinh` 相关的逻辑。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L186 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L187 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L187 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L188 EN**: Continues logic associated with callable symbol `__tg_asinh`.
  **L188 CN**: 继续与可调用符号 `__tg_asinh` 相关的逻辑。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L190 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L190 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L191 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L191 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L192 EN**: Continues logic associated with callable symbol `__tg_asinh`.
  **L192 CN**: 继续与可调用符号 `__tg_asinh` 相关的逻辑。

### Lines 193-216

````c

static long double _Complex
    _TG_ATTRS
    __tg_asinh(long double _Complex __x) {return casinhl(__x);}

#undef asinh
#define asinh(__x) __tg_asinh(__tg_promote1((__x))(__x))

// atanh

static float
    _TG_ATTRS
    __tg_atanh(float __x) {return atanhf(__x);}

static double
    _TG_ATTRS
    __tg_atanh(double __x) {return atanh(__x);}

static long double
    _TG_ATTRS
    __tg_atanh(long double __x) {return atanhl(__x);}

static float _Complex
    _TG_ATTRS
````
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L194 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L194 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L195 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L195 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L196 EN**: Continues logic associated with callable symbol `__tg_asinh`.
  **L196 CN**: 继续与可调用符号 `__tg_asinh` 相关的逻辑。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L198 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef asinh`.
  **L198 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef asinh`。
- **L199 EN**: Defines macro `asinh(__x)` for conditional compilation, shorthand, or API generation.
  **L199 CN**: 定义宏 `asinh(__x)`，用于条件编译、简写或 API 生成。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L201 EN**: Comment explains nearby logic, constraints, or intent: `atanh`.
  **L201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`atanh`。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L203 EN**: Continues the surrounding expression or declaration: `static float`.
  **L203 CN**: 继续构造周围的表达式或声明：`static float`。
- **L204 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L204 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L205 EN**: Continues logic associated with callable symbol `__tg_atanh`.
  **L205 CN**: 继续与可调用符号 `__tg_atanh` 相关的逻辑。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L207 EN**: Continues the surrounding expression or declaration: `static double`.
  **L207 CN**: 继续构造周围的表达式或声明：`static double`。
- **L208 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L208 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L209 EN**: Continues logic associated with callable symbol `__tg_atanh`.
  **L209 CN**: 继续与可调用符号 `__tg_atanh` 相关的逻辑。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L211 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L211 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L212 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L212 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L213 EN**: Continues logic associated with callable symbol `__tg_atanh`.
  **L213 CN**: 继续与可调用符号 `__tg_atanh` 相关的逻辑。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L215 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L215 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L216 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L216 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 217-240

````c
    __tg_atanh(float _Complex __x) {return catanhf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_atanh(double _Complex __x) {return catanh(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_atanh(long double _Complex __x) {return catanhl(__x);}

#undef atanh
#define atanh(__x) __tg_atanh(__tg_promote1((__x))(__x))

// cos

static float
    _TG_ATTRS
    __tg_cos(float __x) {return cosf(__x);}

static double
    _TG_ATTRS
    __tg_cos(double __x) {return cos(__x);}

static long double
````
- **L217 EN**: Continues logic associated with callable symbol `__tg_atanh`.
  **L217 CN**: 继续与可调用符号 `__tg_atanh` 相关的逻辑。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L219 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L219 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L220 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L220 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L221 EN**: Continues logic associated with callable symbol `__tg_atanh`.
  **L221 CN**: 继续与可调用符号 `__tg_atanh` 相关的逻辑。
- **L222 EN**: Blank line separating nearby declarations or logic blocks.
  **L222 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L223 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L223 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L224 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L224 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L225 EN**: Continues logic associated with callable symbol `__tg_atanh`.
  **L225 CN**: 继续与可调用符号 `__tg_atanh` 相关的逻辑。
- **L226 EN**: Blank line separating nearby declarations or logic blocks.
  **L226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L227 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef atanh`.
  **L227 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef atanh`。
- **L228 EN**: Defines macro `atanh(__x)` for conditional compilation, shorthand, or API generation.
  **L228 CN**: 定义宏 `atanh(__x)`，用于条件编译、简写或 API 生成。
- **L229 EN**: Blank line separating nearby declarations or logic blocks.
  **L229 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L230 EN**: Comment explains nearby logic, constraints, or intent: `cos`.
  **L230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`cos`。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L232 EN**: Continues the surrounding expression or declaration: `static float`.
  **L232 CN**: 继续构造周围的表达式或声明：`static float`。
- **L233 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L233 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L234 EN**: Continues logic associated with callable symbol `__tg_cos`.
  **L234 CN**: 继续与可调用符号 `__tg_cos` 相关的逻辑。
- **L235 EN**: Blank line separating nearby declarations or logic blocks.
  **L235 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L236 EN**: Continues the surrounding expression or declaration: `static double`.
  **L236 CN**: 继续构造周围的表达式或声明：`static double`。
- **L237 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L237 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L238 EN**: Continues logic associated with callable symbol `__tg_cos`.
  **L238 CN**: 继续与可调用符号 `__tg_cos` 相关的逻辑。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L240 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L240 CN**: 继续构造周围的表达式或声明：`static long double`。

### Lines 241-264

````c
    _TG_ATTRS
    __tg_cos(long double __x) {return cosl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_cos(float _Complex __x) {return ccosf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_cos(double _Complex __x) {return ccos(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_cos(long double _Complex __x) {return ccosl(__x);}

#undef cos
#define cos(__x) __tg_cos(__tg_promote1((__x))(__x))

// sin

static float
    _TG_ATTRS
    __tg_sin(float __x) {return sinf(__x);}

````
- **L241 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L241 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L242 EN**: Continues logic associated with callable symbol `__tg_cos`.
  **L242 CN**: 继续与可调用符号 `__tg_cos` 相关的逻辑。
- **L243 EN**: Blank line separating nearby declarations or logic blocks.
  **L243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L244 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L244 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L245 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L245 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L246 EN**: Continues logic associated with callable symbol `__tg_cos`.
  **L246 CN**: 继续与可调用符号 `__tg_cos` 相关的逻辑。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L248 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L248 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L249 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L249 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L250 EN**: Continues logic associated with callable symbol `__tg_cos`.
  **L250 CN**: 继续与可调用符号 `__tg_cos` 相关的逻辑。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L252 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L252 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L253 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L253 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L254 EN**: Continues logic associated with callable symbol `__tg_cos`.
  **L254 CN**: 继续与可调用符号 `__tg_cos` 相关的逻辑。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L256 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef cos`.
  **L256 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef cos`。
- **L257 EN**: Defines macro `cos(__x)` for conditional compilation, shorthand, or API generation.
  **L257 CN**: 定义宏 `cos(__x)`，用于条件编译、简写或 API 生成。
- **L258 EN**: Blank line separating nearby declarations or logic blocks.
  **L258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L259 EN**: Comment explains nearby logic, constraints, or intent: `sin`.
  **L259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sin`。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L261 EN**: Continues the surrounding expression or declaration: `static float`.
  **L261 CN**: 继续构造周围的表达式或声明：`static float`。
- **L262 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L262 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L263 EN**: Continues logic associated with callable symbol `__tg_sin`.
  **L263 CN**: 继续与可调用符号 `__tg_sin` 相关的逻辑。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 265-288

````c
static double
    _TG_ATTRS
    __tg_sin(double __x) {return sin(__x);}

static long double
    _TG_ATTRS
    __tg_sin(long double __x) {return sinl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_sin(float _Complex __x) {return csinf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_sin(double _Complex __x) {return csin(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_sin(long double _Complex __x) {return csinl(__x);}

#undef sin
#define sin(__x) __tg_sin(__tg_promote1((__x))(__x))

// tan
````
- **L265 EN**: Continues the surrounding expression or declaration: `static double`.
  **L265 CN**: 继续构造周围的表达式或声明：`static double`。
- **L266 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L266 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L267 EN**: Continues logic associated with callable symbol `__tg_sin`.
  **L267 CN**: 继续与可调用符号 `__tg_sin` 相关的逻辑。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L269 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L269 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L270 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L270 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L271 EN**: Continues logic associated with callable symbol `__tg_sin`.
  **L271 CN**: 继续与可调用符号 `__tg_sin` 相关的逻辑。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L273 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L273 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L274 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L274 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L275 EN**: Continues logic associated with callable symbol `__tg_sin`.
  **L275 CN**: 继续与可调用符号 `__tg_sin` 相关的逻辑。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L277 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L277 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L278 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L278 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L279 EN**: Continues logic associated with callable symbol `__tg_sin`.
  **L279 CN**: 继续与可调用符号 `__tg_sin` 相关的逻辑。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L281 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L281 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L282 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L282 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L283 EN**: Continues logic associated with callable symbol `__tg_sin`.
  **L283 CN**: 继续与可调用符号 `__tg_sin` 相关的逻辑。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L285 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef sin`.
  **L285 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef sin`。
- **L286 EN**: Defines macro `sin(__x)` for conditional compilation, shorthand, or API generation.
  **L286 CN**: 定义宏 `sin(__x)`，用于条件编译、简写或 API 生成。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L288 EN**: Comment explains nearby logic, constraints, or intent: `tan`.
  **L288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tan`。

### Lines 289-312

````c

static float
    _TG_ATTRS
    __tg_tan(float __x) {return tanf(__x);}

static double
    _TG_ATTRS
    __tg_tan(double __x) {return tan(__x);}

static long double
    _TG_ATTRS
    __tg_tan(long double __x) {return tanl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_tan(float _Complex __x) {return ctanf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_tan(double _Complex __x) {return ctan(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_tan(long double _Complex __x) {return ctanl(__x);}
````
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L290 EN**: Continues the surrounding expression or declaration: `static float`.
  **L290 CN**: 继续构造周围的表达式或声明：`static float`。
- **L291 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L291 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L292 EN**: Continues logic associated with callable symbol `__tg_tan`.
  **L292 CN**: 继续与可调用符号 `__tg_tan` 相关的逻辑。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L294 EN**: Continues the surrounding expression or declaration: `static double`.
  **L294 CN**: 继续构造周围的表达式或声明：`static double`。
- **L295 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L295 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L296 EN**: Continues logic associated with callable symbol `__tg_tan`.
  **L296 CN**: 继续与可调用符号 `__tg_tan` 相关的逻辑。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L298 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L298 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L299 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L299 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L300 EN**: Continues logic associated with callable symbol `__tg_tan`.
  **L300 CN**: 继续与可调用符号 `__tg_tan` 相关的逻辑。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L302 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L302 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L303 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L303 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L304 EN**: Continues logic associated with callable symbol `__tg_tan`.
  **L304 CN**: 继续与可调用符号 `__tg_tan` 相关的逻辑。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L306 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L306 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L307 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L307 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L308 EN**: Continues logic associated with callable symbol `__tg_tan`.
  **L308 CN**: 继续与可调用符号 `__tg_tan` 相关的逻辑。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L310 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L311 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L311 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L312 EN**: Continues logic associated with callable symbol `__tg_tan`.
  **L312 CN**: 继续与可调用符号 `__tg_tan` 相关的逻辑。

### Lines 313-336

````c

#undef tan
#define tan(__x) __tg_tan(__tg_promote1((__x))(__x))

// cosh

static float
    _TG_ATTRS
    __tg_cosh(float __x) {return coshf(__x);}

static double
    _TG_ATTRS
    __tg_cosh(double __x) {return cosh(__x);}

static long double
    _TG_ATTRS
    __tg_cosh(long double __x) {return coshl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_cosh(float _Complex __x) {return ccoshf(__x);}

static double _Complex
    _TG_ATTRS
````
- **L313 EN**: Blank line separating nearby declarations or logic blocks.
  **L313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L314 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef tan`.
  **L314 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef tan`。
- **L315 EN**: Defines macro `tan(__x)` for conditional compilation, shorthand, or API generation.
  **L315 CN**: 定义宏 `tan(__x)`，用于条件编译、简写或 API 生成。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L317 EN**: Comment explains nearby logic, constraints, or intent: `cosh`.
  **L317 CN**: 注释解释附近代码的逻辑、约束或设计意图：`cosh`。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L319 EN**: Continues the surrounding expression or declaration: `static float`.
  **L319 CN**: 继续构造周围的表达式或声明：`static float`。
- **L320 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L320 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L321 EN**: Continues logic associated with callable symbol `__tg_cosh`.
  **L321 CN**: 继续与可调用符号 `__tg_cosh` 相关的逻辑。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Continues the surrounding expression or declaration: `static double`.
  **L323 CN**: 继续构造周围的表达式或声明：`static double`。
- **L324 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L324 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L325 EN**: Continues logic associated with callable symbol `__tg_cosh`.
  **L325 CN**: 继续与可调用符号 `__tg_cosh` 相关的逻辑。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L327 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L328 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L328 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L329 EN**: Continues logic associated with callable symbol `__tg_cosh`.
  **L329 CN**: 继续与可调用符号 `__tg_cosh` 相关的逻辑。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L331 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L331 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L332 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L332 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L333 EN**: Continues logic associated with callable symbol `__tg_cosh`.
  **L333 CN**: 继续与可调用符号 `__tg_cosh` 相关的逻辑。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L335 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L335 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L336 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L336 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 337-360

````c
    __tg_cosh(double _Complex __x) {return ccosh(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_cosh(long double _Complex __x) {return ccoshl(__x);}

#undef cosh
#define cosh(__x) __tg_cosh(__tg_promote1((__x))(__x))

// sinh

static float
    _TG_ATTRS
    __tg_sinh(float __x) {return sinhf(__x);}

static double
    _TG_ATTRS
    __tg_sinh(double __x) {return sinh(__x);}

static long double
    _TG_ATTRS
    __tg_sinh(long double __x) {return sinhl(__x);}

static float _Complex
````
- **L337 EN**: Continues logic associated with callable symbol `__tg_cosh`.
  **L337 CN**: 继续与可调用符号 `__tg_cosh` 相关的逻辑。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L339 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L340 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L340 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L341 EN**: Continues logic associated with callable symbol `__tg_cosh`.
  **L341 CN**: 继续与可调用符号 `__tg_cosh` 相关的逻辑。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L343 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef cosh`.
  **L343 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef cosh`。
- **L344 EN**: Defines macro `cosh(__x)` for conditional compilation, shorthand, or API generation.
  **L344 CN**: 定义宏 `cosh(__x)`，用于条件编译、简写或 API 生成。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L346 EN**: Comment explains nearby logic, constraints, or intent: `sinh`.
  **L346 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sinh`。
- **L347 EN**: Blank line separating nearby declarations or logic blocks.
  **L347 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L348 EN**: Continues the surrounding expression or declaration: `static float`.
  **L348 CN**: 继续构造周围的表达式或声明：`static float`。
- **L349 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L349 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L350 EN**: Continues logic associated with callable symbol `__tg_sinh`.
  **L350 CN**: 继续与可调用符号 `__tg_sinh` 相关的逻辑。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L352 EN**: Continues the surrounding expression or declaration: `static double`.
  **L352 CN**: 继续构造周围的表达式或声明：`static double`。
- **L353 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L353 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L354 EN**: Continues logic associated with callable symbol `__tg_sinh`.
  **L354 CN**: 继续与可调用符号 `__tg_sinh` 相关的逻辑。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L356 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L356 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L357 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L357 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L358 EN**: Continues logic associated with callable symbol `__tg_sinh`.
  **L358 CN**: 继续与可调用符号 `__tg_sinh` 相关的逻辑。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L360 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L360 CN**: 继续构造周围的表达式或声明：`static float _Complex`。

### Lines 361-384

````c
    _TG_ATTRS
    __tg_sinh(float _Complex __x) {return csinhf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_sinh(double _Complex __x) {return csinh(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_sinh(long double _Complex __x) {return csinhl(__x);}

#undef sinh
#define sinh(__x) __tg_sinh(__tg_promote1((__x))(__x))

// tanh

static float
    _TG_ATTRS
    __tg_tanh(float __x) {return tanhf(__x);}

static double
    _TG_ATTRS
    __tg_tanh(double __x) {return tanh(__x);}

````
- **L361 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L361 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L362 EN**: Continues logic associated with callable symbol `__tg_sinh`.
  **L362 CN**: 继续与可调用符号 `__tg_sinh` 相关的逻辑。
- **L363 EN**: Blank line separating nearby declarations or logic blocks.
  **L363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L364 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L364 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L365 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L365 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L366 EN**: Continues logic associated with callable symbol `__tg_sinh`.
  **L366 CN**: 继续与可调用符号 `__tg_sinh` 相关的逻辑。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L368 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L368 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L369 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L369 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L370 EN**: Continues logic associated with callable symbol `__tg_sinh`.
  **L370 CN**: 继续与可调用符号 `__tg_sinh` 相关的逻辑。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L372 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef sinh`.
  **L372 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef sinh`。
- **L373 EN**: Defines macro `sinh(__x)` for conditional compilation, shorthand, or API generation.
  **L373 CN**: 定义宏 `sinh(__x)`，用于条件编译、简写或 API 生成。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L375 EN**: Comment explains nearby logic, constraints, or intent: `tanh`.
  **L375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tanh`。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L377 EN**: Continues the surrounding expression or declaration: `static float`.
  **L377 CN**: 继续构造周围的表达式或声明：`static float`。
- **L378 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L378 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L379 EN**: Continues logic associated with callable symbol `__tg_tanh`.
  **L379 CN**: 继续与可调用符号 `__tg_tanh` 相关的逻辑。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L381 EN**: Continues the surrounding expression or declaration: `static double`.
  **L381 CN**: 继续构造周围的表达式或声明：`static double`。
- **L382 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L382 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L383 EN**: Continues logic associated with callable symbol `__tg_tanh`.
  **L383 CN**: 继续与可调用符号 `__tg_tanh` 相关的逻辑。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````c
static long double
    _TG_ATTRS
    __tg_tanh(long double __x) {return tanhl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_tanh(float _Complex __x) {return ctanhf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_tanh(double _Complex __x) {return ctanh(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_tanh(long double _Complex __x) {return ctanhl(__x);}

#undef tanh
#define tanh(__x) __tg_tanh(__tg_promote1((__x))(__x))

// exp

static float
    _TG_ATTRS
    __tg_exp(float __x) {return expf(__x);}
````
- **L385 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L385 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L386 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L386 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L387 EN**: Continues logic associated with callable symbol `__tg_tanh`.
  **L387 CN**: 继续与可调用符号 `__tg_tanh` 相关的逻辑。
- **L388 EN**: Blank line separating nearby declarations or logic blocks.
  **L388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L389 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L389 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L390 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L390 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L391 EN**: Continues logic associated with callable symbol `__tg_tanh`.
  **L391 CN**: 继续与可调用符号 `__tg_tanh` 相关的逻辑。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L393 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L393 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L394 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L394 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L395 EN**: Continues logic associated with callable symbol `__tg_tanh`.
  **L395 CN**: 继续与可调用符号 `__tg_tanh` 相关的逻辑。
- **L396 EN**: Blank line separating nearby declarations or logic blocks.
  **L396 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L397 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L397 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L398 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L398 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L399 EN**: Continues logic associated with callable symbol `__tg_tanh`.
  **L399 CN**: 继续与可调用符号 `__tg_tanh` 相关的逻辑。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L401 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef tanh`.
  **L401 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef tanh`。
- **L402 EN**: Defines macro `tanh(__x)` for conditional compilation, shorthand, or API generation.
  **L402 CN**: 定义宏 `tanh(__x)`，用于条件编译、简写或 API 生成。
- **L403 EN**: Blank line separating nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L404 EN**: Comment explains nearby logic, constraints, or intent: `exp`.
  **L404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exp`。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L406 EN**: Continues the surrounding expression or declaration: `static float`.
  **L406 CN**: 继续构造周围的表达式或声明：`static float`。
- **L407 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L407 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L408 EN**: Continues logic associated with callable symbol `__tg_exp`.
  **L408 CN**: 继续与可调用符号 `__tg_exp` 相关的逻辑。

### Lines 409-432

````c

static double
    _TG_ATTRS
    __tg_exp(double __x) {return exp(__x);}

static long double
    _TG_ATTRS
    __tg_exp(long double __x) {return expl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_exp(float _Complex __x) {return cexpf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_exp(double _Complex __x) {return cexp(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_exp(long double _Complex __x) {return cexpl(__x);}

#undef exp
#define exp(__x) __tg_exp(__tg_promote1((__x))(__x))

````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L410 EN**: Continues the surrounding expression or declaration: `static double`.
  **L410 CN**: 继续构造周围的表达式或声明：`static double`。
- **L411 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L411 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L412 EN**: Continues logic associated with callable symbol `__tg_exp`.
  **L412 CN**: 继续与可调用符号 `__tg_exp` 相关的逻辑。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L414 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L415 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L415 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L416 EN**: Continues logic associated with callable symbol `__tg_exp`.
  **L416 CN**: 继续与可调用符号 `__tg_exp` 相关的逻辑。
- **L417 EN**: Blank line separating nearby declarations or logic blocks.
  **L417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L418 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L418 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L419 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L419 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L420 EN**: Continues logic associated with callable symbol `__tg_exp`.
  **L420 CN**: 继续与可调用符号 `__tg_exp` 相关的逻辑。
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L422 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L422 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L423 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L423 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L424 EN**: Continues logic associated with callable symbol `__tg_exp`.
  **L424 CN**: 继续与可调用符号 `__tg_exp` 相关的逻辑。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L426 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L426 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L427 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L427 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L428 EN**: Continues logic associated with callable symbol `__tg_exp`.
  **L428 CN**: 继续与可调用符号 `__tg_exp` 相关的逻辑。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L430 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef exp`.
  **L430 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef exp`。
- **L431 EN**: Defines macro `exp(__x)` for conditional compilation, shorthand, or API generation.
  **L431 CN**: 定义宏 `exp(__x)`，用于条件编译、简写或 API 生成。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 433-456

````c
// log

static float
    _TG_ATTRS
    __tg_log(float __x) {return logf(__x);}

static double
    _TG_ATTRS
    __tg_log(double __x) {return log(__x);}

static long double
    _TG_ATTRS
    __tg_log(long double __x) {return logl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_log(float _Complex __x) {return clogf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_log(double _Complex __x) {return clog(__x);}

static long double _Complex
    _TG_ATTRS
````
- **L433 EN**: Comment explains nearby logic, constraints, or intent: `log`.
  **L433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`log`。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L435 EN**: Continues the surrounding expression or declaration: `static float`.
  **L435 CN**: 继续构造周围的表达式或声明：`static float`。
- **L436 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L436 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L437 EN**: Continues logic associated with callable symbol `__tg_log`.
  **L437 CN**: 继续与可调用符号 `__tg_log` 相关的逻辑。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L439 EN**: Continues the surrounding expression or declaration: `static double`.
  **L439 CN**: 继续构造周围的表达式或声明：`static double`。
- **L440 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L440 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L441 EN**: Continues logic associated with callable symbol `__tg_log`.
  **L441 CN**: 继续与可调用符号 `__tg_log` 相关的逻辑。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L443 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L443 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L444 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L444 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L445 EN**: Continues logic associated with callable symbol `__tg_log`.
  **L445 CN**: 继续与可调用符号 `__tg_log` 相关的逻辑。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L447 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L447 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L448 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L448 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L449 EN**: Continues logic associated with callable symbol `__tg_log`.
  **L449 CN**: 继续与可调用符号 `__tg_log` 相关的逻辑。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L451 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L451 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L452 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L452 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L453 EN**: Continues logic associated with callable symbol `__tg_log`.
  **L453 CN**: 继续与可调用符号 `__tg_log` 相关的逻辑。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L455 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L455 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L456 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L456 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 457-480

````c
    __tg_log(long double _Complex __x) {return clogl(__x);}

#undef log
#define log(__x) __tg_log(__tg_promote1((__x))(__x))

// pow

static float
    _TG_ATTRS
    __tg_pow(float __x, float __y) {return powf(__x, __y);}

static double
    _TG_ATTRS
    __tg_pow(double __x, double __y) {return pow(__x, __y);}

static long double
    _TG_ATTRS
    __tg_pow(long double __x, long double __y) {return powl(__x, __y);}

static float _Complex
    _TG_ATTRS
    __tg_pow(float _Complex __x, float _Complex __y) {return cpowf(__x, __y);}

static double _Complex
````
- **L457 EN**: Continues logic associated with callable symbol `__tg_log`.
  **L457 CN**: 继续与可调用符号 `__tg_log` 相关的逻辑。
- **L458 EN**: Blank line separating nearby declarations or logic blocks.
  **L458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L459 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef log`.
  **L459 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef log`。
- **L460 EN**: Defines macro `log(__x)` for conditional compilation, shorthand, or API generation.
  **L460 CN**: 定义宏 `log(__x)`，用于条件编译、简写或 API 生成。
- **L461 EN**: Blank line separating nearby declarations or logic blocks.
  **L461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L462 EN**: Comment explains nearby logic, constraints, or intent: `pow`.
  **L462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`pow`。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L464 EN**: Continues the surrounding expression or declaration: `static float`.
  **L464 CN**: 继续构造周围的表达式或声明：`static float`。
- **L465 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L465 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L466 EN**: Continues logic associated with callable symbol `__tg_pow`.
  **L466 CN**: 继续与可调用符号 `__tg_pow` 相关的逻辑。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Continues the surrounding expression or declaration: `static double`.
  **L468 CN**: 继续构造周围的表达式或声明：`static double`。
- **L469 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L469 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L470 EN**: Continues logic associated with callable symbol `__tg_pow`.
  **L470 CN**: 继续与可调用符号 `__tg_pow` 相关的逻辑。
- **L471 EN**: Blank line separating nearby declarations or logic blocks.
  **L471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L472 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L472 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L473 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L473 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L474 EN**: Continues logic associated with callable symbol `__tg_pow`.
  **L474 CN**: 继续与可调用符号 `__tg_pow` 相关的逻辑。
- **L475 EN**: Blank line separating nearby declarations or logic blocks.
  **L475 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L476 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L476 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L477 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L477 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L478 EN**: Continues logic associated with callable symbol `__tg_pow`.
  **L478 CN**: 继续与可调用符号 `__tg_pow` 相关的逻辑。
- **L479 EN**: Blank line separating nearby declarations or logic blocks.
  **L479 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L480 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L480 CN**: 继续构造周围的表达式或声明：`static double _Complex`。

### Lines 481-504

````c
    _TG_ATTRS
    __tg_pow(double _Complex __x, double _Complex __y) {return cpow(__x, __y);}

static long double _Complex
    _TG_ATTRS
    __tg_pow(long double _Complex __x, long double _Complex __y)
    {return cpowl(__x, __y);}

#undef pow
#define pow(__x, __y) __tg_pow(__tg_promote2((__x), (__y))(__x), \
                               __tg_promote2((__x), (__y))(__y))

// sqrt

static float
    _TG_ATTRS
    __tg_sqrt(float __x) {return sqrtf(__x);}

static double
    _TG_ATTRS
    __tg_sqrt(double __x) {return sqrt(__x);}

static long double
    _TG_ATTRS
````
- **L481 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L481 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L482 EN**: Continues logic associated with callable symbol `__tg_pow`.
  **L482 CN**: 继续与可调用符号 `__tg_pow` 相关的逻辑。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L484 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L484 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L485 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L485 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L486 EN**: Continues logic associated with callable symbol `__tg_pow`.
  **L486 CN**: 继续与可调用符号 `__tg_pow` 相关的逻辑。
- **L487 EN**: Continues logic associated with callable symbol `cpowl`.
  **L487 CN**: 继续与可调用符号 `cpowl` 相关的逻辑。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L489 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef pow`.
  **L489 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef pow`。
- **L490 EN**: Defines macro `pow(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L490 CN**: 定义宏 `pow(__x, __y)`，用于条件编译、简写或 API 生成。
- **L491 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L491 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L492 EN**: Blank line separating nearby declarations or logic blocks.
  **L492 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L493 EN**: Comment explains nearby logic, constraints, or intent: `sqrt`.
  **L493 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sqrt`。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L495 EN**: Continues the surrounding expression or declaration: `static float`.
  **L495 CN**: 继续构造周围的表达式或声明：`static float`。
- **L496 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L496 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L497 EN**: Continues logic associated with callable symbol `__tg_sqrt`.
  **L497 CN**: 继续与可调用符号 `__tg_sqrt` 相关的逻辑。
- **L498 EN**: Blank line separating nearby declarations or logic blocks.
  **L498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L499 EN**: Continues the surrounding expression or declaration: `static double`.
  **L499 CN**: 继续构造周围的表达式或声明：`static double`。
- **L500 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L500 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L501 EN**: Continues logic associated with callable symbol `__tg_sqrt`.
  **L501 CN**: 继续与可调用符号 `__tg_sqrt` 相关的逻辑。
- **L502 EN**: Blank line separating nearby declarations or logic blocks.
  **L502 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L503 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L503 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L504 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L504 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 505-528

````c
    __tg_sqrt(long double __x) {return sqrtl(__x);}

static float _Complex
    _TG_ATTRS
    __tg_sqrt(float _Complex __x) {return csqrtf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_sqrt(double _Complex __x) {return csqrt(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_sqrt(long double _Complex __x) {return csqrtl(__x);}

#undef sqrt
#define sqrt(__x) __tg_sqrt(__tg_promote1((__x))(__x))

// fabs

static float
    _TG_ATTRS
    __tg_fabs(float __x) {return fabsf(__x);}

static double
````
- **L505 EN**: Continues logic associated with callable symbol `__tg_sqrt`.
  **L505 CN**: 继续与可调用符号 `__tg_sqrt` 相关的逻辑。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L507 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L507 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L508 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L508 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L509 EN**: Continues logic associated with callable symbol `__tg_sqrt`.
  **L509 CN**: 继续与可调用符号 `__tg_sqrt` 相关的逻辑。
- **L510 EN**: Blank line separating nearby declarations or logic blocks.
  **L510 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L511 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L511 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L512 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L512 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L513 EN**: Continues logic associated with callable symbol `__tg_sqrt`.
  **L513 CN**: 继续与可调用符号 `__tg_sqrt` 相关的逻辑。
- **L514 EN**: Blank line separating nearby declarations or logic blocks.
  **L514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L515 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L515 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L516 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L516 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L517 EN**: Continues logic associated with callable symbol `__tg_sqrt`.
  **L517 CN**: 继续与可调用符号 `__tg_sqrt` 相关的逻辑。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef sqrt`.
  **L519 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef sqrt`。
- **L520 EN**: Defines macro `sqrt(__x)` for conditional compilation, shorthand, or API generation.
  **L520 CN**: 定义宏 `sqrt(__x)`，用于条件编译、简写或 API 生成。
- **L521 EN**: Blank line separating nearby declarations or logic blocks.
  **L521 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L522 EN**: Comment explains nearby logic, constraints, or intent: `fabs`.
  **L522 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fabs`。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L524 EN**: Continues the surrounding expression or declaration: `static float`.
  **L524 CN**: 继续构造周围的表达式或声明：`static float`。
- **L525 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L525 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L526 EN**: Continues logic associated with callable symbol `__tg_fabs`.
  **L526 CN**: 继续与可调用符号 `__tg_fabs` 相关的逻辑。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L528 EN**: Continues the surrounding expression or declaration: `static double`.
  **L528 CN**: 继续构造周围的表达式或声明：`static double`。

### Lines 529-552

````c
    _TG_ATTRS
    __tg_fabs(double __x) {return fabs(__x);}

static long double
    _TG_ATTRS
    __tg_fabs(long double __x) {return fabsl(__x);}

static float
    _TG_ATTRS
    __tg_fabs(float _Complex __x) {return cabsf(__x);}

static double
    _TG_ATTRS
    __tg_fabs(double _Complex __x) {return cabs(__x);}

static long double
    _TG_ATTRS
    __tg_fabs(long double _Complex __x) {return cabsl(__x);}

#undef fabs
#define fabs(__x) __tg_fabs(__tg_promote1((__x))(__x))

// atan2

````
- **L529 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L529 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L530 EN**: Continues logic associated with callable symbol `__tg_fabs`.
  **L530 CN**: 继续与可调用符号 `__tg_fabs` 相关的逻辑。
- **L531 EN**: Blank line separating nearby declarations or logic blocks.
  **L531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L532 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L532 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L533 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L533 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L534 EN**: Continues logic associated with callable symbol `__tg_fabs`.
  **L534 CN**: 继续与可调用符号 `__tg_fabs` 相关的逻辑。
- **L535 EN**: Blank line separating nearby declarations or logic blocks.
  **L535 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L536 EN**: Continues the surrounding expression or declaration: `static float`.
  **L536 CN**: 继续构造周围的表达式或声明：`static float`。
- **L537 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L537 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L538 EN**: Continues logic associated with callable symbol `__tg_fabs`.
  **L538 CN**: 继续与可调用符号 `__tg_fabs` 相关的逻辑。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L540 EN**: Continues the surrounding expression or declaration: `static double`.
  **L540 CN**: 继续构造周围的表达式或声明：`static double`。
- **L541 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L541 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L542 EN**: Continues logic associated with callable symbol `__tg_fabs`.
  **L542 CN**: 继续与可调用符号 `__tg_fabs` 相关的逻辑。
- **L543 EN**: Blank line separating nearby declarations or logic blocks.
  **L543 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L544 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L544 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L545 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L545 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L546 EN**: Continues logic associated with callable symbol `__tg_fabs`.
  **L546 CN**: 继续与可调用符号 `__tg_fabs` 相关的逻辑。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L548 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef fabs`.
  **L548 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef fabs`。
- **L549 EN**: Defines macro `fabs(__x)` for conditional compilation, shorthand, or API generation.
  **L549 CN**: 定义宏 `fabs(__x)`，用于条件编译、简写或 API 生成。
- **L550 EN**: Blank line separating nearby declarations or logic blocks.
  **L550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L551 EN**: Comment explains nearby logic, constraints, or intent: `atan2`.
  **L551 CN**: 注释解释附近代码的逻辑、约束或设计意图：`atan2`。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 553-576

````c
static float
    _TG_ATTRS
    __tg_atan2(float __x, float __y) {return atan2f(__x, __y);}

static double
    _TG_ATTRS
    __tg_atan2(double __x, double __y) {return atan2(__x, __y);}

static long double
    _TG_ATTRS
    __tg_atan2(long double __x, long double __y) {return atan2l(__x, __y);}

#undef atan2
#define atan2(__x, __y) __tg_atan2(__tg_promote2((__x), (__y))(__x), \
                                   __tg_promote2((__x), (__y))(__y))

// cbrt

static float
    _TG_ATTRS
    __tg_cbrt(float __x) {return cbrtf(__x);}

static double
    _TG_ATTRS
````
- **L553 EN**: Continues the surrounding expression or declaration: `static float`.
  **L553 CN**: 继续构造周围的表达式或声明：`static float`。
- **L554 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L554 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L555 EN**: Continues logic associated with callable symbol `__tg_atan2`.
  **L555 CN**: 继续与可调用符号 `__tg_atan2` 相关的逻辑。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L557 EN**: Continues the surrounding expression or declaration: `static double`.
  **L557 CN**: 继续构造周围的表达式或声明：`static double`。
- **L558 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L558 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L559 EN**: Continues logic associated with callable symbol `__tg_atan2`.
  **L559 CN**: 继续与可调用符号 `__tg_atan2` 相关的逻辑。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L561 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L561 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L562 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L562 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L563 EN**: Continues logic associated with callable symbol `__tg_atan2`.
  **L563 CN**: 继续与可调用符号 `__tg_atan2` 相关的逻辑。
- **L564 EN**: Blank line separating nearby declarations or logic blocks.
  **L564 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L565 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef atan2`.
  **L565 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef atan2`。
- **L566 EN**: Defines macro `atan2(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L566 CN**: 定义宏 `atan2(__x, __y)`，用于条件编译、简写或 API 生成。
- **L567 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L567 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L569 EN**: Comment explains nearby logic, constraints, or intent: `cbrt`.
  **L569 CN**: 注释解释附近代码的逻辑、约束或设计意图：`cbrt`。
- **L570 EN**: Blank line separating nearby declarations or logic blocks.
  **L570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L571 EN**: Continues the surrounding expression or declaration: `static float`.
  **L571 CN**: 继续构造周围的表达式或声明：`static float`。
- **L572 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L572 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L573 EN**: Continues logic associated with callable symbol `__tg_cbrt`.
  **L573 CN**: 继续与可调用符号 `__tg_cbrt` 相关的逻辑。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Continues the surrounding expression or declaration: `static double`.
  **L575 CN**: 继续构造周围的表达式或声明：`static double`。
- **L576 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L576 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 577-600

````c
    __tg_cbrt(double __x) {return cbrt(__x);}

static long double
    _TG_ATTRS
    __tg_cbrt(long double __x) {return cbrtl(__x);}

#undef cbrt
#define cbrt(__x) __tg_cbrt(__tg_promote1((__x))(__x))

// ceil

static float
    _TG_ATTRS
    __tg_ceil(float __x) {return ceilf(__x);}

static double
    _TG_ATTRS
    __tg_ceil(double __x) {return ceil(__x);}

static long double
    _TG_ATTRS
    __tg_ceil(long double __x) {return ceill(__x);}

#undef ceil
````
- **L577 EN**: Continues logic associated with callable symbol `__tg_cbrt`.
  **L577 CN**: 继续与可调用符号 `__tg_cbrt` 相关的逻辑。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L579 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L579 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L580 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L580 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L581 EN**: Continues logic associated with callable symbol `__tg_cbrt`.
  **L581 CN**: 继续与可调用符号 `__tg_cbrt` 相关的逻辑。
- **L582 EN**: Blank line separating nearby declarations or logic blocks.
  **L582 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L583 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef cbrt`.
  **L583 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef cbrt`。
- **L584 EN**: Defines macro `cbrt(__x)` for conditional compilation, shorthand, or API generation.
  **L584 CN**: 定义宏 `cbrt(__x)`，用于条件编译、简写或 API 生成。
- **L585 EN**: Blank line separating nearby declarations or logic blocks.
  **L585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L586 EN**: Comment explains nearby logic, constraints, or intent: `ceil`.
  **L586 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ceil`。
- **L587 EN**: Blank line separating nearby declarations or logic blocks.
  **L587 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L588 EN**: Continues the surrounding expression or declaration: `static float`.
  **L588 CN**: 继续构造周围的表达式或声明：`static float`。
- **L589 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L589 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L590 EN**: Continues logic associated with callable symbol `__tg_ceil`.
  **L590 CN**: 继续与可调用符号 `__tg_ceil` 相关的逻辑。
- **L591 EN**: Blank line separating nearby declarations or logic blocks.
  **L591 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L592 EN**: Continues the surrounding expression or declaration: `static double`.
  **L592 CN**: 继续构造周围的表达式或声明：`static double`。
- **L593 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L593 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L594 EN**: Continues logic associated with callable symbol `__tg_ceil`.
  **L594 CN**: 继续与可调用符号 `__tg_ceil` 相关的逻辑。
- **L595 EN**: Blank line separating nearby declarations or logic blocks.
  **L595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L596 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L596 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L597 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L597 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L598 EN**: Continues logic associated with callable symbol `__tg_ceil`.
  **L598 CN**: 继续与可调用符号 `__tg_ceil` 相关的逻辑。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L600 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef ceil`.
  **L600 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef ceil`。

### Lines 601-624

````c
#define ceil(__x) __tg_ceil(__tg_promote1((__x))(__x))

// copysign

static float
    _TG_ATTRS
    __tg_copysign(float __x, float __y) {return copysignf(__x, __y);}

static double
    _TG_ATTRS
    __tg_copysign(double __x, double __y) {return copysign(__x, __y);}

static long double
    _TG_ATTRS
    __tg_copysign(long double __x, long double __y) {return copysignl(__x, __y);}

#undef copysign
#define copysign(__x, __y) __tg_copysign(__tg_promote2((__x), (__y))(__x), \
                                         __tg_promote2((__x), (__y))(__y))

// erf

static float
    _TG_ATTRS
````
- **L601 EN**: Defines macro `ceil(__x)` for conditional compilation, shorthand, or API generation.
  **L601 CN**: 定义宏 `ceil(__x)`，用于条件编译、简写或 API 生成。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L603 EN**: Comment explains nearby logic, constraints, or intent: `copysign`.
  **L603 CN**: 注释解释附近代码的逻辑、约束或设计意图：`copysign`。
- **L604 EN**: Blank line separating nearby declarations or logic blocks.
  **L604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L605 EN**: Continues the surrounding expression or declaration: `static float`.
  **L605 CN**: 继续构造周围的表达式或声明：`static float`。
- **L606 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L606 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L607 EN**: Continues logic associated with callable symbol `__tg_copysign`.
  **L607 CN**: 继续与可调用符号 `__tg_copysign` 相关的逻辑。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Continues the surrounding expression or declaration: `static double`.
  **L609 CN**: 继续构造周围的表达式或声明：`static double`。
- **L610 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L610 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L611 EN**: Continues logic associated with callable symbol `__tg_copysign`.
  **L611 CN**: 继续与可调用符号 `__tg_copysign` 相关的逻辑。
- **L612 EN**: Blank line separating nearby declarations or logic blocks.
  **L612 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L613 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L613 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L614 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L614 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L615 EN**: Continues logic associated with callable symbol `__tg_copysign`.
  **L615 CN**: 继续与可调用符号 `__tg_copysign` 相关的逻辑。
- **L616 EN**: Blank line separating nearby declarations or logic blocks.
  **L616 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L617 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef copysign`.
  **L617 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef copysign`。
- **L618 EN**: Defines macro `copysign(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L618 CN**: 定义宏 `copysign(__x, __y)`，用于条件编译、简写或 API 生成。
- **L619 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L619 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Comment explains nearby logic, constraints, or intent: `erf`.
  **L621 CN**: 注释解释附近代码的逻辑、约束或设计意图：`erf`。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L623 EN**: Continues the surrounding expression or declaration: `static float`.
  **L623 CN**: 继续构造周围的表达式或声明：`static float`。
- **L624 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L624 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 625-648

````c
    __tg_erf(float __x) {return erff(__x);}

static double
    _TG_ATTRS
    __tg_erf(double __x) {return erf(__x);}

static long double
    _TG_ATTRS
    __tg_erf(long double __x) {return erfl(__x);}

#undef erf
#define erf(__x) __tg_erf(__tg_promote1((__x))(__x))

// erfc

static float
    _TG_ATTRS
    __tg_erfc(float __x) {return erfcf(__x);}

static double
    _TG_ATTRS
    __tg_erfc(double __x) {return erfc(__x);}

static long double
````
- **L625 EN**: Continues logic associated with callable symbol `__tg_erf`.
  **L625 CN**: 继续与可调用符号 `__tg_erf` 相关的逻辑。
- **L626 EN**: Blank line separating nearby declarations or logic blocks.
  **L626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L627 EN**: Continues the surrounding expression or declaration: `static double`.
  **L627 CN**: 继续构造周围的表达式或声明：`static double`。
- **L628 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L628 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L629 EN**: Continues logic associated with callable symbol `__tg_erf`.
  **L629 CN**: 继续与可调用符号 `__tg_erf` 相关的逻辑。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L631 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L631 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L632 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L632 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L633 EN**: Continues logic associated with callable symbol `__tg_erf`.
  **L633 CN**: 继续与可调用符号 `__tg_erf` 相关的逻辑。
- **L634 EN**: Blank line separating nearby declarations or logic blocks.
  **L634 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L635 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef erf`.
  **L635 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef erf`。
- **L636 EN**: Defines macro `erf(__x)` for conditional compilation, shorthand, or API generation.
  **L636 CN**: 定义宏 `erf(__x)`，用于条件编译、简写或 API 生成。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L638 EN**: Comment explains nearby logic, constraints, or intent: `erfc`.
  **L638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`erfc`。
- **L639 EN**: Blank line separating nearby declarations or logic blocks.
  **L639 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L640 EN**: Continues the surrounding expression or declaration: `static float`.
  **L640 CN**: 继续构造周围的表达式或声明：`static float`。
- **L641 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L641 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L642 EN**: Continues logic associated with callable symbol `__tg_erfc`.
  **L642 CN**: 继续与可调用符号 `__tg_erfc` 相关的逻辑。
- **L643 EN**: Blank line separating nearby declarations or logic blocks.
  **L643 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L644 EN**: Continues the surrounding expression or declaration: `static double`.
  **L644 CN**: 继续构造周围的表达式或声明：`static double`。
- **L645 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L645 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L646 EN**: Continues logic associated with callable symbol `__tg_erfc`.
  **L646 CN**: 继续与可调用符号 `__tg_erfc` 相关的逻辑。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L648 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L648 CN**: 继续构造周围的表达式或声明：`static long double`。

### Lines 649-672

````c
    _TG_ATTRS
    __tg_erfc(long double __x) {return erfcl(__x);}

#undef erfc
#define erfc(__x) __tg_erfc(__tg_promote1((__x))(__x))

// exp2

static float
    _TG_ATTRS
    __tg_exp2(float __x) {return exp2f(__x);}

static double
    _TG_ATTRS
    __tg_exp2(double __x) {return exp2(__x);}

static long double
    _TG_ATTRS
    __tg_exp2(long double __x) {return exp2l(__x);}

#undef exp2
#define exp2(__x) __tg_exp2(__tg_promote1((__x))(__x))

// expm1
````
- **L649 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L649 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L650 EN**: Continues logic associated with callable symbol `__tg_erfc`.
  **L650 CN**: 继续与可调用符号 `__tg_erfc` 相关的逻辑。
- **L651 EN**: Blank line separating nearby declarations or logic blocks.
  **L651 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L652 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef erfc`.
  **L652 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef erfc`。
- **L653 EN**: Defines macro `erfc(__x)` for conditional compilation, shorthand, or API generation.
  **L653 CN**: 定义宏 `erfc(__x)`，用于条件编译、简写或 API 生成。
- **L654 EN**: Blank line separating nearby declarations or logic blocks.
  **L654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L655 EN**: Comment explains nearby logic, constraints, or intent: `exp2`.
  **L655 CN**: 注释解释附近代码的逻辑、约束或设计意图：`exp2`。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Continues the surrounding expression or declaration: `static float`.
  **L657 CN**: 继续构造周围的表达式或声明：`static float`。
- **L658 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L658 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L659 EN**: Continues logic associated with callable symbol `__tg_exp2`.
  **L659 CN**: 继续与可调用符号 `__tg_exp2` 相关的逻辑。
- **L660 EN**: Blank line separating nearby declarations or logic blocks.
  **L660 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L661 EN**: Continues the surrounding expression or declaration: `static double`.
  **L661 CN**: 继续构造周围的表达式或声明：`static double`。
- **L662 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L662 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L663 EN**: Continues logic associated with callable symbol `__tg_exp2`.
  **L663 CN**: 继续与可调用符号 `__tg_exp2` 相关的逻辑。
- **L664 EN**: Blank line separating nearby declarations or logic blocks.
  **L664 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L665 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L665 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L666 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L666 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L667 EN**: Continues logic associated with callable symbol `__tg_exp2`.
  **L667 CN**: 继续与可调用符号 `__tg_exp2` 相关的逻辑。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef exp2`.
  **L669 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef exp2`。
- **L670 EN**: Defines macro `exp2(__x)` for conditional compilation, shorthand, or API generation.
  **L670 CN**: 定义宏 `exp2(__x)`，用于条件编译、简写或 API 生成。
- **L671 EN**: Blank line separating nearby declarations or logic blocks.
  **L671 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L672 EN**: Comment explains nearby logic, constraints, or intent: `expm1`.
  **L672 CN**: 注释解释附近代码的逻辑、约束或设计意图：`expm1`。

### Lines 673-696

````c

static float
    _TG_ATTRS
    __tg_expm1(float __x) {return expm1f(__x);}

static double
    _TG_ATTRS
    __tg_expm1(double __x) {return expm1(__x);}

static long double
    _TG_ATTRS
    __tg_expm1(long double __x) {return expm1l(__x);}

#undef expm1
#define expm1(__x) __tg_expm1(__tg_promote1((__x))(__x))

// fdim

static float
    _TG_ATTRS
    __tg_fdim(float __x, float __y) {return fdimf(__x, __y);}

static double
    _TG_ATTRS
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Continues the surrounding expression or declaration: `static float`.
  **L674 CN**: 继续构造周围的表达式或声明：`static float`。
- **L675 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L675 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L676 EN**: Continues logic associated with callable symbol `__tg_expm1`.
  **L676 CN**: 继续与可调用符号 `__tg_expm1` 相关的逻辑。
- **L677 EN**: Blank line separating nearby declarations or logic blocks.
  **L677 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L678 EN**: Continues the surrounding expression or declaration: `static double`.
  **L678 CN**: 继续构造周围的表达式或声明：`static double`。
- **L679 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L679 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L680 EN**: Continues logic associated with callable symbol `__tg_expm1`.
  **L680 CN**: 继续与可调用符号 `__tg_expm1` 相关的逻辑。
- **L681 EN**: Blank line separating nearby declarations or logic blocks.
  **L681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L682 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L682 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L683 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L683 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L684 EN**: Continues logic associated with callable symbol `__tg_expm1`.
  **L684 CN**: 继续与可调用符号 `__tg_expm1` 相关的逻辑。
- **L685 EN**: Blank line separating nearby declarations or logic blocks.
  **L685 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L686 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef expm1`.
  **L686 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef expm1`。
- **L687 EN**: Defines macro `expm1(__x)` for conditional compilation, shorthand, or API generation.
  **L687 CN**: 定义宏 `expm1(__x)`，用于条件编译、简写或 API 生成。
- **L688 EN**: Blank line separating nearby declarations or logic blocks.
  **L688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L689 EN**: Comment explains nearby logic, constraints, or intent: `fdim`.
  **L689 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fdim`。
- **L690 EN**: Blank line separating nearby declarations or logic blocks.
  **L690 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L691 EN**: Continues the surrounding expression or declaration: `static float`.
  **L691 CN**: 继续构造周围的表达式或声明：`static float`。
- **L692 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L692 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L693 EN**: Continues logic associated with callable symbol `__tg_fdim`.
  **L693 CN**: 继续与可调用符号 `__tg_fdim` 相关的逻辑。
- **L694 EN**: Blank line separating nearby declarations or logic blocks.
  **L694 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L695 EN**: Continues the surrounding expression or declaration: `static double`.
  **L695 CN**: 继续构造周围的表达式或声明：`static double`。
- **L696 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L696 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 697-720

````c
    __tg_fdim(double __x, double __y) {return fdim(__x, __y);}

static long double
    _TG_ATTRS
    __tg_fdim(long double __x, long double __y) {return fdiml(__x, __y);}

#undef fdim
#define fdim(__x, __y) __tg_fdim(__tg_promote2((__x), (__y))(__x), \
                                 __tg_promote2((__x), (__y))(__y))

// floor

static float
    _TG_ATTRS
    __tg_floor(float __x) {return floorf(__x);}

static double
    _TG_ATTRS
    __tg_floor(double __x) {return floor(__x);}

static long double
    _TG_ATTRS
    __tg_floor(long double __x) {return floorl(__x);}

````
- **L697 EN**: Continues logic associated with callable symbol `__tg_fdim`.
  **L697 CN**: 继续与可调用符号 `__tg_fdim` 相关的逻辑。
- **L698 EN**: Blank line separating nearby declarations or logic blocks.
  **L698 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L699 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L699 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L700 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L700 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L701 EN**: Continues logic associated with callable symbol `__tg_fdim`.
  **L701 CN**: 继续与可调用符号 `__tg_fdim` 相关的逻辑。
- **L702 EN**: Blank line separating nearby declarations or logic blocks.
  **L702 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L703 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef fdim`.
  **L703 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef fdim`。
- **L704 EN**: Defines macro `fdim(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L704 CN**: 定义宏 `fdim(__x, __y)`，用于条件编译、简写或 API 生成。
- **L705 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L705 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L706 EN**: Blank line separating nearby declarations or logic blocks.
  **L706 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L707 EN**: Comment explains nearby logic, constraints, or intent: `floor`.
  **L707 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floor`。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L709 EN**: Continues the surrounding expression or declaration: `static float`.
  **L709 CN**: 继续构造周围的表达式或声明：`static float`。
- **L710 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L710 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L711 EN**: Continues logic associated with callable symbol `__tg_floor`.
  **L711 CN**: 继续与可调用符号 `__tg_floor` 相关的逻辑。
- **L712 EN**: Blank line separating nearby declarations or logic blocks.
  **L712 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L713 EN**: Continues the surrounding expression or declaration: `static double`.
  **L713 CN**: 继续构造周围的表达式或声明：`static double`。
- **L714 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L714 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L715 EN**: Continues logic associated with callable symbol `__tg_floor`.
  **L715 CN**: 继续与可调用符号 `__tg_floor` 相关的逻辑。
- **L716 EN**: Blank line separating nearby declarations or logic blocks.
  **L716 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L717 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L717 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L718 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L718 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L719 EN**: Continues logic associated with callable symbol `__tg_floor`.
  **L719 CN**: 继续与可调用符号 `__tg_floor` 相关的逻辑。
- **L720 EN**: Blank line separating nearby declarations or logic blocks.
  **L720 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 721-744

````c
#undef floor
#define floor(__x) __tg_floor(__tg_promote1((__x))(__x))

// fma

static float
    _TG_ATTRS
    __tg_fma(float __x, float __y, float __z)
    {return fmaf(__x, __y, __z);}

static double
    _TG_ATTRS
    __tg_fma(double __x, double __y, double __z)
    {return fma(__x, __y, __z);}

static long double
    _TG_ATTRS
    __tg_fma(long double __x,long double __y, long double __z)
    {return fmal(__x, __y, __z);}

#undef fma
#define fma(__x, __y, __z)                                \
        __tg_fma(__tg_promote3((__x), (__y), (__z))(__x), \
                 __tg_promote3((__x), (__y), (__z))(__y), \
````
- **L721 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef floor`.
  **L721 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef floor`。
- **L722 EN**: Defines macro `floor(__x)` for conditional compilation, shorthand, or API generation.
  **L722 CN**: 定义宏 `floor(__x)`，用于条件编译、简写或 API 生成。
- **L723 EN**: Blank line separating nearby declarations or logic blocks.
  **L723 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L724 EN**: Comment explains nearby logic, constraints, or intent: `fma`.
  **L724 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fma`。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L726 EN**: Continues the surrounding expression or declaration: `static float`.
  **L726 CN**: 继续构造周围的表达式或声明：`static float`。
- **L727 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L727 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L728 EN**: Continues logic associated with callable symbol `__tg_fma`.
  **L728 CN**: 继续与可调用符号 `__tg_fma` 相关的逻辑。
- **L729 EN**: Continues logic associated with callable symbol `fmaf`.
  **L729 CN**: 继续与可调用符号 `fmaf` 相关的逻辑。
- **L730 EN**: Blank line separating nearby declarations or logic blocks.
  **L730 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L731 EN**: Continues the surrounding expression or declaration: `static double`.
  **L731 CN**: 继续构造周围的表达式或声明：`static double`。
- **L732 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L732 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L733 EN**: Continues logic associated with callable symbol `__tg_fma`.
  **L733 CN**: 继续与可调用符号 `__tg_fma` 相关的逻辑。
- **L734 EN**: Continues logic associated with callable symbol `fma`.
  **L734 CN**: 继续与可调用符号 `fma` 相关的逻辑。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L736 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L736 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L737 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L737 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L738 EN**: Continues logic associated with callable symbol `__tg_fma`.
  **L738 CN**: 继续与可调用符号 `__tg_fma` 相关的逻辑。
- **L739 EN**: Continues logic associated with callable symbol `fmal`.
  **L739 CN**: 继续与可调用符号 `fmal` 相关的逻辑。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef fma`.
  **L741 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef fma`。
- **L742 EN**: Defines macro `fma(__x, __y, __z)` for conditional compilation, shorthand, or API generation.
  **L742 CN**: 定义宏 `fma(__x, __y, __z)`，用于条件编译、简写或 API 生成。
- **L743 EN**: Continues logic associated with callable symbol `__tg_fma`.
  **L743 CN**: 继续与可调用符号 `__tg_fma` 相关的逻辑。
- **L744 EN**: Continues logic associated with callable symbol `__tg_promote3`.
  **L744 CN**: 继续与可调用符号 `__tg_promote3` 相关的逻辑。

### Lines 745-768

````c
                 __tg_promote3((__x), (__y), (__z))(__z))

// fmax

static float
    _TG_ATTRS
    __tg_fmax(float __x, float __y) {return fmaxf(__x, __y);}

static double
    _TG_ATTRS
    __tg_fmax(double __x, double __y) {return fmax(__x, __y);}

static long double
    _TG_ATTRS
    __tg_fmax(long double __x, long double __y) {return fmaxl(__x, __y);}

#undef fmax
#define fmax(__x, __y) __tg_fmax(__tg_promote2((__x), (__y))(__x), \
                                 __tg_promote2((__x), (__y))(__y))

// fmin

static float
    _TG_ATTRS
````
- **L745 EN**: Continues logic associated with callable symbol `__tg_promote3`.
  **L745 CN**: 继续与可调用符号 `__tg_promote3` 相关的逻辑。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L747 EN**: Comment explains nearby logic, constraints, or intent: `fmax`.
  **L747 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fmax`。
- **L748 EN**: Blank line separating nearby declarations or logic blocks.
  **L748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L749 EN**: Continues the surrounding expression or declaration: `static float`.
  **L749 CN**: 继续构造周围的表达式或声明：`static float`。
- **L750 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L750 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L751 EN**: Continues logic associated with callable symbol `__tg_fmax`.
  **L751 CN**: 继续与可调用符号 `__tg_fmax` 相关的逻辑。
- **L752 EN**: Blank line separating nearby declarations or logic blocks.
  **L752 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L753 EN**: Continues the surrounding expression or declaration: `static double`.
  **L753 CN**: 继续构造周围的表达式或声明：`static double`。
- **L754 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L754 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L755 EN**: Continues logic associated with callable symbol `__tg_fmax`.
  **L755 CN**: 继续与可调用符号 `__tg_fmax` 相关的逻辑。
- **L756 EN**: Blank line separating nearby declarations or logic blocks.
  **L756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L757 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L757 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L758 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L758 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L759 EN**: Continues logic associated with callable symbol `__tg_fmax`.
  **L759 CN**: 继续与可调用符号 `__tg_fmax` 相关的逻辑。
- **L760 EN**: Blank line separating nearby declarations or logic blocks.
  **L760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L761 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef fmax`.
  **L761 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef fmax`。
- **L762 EN**: Defines macro `fmax(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L762 CN**: 定义宏 `fmax(__x, __y)`，用于条件编译、简写或 API 生成。
- **L763 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L763 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L764 EN**: Blank line separating nearby declarations or logic blocks.
  **L764 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L765 EN**: Comment explains nearby logic, constraints, or intent: `fmin`.
  **L765 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fmin`。
- **L766 EN**: Blank line separating nearby declarations or logic blocks.
  **L766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L767 EN**: Continues the surrounding expression or declaration: `static float`.
  **L767 CN**: 继续构造周围的表达式或声明：`static float`。
- **L768 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L768 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 769-792

````c
    __tg_fmin(float __x, float __y) {return fminf(__x, __y);}

static double
    _TG_ATTRS
    __tg_fmin(double __x, double __y) {return fmin(__x, __y);}

static long double
    _TG_ATTRS
    __tg_fmin(long double __x, long double __y) {return fminl(__x, __y);}

#undef fmin
#define fmin(__x, __y) __tg_fmin(__tg_promote2((__x), (__y))(__x), \
                                 __tg_promote2((__x), (__y))(__y))

// fmod

static float
    _TG_ATTRS
    __tg_fmod(float __x, float __y) {return fmodf(__x, __y);}

static double
    _TG_ATTRS
    __tg_fmod(double __x, double __y) {return fmod(__x, __y);}

````
- **L769 EN**: Continues logic associated with callable symbol `__tg_fmin`.
  **L769 CN**: 继续与可调用符号 `__tg_fmin` 相关的逻辑。
- **L770 EN**: Blank line separating nearby declarations or logic blocks.
  **L770 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L771 EN**: Continues the surrounding expression or declaration: `static double`.
  **L771 CN**: 继续构造周围的表达式或声明：`static double`。
- **L772 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L772 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L773 EN**: Continues logic associated with callable symbol `__tg_fmin`.
  **L773 CN**: 继续与可调用符号 `__tg_fmin` 相关的逻辑。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L775 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L775 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L776 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L776 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L777 EN**: Continues logic associated with callable symbol `__tg_fmin`.
  **L777 CN**: 继续与可调用符号 `__tg_fmin` 相关的逻辑。
- **L778 EN**: Blank line separating nearby declarations or logic blocks.
  **L778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L779 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef fmin`.
  **L779 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef fmin`。
- **L780 EN**: Defines macro `fmin(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L780 CN**: 定义宏 `fmin(__x, __y)`，用于条件编译、简写或 API 生成。
- **L781 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L781 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L783 EN**: Comment explains nearby logic, constraints, or intent: `fmod`.
  **L783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fmod`。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L785 EN**: Continues the surrounding expression or declaration: `static float`.
  **L785 CN**: 继续构造周围的表达式或声明：`static float`。
- **L786 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L786 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L787 EN**: Continues logic associated with callable symbol `__tg_fmod`.
  **L787 CN**: 继续与可调用符号 `__tg_fmod` 相关的逻辑。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L789 EN**: Continues the surrounding expression or declaration: `static double`.
  **L789 CN**: 继续构造周围的表达式或声明：`static double`。
- **L790 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L790 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L791 EN**: Continues logic associated with callable symbol `__tg_fmod`.
  **L791 CN**: 继续与可调用符号 `__tg_fmod` 相关的逻辑。
- **L792 EN**: Blank line separating nearby declarations or logic blocks.
  **L792 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 793-816

````c
static long double
    _TG_ATTRS
    __tg_fmod(long double __x, long double __y) {return fmodl(__x, __y);}

#undef fmod
#define fmod(__x, __y) __tg_fmod(__tg_promote2((__x), (__y))(__x), \
                                 __tg_promote2((__x), (__y))(__y))

// frexp

static float
    _TG_ATTRS
    __tg_frexp(float __x, int* __y) {return frexpf(__x, __y);}

static double
    _TG_ATTRS
    __tg_frexp(double __x, int* __y) {return frexp(__x, __y);}

static long double
    _TG_ATTRS
    __tg_frexp(long double __x, int* __y) {return frexpl(__x, __y);}

#undef frexp
#define frexp(__x, __y) __tg_frexp(__tg_promote1((__x))(__x), __y)
````
- **L793 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L793 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L794 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L794 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L795 EN**: Continues logic associated with callable symbol `__tg_fmod`.
  **L795 CN**: 继续与可调用符号 `__tg_fmod` 相关的逻辑。
- **L796 EN**: Blank line separating nearby declarations or logic blocks.
  **L796 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L797 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef fmod`.
  **L797 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef fmod`。
- **L798 EN**: Defines macro `fmod(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L798 CN**: 定义宏 `fmod(__x, __y)`，用于条件编译、简写或 API 生成。
- **L799 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L799 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L800 EN**: Blank line separating nearby declarations or logic blocks.
  **L800 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L801 EN**: Comment explains nearby logic, constraints, or intent: `frexp`.
  **L801 CN**: 注释解释附近代码的逻辑、约束或设计意图：`frexp`。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L803 EN**: Continues the surrounding expression or declaration: `static float`.
  **L803 CN**: 继续构造周围的表达式或声明：`static float`。
- **L804 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L804 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L805 EN**: Continues logic associated with callable symbol `__tg_frexp`.
  **L805 CN**: 继续与可调用符号 `__tg_frexp` 相关的逻辑。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L807 EN**: Continues the surrounding expression or declaration: `static double`.
  **L807 CN**: 继续构造周围的表达式或声明：`static double`。
- **L808 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L808 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L809 EN**: Continues logic associated with callable symbol `__tg_frexp`.
  **L809 CN**: 继续与可调用符号 `__tg_frexp` 相关的逻辑。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L811 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L811 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L812 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L812 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L813 EN**: Continues logic associated with callable symbol `__tg_frexp`.
  **L813 CN**: 继续与可调用符号 `__tg_frexp` 相关的逻辑。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef frexp`.
  **L815 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef frexp`。
- **L816 EN**: Defines macro `frexp(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L816 CN**: 定义宏 `frexp(__x, __y)`，用于条件编译、简写或 API 生成。

### Lines 817-840

````c

// hypot

static float
    _TG_ATTRS
    __tg_hypot(float __x, float __y) {return hypotf(__x, __y);}

static double
    _TG_ATTRS
    __tg_hypot(double __x, double __y) {return hypot(__x, __y);}

static long double
    _TG_ATTRS
    __tg_hypot(long double __x, long double __y) {return hypotl(__x, __y);}

#undef hypot
#define hypot(__x, __y) __tg_hypot(__tg_promote2((__x), (__y))(__x), \
                                   __tg_promote2((__x), (__y))(__y))

// ilogb

static int
    _TG_ATTRS
    __tg_ilogb(float __x) {return ilogbf(__x);}
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L818 EN**: Comment explains nearby logic, constraints, or intent: `hypot`.
  **L818 CN**: 注释解释附近代码的逻辑、约束或设计意图：`hypot`。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L820 EN**: Continues the surrounding expression or declaration: `static float`.
  **L820 CN**: 继续构造周围的表达式或声明：`static float`。
- **L821 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L821 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L822 EN**: Continues logic associated with callable symbol `__tg_hypot`.
  **L822 CN**: 继续与可调用符号 `__tg_hypot` 相关的逻辑。
- **L823 EN**: Blank line separating nearby declarations or logic blocks.
  **L823 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L824 EN**: Continues the surrounding expression or declaration: `static double`.
  **L824 CN**: 继续构造周围的表达式或声明：`static double`。
- **L825 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L825 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L826 EN**: Continues logic associated with callable symbol `__tg_hypot`.
  **L826 CN**: 继续与可调用符号 `__tg_hypot` 相关的逻辑。
- **L827 EN**: Blank line separating nearby declarations or logic blocks.
  **L827 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L828 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L828 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L829 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L829 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L830 EN**: Continues logic associated with callable symbol `__tg_hypot`.
  **L830 CN**: 继续与可调用符号 `__tg_hypot` 相关的逻辑。
- **L831 EN**: Blank line separating nearby declarations or logic blocks.
  **L831 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L832 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef hypot`.
  **L832 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef hypot`。
- **L833 EN**: Defines macro `hypot(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L833 CN**: 定义宏 `hypot(__x, __y)`，用于条件编译、简写或 API 生成。
- **L834 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L834 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L835 EN**: Blank line separating nearby declarations or logic blocks.
  **L835 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L836 EN**: Comment explains nearby logic, constraints, or intent: `ilogb`.
  **L836 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ilogb`。
- **L837 EN**: Blank line separating nearby declarations or logic blocks.
  **L837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L838 EN**: Continues the surrounding expression or declaration: `static int`.
  **L838 CN**: 继续构造周围的表达式或声明：`static int`。
- **L839 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L839 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L840 EN**: Continues logic associated with callable symbol `__tg_ilogb`.
  **L840 CN**: 继续与可调用符号 `__tg_ilogb` 相关的逻辑。

### Lines 841-864

````c

static int
    _TG_ATTRS
    __tg_ilogb(double __x) {return ilogb(__x);}

static int
    _TG_ATTRS
    __tg_ilogb(long double __x) {return ilogbl(__x);}

#undef ilogb
#define ilogb(__x) __tg_ilogb(__tg_promote1((__x))(__x))

// ldexp

static float
    _TG_ATTRS
    __tg_ldexp(float __x, int __y) {return ldexpf(__x, __y);}

static double
    _TG_ATTRS
    __tg_ldexp(double __x, int __y) {return ldexp(__x, __y);}

static long double
    _TG_ATTRS
````
- **L841 EN**: Blank line separating nearby declarations or logic blocks.
  **L841 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L842 EN**: Continues the surrounding expression or declaration: `static int`.
  **L842 CN**: 继续构造周围的表达式或声明：`static int`。
- **L843 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L843 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L844 EN**: Continues logic associated with callable symbol `__tg_ilogb`.
  **L844 CN**: 继续与可调用符号 `__tg_ilogb` 相关的逻辑。
- **L845 EN**: Blank line separating nearby declarations or logic blocks.
  **L845 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L846 EN**: Continues the surrounding expression or declaration: `static int`.
  **L846 CN**: 继续构造周围的表达式或声明：`static int`。
- **L847 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L847 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L848 EN**: Continues logic associated with callable symbol `__tg_ilogb`.
  **L848 CN**: 继续与可调用符号 `__tg_ilogb` 相关的逻辑。
- **L849 EN**: Blank line separating nearby declarations or logic blocks.
  **L849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L850 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef ilogb`.
  **L850 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef ilogb`。
- **L851 EN**: Defines macro `ilogb(__x)` for conditional compilation, shorthand, or API generation.
  **L851 CN**: 定义宏 `ilogb(__x)`，用于条件编译、简写或 API 生成。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Comment explains nearby logic, constraints, or intent: `ldexp`.
  **L853 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ldexp`。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L855 EN**: Continues the surrounding expression or declaration: `static float`.
  **L855 CN**: 继续构造周围的表达式或声明：`static float`。
- **L856 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L856 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L857 EN**: Continues logic associated with callable symbol `__tg_ldexp`.
  **L857 CN**: 继续与可调用符号 `__tg_ldexp` 相关的逻辑。
- **L858 EN**: Blank line separating nearby declarations or logic blocks.
  **L858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L859 EN**: Continues the surrounding expression or declaration: `static double`.
  **L859 CN**: 继续构造周围的表达式或声明：`static double`。
- **L860 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L860 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L861 EN**: Continues logic associated with callable symbol `__tg_ldexp`.
  **L861 CN**: 继续与可调用符号 `__tg_ldexp` 相关的逻辑。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L863 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L863 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L864 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L864 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 865-888

````c
    __tg_ldexp(long double __x, int __y) {return ldexpl(__x, __y);}

#undef ldexp
#define ldexp(__x, __y) __tg_ldexp(__tg_promote1((__x))(__x), __y)

// lgamma

static float
    _TG_ATTRS
    __tg_lgamma(float __x) {return lgammaf(__x);}

static double
    _TG_ATTRS
    __tg_lgamma(double __x) {return lgamma(__x);}

static long double
    _TG_ATTRS
    __tg_lgamma(long double __x) {return lgammal(__x);}

#undef lgamma
#define lgamma(__x) __tg_lgamma(__tg_promote1((__x))(__x))

// llrint

````
- **L865 EN**: Continues logic associated with callable symbol `__tg_ldexp`.
  **L865 CN**: 继续与可调用符号 `__tg_ldexp` 相关的逻辑。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef ldexp`.
  **L867 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef ldexp`。
- **L868 EN**: Defines macro `ldexp(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L868 CN**: 定义宏 `ldexp(__x, __y)`，用于条件编译、简写或 API 生成。
- **L869 EN**: Blank line separating nearby declarations or logic blocks.
  **L869 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L870 EN**: Comment explains nearby logic, constraints, or intent: `lgamma`.
  **L870 CN**: 注释解释附近代码的逻辑、约束或设计意图：`lgamma`。
- **L871 EN**: Blank line separating nearby declarations or logic blocks.
  **L871 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L872 EN**: Continues the surrounding expression or declaration: `static float`.
  **L872 CN**: 继续构造周围的表达式或声明：`static float`。
- **L873 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L873 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L874 EN**: Continues logic associated with callable symbol `__tg_lgamma`.
  **L874 CN**: 继续与可调用符号 `__tg_lgamma` 相关的逻辑。
- **L875 EN**: Blank line separating nearby declarations or logic blocks.
  **L875 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L876 EN**: Continues the surrounding expression or declaration: `static double`.
  **L876 CN**: 继续构造周围的表达式或声明：`static double`。
- **L877 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L877 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L878 EN**: Continues logic associated with callable symbol `__tg_lgamma`.
  **L878 CN**: 继续与可调用符号 `__tg_lgamma` 相关的逻辑。
- **L879 EN**: Blank line separating nearby declarations or logic blocks.
  **L879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L880 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L880 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L881 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L881 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L882 EN**: Continues logic associated with callable symbol `__tg_lgamma`.
  **L882 CN**: 继续与可调用符号 `__tg_lgamma` 相关的逻辑。
- **L883 EN**: Blank line separating nearby declarations or logic blocks.
  **L883 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L884 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef lgamma`.
  **L884 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef lgamma`。
- **L885 EN**: Defines macro `lgamma(__x)` for conditional compilation, shorthand, or API generation.
  **L885 CN**: 定义宏 `lgamma(__x)`，用于条件编译、简写或 API 生成。
- **L886 EN**: Blank line separating nearby declarations or logic blocks.
  **L886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L887 EN**: Comment explains nearby logic, constraints, or intent: `llrint`.
  **L887 CN**: 注释解释附近代码的逻辑、约束或设计意图：`llrint`。
- **L888 EN**: Blank line separating nearby declarations or logic blocks.
  **L888 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 889-912

````c
static long long
    _TG_ATTRS
    __tg_llrint(float __x) {return llrintf(__x);}

static long long
    _TG_ATTRS
    __tg_llrint(double __x) {return llrint(__x);}

static long long
    _TG_ATTRS
    __tg_llrint(long double __x) {return llrintl(__x);}

#undef llrint
#define llrint(__x) __tg_llrint(__tg_promote1((__x))(__x))

// llround

static long long
    _TG_ATTRS
    __tg_llround(float __x) {return llroundf(__x);}

static long long
    _TG_ATTRS
    __tg_llround(double __x) {return llround(__x);}
````
- **L889 EN**: Continues the surrounding expression or declaration: `static long long`.
  **L889 CN**: 继续构造周围的表达式或声明：`static long long`。
- **L890 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L890 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L891 EN**: Continues logic associated with callable symbol `__tg_llrint`.
  **L891 CN**: 继续与可调用符号 `__tg_llrint` 相关的逻辑。
- **L892 EN**: Blank line separating nearby declarations or logic blocks.
  **L892 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L893 EN**: Continues the surrounding expression or declaration: `static long long`.
  **L893 CN**: 继续构造周围的表达式或声明：`static long long`。
- **L894 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L894 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L895 EN**: Continues logic associated with callable symbol `__tg_llrint`.
  **L895 CN**: 继续与可调用符号 `__tg_llrint` 相关的逻辑。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L897 EN**: Continues the surrounding expression or declaration: `static long long`.
  **L897 CN**: 继续构造周围的表达式或声明：`static long long`。
- **L898 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L898 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L899 EN**: Continues logic associated with callable symbol `__tg_llrint`.
  **L899 CN**: 继续与可调用符号 `__tg_llrint` 相关的逻辑。
- **L900 EN**: Blank line separating nearby declarations or logic blocks.
  **L900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L901 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef llrint`.
  **L901 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef llrint`。
- **L902 EN**: Defines macro `llrint(__x)` for conditional compilation, shorthand, or API generation.
  **L902 CN**: 定义宏 `llrint(__x)`，用于条件编译、简写或 API 生成。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L904 EN**: Comment explains nearby logic, constraints, or intent: `llround`.
  **L904 CN**: 注释解释附近代码的逻辑、约束或设计意图：`llround`。
- **L905 EN**: Blank line separating nearby declarations or logic blocks.
  **L905 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L906 EN**: Continues the surrounding expression or declaration: `static long long`.
  **L906 CN**: 继续构造周围的表达式或声明：`static long long`。
- **L907 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L907 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L908 EN**: Continues logic associated with callable symbol `__tg_llround`.
  **L908 CN**: 继续与可调用符号 `__tg_llround` 相关的逻辑。
- **L909 EN**: Blank line separating nearby declarations or logic blocks.
  **L909 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L910 EN**: Continues the surrounding expression or declaration: `static long long`.
  **L910 CN**: 继续构造周围的表达式或声明：`static long long`。
- **L911 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L911 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L912 EN**: Continues logic associated with callable symbol `__tg_llround`.
  **L912 CN**: 继续与可调用符号 `__tg_llround` 相关的逻辑。

### Lines 913-936

````c

static long long
    _TG_ATTRS
    __tg_llround(long double __x) {return llroundl(__x);}

#undef llround
#define llround(__x) __tg_llround(__tg_promote1((__x))(__x))

// log10

static float
    _TG_ATTRS
    __tg_log10(float __x) {return log10f(__x);}

static double
    _TG_ATTRS
    __tg_log10(double __x) {return log10(__x);}

static long double
    _TG_ATTRS
    __tg_log10(long double __x) {return log10l(__x);}

#undef log10
#define log10(__x) __tg_log10(__tg_promote1((__x))(__x))
````
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L914 EN**: Continues the surrounding expression or declaration: `static long long`.
  **L914 CN**: 继续构造周围的表达式或声明：`static long long`。
- **L915 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L915 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L916 EN**: Continues logic associated with callable symbol `__tg_llround`.
  **L916 CN**: 继续与可调用符号 `__tg_llround` 相关的逻辑。
- **L917 EN**: Blank line separating nearby declarations or logic blocks.
  **L917 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L918 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef llround`.
  **L918 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef llround`。
- **L919 EN**: Defines macro `llround(__x)` for conditional compilation, shorthand, or API generation.
  **L919 CN**: 定义宏 `llround(__x)`，用于条件编译、简写或 API 生成。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L921 EN**: Comment explains nearby logic, constraints, or intent: `log10`.
  **L921 CN**: 注释解释附近代码的逻辑、约束或设计意图：`log10`。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L923 EN**: Continues the surrounding expression or declaration: `static float`.
  **L923 CN**: 继续构造周围的表达式或声明：`static float`。
- **L924 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L924 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L925 EN**: Continues logic associated with callable symbol `__tg_log10`.
  **L925 CN**: 继续与可调用符号 `__tg_log10` 相关的逻辑。
- **L926 EN**: Blank line separating nearby declarations or logic blocks.
  **L926 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L927 EN**: Continues the surrounding expression or declaration: `static double`.
  **L927 CN**: 继续构造周围的表达式或声明：`static double`。
- **L928 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L928 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L929 EN**: Continues logic associated with callable symbol `__tg_log10`.
  **L929 CN**: 继续与可调用符号 `__tg_log10` 相关的逻辑。
- **L930 EN**: Blank line separating nearby declarations or logic blocks.
  **L930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L931 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L931 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L932 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L932 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L933 EN**: Continues logic associated with callable symbol `__tg_log10`.
  **L933 CN**: 继续与可调用符号 `__tg_log10` 相关的逻辑。
- **L934 EN**: Blank line separating nearby declarations or logic blocks.
  **L934 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L935 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef log10`.
  **L935 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef log10`。
- **L936 EN**: Defines macro `log10(__x)` for conditional compilation, shorthand, or API generation.
  **L936 CN**: 定义宏 `log10(__x)`，用于条件编译、简写或 API 生成。

### Lines 937-960

````c

// log1p

static float
    _TG_ATTRS
    __tg_log1p(float __x) {return log1pf(__x);}

static double
    _TG_ATTRS
    __tg_log1p(double __x) {return log1p(__x);}

static long double
    _TG_ATTRS
    __tg_log1p(long double __x) {return log1pl(__x);}

#undef log1p
#define log1p(__x) __tg_log1p(__tg_promote1((__x))(__x))

// log2

static float
    _TG_ATTRS
    __tg_log2(float __x) {return log2f(__x);}

````
- **L937 EN**: Blank line separating nearby declarations or logic blocks.
  **L937 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L938 EN**: Comment explains nearby logic, constraints, or intent: `log1p`.
  **L938 CN**: 注释解释附近代码的逻辑、约束或设计意图：`log1p`。
- **L939 EN**: Blank line separating nearby declarations or logic blocks.
  **L939 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L940 EN**: Continues the surrounding expression or declaration: `static float`.
  **L940 CN**: 继续构造周围的表达式或声明：`static float`。
- **L941 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L941 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L942 EN**: Continues logic associated with callable symbol `__tg_log1p`.
  **L942 CN**: 继续与可调用符号 `__tg_log1p` 相关的逻辑。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Continues the surrounding expression or declaration: `static double`.
  **L944 CN**: 继续构造周围的表达式或声明：`static double`。
- **L945 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L945 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L946 EN**: Continues logic associated with callable symbol `__tg_log1p`.
  **L946 CN**: 继续与可调用符号 `__tg_log1p` 相关的逻辑。
- **L947 EN**: Blank line separating nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L948 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L948 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L949 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L949 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L950 EN**: Continues logic associated with callable symbol `__tg_log1p`.
  **L950 CN**: 继续与可调用符号 `__tg_log1p` 相关的逻辑。
- **L951 EN**: Blank line separating nearby declarations or logic blocks.
  **L951 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L952 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef log1p`.
  **L952 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef log1p`。
- **L953 EN**: Defines macro `log1p(__x)` for conditional compilation, shorthand, or API generation.
  **L953 CN**: 定义宏 `log1p(__x)`，用于条件编译、简写或 API 生成。
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L955 EN**: Comment explains nearby logic, constraints, or intent: `log2`.
  **L955 CN**: 注释解释附近代码的逻辑、约束或设计意图：`log2`。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L957 EN**: Continues the surrounding expression or declaration: `static float`.
  **L957 CN**: 继续构造周围的表达式或声明：`static float`。
- **L958 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L958 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L959 EN**: Continues logic associated with callable symbol `__tg_log2`.
  **L959 CN**: 继续与可调用符号 `__tg_log2` 相关的逻辑。
- **L960 EN**: Blank line separating nearby declarations or logic blocks.
  **L960 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 961-984

````c
static double
    _TG_ATTRS
    __tg_log2(double __x) {return log2(__x);}

static long double
    _TG_ATTRS
    __tg_log2(long double __x) {return log2l(__x);}

#undef log2
#define log2(__x) __tg_log2(__tg_promote1((__x))(__x))

// logb

static float
    _TG_ATTRS
    __tg_logb(float __x) {return logbf(__x);}

static double
    _TG_ATTRS
    __tg_logb(double __x) {return logb(__x);}

static long double
    _TG_ATTRS
    __tg_logb(long double __x) {return logbl(__x);}
````
- **L961 EN**: Continues the surrounding expression or declaration: `static double`.
  **L961 CN**: 继续构造周围的表达式或声明：`static double`。
- **L962 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L962 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L963 EN**: Continues logic associated with callable symbol `__tg_log2`.
  **L963 CN**: 继续与可调用符号 `__tg_log2` 相关的逻辑。
- **L964 EN**: Blank line separating nearby declarations or logic blocks.
  **L964 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L965 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L965 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L966 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L966 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L967 EN**: Continues logic associated with callable symbol `__tg_log2`.
  **L967 CN**: 继续与可调用符号 `__tg_log2` 相关的逻辑。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef log2`.
  **L969 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef log2`。
- **L970 EN**: Defines macro `log2(__x)` for conditional compilation, shorthand, or API generation.
  **L970 CN**: 定义宏 `log2(__x)`，用于条件编译、简写或 API 生成。
- **L971 EN**: Blank line separating nearby declarations or logic blocks.
  **L971 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L972 EN**: Comment explains nearby logic, constraints, or intent: `logb`.
  **L972 CN**: 注释解释附近代码的逻辑、约束或设计意图：`logb`。
- **L973 EN**: Blank line separating nearby declarations or logic blocks.
  **L973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L974 EN**: Continues the surrounding expression or declaration: `static float`.
  **L974 CN**: 继续构造周围的表达式或声明：`static float`。
- **L975 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L975 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L976 EN**: Continues logic associated with callable symbol `__tg_logb`.
  **L976 CN**: 继续与可调用符号 `__tg_logb` 相关的逻辑。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L978 EN**: Continues the surrounding expression or declaration: `static double`.
  **L978 CN**: 继续构造周围的表达式或声明：`static double`。
- **L979 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L979 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L980 EN**: Continues logic associated with callable symbol `__tg_logb`.
  **L980 CN**: 继续与可调用符号 `__tg_logb` 相关的逻辑。
- **L981 EN**: Blank line separating nearby declarations or logic blocks.
  **L981 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L982 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L982 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L983 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L983 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L984 EN**: Continues logic associated with callable symbol `__tg_logb`.
  **L984 CN**: 继续与可调用符号 `__tg_logb` 相关的逻辑。

### Lines 985-1008

````c

#undef logb
#define logb(__x) __tg_logb(__tg_promote1((__x))(__x))

// lrint

static long
    _TG_ATTRS
    __tg_lrint(float __x) {return lrintf(__x);}

static long
    _TG_ATTRS
    __tg_lrint(double __x) {return lrint(__x);}

static long
    _TG_ATTRS
    __tg_lrint(long double __x) {return lrintl(__x);}

#undef lrint
#define lrint(__x) __tg_lrint(__tg_promote1((__x))(__x))

// lround

static long
````
- **L985 EN**: Blank line separating nearby declarations or logic blocks.
  **L985 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L986 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef logb`.
  **L986 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef logb`。
- **L987 EN**: Defines macro `logb(__x)` for conditional compilation, shorthand, or API generation.
  **L987 CN**: 定义宏 `logb(__x)`，用于条件编译、简写或 API 生成。
- **L988 EN**: Blank line separating nearby declarations or logic blocks.
  **L988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L989 EN**: Comment explains nearby logic, constraints, or intent: `lrint`.
  **L989 CN**: 注释解释附近代码的逻辑、约束或设计意图：`lrint`。
- **L990 EN**: Blank line separating nearby declarations or logic blocks.
  **L990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L991 EN**: Continues the surrounding expression or declaration: `static long`.
  **L991 CN**: 继续构造周围的表达式或声明：`static long`。
- **L992 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L992 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L993 EN**: Continues logic associated with callable symbol `__tg_lrint`.
  **L993 CN**: 继续与可调用符号 `__tg_lrint` 相关的逻辑。
- **L994 EN**: Blank line separating nearby declarations or logic blocks.
  **L994 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L995 EN**: Continues the surrounding expression or declaration: `static long`.
  **L995 CN**: 继续构造周围的表达式或声明：`static long`。
- **L996 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L996 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L997 EN**: Continues logic associated with callable symbol `__tg_lrint`.
  **L997 CN**: 继续与可调用符号 `__tg_lrint` 相关的逻辑。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L999 EN**: Continues the surrounding expression or declaration: `static long`.
  **L999 CN**: 继续构造周围的表达式或声明：`static long`。
- **L1000 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1000 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1001 EN**: Continues logic associated with callable symbol `__tg_lrint`.
  **L1001 CN**: 继续与可调用符号 `__tg_lrint` 相关的逻辑。
- **L1002 EN**: Blank line separating nearby declarations or logic blocks.
  **L1002 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1003 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef lrint`.
  **L1003 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef lrint`。
- **L1004 EN**: Defines macro `lrint(__x)` for conditional compilation, shorthand, or API generation.
  **L1004 CN**: 定义宏 `lrint(__x)`，用于条件编译、简写或 API 生成。
- **L1005 EN**: Blank line separating nearby declarations or logic blocks.
  **L1005 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1006 EN**: Comment explains nearby logic, constraints, or intent: `lround`.
  **L1006 CN**: 注释解释附近代码的逻辑、约束或设计意图：`lround`。
- **L1007 EN**: Blank line separating nearby declarations or logic blocks.
  **L1007 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1008 EN**: Continues the surrounding expression or declaration: `static long`.
  **L1008 CN**: 继续构造周围的表达式或声明：`static long`。

### Lines 1009-1032

````c
    _TG_ATTRS
    __tg_lround(float __x) {return lroundf(__x);}

static long
    _TG_ATTRS
    __tg_lround(double __x) {return lround(__x);}

static long
    _TG_ATTRS
    __tg_lround(long double __x) {return lroundl(__x);}

#undef lround
#define lround(__x) __tg_lround(__tg_promote1((__x))(__x))

// nearbyint

static float
    _TG_ATTRS
    __tg_nearbyint(float __x) {return nearbyintf(__x);}

static double
    _TG_ATTRS
    __tg_nearbyint(double __x) {return nearbyint(__x);}

````
- **L1009 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1009 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1010 EN**: Continues logic associated with callable symbol `__tg_lround`.
  **L1010 CN**: 继续与可调用符号 `__tg_lround` 相关的逻辑。
- **L1011 EN**: Blank line separating nearby declarations or logic blocks.
  **L1011 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1012 EN**: Continues the surrounding expression or declaration: `static long`.
  **L1012 CN**: 继续构造周围的表达式或声明：`static long`。
- **L1013 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1013 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1014 EN**: Continues logic associated with callable symbol `__tg_lround`.
  **L1014 CN**: 继续与可调用符号 `__tg_lround` 相关的逻辑。
- **L1015 EN**: Blank line separating nearby declarations or logic blocks.
  **L1015 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1016 EN**: Continues the surrounding expression or declaration: `static long`.
  **L1016 CN**: 继续构造周围的表达式或声明：`static long`。
- **L1017 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1017 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1018 EN**: Continues logic associated with callable symbol `__tg_lround`.
  **L1018 CN**: 继续与可调用符号 `__tg_lround` 相关的逻辑。
- **L1019 EN**: Blank line separating nearby declarations or logic blocks.
  **L1019 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1020 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef lround`.
  **L1020 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef lround`。
- **L1021 EN**: Defines macro `lround(__x)` for conditional compilation, shorthand, or API generation.
  **L1021 CN**: 定义宏 `lround(__x)`，用于条件编译、简写或 API 生成。
- **L1022 EN**: Blank line separating nearby declarations or logic blocks.
  **L1022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1023 EN**: Comment explains nearby logic, constraints, or intent: `nearbyint`.
  **L1023 CN**: 注释解释附近代码的逻辑、约束或设计意图：`nearbyint`。
- **L1024 EN**: Blank line separating nearby declarations or logic blocks.
  **L1024 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1025 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1025 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1026 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1026 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1027 EN**: Continues logic associated with callable symbol `__tg_nearbyint`.
  **L1027 CN**: 继续与可调用符号 `__tg_nearbyint` 相关的逻辑。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1029 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1029 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1030 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1030 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1031 EN**: Continues logic associated with callable symbol `__tg_nearbyint`.
  **L1031 CN**: 继续与可调用符号 `__tg_nearbyint` 相关的逻辑。
- **L1032 EN**: Blank line separating nearby declarations or logic blocks.
  **L1032 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1033-1056

````c
static long double
    _TG_ATTRS
    __tg_nearbyint(long double __x) {return nearbyintl(__x);}

#undef nearbyint
#define nearbyint(__x) __tg_nearbyint(__tg_promote1((__x))(__x))

// nextafter

static float
    _TG_ATTRS
    __tg_nextafter(float __x, float __y) {return nextafterf(__x, __y);}

static double
    _TG_ATTRS
    __tg_nextafter(double __x, double __y) {return nextafter(__x, __y);}

static long double
    _TG_ATTRS
    __tg_nextafter(long double __x, long double __y) {return nextafterl(__x, __y);}

#undef nextafter
#define nextafter(__x, __y) __tg_nextafter(__tg_promote2((__x), (__y))(__x), \
                                           __tg_promote2((__x), (__y))(__y))
````
- **L1033 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1033 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1034 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1034 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1035 EN**: Continues logic associated with callable symbol `__tg_nearbyint`.
  **L1035 CN**: 继续与可调用符号 `__tg_nearbyint` 相关的逻辑。
- **L1036 EN**: Blank line separating nearby declarations or logic blocks.
  **L1036 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1037 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef nearbyint`.
  **L1037 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef nearbyint`。
- **L1038 EN**: Defines macro `nearbyint(__x)` for conditional compilation, shorthand, or API generation.
  **L1038 CN**: 定义宏 `nearbyint(__x)`，用于条件编译、简写或 API 生成。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1040 EN**: Comment explains nearby logic, constraints, or intent: `nextafter`.
  **L1040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`nextafter`。
- **L1041 EN**: Blank line separating nearby declarations or logic blocks.
  **L1041 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1042 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1042 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1043 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1043 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1044 EN**: Continues logic associated with callable symbol `__tg_nextafter`.
  **L1044 CN**: 继续与可调用符号 `__tg_nextafter` 相关的逻辑。
- **L1045 EN**: Blank line separating nearby declarations or logic blocks.
  **L1045 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1046 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1046 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1047 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1047 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1048 EN**: Continues logic associated with callable symbol `__tg_nextafter`.
  **L1048 CN**: 继续与可调用符号 `__tg_nextafter` 相关的逻辑。
- **L1049 EN**: Blank line separating nearby declarations or logic blocks.
  **L1049 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1050 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1050 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1051 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1051 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1052 EN**: Continues logic associated with callable symbol `__tg_nextafter`.
  **L1052 CN**: 继续与可调用符号 `__tg_nextafter` 相关的逻辑。
- **L1053 EN**: Blank line separating nearby declarations or logic blocks.
  **L1053 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1054 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef nextafter`.
  **L1054 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef nextafter`。
- **L1055 EN**: Defines macro `nextafter(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L1055 CN**: 定义宏 `nextafter(__x, __y)`，用于条件编译、简写或 API 生成。
- **L1056 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L1056 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。

### Lines 1057-1080

````c

// nexttoward

static float
    _TG_ATTRS
    __tg_nexttoward(float __x, long double __y) {return nexttowardf(__x, __y);}

static double
    _TG_ATTRS
    __tg_nexttoward(double __x, long double __y) {return nexttoward(__x, __y);}

static long double
    _TG_ATTRS
    __tg_nexttoward(long double __x, long double __y) {return nexttowardl(__x, __y);}

#undef nexttoward
#define nexttoward(__x, __y) __tg_nexttoward(__tg_promote1((__x))(__x), (__y))

// remainder

static float
    _TG_ATTRS
    __tg_remainder(float __x, float __y) {return remainderf(__x, __y);}

````
- **L1057 EN**: Blank line separating nearby declarations or logic blocks.
  **L1057 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1058 EN**: Comment explains nearby logic, constraints, or intent: `nexttoward`.
  **L1058 CN**: 注释解释附近代码的逻辑、约束或设计意图：`nexttoward`。
- **L1059 EN**: Blank line separating nearby declarations or logic blocks.
  **L1059 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1060 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1060 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1061 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1061 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1062 EN**: Continues logic associated with callable symbol `__tg_nexttoward`.
  **L1062 CN**: 继续与可调用符号 `__tg_nexttoward` 相关的逻辑。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1064 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1064 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1065 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1065 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1066 EN**: Continues logic associated with callable symbol `__tg_nexttoward`.
  **L1066 CN**: 继续与可调用符号 `__tg_nexttoward` 相关的逻辑。
- **L1067 EN**: Blank line separating nearby declarations or logic blocks.
  **L1067 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1068 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1068 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1069 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1069 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1070 EN**: Continues logic associated with callable symbol `__tg_nexttoward`.
  **L1070 CN**: 继续与可调用符号 `__tg_nexttoward` 相关的逻辑。
- **L1071 EN**: Blank line separating nearby declarations or logic blocks.
  **L1071 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1072 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef nexttoward`.
  **L1072 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef nexttoward`。
- **L1073 EN**: Defines macro `nexttoward(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L1073 CN**: 定义宏 `nexttoward(__x, __y)`，用于条件编译、简写或 API 生成。
- **L1074 EN**: Blank line separating nearby declarations or logic blocks.
  **L1074 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1075 EN**: Comment explains nearby logic, constraints, or intent: `remainder`.
  **L1075 CN**: 注释解释附近代码的逻辑、约束或设计意图：`remainder`。
- **L1076 EN**: Blank line separating nearby declarations or logic blocks.
  **L1076 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1077 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1077 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1078 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1078 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1079 EN**: Continues logic associated with callable symbol `__tg_remainder`.
  **L1079 CN**: 继续与可调用符号 `__tg_remainder` 相关的逻辑。
- **L1080 EN**: Blank line separating nearby declarations or logic blocks.
  **L1080 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1081-1104

````c
static double
    _TG_ATTRS
    __tg_remainder(double __x, double __y) {return remainder(__x, __y);}

static long double
    _TG_ATTRS
    __tg_remainder(long double __x, long double __y) {return remainderl(__x, __y);}

#undef remainder
#define remainder(__x, __y) __tg_remainder(__tg_promote2((__x), (__y))(__x), \
                                           __tg_promote2((__x), (__y))(__y))

// remquo

static float
    _TG_ATTRS
    __tg_remquo(float __x, float __y, int* __z)
    {return remquof(__x, __y, __z);}

static double
    _TG_ATTRS
    __tg_remquo(double __x, double __y, int* __z)
    {return remquo(__x, __y, __z);}

````
- **L1081 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1081 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1082 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1082 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1083 EN**: Continues logic associated with callable symbol `__tg_remainder`.
  **L1083 CN**: 继续与可调用符号 `__tg_remainder` 相关的逻辑。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1085 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1085 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1086 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1086 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1087 EN**: Continues logic associated with callable symbol `__tg_remainder`.
  **L1087 CN**: 继续与可调用符号 `__tg_remainder` 相关的逻辑。
- **L1088 EN**: Blank line separating nearby declarations or logic blocks.
  **L1088 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1089 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef remainder`.
  **L1089 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef remainder`。
- **L1090 EN**: Defines macro `remainder(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L1090 CN**: 定义宏 `remainder(__x, __y)`，用于条件编译、简写或 API 生成。
- **L1091 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L1091 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L1092 EN**: Blank line separating nearby declarations or logic blocks.
  **L1092 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1093 EN**: Comment explains nearby logic, constraints, or intent: `remquo`.
  **L1093 CN**: 注释解释附近代码的逻辑、约束或设计意图：`remquo`。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1095 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1095 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1096 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1096 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1097 EN**: Continues logic associated with callable symbol `__tg_remquo`.
  **L1097 CN**: 继续与可调用符号 `__tg_remquo` 相关的逻辑。
- **L1098 EN**: Continues logic associated with callable symbol `remquof`.
  **L1098 CN**: 继续与可调用符号 `remquof` 相关的逻辑。
- **L1099 EN**: Blank line separating nearby declarations or logic blocks.
  **L1099 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1100 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1100 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1101 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1101 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1102 EN**: Continues logic associated with callable symbol `__tg_remquo`.
  **L1102 CN**: 继续与可调用符号 `__tg_remquo` 相关的逻辑。
- **L1103 EN**: Continues logic associated with callable symbol `remquo`.
  **L1103 CN**: 继续与可调用符号 `remquo` 相关的逻辑。
- **L1104 EN**: Blank line separating nearby declarations or logic blocks.
  **L1104 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1105-1128

````c
static long double
    _TG_ATTRS
    __tg_remquo(long double __x,long double __y, int* __z)
    {return remquol(__x, __y, __z);}

#undef remquo
#define remquo(__x, __y, __z)                         \
        __tg_remquo(__tg_promote2((__x), (__y))(__x), \
                    __tg_promote2((__x), (__y))(__y), \
                    (__z))

// rint

static float
    _TG_ATTRS
    __tg_rint(float __x) {return rintf(__x);}

static double
    _TG_ATTRS
    __tg_rint(double __x) {return rint(__x);}

static long double
    _TG_ATTRS
    __tg_rint(long double __x) {return rintl(__x);}
````
- **L1105 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1105 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1106 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1106 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1107 EN**: Continues logic associated with callable symbol `__tg_remquo`.
  **L1107 CN**: 继续与可调用符号 `__tg_remquo` 相关的逻辑。
- **L1108 EN**: Continues logic associated with callable symbol `remquol`.
  **L1108 CN**: 继续与可调用符号 `remquol` 相关的逻辑。
- **L1109 EN**: Blank line separating nearby declarations or logic blocks.
  **L1109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1110 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef remquo`.
  **L1110 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef remquo`。
- **L1111 EN**: Defines macro `remquo(__x, __y, __z)` for conditional compilation, shorthand, or API generation.
  **L1111 CN**: 定义宏 `remquo(__x, __y, __z)`，用于条件编译、简写或 API 生成。
- **L1112 EN**: Continues logic associated with callable symbol `__tg_remquo`.
  **L1112 CN**: 继续与可调用符号 `__tg_remquo` 相关的逻辑。
- **L1113 EN**: Continues logic associated with callable symbol `__tg_promote2`.
  **L1113 CN**: 继续与可调用符号 `__tg_promote2` 相关的逻辑。
- **L1114 EN**: Continues the surrounding expression or declaration: `(__z))`.
  **L1114 CN**: 继续构造周围的表达式或声明：`(__z))`。
- **L1115 EN**: Blank line separating nearby declarations or logic blocks.
  **L1115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1116 EN**: Comment explains nearby logic, constraints, or intent: `rint`.
  **L1116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`rint`。
- **L1117 EN**: Blank line separating nearby declarations or logic blocks.
  **L1117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1118 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1118 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1119 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1119 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1120 EN**: Continues logic associated with callable symbol `__tg_rint`.
  **L1120 CN**: 继续与可调用符号 `__tg_rint` 相关的逻辑。
- **L1121 EN**: Blank line separating nearby declarations or logic blocks.
  **L1121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1122 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1122 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1123 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1123 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1124 EN**: Continues logic associated with callable symbol `__tg_rint`.
  **L1124 CN**: 继续与可调用符号 `__tg_rint` 相关的逻辑。
- **L1125 EN**: Blank line separating nearby declarations or logic blocks.
  **L1125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1126 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1126 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1127 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1127 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1128 EN**: Continues logic associated with callable symbol `__tg_rint`.
  **L1128 CN**: 继续与可调用符号 `__tg_rint` 相关的逻辑。

### Lines 1129-1152

````c

#undef rint
#define rint(__x) __tg_rint(__tg_promote1((__x))(__x))

// round

static float
    _TG_ATTRS
    __tg_round(float __x) {return roundf(__x);}

static double
    _TG_ATTRS
    __tg_round(double __x) {return round(__x);}

static long double
    _TG_ATTRS
    __tg_round(long double __x) {return roundl(__x);}

#undef round
#define round(__x) __tg_round(__tg_promote1((__x))(__x))

// scalbn

static float
````
- **L1129 EN**: Blank line separating nearby declarations or logic blocks.
  **L1129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1130 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef rint`.
  **L1130 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef rint`。
- **L1131 EN**: Defines macro `rint(__x)` for conditional compilation, shorthand, or API generation.
  **L1131 CN**: 定义宏 `rint(__x)`，用于条件编译、简写或 API 生成。
- **L1132 EN**: Blank line separating nearby declarations or logic blocks.
  **L1132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1133 EN**: Comment explains nearby logic, constraints, or intent: `round`.
  **L1133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`round`。
- **L1134 EN**: Blank line separating nearby declarations or logic blocks.
  **L1134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1135 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1135 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1136 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1136 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1137 EN**: Continues logic associated with callable symbol `__tg_round`.
  **L1137 CN**: 继续与可调用符号 `__tg_round` 相关的逻辑。
- **L1138 EN**: Blank line separating nearby declarations or logic blocks.
  **L1138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1139 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1139 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1140 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1140 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1141 EN**: Continues logic associated with callable symbol `__tg_round`.
  **L1141 CN**: 继续与可调用符号 `__tg_round` 相关的逻辑。
- **L1142 EN**: Blank line separating nearby declarations or logic blocks.
  **L1142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1143 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1143 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1144 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1144 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1145 EN**: Continues logic associated with callable symbol `__tg_round`.
  **L1145 CN**: 继续与可调用符号 `__tg_round` 相关的逻辑。
- **L1146 EN**: Blank line separating nearby declarations or logic blocks.
  **L1146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1147 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef round`.
  **L1147 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef round`。
- **L1148 EN**: Defines macro `round(__x)` for conditional compilation, shorthand, or API generation.
  **L1148 CN**: 定义宏 `round(__x)`，用于条件编译、简写或 API 生成。
- **L1149 EN**: Blank line separating nearby declarations or logic blocks.
  **L1149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1150 EN**: Comment explains nearby logic, constraints, or intent: `scalbn`.
  **L1150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`scalbn`。
- **L1151 EN**: Blank line separating nearby declarations or logic blocks.
  **L1151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1152 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1152 CN**: 继续构造周围的表达式或声明：`static float`。

### Lines 1153-1176

````c
    _TG_ATTRS
    __tg_scalbn(float __x, int __y) {return scalbnf(__x, __y);}

static double
    _TG_ATTRS
    __tg_scalbn(double __x, int __y) {return scalbn(__x, __y);}

static long double
    _TG_ATTRS
    __tg_scalbn(long double __x, int __y) {return scalbnl(__x, __y);}

#undef scalbn
#define scalbn(__x, __y) __tg_scalbn(__tg_promote1((__x))(__x), __y)

// scalbln

static float
    _TG_ATTRS
    __tg_scalbln(float __x, long __y) {return scalblnf(__x, __y);}

static double
    _TG_ATTRS
    __tg_scalbln(double __x, long __y) {return scalbln(__x, __y);}

````
- **L1153 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1153 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1154 EN**: Continues logic associated with callable symbol `__tg_scalbn`.
  **L1154 CN**: 继续与可调用符号 `__tg_scalbn` 相关的逻辑。
- **L1155 EN**: Blank line separating nearby declarations or logic blocks.
  **L1155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1156 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1156 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1157 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1157 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1158 EN**: Continues logic associated with callable symbol `__tg_scalbn`.
  **L1158 CN**: 继续与可调用符号 `__tg_scalbn` 相关的逻辑。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1160 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1160 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1161 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1161 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1162 EN**: Continues logic associated with callable symbol `__tg_scalbn`.
  **L1162 CN**: 继续与可调用符号 `__tg_scalbn` 相关的逻辑。
- **L1163 EN**: Blank line separating nearby declarations or logic blocks.
  **L1163 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1164 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef scalbn`.
  **L1164 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef scalbn`。
- **L1165 EN**: Defines macro `scalbn(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L1165 CN**: 定义宏 `scalbn(__x, __y)`，用于条件编译、简写或 API 生成。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1167 EN**: Comment explains nearby logic, constraints, or intent: `scalbln`.
  **L1167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`scalbln`。
- **L1168 EN**: Blank line separating nearby declarations or logic blocks.
  **L1168 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1169 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1169 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1170 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1170 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1171 EN**: Continues logic associated with callable symbol `__tg_scalbln`.
  **L1171 CN**: 继续与可调用符号 `__tg_scalbln` 相关的逻辑。
- **L1172 EN**: Blank line separating nearby declarations or logic blocks.
  **L1172 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1173 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1173 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1174 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1174 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1175 EN**: Continues logic associated with callable symbol `__tg_scalbln`.
  **L1175 CN**: 继续与可调用符号 `__tg_scalbln` 相关的逻辑。
- **L1176 EN**: Blank line separating nearby declarations or logic blocks.
  **L1176 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1177-1200

````c
static long double
    _TG_ATTRS
    __tg_scalbln(long double __x, long __y) {return scalblnl(__x, __y);}

#undef scalbln
#define scalbln(__x, __y) __tg_scalbln(__tg_promote1((__x))(__x), __y)

// tgamma

static float
    _TG_ATTRS
    __tg_tgamma(float __x) {return tgammaf(__x);}

static double
    _TG_ATTRS
    __tg_tgamma(double __x) {return tgamma(__x);}

static long double
    _TG_ATTRS
    __tg_tgamma(long double __x) {return tgammal(__x);}

#undef tgamma
#define tgamma(__x) __tg_tgamma(__tg_promote1((__x))(__x))

````
- **L1177 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1177 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1178 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1178 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1179 EN**: Continues logic associated with callable symbol `__tg_scalbln`.
  **L1179 CN**: 继续与可调用符号 `__tg_scalbln` 相关的逻辑。
- **L1180 EN**: Blank line separating nearby declarations or logic blocks.
  **L1180 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1181 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef scalbln`.
  **L1181 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef scalbln`。
- **L1182 EN**: Defines macro `scalbln(__x, __y)` for conditional compilation, shorthand, or API generation.
  **L1182 CN**: 定义宏 `scalbln(__x, __y)`，用于条件编译、简写或 API 生成。
- **L1183 EN**: Blank line separating nearby declarations or logic blocks.
  **L1183 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1184 EN**: Comment explains nearby logic, constraints, or intent: `tgamma`.
  **L1184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tgamma`。
- **L1185 EN**: Blank line separating nearby declarations or logic blocks.
  **L1185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1186 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1186 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1187 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1187 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1188 EN**: Continues logic associated with callable symbol `__tg_tgamma`.
  **L1188 CN**: 继续与可调用符号 `__tg_tgamma` 相关的逻辑。
- **L1189 EN**: Blank line separating nearby declarations or logic blocks.
  **L1189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1190 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1190 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1191 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1191 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1192 EN**: Continues logic associated with callable symbol `__tg_tgamma`.
  **L1192 CN**: 继续与可调用符号 `__tg_tgamma` 相关的逻辑。
- **L1193 EN**: Blank line separating nearby declarations or logic blocks.
  **L1193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1194 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1194 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1195 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1195 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1196 EN**: Continues logic associated with callable symbol `__tg_tgamma`.
  **L1196 CN**: 继续与可调用符号 `__tg_tgamma` 相关的逻辑。
- **L1197 EN**: Blank line separating nearby declarations or logic blocks.
  **L1197 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1198 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef tgamma`.
  **L1198 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef tgamma`。
- **L1199 EN**: Defines macro `tgamma(__x)` for conditional compilation, shorthand, or API generation.
  **L1199 CN**: 定义宏 `tgamma(__x)`，用于条件编译、简写或 API 生成。
- **L1200 EN**: Blank line separating nearby declarations or logic blocks.
  **L1200 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1201-1224

````c
// trunc

static float
    _TG_ATTRS
    __tg_trunc(float __x) {return truncf(__x);}

static double
    _TG_ATTRS
    __tg_trunc(double __x) {return trunc(__x);}

static long double
    _TG_ATTRS
    __tg_trunc(long double __x) {return truncl(__x);}

#undef trunc
#define trunc(__x) __tg_trunc(__tg_promote1((__x))(__x))

// carg

static float
    _TG_ATTRS
    __tg_carg(float __x) {return atan2f(0.F, __x);}

static double
````
- **L1201 EN**: Comment explains nearby logic, constraints, or intent: `trunc`.
  **L1201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`trunc`。
- **L1202 EN**: Blank line separating nearby declarations or logic blocks.
  **L1202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1203 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1203 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1204 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1204 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1205 EN**: Continues logic associated with callable symbol `__tg_trunc`.
  **L1205 CN**: 继续与可调用符号 `__tg_trunc` 相关的逻辑。
- **L1206 EN**: Blank line separating nearby declarations or logic blocks.
  **L1206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1207 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1207 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1208 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1208 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1209 EN**: Continues logic associated with callable symbol `__tg_trunc`.
  **L1209 CN**: 继续与可调用符号 `__tg_trunc` 相关的逻辑。
- **L1210 EN**: Blank line separating nearby declarations or logic blocks.
  **L1210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1211 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1211 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1212 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1212 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1213 EN**: Continues logic associated with callable symbol `__tg_trunc`.
  **L1213 CN**: 继续与可调用符号 `__tg_trunc` 相关的逻辑。
- **L1214 EN**: Blank line separating nearby declarations or logic blocks.
  **L1214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1215 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef trunc`.
  **L1215 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef trunc`。
- **L1216 EN**: Defines macro `trunc(__x)` for conditional compilation, shorthand, or API generation.
  **L1216 CN**: 定义宏 `trunc(__x)`，用于条件编译、简写或 API 生成。
- **L1217 EN**: Blank line separating nearby declarations or logic blocks.
  **L1217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1218 EN**: Comment explains nearby logic, constraints, or intent: `carg`.
  **L1218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`carg`。
- **L1219 EN**: Blank line separating nearby declarations or logic blocks.
  **L1219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1220 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1220 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1221 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1221 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1222 EN**: Continues logic associated with callable symbol `__tg_carg`.
  **L1222 CN**: 继续与可调用符号 `__tg_carg` 相关的逻辑。
- **L1223 EN**: Blank line separating nearby declarations or logic blocks.
  **L1223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1224 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1224 CN**: 继续构造周围的表达式或声明：`static double`。

### Lines 1225-1248

````c
    _TG_ATTRS
    __tg_carg(double __x) {return atan2(0., __x);}

static long double
    _TG_ATTRS
    __tg_carg(long double __x) {return atan2l(0.L, __x);}

static float
    _TG_ATTRS
    __tg_carg(float _Complex __x) {return cargf(__x);}

static double
    _TG_ATTRS
    __tg_carg(double _Complex __x) {return carg(__x);}

static long double
    _TG_ATTRS
    __tg_carg(long double _Complex __x) {return cargl(__x);}

#undef carg
#define carg(__x) __tg_carg(__tg_promote1((__x))(__x))

// cimag

````
- **L1225 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1225 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1226 EN**: Continues logic associated with callable symbol `__tg_carg`.
  **L1226 CN**: 继续与可调用符号 `__tg_carg` 相关的逻辑。
- **L1227 EN**: Blank line separating nearby declarations or logic blocks.
  **L1227 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1228 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1228 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1229 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1229 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1230 EN**: Continues logic associated with callable symbol `__tg_carg`.
  **L1230 CN**: 继续与可调用符号 `__tg_carg` 相关的逻辑。
- **L1231 EN**: Blank line separating nearby declarations or logic blocks.
  **L1231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1232 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1232 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1233 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1233 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1234 EN**: Continues logic associated with callable symbol `__tg_carg`.
  **L1234 CN**: 继续与可调用符号 `__tg_carg` 相关的逻辑。
- **L1235 EN**: Blank line separating nearby declarations or logic blocks.
  **L1235 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1236 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1236 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1237 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1237 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1238 EN**: Continues logic associated with callable symbol `__tg_carg`.
  **L1238 CN**: 继续与可调用符号 `__tg_carg` 相关的逻辑。
- **L1239 EN**: Blank line separating nearby declarations or logic blocks.
  **L1239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1240 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1240 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1241 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1241 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1242 EN**: Continues logic associated with callable symbol `__tg_carg`.
  **L1242 CN**: 继续与可调用符号 `__tg_carg` 相关的逻辑。
- **L1243 EN**: Blank line separating nearby declarations or logic blocks.
  **L1243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1244 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef carg`.
  **L1244 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef carg`。
- **L1245 EN**: Defines macro `carg(__x)` for conditional compilation, shorthand, or API generation.
  **L1245 CN**: 定义宏 `carg(__x)`，用于条件编译、简写或 API 生成。
- **L1246 EN**: Blank line separating nearby declarations or logic blocks.
  **L1246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1247 EN**: Comment explains nearby logic, constraints, or intent: `cimag`.
  **L1247 CN**: 注释解释附近代码的逻辑、约束或设计意图：`cimag`。
- **L1248 EN**: Blank line separating nearby declarations or logic blocks.
  **L1248 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1249-1272

````c
static float
    _TG_ATTRS
    __tg_cimag(float __x) {return 0;}

static double
    _TG_ATTRS
    __tg_cimag(double __x) {return 0;}

static long double
    _TG_ATTRS
    __tg_cimag(long double __x) {return 0;}

static float
    _TG_ATTRS
    __tg_cimag(float _Complex __x) {return cimagf(__x);}

static double
    _TG_ATTRS
    __tg_cimag(double _Complex __x) {return cimag(__x);}

static long double
    _TG_ATTRS
    __tg_cimag(long double _Complex __x) {return cimagl(__x);}

````
- **L1249 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1249 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1250 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1250 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1251 EN**: Continues logic associated with callable symbol `__tg_cimag`.
  **L1251 CN**: 继续与可调用符号 `__tg_cimag` 相关的逻辑。
- **L1252 EN**: Blank line separating nearby declarations or logic blocks.
  **L1252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1253 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1253 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1254 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1254 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1255 EN**: Continues logic associated with callable symbol `__tg_cimag`.
  **L1255 CN**: 继续与可调用符号 `__tg_cimag` 相关的逻辑。
- **L1256 EN**: Blank line separating nearby declarations or logic blocks.
  **L1256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1257 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1257 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1258 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1258 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1259 EN**: Continues logic associated with callable symbol `__tg_cimag`.
  **L1259 CN**: 继续与可调用符号 `__tg_cimag` 相关的逻辑。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1261 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1261 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1262 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1262 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1263 EN**: Continues logic associated with callable symbol `__tg_cimag`.
  **L1263 CN**: 继续与可调用符号 `__tg_cimag` 相关的逻辑。
- **L1264 EN**: Blank line separating nearby declarations or logic blocks.
  **L1264 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1265 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1265 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1266 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1266 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1267 EN**: Continues logic associated with callable symbol `__tg_cimag`.
  **L1267 CN**: 继续与可调用符号 `__tg_cimag` 相关的逻辑。
- **L1268 EN**: Blank line separating nearby declarations or logic blocks.
  **L1268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1269 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1269 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1270 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1270 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1271 EN**: Continues logic associated with callable symbol `__tg_cimag`.
  **L1271 CN**: 继续与可调用符号 `__tg_cimag` 相关的逻辑。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1273-1296

````c
#undef cimag
#define cimag(__x) __tg_cimag(__tg_promote1((__x))(__x))

// conj

static float _Complex
    _TG_ATTRS
    __tg_conj(float __x) {return __x;}

static double _Complex
    _TG_ATTRS
    __tg_conj(double __x) {return __x;}

static long double _Complex
    _TG_ATTRS
    __tg_conj(long double __x) {return __x;}

static float _Complex
    _TG_ATTRS
    __tg_conj(float _Complex __x) {return conjf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_conj(double _Complex __x) {return conj(__x);}
````
- **L1273 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef cimag`.
  **L1273 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef cimag`。
- **L1274 EN**: Defines macro `cimag(__x)` for conditional compilation, shorthand, or API generation.
  **L1274 CN**: 定义宏 `cimag(__x)`，用于条件编译、简写或 API 生成。
- **L1275 EN**: Blank line separating nearby declarations or logic blocks.
  **L1275 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1276 EN**: Comment explains nearby logic, constraints, or intent: `conj`.
  **L1276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`conj`。
- **L1277 EN**: Blank line separating nearby declarations or logic blocks.
  **L1277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1278 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L1278 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L1279 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1279 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1280 EN**: Continues logic associated with callable symbol `__tg_conj`.
  **L1280 CN**: 继续与可调用符号 `__tg_conj` 相关的逻辑。
- **L1281 EN**: Blank line separating nearby declarations or logic blocks.
  **L1281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1282 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L1282 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L1283 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1283 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1284 EN**: Continues logic associated with callable symbol `__tg_conj`.
  **L1284 CN**: 继续与可调用符号 `__tg_conj` 相关的逻辑。
- **L1285 EN**: Blank line separating nearby declarations or logic blocks.
  **L1285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1286 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L1286 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L1287 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1287 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1288 EN**: Continues logic associated with callable symbol `__tg_conj`.
  **L1288 CN**: 继续与可调用符号 `__tg_conj` 相关的逻辑。
- **L1289 EN**: Blank line separating nearby declarations or logic blocks.
  **L1289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1290 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L1290 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L1291 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1291 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1292 EN**: Continues logic associated with callable symbol `__tg_conj`.
  **L1292 CN**: 继续与可调用符号 `__tg_conj` 相关的逻辑。
- **L1293 EN**: Blank line separating nearby declarations or logic blocks.
  **L1293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1294 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L1294 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L1295 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1295 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1296 EN**: Continues logic associated with callable symbol `__tg_conj`.
  **L1296 CN**: 继续与可调用符号 `__tg_conj` 相关的逻辑。

### Lines 1297-1320

````c

static long double _Complex
    _TG_ATTRS
    __tg_conj(long double _Complex __x) {return conjl(__x);}

#undef conj
#define conj(__x) __tg_conj(__tg_promote1((__x))(__x))

// cproj

static float _Complex
    _TG_ATTRS
    __tg_cproj(float __x) {return cprojf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_cproj(double __x) {return cproj(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_cproj(long double __x) {return cprojl(__x);}

static float _Complex
    _TG_ATTRS
````
- **L1297 EN**: Blank line separating nearby declarations or logic blocks.
  **L1297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1298 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L1298 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L1299 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1299 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1300 EN**: Continues logic associated with callable symbol `__tg_conj`.
  **L1300 CN**: 继续与可调用符号 `__tg_conj` 相关的逻辑。
- **L1301 EN**: Blank line separating nearby declarations or logic blocks.
  **L1301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1302 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef conj`.
  **L1302 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef conj`。
- **L1303 EN**: Defines macro `conj(__x)` for conditional compilation, shorthand, or API generation.
  **L1303 CN**: 定义宏 `conj(__x)`，用于条件编译、简写或 API 生成。
- **L1304 EN**: Blank line separating nearby declarations or logic blocks.
  **L1304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1305 EN**: Comment explains nearby logic, constraints, or intent: `cproj`.
  **L1305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`cproj`。
- **L1306 EN**: Blank line separating nearby declarations or logic blocks.
  **L1306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1307 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L1307 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L1308 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1308 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1309 EN**: Continues logic associated with callable symbol `__tg_cproj`.
  **L1309 CN**: 继续与可调用符号 `__tg_cproj` 相关的逻辑。
- **L1310 EN**: Blank line separating nearby declarations or logic blocks.
  **L1310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1311 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L1311 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L1312 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1312 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1313 EN**: Continues logic associated with callable symbol `__tg_cproj`.
  **L1313 CN**: 继续与可调用符号 `__tg_cproj` 相关的逻辑。
- **L1314 EN**: Blank line separating nearby declarations or logic blocks.
  **L1314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1315 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L1315 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L1316 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1316 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1317 EN**: Continues logic associated with callable symbol `__tg_cproj`.
  **L1317 CN**: 继续与可调用符号 `__tg_cproj` 相关的逻辑。
- **L1318 EN**: Blank line separating nearby declarations or logic blocks.
  **L1318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1319 EN**: Continues the surrounding expression or declaration: `static float _Complex`.
  **L1319 CN**: 继续构造周围的表达式或声明：`static float _Complex`。
- **L1320 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1320 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。

### Lines 1321-1344

````c
    __tg_cproj(float _Complex __x) {return cprojf(__x);}

static double _Complex
    _TG_ATTRS
    __tg_cproj(double _Complex __x) {return cproj(__x);}

static long double _Complex
    _TG_ATTRS
    __tg_cproj(long double _Complex __x) {return cprojl(__x);}

#undef cproj
#define cproj(__x) __tg_cproj(__tg_promote1((__x))(__x))

// creal

static float
    _TG_ATTRS
    __tg_creal(float __x) {return __x;}

static double
    _TG_ATTRS
    __tg_creal(double __x) {return __x;}

static long double
````
- **L1321 EN**: Continues logic associated with callable symbol `__tg_cproj`.
  **L1321 CN**: 继续与可调用符号 `__tg_cproj` 相关的逻辑。
- **L1322 EN**: Blank line separating nearby declarations or logic blocks.
  **L1322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1323 EN**: Continues the surrounding expression or declaration: `static double _Complex`.
  **L1323 CN**: 继续构造周围的表达式或声明：`static double _Complex`。
- **L1324 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1324 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1325 EN**: Continues logic associated with callable symbol `__tg_cproj`.
  **L1325 CN**: 继续与可调用符号 `__tg_cproj` 相关的逻辑。
- **L1326 EN**: Blank line separating nearby declarations or logic blocks.
  **L1326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1327 EN**: Continues the surrounding expression or declaration: `static long double _Complex`.
  **L1327 CN**: 继续构造周围的表达式或声明：`static long double _Complex`。
- **L1328 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1328 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1329 EN**: Continues logic associated with callable symbol `__tg_cproj`.
  **L1329 CN**: 继续与可调用符号 `__tg_cproj` 相关的逻辑。
- **L1330 EN**: Blank line separating nearby declarations or logic blocks.
  **L1330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1331 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef cproj`.
  **L1331 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef cproj`。
- **L1332 EN**: Defines macro `cproj(__x)` for conditional compilation, shorthand, or API generation.
  **L1332 CN**: 定义宏 `cproj(__x)`，用于条件编译、简写或 API 生成。
- **L1333 EN**: Blank line separating nearby declarations or logic blocks.
  **L1333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1334 EN**: Comment explains nearby logic, constraints, or intent: `creal`.
  **L1334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`creal`。
- **L1335 EN**: Blank line separating nearby declarations or logic blocks.
  **L1335 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1336 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1336 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1337 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1337 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1338 EN**: Continues logic associated with callable symbol `__tg_creal`.
  **L1338 CN**: 继续与可调用符号 `__tg_creal` 相关的逻辑。
- **L1339 EN**: Blank line separating nearby declarations or logic blocks.
  **L1339 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1340 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1340 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1341 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1341 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1342 EN**: Continues logic associated with callable symbol `__tg_creal`.
  **L1342 CN**: 继续与可调用符号 `__tg_creal` 相关的逻辑。
- **L1343 EN**: Blank line separating nearby declarations or logic blocks.
  **L1343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1344 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1344 CN**: 继续构造周围的表达式或声明：`static long double`。

### Lines 1345-1368

````c
    _TG_ATTRS
    __tg_creal(long double __x) {return __x;}

static float
    _TG_ATTRS
    __tg_creal(float _Complex __x) {return crealf(__x);}

static double
    _TG_ATTRS
    __tg_creal(double _Complex __x) {return creal(__x);}

static long double
    _TG_ATTRS
    __tg_creal(long double _Complex __x) {return creall(__x);}

#undef creal
#define creal(__x) __tg_creal(__tg_promote1((__x))(__x))

#undef _TG_ATTRSp
#undef _TG_ATTRS

#endif /* __cplusplus */
#endif /* __has_include_next */
#endif /* __CLANG_TGMATH_H */
````
- **L1345 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1345 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1346 EN**: Continues logic associated with callable symbol `__tg_creal`.
  **L1346 CN**: 继续与可调用符号 `__tg_creal` 相关的逻辑。
- **L1347 EN**: Blank line separating nearby declarations or logic blocks.
  **L1347 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1348 EN**: Continues the surrounding expression or declaration: `static float`.
  **L1348 CN**: 继续构造周围的表达式或声明：`static float`。
- **L1349 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1349 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1350 EN**: Continues logic associated with callable symbol `__tg_creal`.
  **L1350 CN**: 继续与可调用符号 `__tg_creal` 相关的逻辑。
- **L1351 EN**: Blank line separating nearby declarations or logic blocks.
  **L1351 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1352 EN**: Continues the surrounding expression or declaration: `static double`.
  **L1352 CN**: 继续构造周围的表达式或声明：`static double`。
- **L1353 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1353 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1354 EN**: Continues logic associated with callable symbol `__tg_creal`.
  **L1354 CN**: 继续与可调用符号 `__tg_creal` 相关的逻辑。
- **L1355 EN**: Blank line separating nearby declarations or logic blocks.
  **L1355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1356 EN**: Continues the surrounding expression or declaration: `static long double`.
  **L1356 CN**: 继续构造周围的表达式或声明：`static long double`。
- **L1357 EN**: Continues the surrounding expression or declaration: `_TG_ATTRS`.
  **L1357 CN**: 继续构造周围的表达式或声明：`_TG_ATTRS`。
- **L1358 EN**: Continues logic associated with callable symbol `__tg_creal`.
  **L1358 CN**: 继续与可调用符号 `__tg_creal` 相关的逻辑。
- **L1359 EN**: Blank line separating nearby declarations or logic blocks.
  **L1359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1360 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef creal`.
  **L1360 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef creal`。
- **L1361 EN**: Defines macro `creal(__x)` for conditional compilation, shorthand, or API generation.
  **L1361 CN**: 定义宏 `creal(__x)`，用于条件编译、简写或 API 生成。
- **L1362 EN**: Blank line separating nearby declarations or logic blocks.
  **L1362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1363 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef _TG_ATTRSp`.
  **L1363 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef _TG_ATTRSp`。
- **L1364 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef _TG_ATTRS`.
  **L1364 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef _TG_ATTRS`。
- **L1365 EN**: Blank line separating nearby declarations or logic blocks.
  **L1365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1366 EN**: Closes the current preprocessor conditional block.
  **L1366 CN**: 结束当前预处理条件块。
- **L1367 EN**: Closes the current preprocessor conditional block.
  **L1367 CN**: 结束当前预处理条件块。
- **L1368 EN**: Closes the current preprocessor conditional block.
  **L1368 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **C/C++ compatibility headers / C/C++ 兼容头文件**
- **Header composition through includes / 通过 include 组合头文件**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `math.h`: Provides related header declarations. / 提供相关头文件声明。
  - `tgmath.h`: Provides related header declarations. / 提供相关头文件声明。
  - `complex.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `__CLANG_TGMATH_H`, `__APPLE__`, `__cplusplus`
- **External builtins / 外部 builtin**: No direct builtin forwarding detected. / 未检测到直接的 builtin 转发。
