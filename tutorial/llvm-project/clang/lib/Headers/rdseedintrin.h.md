# rdseedintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/rdseedintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: RDSEED intrinsics.
- **Purpose (CN)**: 提供 RDSEED intrinsic 接口。
- **Line Count / 行数**: 105

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````c
/*===---- rdseedintrin.h - RDSEED intrinsics -------------------------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __IMMINTRIN_H
#error "Never use <rdseedintrin.h> directly; include <immintrin.h> instead."
#endif

#ifndef __RDSEEDINTRIN_H
#define __RDSEEDINTRIN_H

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
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <rdseedintrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error "Never use <rdseedintrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifndef __RDSEEDINTRIN_H`.
  **L14 CN**: 开始一个预处理条件块：`#ifndef __RDSEEDINTRIN_H`。
- **L15 EN**: Defines macro `__RDSEEDINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `__RDSEEDINTRIN_H`，用于条件编译、简写或 API 生成。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 17-32

````c
/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS __attribute__((__always_inline__, __nodebug__, __target__("rdseed")))

/// Stores a hardware-generated 16-bit random value in the memory at \a __p.
///
///    The random number generator complies with NIST SP800-90B and SP800-90C.
///
/// \code{.operation}
/// IF HW_NRND_GEN.ready == 1
///   Store16(__p, HW_NRND_GEN.data)
///   result := 1
/// ELSE
///   Store16(__p, 0)
///   result := 0
/// END
/// \endcode
````
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L18 EN**: Defines macro `__DEFAULT_FN_ATTRS` for conditional compilation, shorthand, or API generation.
  **L18 CN**: 定义宏 `__DEFAULT_FN_ATTRS`，用于条件编译、简写或 API 生成。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L20 EN**: Comment explains nearby logic, constraints, or intent: `Stores a hardware-generated 16-bit random value in the memory at a __p.`.
  **L20 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores a hardware-generated 16-bit random value in the memory at a __p.`。
- **L21 EN**: Separator comment used for visual grouping.
  **L21 CN**: 用于视觉分组的分隔注释。
- **L22 EN**: Comment explains nearby logic, constraints, or intent: `The random number generator complies with NIST SP800-90B and SP800-90C.`.
  **L22 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The random number generator complies with NIST SP800-90B and SP800-90C.`。
- **L23 EN**: Separator comment used for visual grouping.
  **L23 CN**: 用于视觉分组的分隔注释。
- **L24 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L24 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L25 EN**: Comment explains nearby logic, constraints, or intent: `IF HW_NRND_GEN.ready 1`.
  **L25 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF HW_NRND_GEN.ready 1`。
- **L26 EN**: Comment explains nearby logic, constraints, or intent: `Store16(__p, HW_NRND_GEN.data)`.
  **L26 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store16(__p, HW_NRND_GEN.data)`。
- **L27 EN**: Comment explains nearby logic, constraints, or intent: `result : 1`.
  **L27 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result : 1`。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L29 EN**: Comment explains nearby logic, constraints, or intent: `Store16(__p, 0)`.
  **L29 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store16(__p, 0)`。
- **L30 EN**: Comment explains nearby logic, constraints, or intent: `result : 0`.
  **L30 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result : 0`。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `END`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`END`。
- **L32 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L32 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。

### Lines 33-48

````c
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c RDSEED instruction.
///
/// \param __p
///    Pointer to memory for storing the 16-bit random number.
/// \returns 1 if a random number was generated, 0 if not.
static __inline__ int __DEFAULT_FN_ATTRS
_rdseed16_step(unsigned short *__p)
{
  return (int) __builtin_ia32_rdseed16_step(__p);
}

/// Stores a hardware-generated 32-bit random value in the memory at \a __p.
///
````
- **L33 EN**: Separator comment used for visual grouping.
  **L33 CN**: 用于视觉分组的分隔注释。
- **L34 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L34 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L35 EN**: Separator comment used for visual grouping.
  **L35 CN**: 用于视觉分组的分隔注释。
- **L36 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c RDSEED instruction.`.
  **L36 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c RDSEED instruction.`。
- **L37 EN**: Separator comment used for visual grouping.
  **L37 CN**: 用于视觉分组的分隔注释。
- **L38 EN**: Comment explains nearby logic, constraints, or intent: `param __p`.
  **L38 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __p`。
- **L39 EN**: Comment explains nearby logic, constraints, or intent: `Pointer to memory for storing the 16-bit random number.`.
  **L39 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pointer to memory for storing the 16-bit random number.`。
- **L40 EN**: Comment explains nearby logic, constraints, or intent: `returns 1 if a random number was generated, 0 if not.`.
  **L40 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns 1 if a random number was generated, 0 if not.`。
- **L41 EN**: Continues the surrounding expression or declaration: `static __inline__ int __DEFAULT_FN_ATTRS`.
  **L41 CN**: 继续构造周围的表达式或声明：`static __inline__ int __DEFAULT_FN_ATTRS`。
- **L42 EN**: Continues logic associated with callable symbol `_rdseed16_step`.
  **L42 CN**: 继续与可调用符号 `_rdseed16_step` 相关的逻辑。
- **L43 EN**: Opens a new lexical scope or compound statement.
  **L43 CN**: 打开一个新的词法作用域或复合语句块。
- **L44 EN**: Returns from the current function with `(int) __builtin_ia32_rdseed16_step(__p)`.
  **L44 CN**: 以 `(int) __builtin_ia32_rdseed16_step(__p)` 从当前函数返回。
- **L45 EN**: Closes the current lexical scope or compound statement.
  **L45 CN**: 结束当前词法作用域或复合语句块。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Comment explains nearby logic, constraints, or intent: `Stores a hardware-generated 32-bit random value in the memory at a __p.`.
  **L47 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores a hardware-generated 32-bit random value in the memory at a __p.`。
- **L48 EN**: Separator comment used for visual grouping.
  **L48 CN**: 用于视觉分组的分隔注释。

### Lines 49-64

````c
///    The random number generator complies with NIST SP800-90B and SP800-90C.
///
/// \code{.operation}
/// IF HW_NRND_GEN.ready == 1
///   Store32(__p, HW_NRND_GEN.data)
///   result := 1
/// ELSE
///   Store32(__p, 0)
///   result := 0
/// END
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c RDSEED instruction.
///
````
- **L49 EN**: Comment explains nearby logic, constraints, or intent: `The random number generator complies with NIST SP800-90B and SP800-90C.`.
  **L49 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The random number generator complies with NIST SP800-90B and SP800-90C.`。
- **L50 EN**: Separator comment used for visual grouping.
  **L50 CN**: 用于视觉分组的分隔注释。
- **L51 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L51 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L52 EN**: Comment explains nearby logic, constraints, or intent: `IF HW_NRND_GEN.ready 1`.
  **L52 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF HW_NRND_GEN.ready 1`。
- **L53 EN**: Comment explains nearby logic, constraints, or intent: `Store32(__p, HW_NRND_GEN.data)`.
  **L53 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store32(__p, HW_NRND_GEN.data)`。
- **L54 EN**: Comment explains nearby logic, constraints, or intent: `result : 1`.
  **L54 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result : 1`。
- **L55 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L55 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L56 EN**: Comment explains nearby logic, constraints, or intent: `Store32(__p, 0)`.
  **L56 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store32(__p, 0)`。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `result : 0`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result : 0`。
- **L58 EN**: Comment explains nearby logic, constraints, or intent: `END`.
  **L58 CN**: 注释解释附近代码的逻辑、约束或设计意图：`END`。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L60 EN**: Separator comment used for visual grouping.
  **L60 CN**: 用于视觉分组的分隔注释。
- **L61 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L61 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L62 EN**: Separator comment used for visual grouping.
  **L62 CN**: 用于视觉分组的分隔注释。
- **L63 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c RDSEED instruction.`.
  **L63 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c RDSEED instruction.`。
- **L64 EN**: Separator comment used for visual grouping.
  **L64 CN**: 用于视觉分组的分隔注释。

### Lines 65-80

````c
/// \param __p
///    Pointer to memory for storing the 32-bit random number.
/// \returns 1 if a random number was generated, 0 if not.
static __inline__ int __DEFAULT_FN_ATTRS
_rdseed32_step(unsigned int *__p)
{
  return (int) __builtin_ia32_rdseed32_step(__p);
}

#ifdef __x86_64__
/// Stores a hardware-generated 64-bit random value in the memory at \a __p.
///
///    The random number generator complies with NIST SP800-90B and SP800-90C.
///
/// \code{.operation}
/// IF HW_NRND_GEN.ready == 1
````
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `param __p`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __p`。
- **L66 EN**: Comment explains nearby logic, constraints, or intent: `Pointer to memory for storing the 32-bit random number.`.
  **L66 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pointer to memory for storing the 32-bit random number.`。
- **L67 EN**: Comment explains nearby logic, constraints, or intent: `returns 1 if a random number was generated, 0 if not.`.
  **L67 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns 1 if a random number was generated, 0 if not.`。
- **L68 EN**: Continues the surrounding expression or declaration: `static __inline__ int __DEFAULT_FN_ATTRS`.
  **L68 CN**: 继续构造周围的表达式或声明：`static __inline__ int __DEFAULT_FN_ATTRS`。
- **L69 EN**: Continues logic associated with callable symbol `_rdseed32_step`.
  **L69 CN**: 继续与可调用符号 `_rdseed32_step` 相关的逻辑。
- **L70 EN**: Opens a new lexical scope or compound statement.
  **L70 CN**: 打开一个新的词法作用域或复合语句块。
- **L71 EN**: Returns from the current function with `(int) __builtin_ia32_rdseed32_step(__p)`.
  **L71 CN**: 以 `(int) __builtin_ia32_rdseed32_step(__p)` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L74 EN**: Starts a preprocessor conditional block: `#ifdef __x86_64__`.
  **L74 CN**: 开始一个预处理条件块：`#ifdef __x86_64__`。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `Stores a hardware-generated 64-bit random value in the memory at a __p.`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores a hardware-generated 64-bit random value in the memory at a __p.`。
- **L76 EN**: Separator comment used for visual grouping.
  **L76 CN**: 用于视觉分组的分隔注释。
- **L77 EN**: Comment explains nearby logic, constraints, or intent: `The random number generator complies with NIST SP800-90B and SP800-90C.`.
  **L77 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The random number generator complies with NIST SP800-90B and SP800-90C.`。
- **L78 EN**: Separator comment used for visual grouping.
  **L78 CN**: 用于视觉分组的分隔注释。
- **L79 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L79 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `IF HW_NRND_GEN.ready 1`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`IF HW_NRND_GEN.ready 1`。

### Lines 81-96

````c
///   Store64(__p, HW_NRND_GEN.data)
///   result := 1
/// ELSE
///   Store64(__p, 0)
///   result := 0
/// END
/// \endcode
///
/// \headerfile <immintrin.h>
///
/// This intrinsic corresponds to the \c RDSEED instruction.
///
/// \param __p
///    Pointer to memory for storing the 64-bit random number.
/// \returns 1 if a random number was generated, 0 if not.
static __inline__ int __DEFAULT_FN_ATTRS
````
- **L81 EN**: Comment explains nearby logic, constraints, or intent: `Store64(__p, HW_NRND_GEN.data)`.
  **L81 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store64(__p, HW_NRND_GEN.data)`。
- **L82 EN**: Comment explains nearby logic, constraints, or intent: `result : 1`.
  **L82 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result : 1`。
- **L83 EN**: Comment explains nearby logic, constraints, or intent: `ELSE`.
  **L83 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ELSE`。
- **L84 EN**: Comment explains nearby logic, constraints, or intent: `Store64(__p, 0)`.
  **L84 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store64(__p, 0)`。
- **L85 EN**: Comment explains nearby logic, constraints, or intent: `result : 0`.
  **L85 CN**: 注释解释附近代码的逻辑、约束或设计意图：`result : 0`。
- **L86 EN**: Comment explains nearby logic, constraints, or intent: `END`.
  **L86 CN**: 注释解释附近代码的逻辑、约束或设计意图：`END`。
- **L87 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L87 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L88 EN**: Separator comment used for visual grouping.
  **L88 CN**: 用于视觉分组的分隔注释。
- **L89 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L89 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L90 EN**: Separator comment used for visual grouping.
  **L90 CN**: 用于视觉分组的分隔注释。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c RDSEED instruction.`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c RDSEED instruction.`。
- **L92 EN**: Separator comment used for visual grouping.
  **L92 CN**: 用于视觉分组的分隔注释。
- **L93 EN**: Comment explains nearby logic, constraints, or intent: `param __p`.
  **L93 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __p`。
- **L94 EN**: Comment explains nearby logic, constraints, or intent: `Pointer to memory for storing the 64-bit random number.`.
  **L94 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pointer to memory for storing the 64-bit random number.`。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `returns 1 if a random number was generated, 0 if not.`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns 1 if a random number was generated, 0 if not.`。
- **L96 EN**: Continues the surrounding expression or declaration: `static __inline__ int __DEFAULT_FN_ATTRS`.
  **L96 CN**: 继续构造周围的表达式或声明：`static __inline__ int __DEFAULT_FN_ATTRS`。

### Lines 97-105

````c
_rdseed64_step(unsigned long long *__p)
{
  return (int) __builtin_ia32_rdseed64_step(__p);
}
#endif

#undef __DEFAULT_FN_ATTRS

#endif /* __RDSEEDINTRIN_H */
````
- **L97 EN**: Continues logic associated with callable symbol `_rdseed64_step`.
  **L97 CN**: 继续与可调用符号 `_rdseed64_step` 相关的逻辑。
- **L98 EN**: Opens a new lexical scope or compound statement.
  **L98 CN**: 打开一个新的词法作用域或复合语句块。
- **L99 EN**: Returns from the current function with `(int) __builtin_ia32_rdseed64_step(__p)`.
  **L99 CN**: 以 `(int) __builtin_ia32_rdseed64_step(__p)` 从当前函数返回。
- **L100 EN**: Closes the current lexical scope or compound statement.
  **L100 CN**: 结束当前词法作用域或复合语句块。
- **L101 EN**: Closes the current preprocessor conditional block.
  **L101 CN**: 结束当前预处理条件块。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L103 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS`.
  **L103 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS`。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Closes the current preprocessor conditional block.
  **L105 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Inline wrapper functions / 内联包装函数**
- **Compiler-specific attributes / 编译器特定属性**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__RDSEEDINTRIN_H`, `__x86_64__`
- **External builtins / 外部 builtin**: `__builtin_ia32_rdseed16_step`, `__builtin_ia32_rdseed32_step`, `__builtin_ia32_rdseed64_step`
