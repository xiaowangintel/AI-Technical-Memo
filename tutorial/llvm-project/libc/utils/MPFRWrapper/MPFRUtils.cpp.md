# MPFRUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `libc/utils/MPFRWrapper/MPFRUtils.cpp` | `libc/utils/MPFRWrapper/MPFRUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements logic associated with `MPFRUtils`. Provides MPFR-backed utilities used to validate, compare, or support LLVM libc floating-point behavior. | 实现与 `MPFRUtils` 相关的逻辑。提供基于 MPFR 的工具，用于验证、比较或支撑 LLVM libc 的浮点行为。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- Utils which wrap MPFR ---------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MPFRUtils.h"
#include "MPCommon.h"

#include "src/__support/CPP/array.h"
#include "src/__support/CPP/stringstream.h"
#include "src/__support/FPUtil/bfloat16.h"
#include "src/__support/FPUtil/fpbits_str.h"
#include "src/__support/macros/config.h"
#include "src/__support/macros/properties/types.h"

namespace LIBC_NAMESPACE_DECL {
namespace testing {
namespace mpfr {
namespace internal {

template <typename InputType>
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes "MPFRUtils.h" to access nearby helper declarations.
  **L9 CN**: 引入 "MPFRUtils.h" 以获得附近的辅助声明。
- **L10 EN**: Includes "MPCommon.h" to access nearby helper declarations.
  **L10 CN**: 引入 "MPCommon.h" 以获得附近的辅助声明。
- **L11 EN**: Blank line separating nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L12 EN**: Includes "src/__support/CPP/array.h" to access LLVM libc internal C++ support utilities.
  **L12 CN**: 引入 "src/__support/CPP/array.h" 以获得LLVM libc 内部 C++ 支撑工具。
- **L13 EN**: Includes "src/__support/CPP/stringstream.h" to access LLVM libc internal C++ support utilities.
  **L13 CN**: 引入 "src/__support/CPP/stringstream.h" 以获得LLVM libc 内部 C++ 支撑工具。
- **L14 EN**: Includes "src/__support/FPUtil/bfloat16.h" to access LLVM libc internal support utilities.
  **L14 CN**: 引入 "src/__support/FPUtil/bfloat16.h" 以获得LLVM libc 内部支撑工具。
- **L15 EN**: Includes "src/__support/FPUtil/fpbits_str.h" to access LLVM libc internal support utilities.
  **L15 CN**: 引入 "src/__support/FPUtil/fpbits_str.h" 以获得LLVM libc 内部支撑工具。
- **L16 EN**: Includes "src/__support/macros/config.h" to access LLVM libc internal support utilities.
  **L16 CN**: 引入 "src/__support/macros/config.h" 以获得LLVM libc 内部支撑工具。
- **L17 EN**: Includes "src/__support/macros/properties/types.h" to access LLVM libc internal support utilities.
  **L17 CN**: 引入 "src/__support/macros/properties/types.h" 以获得LLVM libc 内部支撑工具。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Opens namespace scope `LIBC_NAMESPACE_DECL`.
  **L19 CN**: 打开命名空间作用域 `LIBC_NAMESPACE_DECL`。
- **L20 EN**: Opens namespace scope `testing`.
  **L20 CN**: 打开命名空间作用域 `testing`。
- **L21 EN**: Opens namespace scope `mpfr`.
  **L21 CN**: 打开命名空间作用域 `mpfr`。
- **L22 EN**: Opens namespace scope `internal`.
  **L22 CN**: 打开命名空间作用域 `internal`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Introduces template parameters or specialization context: `template <typename InputType>`.
  **L24 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType>`。

### Lines 25-48

````cpp
cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>
unary_operation(Operation op, InputType input, unsigned int precision,
                RoundingMode rounding) {
  MPFRNumber mpfrInput(input, precision, rounding);
  switch (op) {
  case Operation::Abs:
    return mpfrInput.abs();
  case Operation::Acos:
    return mpfrInput.acos();
  case Operation::Acosh:
    return mpfrInput.acosh();
  case Operation::Acospi:
    return mpfrInput.acospi();
  case Operation::Asin:
    return mpfrInput.asin();
  case Operation::Asinh:
    return mpfrInput.asinh();
  case Operation::Asinpi:
    return mpfrInput.asinpi();
  case Operation::Atan:
    return mpfrInput.atan();
  case Operation::Atanh:
    return mpfrInput.atanh();
  case Operation::Atanpi:
````
- **L25 EN**: Continues the surrounding expression or declaration: `cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`.
  **L25 CN**: 继续构造周围的表达式或声明：`cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`。
- **L26 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unary_operation(Operation op, InputType input, unsigned int precision,`.
  **L26 CN**: 继续一个多行参数列表、初始化器或聚合项：`unary_operation(Operation op, InputType input, unsigned int precision,`。
- **L27 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L27 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L28 EN**: Constructs or initializes local object `mpfrInput` with parenthesized arguments.
  **L28 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrInput`。
- **L29 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L29 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L30 EN**: Introduces a switch dispatch label: `case Operation::Abs:`.
  **L30 CN**: 引入一个 switch 分发标签：`case Operation::Abs:`。
- **L31 EN**: Returns from the current function with `mpfrInput.abs()`.
  **L31 CN**: 以 `mpfrInput.abs()` 从当前函数返回。
- **L32 EN**: Introduces a switch dispatch label: `case Operation::Acos:`.
  **L32 CN**: 引入一个 switch 分发标签：`case Operation::Acos:`。
- **L33 EN**: Returns from the current function with `mpfrInput.acos()`.
  **L33 CN**: 以 `mpfrInput.acos()` 从当前函数返回。
- **L34 EN**: Introduces a switch dispatch label: `case Operation::Acosh:`.
  **L34 CN**: 引入一个 switch 分发标签：`case Operation::Acosh:`。
- **L35 EN**: Returns from the current function with `mpfrInput.acosh()`.
  **L35 CN**: 以 `mpfrInput.acosh()` 从当前函数返回。
- **L36 EN**: Introduces a switch dispatch label: `case Operation::Acospi:`.
  **L36 CN**: 引入一个 switch 分发标签：`case Operation::Acospi:`。
- **L37 EN**: Returns from the current function with `mpfrInput.acospi()`.
  **L37 CN**: 以 `mpfrInput.acospi()` 从当前函数返回。
- **L38 EN**: Introduces a switch dispatch label: `case Operation::Asin:`.
  **L38 CN**: 引入一个 switch 分发标签：`case Operation::Asin:`。
- **L39 EN**: Returns from the current function with `mpfrInput.asin()`.
  **L39 CN**: 以 `mpfrInput.asin()` 从当前函数返回。
- **L40 EN**: Introduces a switch dispatch label: `case Operation::Asinh:`.
  **L40 CN**: 引入一个 switch 分发标签：`case Operation::Asinh:`。
- **L41 EN**: Returns from the current function with `mpfrInput.asinh()`.
  **L41 CN**: 以 `mpfrInput.asinh()` 从当前函数返回。
- **L42 EN**: Introduces a switch dispatch label: `case Operation::Asinpi:`.
  **L42 CN**: 引入一个 switch 分发标签：`case Operation::Asinpi:`。
- **L43 EN**: Returns from the current function with `mpfrInput.asinpi()`.
  **L43 CN**: 以 `mpfrInput.asinpi()` 从当前函数返回。
- **L44 EN**: Introduces a switch dispatch label: `case Operation::Atan:`.
  **L44 CN**: 引入一个 switch 分发标签：`case Operation::Atan:`。
- **L45 EN**: Returns from the current function with `mpfrInput.atan()`.
  **L45 CN**: 以 `mpfrInput.atan()` 从当前函数返回。
- **L46 EN**: Introduces a switch dispatch label: `case Operation::Atanh:`.
  **L46 CN**: 引入一个 switch 分发标签：`case Operation::Atanh:`。
- **L47 EN**: Returns from the current function with `mpfrInput.atanh()`.
  **L47 CN**: 以 `mpfrInput.atanh()` 从当前函数返回。
- **L48 EN**: Introduces a switch dispatch label: `case Operation::Atanpi:`.
  **L48 CN**: 引入一个 switch 分发标签：`case Operation::Atanpi:`。

### Lines 49-72

````cpp
    return mpfrInput.atanpi();
  case Operation::Cbrt:
    return mpfrInput.cbrt();
  case Operation::Ceil:
    return mpfrInput.ceil();
  case Operation::Cos:
    return mpfrInput.cos();
  case Operation::Cosh:
    return mpfrInput.cosh();
  case Operation::Cospi:
    return mpfrInput.cospi();
  case Operation::Erf:
    return mpfrInput.erf();
  case Operation::Erfc:
    return mpfrInput.erfc();
  case Operation::Exp:
    return mpfrInput.exp();
  case Operation::Exp2:
    return mpfrInput.exp2();
  case Operation::Exp2m1:
    return mpfrInput.exp2m1();
  case Operation::Exp10:
    return mpfrInput.exp10();
  case Operation::Exp10m1:
````
- **L49 EN**: Returns from the current function with `mpfrInput.atanpi()`.
  **L49 CN**: 以 `mpfrInput.atanpi()` 从当前函数返回。
- **L50 EN**: Introduces a switch dispatch label: `case Operation::Cbrt:`.
  **L50 CN**: 引入一个 switch 分发标签：`case Operation::Cbrt:`。
- **L51 EN**: Returns from the current function with `mpfrInput.cbrt()`.
  **L51 CN**: 以 `mpfrInput.cbrt()` 从当前函数返回。
- **L52 EN**: Introduces a switch dispatch label: `case Operation::Ceil:`.
  **L52 CN**: 引入一个 switch 分发标签：`case Operation::Ceil:`。
- **L53 EN**: Returns from the current function with `mpfrInput.ceil()`.
  **L53 CN**: 以 `mpfrInput.ceil()` 从当前函数返回。
- **L54 EN**: Introduces a switch dispatch label: `case Operation::Cos:`.
  **L54 CN**: 引入一个 switch 分发标签：`case Operation::Cos:`。
- **L55 EN**: Returns from the current function with `mpfrInput.cos()`.
  **L55 CN**: 以 `mpfrInput.cos()` 从当前函数返回。
- **L56 EN**: Introduces a switch dispatch label: `case Operation::Cosh:`.
  **L56 CN**: 引入一个 switch 分发标签：`case Operation::Cosh:`。
- **L57 EN**: Returns from the current function with `mpfrInput.cosh()`.
  **L57 CN**: 以 `mpfrInput.cosh()` 从当前函数返回。
- **L58 EN**: Introduces a switch dispatch label: `case Operation::Cospi:`.
  **L58 CN**: 引入一个 switch 分发标签：`case Operation::Cospi:`。
- **L59 EN**: Returns from the current function with `mpfrInput.cospi()`.
  **L59 CN**: 以 `mpfrInput.cospi()` 从当前函数返回。
- **L60 EN**: Introduces a switch dispatch label: `case Operation::Erf:`.
  **L60 CN**: 引入一个 switch 分发标签：`case Operation::Erf:`。
- **L61 EN**: Returns from the current function with `mpfrInput.erf()`.
  **L61 CN**: 以 `mpfrInput.erf()` 从当前函数返回。
- **L62 EN**: Introduces a switch dispatch label: `case Operation::Erfc:`.
  **L62 CN**: 引入一个 switch 分发标签：`case Operation::Erfc:`。
- **L63 EN**: Returns from the current function with `mpfrInput.erfc()`.
  **L63 CN**: 以 `mpfrInput.erfc()` 从当前函数返回。
- **L64 EN**: Introduces a switch dispatch label: `case Operation::Exp:`.
  **L64 CN**: 引入一个 switch 分发标签：`case Operation::Exp:`。
- **L65 EN**: Returns from the current function with `mpfrInput.exp()`.
  **L65 CN**: 以 `mpfrInput.exp()` 从当前函数返回。
- **L66 EN**: Introduces a switch dispatch label: `case Operation::Exp2:`.
  **L66 CN**: 引入一个 switch 分发标签：`case Operation::Exp2:`。
- **L67 EN**: Returns from the current function with `mpfrInput.exp2()`.
  **L67 CN**: 以 `mpfrInput.exp2()` 从当前函数返回。
- **L68 EN**: Introduces a switch dispatch label: `case Operation::Exp2m1:`.
  **L68 CN**: 引入一个 switch 分发标签：`case Operation::Exp2m1:`。
- **L69 EN**: Returns from the current function with `mpfrInput.exp2m1()`.
  **L69 CN**: 以 `mpfrInput.exp2m1()` 从当前函数返回。
- **L70 EN**: Introduces a switch dispatch label: `case Operation::Exp10:`.
  **L70 CN**: 引入一个 switch 分发标签：`case Operation::Exp10:`。
- **L71 EN**: Returns from the current function with `mpfrInput.exp10()`.
  **L71 CN**: 以 `mpfrInput.exp10()` 从当前函数返回。
- **L72 EN**: Introduces a switch dispatch label: `case Operation::Exp10m1:`.
  **L72 CN**: 引入一个 switch 分发标签：`case Operation::Exp10m1:`。

### Lines 73-96

````cpp
    return mpfrInput.exp10m1();
  case Operation::Expm1:
    return mpfrInput.expm1();
  case Operation::Floor:
    return mpfrInput.floor();
  case Operation::Log:
    return mpfrInput.log();
  case Operation::Log2:
    return mpfrInput.log2();
  case Operation::Log2p1:
    return mpfrInput.log2p1();
  case Operation::Log10:
    return mpfrInput.log10();
  case Operation::Log10p1:
    return mpfrInput.log10p1();
  case Operation::Log1p:
    return mpfrInput.log1p();
  case Operation::Mod2PI:
    return mpfrInput.mod_2pi();
  case Operation::ModPIOver2:
    return mpfrInput.mod_pi_over_2();
  case Operation::ModPIOver4:
    return mpfrInput.mod_pi_over_4();
  case Operation::Round:
````
- **L73 EN**: Returns from the current function with `mpfrInput.exp10m1()`.
  **L73 CN**: 以 `mpfrInput.exp10m1()` 从当前函数返回。
- **L74 EN**: Introduces a switch dispatch label: `case Operation::Expm1:`.
  **L74 CN**: 引入一个 switch 分发标签：`case Operation::Expm1:`。
- **L75 EN**: Returns from the current function with `mpfrInput.expm1()`.
  **L75 CN**: 以 `mpfrInput.expm1()` 从当前函数返回。
- **L76 EN**: Introduces a switch dispatch label: `case Operation::Floor:`.
  **L76 CN**: 引入一个 switch 分发标签：`case Operation::Floor:`。
- **L77 EN**: Returns from the current function with `mpfrInput.floor()`.
  **L77 CN**: 以 `mpfrInput.floor()` 从当前函数返回。
- **L78 EN**: Introduces a switch dispatch label: `case Operation::Log:`.
  **L78 CN**: 引入一个 switch 分发标签：`case Operation::Log:`。
- **L79 EN**: Returns from the current function with `mpfrInput.log()`.
  **L79 CN**: 以 `mpfrInput.log()` 从当前函数返回。
- **L80 EN**: Introduces a switch dispatch label: `case Operation::Log2:`.
  **L80 CN**: 引入一个 switch 分发标签：`case Operation::Log2:`。
- **L81 EN**: Returns from the current function with `mpfrInput.log2()`.
  **L81 CN**: 以 `mpfrInput.log2()` 从当前函数返回。
- **L82 EN**: Introduces a switch dispatch label: `case Operation::Log2p1:`.
  **L82 CN**: 引入一个 switch 分发标签：`case Operation::Log2p1:`。
- **L83 EN**: Returns from the current function with `mpfrInput.log2p1()`.
  **L83 CN**: 以 `mpfrInput.log2p1()` 从当前函数返回。
- **L84 EN**: Introduces a switch dispatch label: `case Operation::Log10:`.
  **L84 CN**: 引入一个 switch 分发标签：`case Operation::Log10:`。
- **L85 EN**: Returns from the current function with `mpfrInput.log10()`.
  **L85 CN**: 以 `mpfrInput.log10()` 从当前函数返回。
- **L86 EN**: Introduces a switch dispatch label: `case Operation::Log10p1:`.
  **L86 CN**: 引入一个 switch 分发标签：`case Operation::Log10p1:`。
- **L87 EN**: Returns from the current function with `mpfrInput.log10p1()`.
  **L87 CN**: 以 `mpfrInput.log10p1()` 从当前函数返回。
- **L88 EN**: Introduces a switch dispatch label: `case Operation::Log1p:`.
  **L88 CN**: 引入一个 switch 分发标签：`case Operation::Log1p:`。
- **L89 EN**: Returns from the current function with `mpfrInput.log1p()`.
  **L89 CN**: 以 `mpfrInput.log1p()` 从当前函数返回。
- **L90 EN**: Introduces a switch dispatch label: `case Operation::Mod2PI:`.
  **L90 CN**: 引入一个 switch 分发标签：`case Operation::Mod2PI:`。
- **L91 EN**: Returns from the current function with `mpfrInput.mod_2pi()`.
  **L91 CN**: 以 `mpfrInput.mod_2pi()` 从当前函数返回。
- **L92 EN**: Introduces a switch dispatch label: `case Operation::ModPIOver2:`.
  **L92 CN**: 引入一个 switch 分发标签：`case Operation::ModPIOver2:`。
- **L93 EN**: Returns from the current function with `mpfrInput.mod_pi_over_2()`.
  **L93 CN**: 以 `mpfrInput.mod_pi_over_2()` 从当前函数返回。
- **L94 EN**: Introduces a switch dispatch label: `case Operation::ModPIOver4:`.
  **L94 CN**: 引入一个 switch 分发标签：`case Operation::ModPIOver4:`。
- **L95 EN**: Returns from the current function with `mpfrInput.mod_pi_over_4()`.
  **L95 CN**: 以 `mpfrInput.mod_pi_over_4()` 从当前函数返回。
- **L96 EN**: Introduces a switch dispatch label: `case Operation::Round:`.
  **L96 CN**: 引入一个 switch 分发标签：`case Operation::Round:`。

### Lines 97-120

````cpp
    return mpfrInput.round();
  case Operation::RoundEven:
    return mpfrInput.roundeven();
  case Operation::Rsqrt:
    return mpfrInput.rsqrt();
  case Operation::Sin:
    return mpfrInput.sin();
  case Operation::Sinpi:
    return mpfrInput.sinpi();
  case Operation::Sinh:
    return mpfrInput.sinh();
  case Operation::Sqrt:
    return mpfrInput.sqrt();
  case Operation::Tan:
    return mpfrInput.tan();
  case Operation::Tanh:
    return mpfrInput.tanh();
  case Operation::Tanpi:
    return mpfrInput.tanpi();
  case Operation::Trunc:
    return mpfrInput.trunc();
  default:
    __builtin_unreachable();
  }
````
- **L97 EN**: Returns from the current function with `mpfrInput.round()`.
  **L97 CN**: 以 `mpfrInput.round()` 从当前函数返回。
- **L98 EN**: Introduces a switch dispatch label: `case Operation::RoundEven:`.
  **L98 CN**: 引入一个 switch 分发标签：`case Operation::RoundEven:`。
- **L99 EN**: Returns from the current function with `mpfrInput.roundeven()`.
  **L99 CN**: 以 `mpfrInput.roundeven()` 从当前函数返回。
- **L100 EN**: Introduces a switch dispatch label: `case Operation::Rsqrt:`.
  **L100 CN**: 引入一个 switch 分发标签：`case Operation::Rsqrt:`。
- **L101 EN**: Returns from the current function with `mpfrInput.rsqrt()`.
  **L101 CN**: 以 `mpfrInput.rsqrt()` 从当前函数返回。
- **L102 EN**: Introduces a switch dispatch label: `case Operation::Sin:`.
  **L102 CN**: 引入一个 switch 分发标签：`case Operation::Sin:`。
- **L103 EN**: Returns from the current function with `mpfrInput.sin()`.
  **L103 CN**: 以 `mpfrInput.sin()` 从当前函数返回。
- **L104 EN**: Introduces a switch dispatch label: `case Operation::Sinpi:`.
  **L104 CN**: 引入一个 switch 分发标签：`case Operation::Sinpi:`。
- **L105 EN**: Returns from the current function with `mpfrInput.sinpi()`.
  **L105 CN**: 以 `mpfrInput.sinpi()` 从当前函数返回。
- **L106 EN**: Introduces a switch dispatch label: `case Operation::Sinh:`.
  **L106 CN**: 引入一个 switch 分发标签：`case Operation::Sinh:`。
- **L107 EN**: Returns from the current function with `mpfrInput.sinh()`.
  **L107 CN**: 以 `mpfrInput.sinh()` 从当前函数返回。
- **L108 EN**: Introduces a switch dispatch label: `case Operation::Sqrt:`.
  **L108 CN**: 引入一个 switch 分发标签：`case Operation::Sqrt:`。
- **L109 EN**: Returns from the current function with `mpfrInput.sqrt()`.
  **L109 CN**: 以 `mpfrInput.sqrt()` 从当前函数返回。
- **L110 EN**: Introduces a switch dispatch label: `case Operation::Tan:`.
  **L110 CN**: 引入一个 switch 分发标签：`case Operation::Tan:`。
- **L111 EN**: Returns from the current function with `mpfrInput.tan()`.
  **L111 CN**: 以 `mpfrInput.tan()` 从当前函数返回。
- **L112 EN**: Introduces a switch dispatch label: `case Operation::Tanh:`.
  **L112 CN**: 引入一个 switch 分发标签：`case Operation::Tanh:`。
- **L113 EN**: Returns from the current function with `mpfrInput.tanh()`.
  **L113 CN**: 以 `mpfrInput.tanh()` 从当前函数返回。
- **L114 EN**: Introduces a switch dispatch label: `case Operation::Tanpi:`.
  **L114 CN**: 引入一个 switch 分发标签：`case Operation::Tanpi:`。
- **L115 EN**: Returns from the current function with `mpfrInput.tanpi()`.
  **L115 CN**: 以 `mpfrInput.tanpi()` 从当前函数返回。
- **L116 EN**: Introduces a switch dispatch label: `case Operation::Trunc:`.
  **L116 CN**: 引入一个 switch 分发标签：`case Operation::Trunc:`。
- **L117 EN**: Returns from the current function with `mpfrInput.trunc()`.
  **L117 CN**: 以 `mpfrInput.trunc()` 从当前函数返回。
- **L118 EN**: Introduces a switch dispatch label: `default:`.
  **L118 CN**: 引入一个 switch 分发标签：`default:`。
- **L119 EN**: Executes a call or declaration centered on `__builtin_unreachable`.
  **L119 CN**: 执行以 `__builtin_unreachable` 为核心的调用或声明。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。

### Lines 121-144

````cpp
}

template <typename InputType>
cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>
unary_operation_two_outputs(Operation op, InputType input, int &output,
                            unsigned int precision, RoundingMode rounding) {
  MPFRNumber mpfrInput(input, precision, rounding);
  switch (op) {
  case Operation::Frexp:
    return mpfrInput.frexp(output);
  default:
    __builtin_unreachable();
  }
}

template <typename InputType>
cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>
binary_operation_one_output(Operation op, InputType x, InputType y,
                            unsigned int precision, RoundingMode rounding) {
  MPFRNumber inputX(x, precision, rounding);
  MPFRNumber inputY(y, precision, rounding);
  switch (op) {
  case Operation::Add:
    return inputX.add(inputY);
````
- **L121 EN**: Closes the current lexical scope or compound statement.
  **L121 CN**: 结束当前词法作用域或复合语句块。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L123 EN**: Introduces template parameters or specialization context: `template <typename InputType>`.
  **L123 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType>`。
- **L124 EN**: Continues the surrounding expression or declaration: `cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`.
  **L124 CN**: 继续构造周围的表达式或声明：`cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unary_operation_two_outputs(Operation op, InputType input, int &output,`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`unary_operation_two_outputs(Operation op, InputType input, int &output,`。
- **L126 EN**: Continues the surrounding expression or declaration: `unsigned int precision, RoundingMode rounding) {`.
  **L126 CN**: 继续构造周围的表达式或声明：`unsigned int precision, RoundingMode rounding) {`。
- **L127 EN**: Constructs or initializes local object `mpfrInput` with parenthesized arguments.
  **L127 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrInput`。
- **L128 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L128 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L129 EN**: Introduces a switch dispatch label: `case Operation::Frexp:`.
  **L129 CN**: 引入一个 switch 分发标签：`case Operation::Frexp:`。
- **L130 EN**: Returns from the current function with `mpfrInput.frexp(output)`.
  **L130 CN**: 以 `mpfrInput.frexp(output)` 从当前函数返回。
- **L131 EN**: Introduces a switch dispatch label: `default:`.
  **L131 CN**: 引入一个 switch 分发标签：`default:`。
- **L132 EN**: Executes a call or declaration centered on `__builtin_unreachable`.
  **L132 CN**: 执行以 `__builtin_unreachable` 为核心的调用或声明。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Closes the current lexical scope or compound statement.
  **L134 CN**: 结束当前词法作用域或复合语句块。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L136 EN**: Introduces template parameters or specialization context: `template <typename InputType>`.
  **L136 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType>`。
- **L137 EN**: Continues the surrounding expression or declaration: `cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`.
  **L137 CN**: 继续构造周围的表达式或声明：`cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `binary_operation_one_output(Operation op, InputType x, InputType y,`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`binary_operation_one_output(Operation op, InputType x, InputType y,`。
- **L139 EN**: Continues the surrounding expression or declaration: `unsigned int precision, RoundingMode rounding) {`.
  **L139 CN**: 继续构造周围的表达式或声明：`unsigned int precision, RoundingMode rounding) {`。
- **L140 EN**: Constructs or initializes local object `inputX` with parenthesized arguments.
  **L140 CN**: 使用带括号的参数构造或初始化局部对象 `inputX`。
- **L141 EN**: Constructs or initializes local object `inputY` with parenthesized arguments.
  **L141 CN**: 使用带括号的参数构造或初始化局部对象 `inputY`。
- **L142 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L142 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L143 EN**: Introduces a switch dispatch label: `case Operation::Add:`.
  **L143 CN**: 引入一个 switch 分发标签：`case Operation::Add:`。
- **L144 EN**: Returns from the current function with `inputX.add(inputY)`.
  **L144 CN**: 以 `inputX.add(inputY)` 从当前函数返回。

### Lines 145-168

````cpp
  case Operation::Atan2:
    return inputX.atan2(inputY);
  case Operation::Div:
    return inputX.div(inputY);
  case Operation::Fmod:
    return inputX.fmod(inputY);
  case Operation::Hypot:
    return inputX.hypot(inputY);
  case Operation::Mul:
    return inputX.mul(inputY);
  case Operation::Pow:
    return inputX.pow(inputY);
  case Operation::Sub:
    return inputX.sub(inputY);
  default:
    __builtin_unreachable();
  }
}

template <typename InputType>
cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>
binary_operation_two_outputs(Operation op, InputType x, InputType y,
                             int &output, unsigned int precision,
                             RoundingMode rounding) {
````
- **L145 EN**: Introduces a switch dispatch label: `case Operation::Atan2:`.
  **L145 CN**: 引入一个 switch 分发标签：`case Operation::Atan2:`。
- **L146 EN**: Returns from the current function with `inputX.atan2(inputY)`.
  **L146 CN**: 以 `inputX.atan2(inputY)` 从当前函数返回。
- **L147 EN**: Introduces a switch dispatch label: `case Operation::Div:`.
  **L147 CN**: 引入一个 switch 分发标签：`case Operation::Div:`。
- **L148 EN**: Returns from the current function with `inputX.div(inputY)`.
  **L148 CN**: 以 `inputX.div(inputY)` 从当前函数返回。
- **L149 EN**: Introduces a switch dispatch label: `case Operation::Fmod:`.
  **L149 CN**: 引入一个 switch 分发标签：`case Operation::Fmod:`。
- **L150 EN**: Returns from the current function with `inputX.fmod(inputY)`.
  **L150 CN**: 以 `inputX.fmod(inputY)` 从当前函数返回。
- **L151 EN**: Introduces a switch dispatch label: `case Operation::Hypot:`.
  **L151 CN**: 引入一个 switch 分发标签：`case Operation::Hypot:`。
- **L152 EN**: Returns from the current function with `inputX.hypot(inputY)`.
  **L152 CN**: 以 `inputX.hypot(inputY)` 从当前函数返回。
- **L153 EN**: Introduces a switch dispatch label: `case Operation::Mul:`.
  **L153 CN**: 引入一个 switch 分发标签：`case Operation::Mul:`。
- **L154 EN**: Returns from the current function with `inputX.mul(inputY)`.
  **L154 CN**: 以 `inputX.mul(inputY)` 从当前函数返回。
- **L155 EN**: Introduces a switch dispatch label: `case Operation::Pow:`.
  **L155 CN**: 引入一个 switch 分发标签：`case Operation::Pow:`。
- **L156 EN**: Returns from the current function with `inputX.pow(inputY)`.
  **L156 CN**: 以 `inputX.pow(inputY)` 从当前函数返回。
- **L157 EN**: Introduces a switch dispatch label: `case Operation::Sub:`.
  **L157 CN**: 引入一个 switch 分发标签：`case Operation::Sub:`。
- **L158 EN**: Returns from the current function with `inputX.sub(inputY)`.
  **L158 CN**: 以 `inputX.sub(inputY)` 从当前函数返回。
- **L159 EN**: Introduces a switch dispatch label: `default:`.
  **L159 CN**: 引入一个 switch 分发标签：`default:`。
- **L160 EN**: Executes a call or declaration centered on `__builtin_unreachable`.
  **L160 CN**: 执行以 `__builtin_unreachable` 为核心的调用或声明。
- **L161 EN**: Closes the current lexical scope or compound statement.
  **L161 CN**: 结束当前词法作用域或复合语句块。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Introduces template parameters or specialization context: `template <typename InputType>`.
  **L164 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType>`。
- **L165 EN**: Continues the surrounding expression or declaration: `cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`.
  **L165 CN**: 继续构造周围的表达式或声明：`cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `binary_operation_two_outputs(Operation op, InputType x, InputType y,`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`binary_operation_two_outputs(Operation op, InputType x, InputType y,`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int &output, unsigned int precision,`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`int &output, unsigned int precision,`。
- **L168 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L168 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。

### Lines 169-192

````cpp
  MPFRNumber inputX(x, precision, rounding);
  MPFRNumber inputY(y, precision, rounding);
  switch (op) {
  case Operation::RemQuo:
    return inputX.remquo(inputY, output);
  default:
    __builtin_unreachable();
  }
}

template <typename InputType>
cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>
ternary_operation_one_output(Operation op, InputType x, InputType y,
                             InputType z, unsigned int precision,
                             RoundingMode rounding) {
  // For FMA function, we just need to compare with the mpfr_fma with the same
  // precision as InputType.  Using higher precision as the intermediate results
  // to compare might incorrectly fail due to double-rounding errors.
  MPFRNumber inputX(x, precision, rounding);
  MPFRNumber inputY(y, precision, rounding);
  MPFRNumber inputZ(z, precision, rounding);
  switch (op) {
  case Operation::Fma:
    return inputX.fma(inputY, inputZ);
````
- **L169 EN**: Constructs or initializes local object `inputX` with parenthesized arguments.
  **L169 CN**: 使用带括号的参数构造或初始化局部对象 `inputX`。
- **L170 EN**: Constructs or initializes local object `inputY` with parenthesized arguments.
  **L170 CN**: 使用带括号的参数构造或初始化局部对象 `inputY`。
- **L171 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L171 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L172 EN**: Introduces a switch dispatch label: `case Operation::RemQuo:`.
  **L172 CN**: 引入一个 switch 分发标签：`case Operation::RemQuo:`。
- **L173 EN**: Returns from the current function with `inputX.remquo(inputY, output)`.
  **L173 CN**: 以 `inputX.remquo(inputY, output)` 从当前函数返回。
- **L174 EN**: Introduces a switch dispatch label: `default:`.
  **L174 CN**: 引入一个 switch 分发标签：`default:`。
- **L175 EN**: Executes a call or declaration centered on `__builtin_unreachable`.
  **L175 CN**: 执行以 `__builtin_unreachable` 为核心的调用或声明。
- **L176 EN**: Closes the current lexical scope or compound statement.
  **L176 CN**: 结束当前词法作用域或复合语句块。
- **L177 EN**: Closes the current lexical scope or compound statement.
  **L177 CN**: 结束当前词法作用域或复合语句块。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Introduces template parameters or specialization context: `template <typename InputType>`.
  **L179 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType>`。
- **L180 EN**: Continues the surrounding expression or declaration: `cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`.
  **L180 CN**: 继续构造周围的表达式或声明：`cpp::enable_if_t<cpp::is_floating_point_v<InputType>, MPFRNumber>`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ternary_operation_one_output(Operation op, InputType x, InputType y,`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`ternary_operation_one_output(Operation op, InputType x, InputType y,`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `InputType z, unsigned int precision,`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`InputType z, unsigned int precision,`。
- **L183 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L183 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L184 EN**: Comment explains nearby logic, invariants, or intent: `For FMA function, we just need to compare with the mpfr_fma with the same`.
  **L184 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For FMA function, we just need to compare with the mpfr_fma with the same`。
- **L185 EN**: Comment explains nearby logic, invariants, or intent: `precision as InputType.  Using higher precision as the intermediate results`.
  **L185 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`precision as InputType.  Using higher precision as the intermediate results`。
- **L186 EN**: Comment explains nearby logic, invariants, or intent: `to compare might incorrectly fail due to double-rounding errors.`.
  **L186 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to compare might incorrectly fail due to double-rounding errors.`。
- **L187 EN**: Constructs or initializes local object `inputX` with parenthesized arguments.
  **L187 CN**: 使用带括号的参数构造或初始化局部对象 `inputX`。
- **L188 EN**: Constructs or initializes local object `inputY` with parenthesized arguments.
  **L188 CN**: 使用带括号的参数构造或初始化局部对象 `inputY`。
- **L189 EN**: Constructs or initializes local object `inputZ` with parenthesized arguments.
  **L189 CN**: 使用带括号的参数构造或初始化局部对象 `inputZ`。
- **L190 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L190 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L191 EN**: Introduces a switch dispatch label: `case Operation::Fma:`.
  **L191 CN**: 引入一个 switch 分发标签：`case Operation::Fma:`。
- **L192 EN**: Returns from the current function with `inputX.fma(inputY, inputZ)`.
  **L192 CN**: 以 `inputX.fma(inputY, inputZ)` 从当前函数返回。

### Lines 193-216

````cpp
  default:
    __builtin_unreachable();
  }
}

// Remark: For all the explain_*_error functions, we will use std::stringstream
// to build the complete error messages before sending it to the outstream `OS`
// once at the end.  This will stop the error messages from interleaving when
// the tests are running concurrently.
template <typename InputType, typename OutputType>
void explain_unary_operation_single_output_error(Operation op, InputType input,
                                                 OutputType matchValue,
                                                 double ulp_tolerance,
                                                 RoundingMode rounding) {
  unsigned int precision = get_precision<InputType>(ulp_tolerance);
  MPFRNumber mpfrInput(input, precision);
  MPFRNumber mpfr_result;
  mpfr_result = unary_operation(op, input, precision, rounding);
  MPFRNumber mpfrMatchValue(matchValue);
  cpp::array<char, 1024> msg_buf;
  cpp::StringStream msg(msg_buf);
  msg << "Match value not within tolerance value of MPFR result:\n"
      << "  Input decimal: " << mpfrInput.str() << '\n';
  msg << "     Input bits: " << str(FPBits<InputType>(input)) << '\n';
````
- **L193 EN**: Introduces a switch dispatch label: `default:`.
  **L193 CN**: 引入一个 switch 分发标签：`default:`。
- **L194 EN**: Executes a call or declaration centered on `__builtin_unreachable`.
  **L194 CN**: 执行以 `__builtin_unreachable` 为核心的调用或声明。
- **L195 EN**: Closes the current lexical scope or compound statement.
  **L195 CN**: 结束当前词法作用域或复合语句块。
- **L196 EN**: Closes the current lexical scope or compound statement.
  **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Comment explains nearby logic, invariants, or intent: `Remark: For all the explain_*_error functions, we will use std::stringstream`.
  **L198 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Remark: For all the explain_*_error functions, we will use std::stringstream`。
- **L199 EN**: Comment explains nearby logic, invariants, or intent: `to build the complete error messages before sending it to the outstream `OS``.
  **L199 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to build the complete error messages before sending it to the outstream `OS``。
- **L200 EN**: Comment explains nearby logic, invariants, or intent: `once at the end.  This will stop the error messages from interleaving when`.
  **L200 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`once at the end.  This will stop the error messages from interleaving when`。
- **L201 EN**: Comment explains nearby logic, invariants, or intent: `the tests are running concurrently.`.
  **L201 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the tests are running concurrently.`。
- **L202 EN**: Introduces template parameters or specialization context: `template <typename InputType, typename OutputType>`.
  **L202 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType, typename OutputType>`。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void explain_unary_operation_single_output_error(Operation op, InputType input,`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`void explain_unary_operation_single_output_error(Operation op, InputType input,`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputType matchValue,`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputType matchValue,`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double ulp_tolerance,`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`double ulp_tolerance,`。
- **L206 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L206 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L207 EN**: Initializes variable `precision` from the right-hand expression.
  **L207 CN**: 使用右侧表达式初始化变量 `precision`。
- **L208 EN**: Constructs or initializes local object `mpfrInput` with parenthesized arguments.
  **L208 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrInput`。
- **L209 EN**: Executes a standalone statement or declaration: `MPFRNumber mpfr_result;`.
  **L209 CN**: 执行一条独立语句或声明：`MPFRNumber mpfr_result;`。
- **L210 EN**: Executes a call or declaration centered on `unary_operation`.
  **L210 CN**: 执行以 `unary_operation` 为核心的调用或声明。
- **L211 EN**: Constructs or initializes local object `mpfrMatchValue` with parenthesized arguments.
  **L211 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrMatchValue`。
- **L212 EN**: Executes a standalone statement or declaration: `cpp::array<char, 1024> msg_buf;`.
  **L212 CN**: 执行一条独立语句或声明：`cpp::array<char, 1024> msg_buf;`。
- **L213 EN**: Constructs or initializes local object `msg` with parenthesized arguments.
  **L213 CN**: 使用带括号的参数构造或初始化局部对象 `msg`。
- **L214 EN**: Continues the surrounding expression or declaration: `msg << "Match value not within tolerance value of MPFR result:\n"`.
  **L214 CN**: 继续构造周围的表达式或声明：`msg << "Match value not within tolerance value of MPFR result:\n"`。
- **L215 EN**: Executes a call or declaration centered on `str`.
  **L215 CN**: 执行以 `str` 为核心的调用或声明。
- **L216 EN**: Executes a call or declaration centered on `str`.
  **L216 CN**: 执行以 `str` 为核心的调用或声明。

### Lines 217-240

````cpp
  msg << '\n' << "  Match decimal: " << mpfrMatchValue.str() << '\n';
  msg << "     Match bits: " << str(FPBits<OutputType>(matchValue)) << '\n';
  msg << '\n' << "    MPFR result: " << mpfr_result.str() << '\n';
  msg << "   MPFR rounded: "
      << str(FPBits<OutputType>(mpfr_result.as<OutputType>())) << '\n';
  msg << '\n';
  msg << "      ULP error: " << mpfr_result.ulp_as_mpfr_number(matchValue).str()
      << '\n';
  if (msg.overflow())
    __builtin_unreachable();
  tlog << msg.str();
}

template void explain_unary_operation_single_output_error(Operation op, float,
                                                          float, double,
                                                          RoundingMode);
template void explain_unary_operation_single_output_error(Operation op, double,
                                                          double, double,
                                                          RoundingMode);
template void explain_unary_operation_single_output_error(Operation op,
                                                          long double,
                                                          long double, double,
                                                          RoundingMode);
template void explain_unary_operation_single_output_error(Operation op, double,
````
- **L217 EN**: Executes a call or declaration centered on `str`.
  **L217 CN**: 执行以 `str` 为核心的调用或声明。
- **L218 EN**: Executes a call or declaration centered on `str`.
  **L218 CN**: 执行以 `str` 为核心的调用或声明。
- **L219 EN**: Executes a call or declaration centered on `str`.
  **L219 CN**: 执行以 `str` 为核心的调用或声明。
- **L220 EN**: Continues the surrounding expression or declaration: `msg << "   MPFR rounded: "`.
  **L220 CN**: 继续构造周围的表达式或声明：`msg << "   MPFR rounded: "`。
- **L221 EN**: Executes a call or declaration centered on `str`.
  **L221 CN**: 执行以 `str` 为核心的调用或声明。
- **L222 EN**: Executes a standalone statement or declaration: `msg << '\n';`.
  **L222 CN**: 执行一条独立语句或声明：`msg << '\n';`。
- **L223 EN**: Continues logic associated with callable symbol `ulp_as_mpfr_number`.
  **L223 CN**: 继续与可调用符号 `ulp_as_mpfr_number` 相关的逻辑。
- **L224 EN**: Executes a standalone statement or declaration: `<< '\n';`.
  **L224 CN**: 执行一条独立语句或声明：`<< '\n';`。
- **L225 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L225 CN**: 开始 `if` 控制流语句并计算其条件。
- **L226 EN**: Executes a call or declaration centered on `__builtin_unreachable`.
  **L226 CN**: 执行以 `__builtin_unreachable` 为核心的调用或声明。
- **L227 EN**: Executes a call or declaration centered on `str`.
  **L227 CN**: 执行以 `str` 为核心的调用或声明。
- **L228 EN**: Closes the current lexical scope or compound statement.
  **L228 CN**: 结束当前词法作用域或复合语句块。
- **L229 EN**: Blank line separating nearby declarations or logic blocks.
  **L229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L230 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op, float,`.
  **L230 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op, float,`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float, double,`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`float, double,`。
- **L232 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L232 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L233 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op, double,`.
  **L233 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op, double,`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double, double,`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`double, double,`。
- **L235 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L235 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L236 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L236 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `long double,`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`long double,`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `long double, double,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`long double, double,`。
- **L239 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L239 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L240 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op, double,`.
  **L240 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op, double,`。

### Lines 241-264

````cpp
                                                          float, double,
                                                          RoundingMode);
template void explain_unary_operation_single_output_error(Operation op,
                                                          long double, float,
                                                          double, RoundingMode);
template void explain_unary_operation_single_output_error(Operation op,
                                                          long double, double,
                                                          double, RoundingMode);

#ifdef LIBC_TYPES_HAS_FLOAT16
template void explain_unary_operation_single_output_error(Operation op, float16,
                                                          float16, double,
                                                          RoundingMode);
template void explain_unary_operation_single_output_error(Operation op, float,
                                                          float16, double,
                                                          RoundingMode);
template void explain_unary_operation_single_output_error(Operation op, double,
                                                          float16, double,
                                                          RoundingMode);
template void explain_unary_operation_single_output_error(Operation op,
                                                          long double, float16,
                                                          double, RoundingMode);
#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
template void explain_unary_operation_single_output_error(Operation op,
````
- **L241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float, double,`.
  **L241 CN**: 继续一个多行参数列表、初始化器或聚合项：`float, double,`。
- **L242 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L242 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L243 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L243 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `long double, float,`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`long double, float,`。
- **L245 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L245 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L246 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L246 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `long double, double,`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`long double, double,`。
- **L248 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L248 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L250 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L250 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L251 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op, float16,`.
  **L251 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op, float16,`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L253 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L253 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L254 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op, float,`.
  **L254 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op, float,`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L256 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L256 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L257 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op, double,`.
  **L257 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op, double,`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L259 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L259 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L260 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L260 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `long double, float16,`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`long double, float16,`。
- **L262 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L262 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L263 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`.
  **L263 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`。
- **L264 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L264 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。

### Lines 265-288

````cpp
                                                          float128, float16,
                                                          double, RoundingMode);
#endif // LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
#endif // LIBC_TYPES_HAS_FLOAT16

#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
template void explain_unary_operation_single_output_error(Operation op,
                                                          float128, float128,
                                                          double, RoundingMode);
template void explain_unary_operation_single_output_error(Operation op,
                                                          float128, float,
                                                          double, RoundingMode);
template void explain_unary_operation_single_output_error(Operation op,
                                                          float128, double,
                                                          double, RoundingMode);
template void explain_unary_operation_single_output_error(Operation op,
                                                          float128, long double,
                                                          double, RoundingMode);
#endif // LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE

template void explain_unary_operation_single_output_error(Operation op,
                                                          bfloat16, bfloat16,
                                                          double, RoundingMode);

````
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float128, float16,`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`float128, float16,`。
- **L266 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L266 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L267 EN**: Closes the current preprocessor conditional block.
  **L267 CN**: 结束当前的预处理条件块。
- **L268 EN**: Closes the current preprocessor conditional block.
  **L268 CN**: 结束当前的预处理条件块。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`.
  **L270 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`。
- **L271 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L271 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float128, float128,`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`float128, float128,`。
- **L273 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L273 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L274 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L274 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float128, float,`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`float128, float,`。
- **L276 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L276 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L277 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L277 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float128, double,`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`float128, double,`。
- **L279 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L279 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L280 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L280 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float128, long double,`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`float128, long double,`。
- **L282 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L282 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L283 EN**: Closes the current preprocessor conditional block.
  **L283 CN**: 结束当前的预处理条件块。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L285 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_single_output_error(Operation op,`.
  **L285 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_single_output_error(Operation op,`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bfloat16, bfloat16,`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`bfloat16, bfloat16,`。
- **L287 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L287 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 289-312

````cpp
template <typename T>
void explain_unary_operation_two_outputs_error(
    Operation op, T input, const BinaryOutput<T> &libc_result,
    double ulp_tolerance, RoundingMode rounding) {
  unsigned int precision = get_precision<T>(ulp_tolerance);
  MPFRNumber mpfrInput(input, precision);
  int mpfrIntResult;
  MPFRNumber mpfr_result = unary_operation_two_outputs(op, input, mpfrIntResult,
                                                       precision, rounding);

  if (mpfrIntResult != libc_result.i) {
    tlog << "MPFR integral result: " << mpfrIntResult << '\n'
         << "Libc integral result: " << libc_result.i << '\n';
  } else {
    tlog << "Integral result from libc matches integral result from MPFR.\n";
  }

  MPFRNumber mpfrMatchValue(libc_result.f);
  tlog
      << "Libc floating point result is not within tolerance value of the MPFR "
      << "result.\n\n";

  tlog << "            Input decimal: " << mpfrInput.str() << "\n\n";

````
- **L289 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  **L289 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L290 EN**: Continues logic associated with callable symbol `explain_unary_operation_two_outputs_error`.
  **L290 CN**: 继续与可调用符号 `explain_unary_operation_two_outputs_error` 相关的逻辑。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation op, T input, const BinaryOutput<T> &libc_result,`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation op, T input, const BinaryOutput<T> &libc_result,`。
- **L292 EN**: Continues the surrounding expression or declaration: `double ulp_tolerance, RoundingMode rounding) {`.
  **L292 CN**: 继续构造周围的表达式或声明：`double ulp_tolerance, RoundingMode rounding) {`。
- **L293 EN**: Initializes variable `precision` from the right-hand expression.
  **L293 CN**: 使用右侧表达式初始化变量 `precision`。
- **L294 EN**: Constructs or initializes local object `mpfrInput` with parenthesized arguments.
  **L294 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrInput`。
- **L295 EN**: Executes a standalone statement or declaration: `int mpfrIntResult;`.
  **L295 CN**: 执行一条独立语句或声明：`int mpfrIntResult;`。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MPFRNumber mpfr_result = unary_operation_two_outputs(op, input, mpfrIntResult,`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`MPFRNumber mpfr_result = unary_operation_two_outputs(op, input, mpfrIntResult,`。
- **L297 EN**: Executes a standalone statement or declaration: `precision, rounding);`.
  **L297 CN**: 执行一条独立语句或声明：`precision, rounding);`。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L299 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L299 CN**: 开始 `if` 控制流语句并计算其条件。
- **L300 EN**: Continues the surrounding expression or declaration: `tlog << "MPFR integral result: " << mpfrIntResult << '\n'`.
  **L300 CN**: 继续构造周围的表达式或声明：`tlog << "MPFR integral result: " << mpfrIntResult << '\n'`。
- **L301 EN**: Executes a standalone statement or declaration: `<< "Libc integral result: " << libc_result.i << '\n';`.
  **L301 CN**: 执行一条独立语句或声明：`<< "Libc integral result: " << libc_result.i << '\n';`。
- **L302 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L302 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L303 EN**: Executes a standalone statement or declaration: `tlog << "Integral result from libc matches integral result from MPFR.\n";`.
  **L303 CN**: 执行一条独立语句或声明：`tlog << "Integral result from libc matches integral result from MPFR.\n";`。
- **L304 EN**: Closes the current lexical scope or compound statement.
  **L304 CN**: 结束当前词法作用域或复合语句块。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L306 EN**: Constructs or initializes local object `mpfrMatchValue` with parenthesized arguments.
  **L306 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrMatchValue`。
- **L307 EN**: Continues the surrounding expression or declaration: `tlog`.
  **L307 CN**: 继续构造周围的表达式或声明：`tlog`。
- **L308 EN**: Continues the surrounding expression or declaration: `<< "Libc floating point result is not within tolerance value of the MPFR "`.
  **L308 CN**: 继续构造周围的表达式或声明：`<< "Libc floating point result is not within tolerance value of the MPFR "`。
- **L309 EN**: Executes a standalone statement or declaration: `<< "result.\n\n";`.
  **L309 CN**: 执行一条独立语句或声明：`<< "result.\n\n";`。
- **L310 EN**: Blank line separating nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L311 EN**: Executes a call or declaration centered on `str`.
  **L311 CN**: 执行以 `str` 为核心的调用或声明。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 313-336

````cpp
  tlog << "Libc floating point value: " << mpfrMatchValue.str() << '\n';
  tlog << " Libc floating point bits: " << str(FPBits<T>(libc_result.f))
       << '\n';
  tlog << "\n\n";

  tlog << "              MPFR result: " << mpfr_result.str() << '\n';
  tlog << "             MPFR rounded: " << str(FPBits<T>(mpfr_result.as<T>()))
       << '\n';
  tlog << '\n'
       << "                ULP error: "
       << mpfr_result.ulp_as_mpfr_number(libc_result.f).str() << '\n';
}

template void explain_unary_operation_two_outputs_error<float>(
    Operation, float, const BinaryOutput<float> &, double, RoundingMode);
template void explain_unary_operation_two_outputs_error<double>(
    Operation, double, const BinaryOutput<double> &, double, RoundingMode);
template void explain_unary_operation_two_outputs_error<long double>(
    Operation, long double, const BinaryOutput<long double> &, double,
    RoundingMode);

template <typename T>
void explain_binary_operation_two_outputs_error(
    Operation op, const BinaryInput<T> &input,
````
- **L313 EN**: Executes a call or declaration centered on `str`.
  **L313 CN**: 执行以 `str` 为核心的调用或声明。
- **L314 EN**: Continues logic associated with callable symbol `str`.
  **L314 CN**: 继续与可调用符号 `str` 相关的逻辑。
- **L315 EN**: Executes a standalone statement or declaration: `<< '\n';`.
  **L315 CN**: 执行一条独立语句或声明：`<< '\n';`。
- **L316 EN**: Executes a standalone statement or declaration: `tlog << "\n\n";`.
  **L316 CN**: 执行一条独立语句或声明：`tlog << "\n\n";`。
- **L317 EN**: Blank line separating nearby declarations or logic blocks.
  **L317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L318 EN**: Executes a call or declaration centered on `str`.
  **L318 CN**: 执行以 `str` 为核心的调用或声明。
- **L319 EN**: Continues logic associated with callable symbol `str`.
  **L319 CN**: 继续与可调用符号 `str` 相关的逻辑。
- **L320 EN**: Executes a standalone statement or declaration: `<< '\n';`.
  **L320 CN**: 执行一条独立语句或声明：`<< '\n';`。
- **L321 EN**: Continues the surrounding expression or declaration: `tlog << '\n'`.
  **L321 CN**: 继续构造周围的表达式或声明：`tlog << '\n'`。
- **L322 EN**: Continues the surrounding expression or declaration: `<< "                ULP error: "`.
  **L322 CN**: 继续构造周围的表达式或声明：`<< "                ULP error: "`。
- **L323 EN**: Executes a call or declaration centered on `ulp_as_mpfr_number`.
  **L323 CN**: 执行以 `ulp_as_mpfr_number` 为核心的调用或声明。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L326 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_two_outputs_error<float>(`.
  **L326 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_two_outputs_error<float>(`。
- **L327 EN**: Executes a standalone statement or declaration: `Operation, float, const BinaryOutput<float> &, double, RoundingMode);`.
  **L327 CN**: 执行一条独立语句或声明：`Operation, float, const BinaryOutput<float> &, double, RoundingMode);`。
- **L328 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_two_outputs_error<double>(`.
  **L328 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_two_outputs_error<double>(`。
- **L329 EN**: Executes a standalone statement or declaration: `Operation, double, const BinaryOutput<double> &, double, RoundingMode);`.
  **L329 CN**: 执行一条独立语句或声明：`Operation, double, const BinaryOutput<double> &, double, RoundingMode);`。
- **L330 EN**: Introduces template parameters or specialization context: `template void explain_unary_operation_two_outputs_error<long double>(`.
  **L330 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_unary_operation_two_outputs_error<long double>(`。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation, long double, const BinaryOutput<long double> &, double,`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation, long double, const BinaryOutput<long double> &, double,`。
- **L332 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L332 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L334 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  **L334 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L335 EN**: Continues logic associated with callable symbol `explain_binary_operation_two_outputs_error`.
  **L335 CN**: 继续与可调用符号 `explain_binary_operation_two_outputs_error` 相关的逻辑。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation op, const BinaryInput<T> &input,`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation op, const BinaryInput<T> &input,`。

### Lines 337-360

````cpp
    const BinaryOutput<T> &libc_result, double ulp_tolerance,
    RoundingMode rounding) {
  unsigned int precision = get_precision<T>(ulp_tolerance);
  MPFRNumber mpfrX(input.x, precision);
  MPFRNumber mpfrY(input.y, precision);
  int mpfrIntResult;
  MPFRNumber mpfr_result = binary_operation_two_outputs(
      op, input.x, input.y, mpfrIntResult, precision, rounding);
  MPFRNumber mpfrMatchValue(libc_result.f);

  tlog << "Input decimal: x: " << mpfrX.str() << " y: " << mpfrY.str() << '\n'
       << "MPFR integral result: " << mpfrIntResult << '\n'
       << "Libc integral result: " << libc_result.i << '\n'
       << "Libc floating point result: " << mpfrMatchValue.str() << '\n'
       << "               MPFR result: " << mpfr_result.str() << '\n';
  tlog << "Libc floating point result bits: " << str(FPBits<T>(libc_result.f))
       << '\n';
  tlog << "              MPFR rounded bits: "
       << str(FPBits<T>(mpfr_result.as<T>())) << '\n';
  tlog << "ULP error: " << mpfr_result.ulp_as_mpfr_number(libc_result.f).str()
       << '\n';
}

template void explain_binary_operation_two_outputs_error<float>(
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryOutput<T> &libc_result, double ulp_tolerance,`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryOutput<T> &libc_result, double ulp_tolerance,`。
- **L338 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L338 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L339 EN**: Initializes variable `precision` from the right-hand expression.
  **L339 CN**: 使用右侧表达式初始化变量 `precision`。
- **L340 EN**: Constructs or initializes local object `mpfrX` with parenthesized arguments.
  **L340 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrX`。
- **L341 EN**: Constructs or initializes local object `mpfrY` with parenthesized arguments.
  **L341 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrY`。
- **L342 EN**: Executes a standalone statement or declaration: `int mpfrIntResult;`.
  **L342 CN**: 执行一条独立语句或声明：`int mpfrIntResult;`。
- **L343 EN**: Continues logic associated with callable symbol `binary_operation_two_outputs`.
  **L343 CN**: 继续与可调用符号 `binary_operation_two_outputs` 相关的逻辑。
- **L344 EN**: Executes a standalone statement or declaration: `op, input.x, input.y, mpfrIntResult, precision, rounding);`.
  **L344 CN**: 执行一条独立语句或声明：`op, input.x, input.y, mpfrIntResult, precision, rounding);`。
- **L345 EN**: Constructs or initializes local object `mpfrMatchValue` with parenthesized arguments.
  **L345 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrMatchValue`。
- **L346 EN**: Blank line separating nearby declarations or logic blocks.
  **L346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L347 EN**: Continues logic associated with callable symbol `str`.
  **L347 CN**: 继续与可调用符号 `str` 相关的逻辑。
- **L348 EN**: Continues the surrounding expression or declaration: `<< "MPFR integral result: " << mpfrIntResult << '\n'`.
  **L348 CN**: 继续构造周围的表达式或声明：`<< "MPFR integral result: " << mpfrIntResult << '\n'`。
- **L349 EN**: Continues the surrounding expression or declaration: `<< "Libc integral result: " << libc_result.i << '\n'`.
  **L349 CN**: 继续构造周围的表达式或声明：`<< "Libc integral result: " << libc_result.i << '\n'`。
- **L350 EN**: Continues logic associated with callable symbol `str`.
  **L350 CN**: 继续与可调用符号 `str` 相关的逻辑。
- **L351 EN**: Executes a call or declaration centered on `str`.
  **L351 CN**: 执行以 `str` 为核心的调用或声明。
- **L352 EN**: Continues logic associated with callable symbol `str`.
  **L352 CN**: 继续与可调用符号 `str` 相关的逻辑。
- **L353 EN**: Executes a standalone statement or declaration: `<< '\n';`.
  **L353 CN**: 执行一条独立语句或声明：`<< '\n';`。
- **L354 EN**: Continues the surrounding expression or declaration: `tlog << "              MPFR rounded bits: "`.
  **L354 CN**: 继续构造周围的表达式或声明：`tlog << "              MPFR rounded bits: "`。
- **L355 EN**: Executes a call or declaration centered on `str`.
  **L355 CN**: 执行以 `str` 为核心的调用或声明。
- **L356 EN**: Continues logic associated with callable symbol `ulp_as_mpfr_number`.
  **L356 CN**: 继续与可调用符号 `ulp_as_mpfr_number` 相关的逻辑。
- **L357 EN**: Executes a standalone statement or declaration: `<< '\n';`.
  **L357 CN**: 执行一条独立语句或声明：`<< '\n';`。
- **L358 EN**: Closes the current lexical scope or compound statement.
  **L358 CN**: 结束当前词法作用域或复合语句块。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L360 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_two_outputs_error<float>(`.
  **L360 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_two_outputs_error<float>(`。

### Lines 361-384

````cpp
    Operation, const BinaryInput<float> &, const BinaryOutput<float> &, double,
    RoundingMode);
template void explain_binary_operation_two_outputs_error<double>(
    Operation, const BinaryInput<double> &, const BinaryOutput<double> &,
    double, RoundingMode);
template void explain_binary_operation_two_outputs_error<long double>(
    Operation, const BinaryInput<long double> &,
    const BinaryOutput<long double> &, double, RoundingMode);

template <typename InputType, typename OutputType>
void explain_binary_operation_one_output_error(
    Operation op, const BinaryInput<InputType> &input, OutputType libc_result,
    double ulp_tolerance, RoundingMode rounding) {
  unsigned int precision = get_precision<InputType>(ulp_tolerance);
  MPFRNumber mpfrX(input.x, precision);
  MPFRNumber mpfrY(input.y, precision);
  FPBits<InputType> xbits(input.x);
  FPBits<InputType> ybits(input.y);
  MPFRNumber mpfr_result =
      binary_operation_one_output(op, input.x, input.y, precision, rounding);
  MPFRNumber mpfrMatchValue(libc_result);

  tlog << "Input decimal: x: " << mpfrX.str() << " y: " << mpfrY.str() << '\n';
  tlog << "First input bits: " << str(FPBits<InputType>(input.x)) << '\n';
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation, const BinaryInput<float> &, const BinaryOutput<float> &, double,`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation, const BinaryInput<float> &, const BinaryOutput<float> &, double,`。
- **L362 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L362 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L363 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_two_outputs_error<double>(`.
  **L363 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_two_outputs_error<double>(`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation, const BinaryInput<double> &, const BinaryOutput<double> &,`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation, const BinaryInput<double> &, const BinaryOutput<double> &,`。
- **L365 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L365 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L366 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_two_outputs_error<long double>(`.
  **L366 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_two_outputs_error<long double>(`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation, const BinaryInput<long double> &,`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation, const BinaryInput<long double> &,`。
- **L368 EN**: Executes a standalone statement or declaration: `const BinaryOutput<long double> &, double, RoundingMode);`.
  **L368 CN**: 执行一条独立语句或声明：`const BinaryOutput<long double> &, double, RoundingMode);`。
- **L369 EN**: Blank line separating nearby declarations or logic blocks.
  **L369 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L370 EN**: Introduces template parameters or specialization context: `template <typename InputType, typename OutputType>`.
  **L370 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType, typename OutputType>`。
- **L371 EN**: Continues logic associated with callable symbol `explain_binary_operation_one_output_error`.
  **L371 CN**: 继续与可调用符号 `explain_binary_operation_one_output_error` 相关的逻辑。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation op, const BinaryInput<InputType> &input, OutputType libc_result,`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation op, const BinaryInput<InputType> &input, OutputType libc_result,`。
- **L373 EN**: Continues the surrounding expression or declaration: `double ulp_tolerance, RoundingMode rounding) {`.
  **L373 CN**: 继续构造周围的表达式或声明：`double ulp_tolerance, RoundingMode rounding) {`。
- **L374 EN**: Initializes variable `precision` from the right-hand expression.
  **L374 CN**: 使用右侧表达式初始化变量 `precision`。
- **L375 EN**: Constructs or initializes local object `mpfrX` with parenthesized arguments.
  **L375 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrX`。
- **L376 EN**: Constructs or initializes local object `mpfrY` with parenthesized arguments.
  **L376 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrY`。
- **L377 EN**: Constructs or initializes local object `xbits` with parenthesized arguments.
  **L377 CN**: 使用带括号的参数构造或初始化局部对象 `xbits`。
- **L378 EN**: Constructs or initializes local object `ybits` with parenthesized arguments.
  **L378 CN**: 使用带括号的参数构造或初始化局部对象 `ybits`。
- **L379 EN**: Continues the surrounding expression or declaration: `MPFRNumber mpfr_result =`.
  **L379 CN**: 继续构造周围的表达式或声明：`MPFRNumber mpfr_result =`。
- **L380 EN**: Executes a call or declaration centered on `binary_operation_one_output`.
  **L380 CN**: 执行以 `binary_operation_one_output` 为核心的调用或声明。
- **L381 EN**: Constructs or initializes local object `mpfrMatchValue` with parenthesized arguments.
  **L381 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrMatchValue`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L383 EN**: Executes a call or declaration centered on `str`.
  **L383 CN**: 执行以 `str` 为核心的调用或声明。
- **L384 EN**: Executes a call or declaration centered on `str`.
  **L384 CN**: 执行以 `str` 为核心的调用或声明。

### Lines 385-408

````cpp
  tlog << "Second input bits: " << str(FPBits<InputType>(input.y)) << '\n';

  tlog << "Libc result: " << mpfrMatchValue.str() << '\n'
       << "MPFR result: " << mpfr_result.str() << '\n';
  tlog << "Libc floating point result bits: "
       << str(FPBits<OutputType>(libc_result)) << '\n';
  tlog << "              MPFR rounded bits: "
       << str(FPBits<OutputType>(mpfr_result.as<OutputType>())) << '\n';
  tlog << "ULP error: " << mpfr_result.ulp_as_mpfr_number(libc_result).str()
       << '\n';
}

template void
explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,
                                          float, double, RoundingMode);
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<double> &, float, double, RoundingMode);
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<double> &, double, double, RoundingMode);
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<long double> &, float, double, RoundingMode);
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<long double> &, double, double, RoundingMode);
template void
````
- **L385 EN**: Executes a call or declaration centered on `str`.
  **L385 CN**: 执行以 `str` 为核心的调用或声明。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L387 EN**: Continues logic associated with callable symbol `str`.
  **L387 CN**: 继续与可调用符号 `str` 相关的逻辑。
- **L388 EN**: Executes a call or declaration centered on `str`.
  **L388 CN**: 执行以 `str` 为核心的调用或声明。
- **L389 EN**: Continues the surrounding expression or declaration: `tlog << "Libc floating point result bits: "`.
  **L389 CN**: 继续构造周围的表达式或声明：`tlog << "Libc floating point result bits: "`。
- **L390 EN**: Executes a call or declaration centered on `str`.
  **L390 CN**: 执行以 `str` 为核心的调用或声明。
- **L391 EN**: Continues the surrounding expression or declaration: `tlog << "              MPFR rounded bits: "`.
  **L391 CN**: 继续构造周围的表达式或声明：`tlog << "              MPFR rounded bits: "`。
- **L392 EN**: Executes a call or declaration centered on `str`.
  **L392 CN**: 执行以 `str` 为核心的调用或声明。
- **L393 EN**: Continues logic associated with callable symbol `ulp_as_mpfr_number`.
  **L393 CN**: 继续与可调用符号 `ulp_as_mpfr_number` 相关的逻辑。
- **L394 EN**: Executes a standalone statement or declaration: `<< '\n';`.
  **L394 CN**: 执行一条独立语句或声明：`<< '\n';`。
- **L395 EN**: Closes the current lexical scope or compound statement.
  **L395 CN**: 结束当前词法作用域或复合语句块。
- **L396 EN**: Blank line separating nearby declarations or logic blocks.
  **L396 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L397 EN**: Introduces template parameters or specialization context: `template void`.
  **L397 CN**: 为后续声明引入模板参数或特化上下文：`template void`。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,`。
- **L399 EN**: Executes a standalone statement or declaration: `float, double, RoundingMode);`.
  **L399 CN**: 执行一条独立语句或声明：`float, double, RoundingMode);`。
- **L400 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L400 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L401 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<double> &, float, double, RoundingMode);`.
  **L401 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<double> &, float, double, RoundingMode);`。
- **L402 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L402 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L403 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<double> &, double, double, RoundingMode);`.
  **L403 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<double> &, double, double, RoundingMode);`。
- **L404 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L404 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L405 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<long double> &, float, double, RoundingMode);`.
  **L405 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<long double> &, float, double, RoundingMode);`。
- **L406 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L406 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L407 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<long double> &, double, double, RoundingMode);`.
  **L407 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<long double> &, double, double, RoundingMode);`。
- **L408 EN**: Introduces template parameters or specialization context: `template void`.
  **L408 CN**: 为后续声明引入模板参数或特化上下文：`template void`。

### Lines 409-432

````cpp
explain_binary_operation_one_output_error(Operation,
                                          const BinaryInput<long double> &,
                                          long double, double, RoundingMode);
#ifdef LIBC_TYPES_HAS_FLOAT16
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<float16> &, float16, double, RoundingMode);
template void
explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,
                                          float16, double, RoundingMode);
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<double> &, float16, double, RoundingMode);
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<long double> &, float16, double, RoundingMode);
#endif
#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \
    defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<float128> &, float128, double, RoundingMode);
#endif
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<bfloat16> &, bfloat16, double, RoundingMode);
template void
explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,
                                          bfloat16, double, RoundingMode);
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explain_binary_operation_one_output_error(Operation,`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`explain_binary_operation_one_output_error(Operation,`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<long double> &,`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<long double> &,`。
- **L411 EN**: Executes a standalone statement or declaration: `long double, double, RoundingMode);`.
  **L411 CN**: 执行一条独立语句或声明：`long double, double, RoundingMode);`。
- **L412 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L412 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L413 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L413 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L414 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<float16> &, float16, double, RoundingMode);`.
  **L414 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<float16> &, float16, double, RoundingMode);`。
- **L415 EN**: Introduces template parameters or specialization context: `template void`.
  **L415 CN**: 为后续声明引入模板参数或特化上下文：`template void`。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,`。
- **L417 EN**: Executes a standalone statement or declaration: `float16, double, RoundingMode);`.
  **L417 CN**: 执行一条独立语句或声明：`float16, double, RoundingMode);`。
- **L418 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L418 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L419 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<double> &, float16, double, RoundingMode);`.
  **L419 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<double> &, float16, double, RoundingMode);`。
- **L420 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L420 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L421 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<long double> &, float16, double, RoundingMode);`.
  **L421 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<long double> &, float16, double, RoundingMode);`。
- **L422 EN**: Closes the current preprocessor conditional block.
  **L422 CN**: 结束当前的预处理条件块。
- **L423 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`.
  **L423 CN**: 开始一个预处理条件块：`#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`。
- **L424 EN**: Continues logic associated with callable symbol `defined`.
  **L424 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L425 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L425 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L426 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<float128> &, float128, double, RoundingMode);`.
  **L426 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<float128> &, float128, double, RoundingMode);`。
- **L427 EN**: Closes the current preprocessor conditional block.
  **L427 CN**: 结束当前的预处理条件块。
- **L428 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L428 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L429 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<bfloat16> &, bfloat16, double, RoundingMode);`.
  **L429 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<bfloat16> &, bfloat16, double, RoundingMode);`。
- **L430 EN**: Introduces template parameters or specialization context: `template void`.
  **L430 CN**: 为后续声明引入模板参数或特化上下文：`template void`。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`explain_binary_operation_one_output_error(Operation, const BinaryInput<float> &,`。
- **L432 EN**: Executes a standalone statement or declaration: `bfloat16, double, RoundingMode);`.
  **L432 CN**: 执行一条独立语句或声明：`bfloat16, double, RoundingMode);`。

### Lines 433-456

````cpp
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<double> &, bfloat16, double, RoundingMode);
template void
explain_binary_operation_one_output_error(Operation,
                                          const BinaryInput<long double> &,
                                          bfloat16, double, RoundingMode);
#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \
    defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)
template void explain_binary_operation_one_output_error(
    Operation, const BinaryInput<float128> &, bfloat16, double, RoundingMode);
#endif // defined(LIBC_TYPES_HAS_FLOAT128) &&
       // defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)

template <typename InputType, typename OutputType>
void explain_ternary_operation_one_output_error(
    Operation op, const TernaryInput<InputType> &input, OutputType libc_result,
    double ulp_tolerance, RoundingMode rounding) {
  unsigned int precision = get_precision<InputType>(ulp_tolerance);
  MPFRNumber mpfrX(input.x, precision);
  MPFRNumber mpfrY(input.y, precision);
  MPFRNumber mpfrZ(input.z, precision);
  FPBits<InputType> xbits(input.x);
  FPBits<InputType> ybits(input.y);
  FPBits<InputType> zbits(input.z);
````
- **L433 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L433 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L434 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<double> &, bfloat16, double, RoundingMode);`.
  **L434 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<double> &, bfloat16, double, RoundingMode);`。
- **L435 EN**: Introduces template parameters or specialization context: `template void`.
  **L435 CN**: 为后续声明引入模板参数或特化上下文：`template void`。
- **L436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explain_binary_operation_one_output_error(Operation,`.
  **L436 CN**: 继续一个多行参数列表、初始化器或聚合项：`explain_binary_operation_one_output_error(Operation,`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<long double> &,`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<long double> &,`。
- **L438 EN**: Executes a standalone statement or declaration: `bfloat16, double, RoundingMode);`.
  **L438 CN**: 执行一条独立语句或声明：`bfloat16, double, RoundingMode);`。
- **L439 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`.
  **L439 CN**: 开始一个预处理条件块：`#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`。
- **L440 EN**: Continues logic associated with callable symbol `defined`.
  **L440 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L441 EN**: Introduces template parameters or specialization context: `template void explain_binary_operation_one_output_error(`.
  **L441 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_binary_operation_one_output_error(`。
- **L442 EN**: Executes a standalone statement or declaration: `Operation, const BinaryInput<float128> &, bfloat16, double, RoundingMode);`.
  **L442 CN**: 执行一条独立语句或声明：`Operation, const BinaryInput<float128> &, bfloat16, double, RoundingMode);`。
- **L443 EN**: Closes the current preprocessor conditional block.
  **L443 CN**: 结束当前的预处理条件块。
- **L444 EN**: Comment explains nearby logic, invariants, or intent: `defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)`.
  **L444 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)`。
- **L445 EN**: Blank line separating nearby declarations or logic blocks.
  **L445 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L446 EN**: Introduces template parameters or specialization context: `template <typename InputType, typename OutputType>`.
  **L446 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType, typename OutputType>`。
- **L447 EN**: Continues logic associated with callable symbol `explain_ternary_operation_one_output_error`.
  **L447 CN**: 继续与可调用符号 `explain_ternary_operation_one_output_error` 相关的逻辑。
- **L448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation op, const TernaryInput<InputType> &input, OutputType libc_result,`.
  **L448 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation op, const TernaryInput<InputType> &input, OutputType libc_result,`。
- **L449 EN**: Continues the surrounding expression or declaration: `double ulp_tolerance, RoundingMode rounding) {`.
  **L449 CN**: 继续构造周围的表达式或声明：`double ulp_tolerance, RoundingMode rounding) {`。
- **L450 EN**: Initializes variable `precision` from the right-hand expression.
  **L450 CN**: 使用右侧表达式初始化变量 `precision`。
- **L451 EN**: Constructs or initializes local object `mpfrX` with parenthesized arguments.
  **L451 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrX`。
- **L452 EN**: Constructs or initializes local object `mpfrY` with parenthesized arguments.
  **L452 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrY`。
- **L453 EN**: Constructs or initializes local object `mpfrZ` with parenthesized arguments.
  **L453 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrZ`。
- **L454 EN**: Constructs or initializes local object `xbits` with parenthesized arguments.
  **L454 CN**: 使用带括号的参数构造或初始化局部对象 `xbits`。
- **L455 EN**: Constructs or initializes local object `ybits` with parenthesized arguments.
  **L455 CN**: 使用带括号的参数构造或初始化局部对象 `ybits`。
- **L456 EN**: Constructs or initializes local object `zbits` with parenthesized arguments.
  **L456 CN**: 使用带括号的参数构造或初始化局部对象 `zbits`。

### Lines 457-480

````cpp
  MPFRNumber mpfr_result = ternary_operation_one_output(
      op, input.x, input.y, input.z, precision, rounding);
  MPFRNumber mpfrMatchValue(libc_result);

  tlog << "Input decimal: x: " << mpfrX.str() << " y: " << mpfrY.str()
       << " z: " << mpfrZ.str() << '\n';
  tlog << " First input bits: " << str(FPBits<InputType>(input.x)) << '\n';
  tlog << "Second input bits: " << str(FPBits<InputType>(input.y)) << '\n';
  tlog << " Third input bits: " << str(FPBits<InputType>(input.z)) << '\n';

  tlog << "Libc result: " << mpfrMatchValue.str() << '\n'
       << "MPFR result: " << mpfr_result.str() << '\n';
  tlog << "Libc floating point result bits: "
       << str(FPBits<OutputType>(libc_result)) << '\n';
  tlog << "              MPFR rounded bits: "
       << str(FPBits<OutputType>(mpfr_result.as<OutputType>())) << '\n';
  tlog << "ULP error: " << mpfr_result.ulp_as_mpfr_number(libc_result).str()
       << '\n';
}

template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<float> &, float, double, RoundingMode);
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<double> &, float, double, RoundingMode);
````
- **L457 EN**: Continues logic associated with callable symbol `ternary_operation_one_output`.
  **L457 CN**: 继续与可调用符号 `ternary_operation_one_output` 相关的逻辑。
- **L458 EN**: Executes a standalone statement or declaration: `op, input.x, input.y, input.z, precision, rounding);`.
  **L458 CN**: 执行一条独立语句或声明：`op, input.x, input.y, input.z, precision, rounding);`。
- **L459 EN**: Constructs or initializes local object `mpfrMatchValue` with parenthesized arguments.
  **L459 CN**: 使用带括号的参数构造或初始化局部对象 `mpfrMatchValue`。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L461 EN**: Continues logic associated with callable symbol `str`.
  **L461 CN**: 继续与可调用符号 `str` 相关的逻辑。
- **L462 EN**: Executes a call or declaration centered on `str`.
  **L462 CN**: 执行以 `str` 为核心的调用或声明。
- **L463 EN**: Executes a call or declaration centered on `str`.
  **L463 CN**: 执行以 `str` 为核心的调用或声明。
- **L464 EN**: Executes a call or declaration centered on `str`.
  **L464 CN**: 执行以 `str` 为核心的调用或声明。
- **L465 EN**: Executes a call or declaration centered on `str`.
  **L465 CN**: 执行以 `str` 为核心的调用或声明。
- **L466 EN**: Blank line separating nearby declarations or logic blocks.
  **L466 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L467 EN**: Continues logic associated with callable symbol `str`.
  **L467 CN**: 继续与可调用符号 `str` 相关的逻辑。
- **L468 EN**: Executes a call or declaration centered on `str`.
  **L468 CN**: 执行以 `str` 为核心的调用或声明。
- **L469 EN**: Continues the surrounding expression or declaration: `tlog << "Libc floating point result bits: "`.
  **L469 CN**: 继续构造周围的表达式或声明：`tlog << "Libc floating point result bits: "`。
- **L470 EN**: Executes a call or declaration centered on `str`.
  **L470 CN**: 执行以 `str` 为核心的调用或声明。
- **L471 EN**: Continues the surrounding expression or declaration: `tlog << "              MPFR rounded bits: "`.
  **L471 CN**: 继续构造周围的表达式或声明：`tlog << "              MPFR rounded bits: "`。
- **L472 EN**: Executes a call or declaration centered on `str`.
  **L472 CN**: 执行以 `str` 为核心的调用或声明。
- **L473 EN**: Continues logic associated with callable symbol `ulp_as_mpfr_number`.
  **L473 CN**: 继续与可调用符号 `ulp_as_mpfr_number` 相关的逻辑。
- **L474 EN**: Executes a standalone statement or declaration: `<< '\n';`.
  **L474 CN**: 执行一条独立语句或声明：`<< '\n';`。
- **L475 EN**: Closes the current lexical scope or compound statement.
  **L475 CN**: 结束当前词法作用域或复合语句块。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L477 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L477 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L478 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<float> &, float, double, RoundingMode);`.
  **L478 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<float> &, float, double, RoundingMode);`。
- **L479 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L479 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L480 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<double> &, float, double, RoundingMode);`.
  **L480 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<double> &, float, double, RoundingMode);`。

### Lines 481-504

````cpp
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<double> &, double, double, RoundingMode);
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<long double> &, float, double, RoundingMode);
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<long double> &, double, double, RoundingMode);
template void
explain_ternary_operation_one_output_error(Operation,
                                           const TernaryInput<long double> &,
                                           long double, double, RoundingMode);

#ifdef LIBC_TYPES_HAS_FLOAT16
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<float16> &, float16, double, RoundingMode);
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<float> &, float16, double, RoundingMode);
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<double> &, float16, double, RoundingMode);
template void
explain_ternary_operation_one_output_error(Operation,
                                           const TernaryInput<long double> &,
                                           float16, double, RoundingMode);
#endif

````
- **L481 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L481 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L482 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<double> &, double, double, RoundingMode);`.
  **L482 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<double> &, double, double, RoundingMode);`。
- **L483 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L483 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L484 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<long double> &, float, double, RoundingMode);`.
  **L484 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<long double> &, float, double, RoundingMode);`。
- **L485 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L485 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L486 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<long double> &, double, double, RoundingMode);`.
  **L486 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<long double> &, double, double, RoundingMode);`。
- **L487 EN**: Introduces template parameters or specialization context: `template void`.
  **L487 CN**: 为后续声明引入模板参数或特化上下文：`template void`。
- **L488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explain_ternary_operation_one_output_error(Operation,`.
  **L488 CN**: 继续一个多行参数列表、初始化器或聚合项：`explain_ternary_operation_one_output_error(Operation,`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<long double> &,`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<long double> &,`。
- **L490 EN**: Executes a standalone statement or declaration: `long double, double, RoundingMode);`.
  **L490 CN**: 执行一条独立语句或声明：`long double, double, RoundingMode);`。
- **L491 EN**: Blank line separating nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L492 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L492 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L493 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L493 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L494 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<float16> &, float16, double, RoundingMode);`.
  **L494 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<float16> &, float16, double, RoundingMode);`。
- **L495 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L495 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L496 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<float> &, float16, double, RoundingMode);`.
  **L496 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<float> &, float16, double, RoundingMode);`。
- **L497 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L497 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L498 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<double> &, float16, double, RoundingMode);`.
  **L498 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<double> &, float16, double, RoundingMode);`。
- **L499 EN**: Introduces template parameters or specialization context: `template void`.
  **L499 CN**: 为后续声明引入模板参数或特化上下文：`template void`。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explain_ternary_operation_one_output_error(Operation,`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`explain_ternary_operation_one_output_error(Operation,`。
- **L501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<long double> &,`.
  **L501 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<long double> &,`。
- **L502 EN**: Executes a standalone statement or declaration: `float16, double, RoundingMode);`.
  **L502 CN**: 执行一条独立语句或声明：`float16, double, RoundingMode);`。
- **L503 EN**: Closes the current preprocessor conditional block.
  **L503 CN**: 结束当前的预处理条件块。
- **L504 EN**: Blank line separating nearby declarations or logic blocks.
  **L504 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 505-528

````cpp
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<bfloat16> &, bfloat16, double, RoundingMode);
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<float> &, bfloat16, double, RoundingMode);
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<double> &, bfloat16, double, RoundingMode);
template void
explain_ternary_operation_one_output_error(Operation,
                                           const TernaryInput<long double> &,
                                           bfloat16, double, RoundingMode);
#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \
    defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)
template void explain_ternary_operation_one_output_error(
    Operation, const TernaryInput<float128> &, bfloat16, double, RoundingMode);
#endif // defined(LIBC_TYPES_HAS_FLOAT128) &&
       // defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)

template <typename InputType, typename OutputType>
bool compare_unary_operation_single_output(Operation op, InputType input,
                                           OutputType libc_result,
                                           double ulp_tolerance,
                                           RoundingMode rounding) {
  unsigned int precision = get_precision<InputType>(ulp_tolerance);
  MPFRNumber mpfr_result;
````
- **L505 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L505 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L506 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<bfloat16> &, bfloat16, double, RoundingMode);`.
  **L506 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<bfloat16> &, bfloat16, double, RoundingMode);`。
- **L507 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L507 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L508 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<float> &, bfloat16, double, RoundingMode);`.
  **L508 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<float> &, bfloat16, double, RoundingMode);`。
- **L509 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L509 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L510 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<double> &, bfloat16, double, RoundingMode);`.
  **L510 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<double> &, bfloat16, double, RoundingMode);`。
- **L511 EN**: Introduces template parameters or specialization context: `template void`.
  **L511 CN**: 为后续声明引入模板参数或特化上下文：`template void`。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explain_ternary_operation_one_output_error(Operation,`.
  **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`explain_ternary_operation_one_output_error(Operation,`。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<long double> &,`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<long double> &,`。
- **L514 EN**: Executes a standalone statement or declaration: `bfloat16, double, RoundingMode);`.
  **L514 CN**: 执行一条独立语句或声明：`bfloat16, double, RoundingMode);`。
- **L515 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`.
  **L515 CN**: 开始一个预处理条件块：`#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`。
- **L516 EN**: Continues logic associated with callable symbol `defined`.
  **L516 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L517 EN**: Introduces template parameters or specialization context: `template void explain_ternary_operation_one_output_error(`.
  **L517 CN**: 为后续声明引入模板参数或特化上下文：`template void explain_ternary_operation_one_output_error(`。
- **L518 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<float128> &, bfloat16, double, RoundingMode);`.
  **L518 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<float128> &, bfloat16, double, RoundingMode);`。
- **L519 EN**: Closes the current preprocessor conditional block.
  **L519 CN**: 结束当前的预处理条件块。
- **L520 EN**: Comment explains nearby logic, invariants, or intent: `defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)`.
  **L520 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)`。
- **L521 EN**: Blank line separating nearby declarations or logic blocks.
  **L521 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L522 EN**: Introduces template parameters or specialization context: `template <typename InputType, typename OutputType>`.
  **L522 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType, typename OutputType>`。
- **L523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool compare_unary_operation_single_output(Operation op, InputType input,`.
  **L523 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool compare_unary_operation_single_output(Operation op, InputType input,`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputType libc_result,`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputType libc_result,`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double ulp_tolerance,`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`double ulp_tolerance,`。
- **L526 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L526 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L527 EN**: Initializes variable `precision` from the right-hand expression.
  **L527 CN**: 使用右侧表达式初始化变量 `precision`。
- **L528 EN**: Executes a standalone statement or declaration: `MPFRNumber mpfr_result;`.
  **L528 CN**: 执行一条独立语句或声明：`MPFRNumber mpfr_result;`。

### Lines 529-552

````cpp
  mpfr_result = unary_operation(op, input, precision, rounding);
  double ulp = mpfr_result.ulp(libc_result);
  return (ulp <= ulp_tolerance);
}

template bool compare_unary_operation_single_output(Operation, float, float,
                                                    double, RoundingMode);
template bool compare_unary_operation_single_output(Operation, double, double,
                                                    double, RoundingMode);
template bool compare_unary_operation_single_output(Operation, long double,
                                                    long double, double,
                                                    RoundingMode);
template bool compare_unary_operation_single_output(Operation, double, float,
                                                    double, RoundingMode);
template bool compare_unary_operation_single_output(Operation, long double,
                                                    float, double,
                                                    RoundingMode);
template bool compare_unary_operation_single_output(Operation, long double,
                                                    double, double,
                                                    RoundingMode);
#ifdef LIBC_TYPES_HAS_FLOAT16
template bool compare_unary_operation_single_output(Operation, float16, float16,
                                                    double, RoundingMode);
template bool compare_unary_operation_single_output(Operation, float, float16,
````
- **L529 EN**: Executes a call or declaration centered on `unary_operation`.
  **L529 CN**: 执行以 `unary_operation` 为核心的调用或声明。
- **L530 EN**: Initializes variable `ulp` from the right-hand expression.
  **L530 CN**: 使用右侧表达式初始化变量 `ulp`。
- **L531 EN**: Returns from the current function with `(ulp <= ulp_tolerance)`.
  **L531 CN**: 以 `(ulp <= ulp_tolerance)` 从当前函数返回。
- **L532 EN**: Closes the current lexical scope or compound statement.
  **L532 CN**: 结束当前词法作用域或复合语句块。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L534 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, float, float,`.
  **L534 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, float, float,`。
- **L535 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L535 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L536 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, double, double,`.
  **L536 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, double, double,`。
- **L537 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L537 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L538 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, long double,`.
  **L538 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, long double,`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `long double, double,`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`long double, double,`。
- **L540 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L540 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L541 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, double, float,`.
  **L541 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, double, float,`。
- **L542 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L542 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L543 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, long double,`.
  **L543 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, long double,`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float, double,`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`float, double,`。
- **L545 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L545 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L546 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, long double,`.
  **L546 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, long double,`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double, double,`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`double, double,`。
- **L548 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L548 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L549 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L549 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L550 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, float16, float16,`.
  **L550 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, float16, float16,`。
- **L551 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L551 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L552 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, float, float16,`.
  **L552 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, float, float16,`。

### Lines 553-576

````cpp
                                                    double, RoundingMode);
template bool compare_unary_operation_single_output(Operation, double, float16,
                                                    double, RoundingMode);
template bool compare_unary_operation_single_output(Operation, long double,
                                                    float16, double,
                                                    RoundingMode);
#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
template bool compare_unary_operation_single_output(Operation, float128,
                                                    float16, double,
                                                    RoundingMode);
#endif // LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
#endif // LIBC_TYPES_HAS_FLOAT16

#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
template bool compare_unary_operation_single_output(Operation, float128,
                                                    float128, double,
                                                    RoundingMode);
template bool compare_unary_operation_single_output(Operation, float128, float,
                                                    double, RoundingMode);
template bool compare_unary_operation_single_output(Operation, float128, double,
                                                    double, RoundingMode);
template bool compare_unary_operation_single_output(Operation, float128,
                                                    long double, double,
                                                    RoundingMode);
````
- **L553 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L553 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L554 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, double, float16,`.
  **L554 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, double, float16,`。
- **L555 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L555 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L556 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, long double,`.
  **L556 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, long double,`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L558 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L558 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L559 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`.
  **L559 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`。
- **L560 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, float128,`.
  **L560 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, float128,`。
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L562 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L562 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L563 EN**: Closes the current preprocessor conditional block.
  **L563 CN**: 结束当前的预处理条件块。
- **L564 EN**: Closes the current preprocessor conditional block.
  **L564 CN**: 结束当前的预处理条件块。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L566 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`.
  **L566 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`。
- **L567 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, float128,`.
  **L567 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, float128,`。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float128, double,`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`float128, double,`。
- **L569 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L569 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L570 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, float128, float,`.
  **L570 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, float128, float,`。
- **L571 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L571 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L572 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, float128, double,`.
  **L572 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, float128, double,`。
- **L573 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L573 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L574 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, float128,`.
  **L574 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, float128,`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `long double, double,`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`long double, double,`。
- **L576 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L576 CN**: 执行一条独立语句或声明：`RoundingMode);`。

### Lines 577-600

````cpp
#endif // LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
template bool compare_unary_operation_single_output(Operation, bfloat16,
                                                    bfloat16, double,
                                                    RoundingMode);

template <typename T>
bool compare_unary_operation_two_outputs(Operation op, T input,
                                         const BinaryOutput<T> &libc_result,
                                         double ulp_tolerance,
                                         RoundingMode rounding) {
  int mpfrIntResult;
  unsigned int precision = get_precision<T>(ulp_tolerance);
  MPFRNumber mpfr_result = unary_operation_two_outputs(op, input, mpfrIntResult,
                                                       precision, rounding);
  double ulp = mpfr_result.ulp(libc_result.f);

  if (mpfrIntResult != libc_result.i)
    return false;

  return (ulp <= ulp_tolerance);
}

template bool compare_unary_operation_two_outputs<float>(
    Operation, float, const BinaryOutput<float> &, double, RoundingMode);
````
- **L577 EN**: Closes the current preprocessor conditional block.
  **L577 CN**: 结束当前的预处理条件块。
- **L578 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_single_output(Operation, bfloat16,`.
  **L578 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_single_output(Operation, bfloat16,`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bfloat16, double,`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`bfloat16, double,`。
- **L580 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L580 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L582 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  **L582 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool compare_unary_operation_two_outputs(Operation op, T input,`.
  **L583 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool compare_unary_operation_two_outputs(Operation op, T input,`。
- **L584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryOutput<T> &libc_result,`.
  **L584 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryOutput<T> &libc_result,`。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double ulp_tolerance,`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`double ulp_tolerance,`。
- **L586 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L586 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L587 EN**: Executes a standalone statement or declaration: `int mpfrIntResult;`.
  **L587 CN**: 执行一条独立语句或声明：`int mpfrIntResult;`。
- **L588 EN**: Initializes variable `precision` from the right-hand expression.
  **L588 CN**: 使用右侧表达式初始化变量 `precision`。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MPFRNumber mpfr_result = unary_operation_two_outputs(op, input, mpfrIntResult,`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`MPFRNumber mpfr_result = unary_operation_two_outputs(op, input, mpfrIntResult,`。
- **L590 EN**: Executes a standalone statement or declaration: `precision, rounding);`.
  **L590 CN**: 执行一条独立语句或声明：`precision, rounding);`。
- **L591 EN**: Initializes variable `ulp` from the right-hand expression.
  **L591 CN**: 使用右侧表达式初始化变量 `ulp`。
- **L592 EN**: Blank line separating nearby declarations or logic blocks.
  **L592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L593 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L593 CN**: 开始 `if` 控制流语句并计算其条件。
- **L594 EN**: Returns from the current function with `false`.
  **L594 CN**: 以 `false` 从当前函数返回。
- **L595 EN**: Blank line separating nearby declarations or logic blocks.
  **L595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L596 EN**: Returns from the current function with `(ulp <= ulp_tolerance)`.
  **L596 CN**: 以 `(ulp <= ulp_tolerance)` 从当前函数返回。
- **L597 EN**: Closes the current lexical scope or compound statement.
  **L597 CN**: 结束当前词法作用域或复合语句块。
- **L598 EN**: Blank line separating nearby declarations or logic blocks.
  **L598 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L599 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_two_outputs<float>(`.
  **L599 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_two_outputs<float>(`。
- **L600 EN**: Executes a standalone statement or declaration: `Operation, float, const BinaryOutput<float> &, double, RoundingMode);`.
  **L600 CN**: 执行一条独立语句或声明：`Operation, float, const BinaryOutput<float> &, double, RoundingMode);`。

### Lines 601-624

````cpp
template bool compare_unary_operation_two_outputs<double>(
    Operation, double, const BinaryOutput<double> &, double, RoundingMode);
template bool compare_unary_operation_two_outputs<long double>(
    Operation, long double, const BinaryOutput<long double> &, double,
    RoundingMode);

template <typename T>
bool compare_binary_operation_two_outputs(Operation op,
                                          const BinaryInput<T> &input,
                                          const BinaryOutput<T> &libc_result,
                                          double ulp_tolerance,
                                          RoundingMode rounding) {
  int mpfrIntResult;
  unsigned int precision = get_precision<T>(ulp_tolerance);
  MPFRNumber mpfr_result = binary_operation_two_outputs(
      op, input.x, input.y, mpfrIntResult, precision, rounding);
  double ulp = mpfr_result.ulp(libc_result.f);

  if (mpfrIntResult != libc_result.i) {
    if (op == Operation::RemQuo) {
      if ((0x7 & mpfrIntResult) != (0x7 & libc_result.i))
        return false;
    } else {
      return false;
````
- **L601 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_two_outputs<double>(`.
  **L601 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_two_outputs<double>(`。
- **L602 EN**: Executes a standalone statement or declaration: `Operation, double, const BinaryOutput<double> &, double, RoundingMode);`.
  **L602 CN**: 执行一条独立语句或声明：`Operation, double, const BinaryOutput<double> &, double, RoundingMode);`。
- **L603 EN**: Introduces template parameters or specialization context: `template bool compare_unary_operation_two_outputs<long double>(`.
  **L603 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_unary_operation_two_outputs<long double>(`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation, long double, const BinaryOutput<long double> &, double,`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation, long double, const BinaryOutput<long double> &, double,`。
- **L605 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L605 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L606 EN**: Blank line separating nearby declarations or logic blocks.
  **L606 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L607 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  **L607 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool compare_binary_operation_two_outputs(Operation op,`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool compare_binary_operation_two_outputs(Operation op,`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<T> &input,`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<T> &input,`。
- **L610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryOutput<T> &libc_result,`.
  **L610 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryOutput<T> &libc_result,`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double ulp_tolerance,`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`double ulp_tolerance,`。
- **L612 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L612 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L613 EN**: Executes a standalone statement or declaration: `int mpfrIntResult;`.
  **L613 CN**: 执行一条独立语句或声明：`int mpfrIntResult;`。
- **L614 EN**: Initializes variable `precision` from the right-hand expression.
  **L614 CN**: 使用右侧表达式初始化变量 `precision`。
- **L615 EN**: Continues logic associated with callable symbol `binary_operation_two_outputs`.
  **L615 CN**: 继续与可调用符号 `binary_operation_two_outputs` 相关的逻辑。
- **L616 EN**: Executes a standalone statement or declaration: `op, input.x, input.y, mpfrIntResult, precision, rounding);`.
  **L616 CN**: 执行一条独立语句或声明：`op, input.x, input.y, mpfrIntResult, precision, rounding);`。
- **L617 EN**: Initializes variable `ulp` from the right-hand expression.
  **L617 CN**: 使用右侧表达式初始化变量 `ulp`。
- **L618 EN**: Blank line separating nearby declarations or logic blocks.
  **L618 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L619 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L619 CN**: 开始 `if` 控制流语句并计算其条件。
- **L620 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L620 CN**: 开始 `if` 控制流语句并计算其条件。
- **L621 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L621 CN**: 开始 `if` 控制流语句并计算其条件。
- **L622 EN**: Returns from the current function with `false`.
  **L622 CN**: 以 `false` 从当前函数返回。
- **L623 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L623 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L624 EN**: Returns from the current function with `false`.
  **L624 CN**: 以 `false` 从当前函数返回。

### Lines 625-648

````cpp
    }
  }

  return (ulp <= ulp_tolerance);
}

template bool compare_binary_operation_two_outputs<float>(
    Operation, const BinaryInput<float> &, const BinaryOutput<float> &, double,
    RoundingMode);
template bool compare_binary_operation_two_outputs<double>(
    Operation, const BinaryInput<double> &, const BinaryOutput<double> &,
    double, RoundingMode);
template bool compare_binary_operation_two_outputs<long double>(
    Operation, const BinaryInput<long double> &,
    const BinaryOutput<long double> &, double, RoundingMode);

template <typename InputType, typename OutputType>
bool compare_binary_operation_one_output(Operation op,
                                         const BinaryInput<InputType> &input,
                                         OutputType libc_result,
                                         double ulp_tolerance,
                                         RoundingMode rounding) {
  unsigned int precision = get_precision<InputType>(ulp_tolerance);
  MPFRNumber mpfr_result =
````
- **L625 EN**: Closes the current lexical scope or compound statement.
  **L625 CN**: 结束当前词法作用域或复合语句块。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Blank line separating nearby declarations or logic blocks.
  **L627 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L628 EN**: Returns from the current function with `(ulp <= ulp_tolerance)`.
  **L628 CN**: 以 `(ulp <= ulp_tolerance)` 从当前函数返回。
- **L629 EN**: Closes the current lexical scope or compound statement.
  **L629 CN**: 结束当前词法作用域或复合语句块。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L631 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_two_outputs<float>(`.
  **L631 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_two_outputs<float>(`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation, const BinaryInput<float> &, const BinaryOutput<float> &, double,`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation, const BinaryInput<float> &, const BinaryOutput<float> &, double,`。
- **L633 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L633 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L634 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_two_outputs<double>(`.
  **L634 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_two_outputs<double>(`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation, const BinaryInput<double> &, const BinaryOutput<double> &,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation, const BinaryInput<double> &, const BinaryOutput<double> &,`。
- **L636 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L636 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L637 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_two_outputs<long double>(`.
  **L637 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_two_outputs<long double>(`。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation, const BinaryInput<long double> &,`.
  **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation, const BinaryInput<long double> &,`。
- **L639 EN**: Executes a standalone statement or declaration: `const BinaryOutput<long double> &, double, RoundingMode);`.
  **L639 CN**: 执行一条独立语句或声明：`const BinaryOutput<long double> &, double, RoundingMode);`。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L641 EN**: Introduces template parameters or specialization context: `template <typename InputType, typename OutputType>`.
  **L641 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType, typename OutputType>`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool compare_binary_operation_one_output(Operation op,`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool compare_binary_operation_one_output(Operation op,`。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<InputType> &input,`.
  **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<InputType> &input,`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputType libc_result,`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputType libc_result,`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double ulp_tolerance,`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`double ulp_tolerance,`。
- **L646 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L646 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L647 EN**: Initializes variable `precision` from the right-hand expression.
  **L647 CN**: 使用右侧表达式初始化变量 `precision`。
- **L648 EN**: Continues the surrounding expression or declaration: `MPFRNumber mpfr_result =`.
  **L648 CN**: 继续构造周围的表达式或声明：`MPFRNumber mpfr_result =`。

### Lines 649-672

````cpp
      binary_operation_one_output(op, input.x, input.y, precision, rounding);
  double ulp = mpfr_result.ulp(libc_result);

  return (ulp <= ulp_tolerance);
}

template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<float> &,
                                                  float, double, RoundingMode);
template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<double> &,
                                                  double, double, RoundingMode);
template bool
compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,
                                    float, double, RoundingMode);
template bool
compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,
                                    double, double, RoundingMode);
template bool
compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,
                                    long double, double, RoundingMode);

template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<double> &,
````
- **L649 EN**: Executes a call or declaration centered on `binary_operation_one_output`.
  **L649 CN**: 执行以 `binary_operation_one_output` 为核心的调用或声明。
- **L650 EN**: Initializes variable `ulp` from the right-hand expression.
  **L650 CN**: 使用右侧表达式初始化变量 `ulp`。
- **L651 EN**: Blank line separating nearby declarations or logic blocks.
  **L651 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L652 EN**: Returns from the current function with `(ulp <= ulp_tolerance)`.
  **L652 CN**: 以 `(ulp <= ulp_tolerance)` 从当前函数返回。
- **L653 EN**: Closes the current lexical scope or compound statement.
  **L653 CN**: 结束当前词法作用域或复合语句块。
- **L654 EN**: Blank line separating nearby declarations or logic blocks.
  **L654 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L655 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L655 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<float> &,`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<float> &,`。
- **L657 EN**: Executes a standalone statement or declaration: `float, double, RoundingMode);`.
  **L657 CN**: 执行一条独立语句或声明：`float, double, RoundingMode);`。
- **L658 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L658 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<double> &,`.
  **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<double> &,`。
- **L660 EN**: Executes a standalone statement or declaration: `double, double, RoundingMode);`.
  **L660 CN**: 执行一条独立语句或声明：`double, double, RoundingMode);`。
- **L661 EN**: Introduces template parameters or specialization context: `template bool`.
  **L661 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`.
  **L662 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`。
- **L663 EN**: Executes a standalone statement or declaration: `float, double, RoundingMode);`.
  **L663 CN**: 执行一条独立语句或声明：`float, double, RoundingMode);`。
- **L664 EN**: Introduces template parameters or specialization context: `template bool`.
  **L664 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`。
- **L666 EN**: Executes a standalone statement or declaration: `double, double, RoundingMode);`.
  **L666 CN**: 执行一条独立语句或声明：`double, double, RoundingMode);`。
- **L667 EN**: Introduces template parameters or specialization context: `template bool`.
  **L667 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`.
  **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`。
- **L669 EN**: Executes a standalone statement or declaration: `long double, double, RoundingMode);`.
  **L669 CN**: 执行一条独立语句或声明：`long double, double, RoundingMode);`。
- **L670 EN**: Blank line separating nearby declarations or logic blocks.
  **L670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L671 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L671 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<double> &,`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<double> &,`。

### Lines 673-696

````cpp
                                                  float, double, RoundingMode);
#ifdef LIBC_TYPES_HAS_FLOAT16
template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<float16> &,
                                                  float16, double,
                                                  RoundingMode);
template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<float> &,
                                                  float16, double,
                                                  RoundingMode);
template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<double> &,
                                                  float16, double,
                                                  RoundingMode);
template bool
compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,
                                    float16, double, RoundingMode);
#endif
#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \
    defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)
template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<float128> &,
                                                  float128, double,
                                                  RoundingMode);
````
- **L673 EN**: Executes a standalone statement or declaration: `float, double, RoundingMode);`.
  **L673 CN**: 执行一条独立语句或声明：`float, double, RoundingMode);`。
- **L674 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L674 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L675 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L675 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<float16> &,`.
  **L676 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<float16> &,`。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L678 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L678 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L679 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L679 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<float> &,`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<float> &,`。
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L682 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L682 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L683 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L683 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<double> &,`.
  **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<double> &,`。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L686 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L686 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L687 EN**: Introduces template parameters or specialization context: `template bool`.
  **L687 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`。
- **L689 EN**: Executes a standalone statement or declaration: `float16, double, RoundingMode);`.
  **L689 CN**: 执行一条独立语句或声明：`float16, double, RoundingMode);`。
- **L690 EN**: Closes the current preprocessor conditional block.
  **L690 CN**: 结束当前的预处理条件块。
- **L691 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`.
  **L691 CN**: 开始一个预处理条件块：`#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`。
- **L692 EN**: Continues logic associated with callable symbol `defined`.
  **L692 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L693 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L693 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<float128> &,`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<float128> &,`。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float128, double,`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`float128, double,`。
- **L696 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L696 CN**: 执行一条独立语句或声明：`RoundingMode);`。

### Lines 697-720

````cpp
#endif
template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<bfloat16> &,
                                                  bfloat16, double,
                                                  RoundingMode);

template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<float> &,
                                                  bfloat16, double,
                                                  RoundingMode);
template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<double> &,
                                                  bfloat16, double,
                                                  RoundingMode);
template bool
compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,
                                    bfloat16, double, RoundingMode);
#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \
    defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)
template bool compare_binary_operation_one_output(Operation,
                                                  const BinaryInput<float128> &,
                                                  bfloat16, double,
                                                  RoundingMode);
#endif // defined(LIBC_TYPES_HAS_FLOAT128) &&
````
- **L697 EN**: Closes the current preprocessor conditional block.
  **L697 CN**: 结束当前的预处理条件块。
- **L698 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L698 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<bfloat16> &,`.
  **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<bfloat16> &,`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bfloat16, double,`.
  **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`bfloat16, double,`。
- **L701 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L701 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L702 EN**: Blank line separating nearby declarations or logic blocks.
  **L702 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L703 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L703 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<float> &,`.
  **L704 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<float> &,`。
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bfloat16, double,`.
  **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`bfloat16, double,`。
- **L706 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L706 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L707 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L707 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<double> &,`.
  **L708 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<double> &,`。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bfloat16, double,`.
  **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`bfloat16, double,`。
- **L710 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L710 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L711 EN**: Introduces template parameters or specialization context: `template bool`.
  **L711 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`.
  **L712 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_binary_operation_one_output(Operation, const BinaryInput<long double> &,`。
- **L713 EN**: Executes a standalone statement or declaration: `bfloat16, double, RoundingMode);`.
  **L713 CN**: 执行一条独立语句或声明：`bfloat16, double, RoundingMode);`。
- **L714 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`.
  **L714 CN**: 开始一个预处理条件块：`#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`。
- **L715 EN**: Continues logic associated with callable symbol `defined`.
  **L715 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L716 EN**: Introduces template parameters or specialization context: `template bool compare_binary_operation_one_output(Operation,`.
  **L716 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_binary_operation_one_output(Operation,`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const BinaryInput<float128> &,`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`const BinaryInput<float128> &,`。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bfloat16, double,`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`bfloat16, double,`。
- **L719 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L719 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L720 EN**: Closes the current preprocessor conditional block.
  **L720 CN**: 结束当前的预处理条件块。

### Lines 721-744

````cpp
       // defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)
template <typename InputType, typename OutputType>
bool compare_ternary_operation_one_output(Operation op,
                                          const TernaryInput<InputType> &input,
                                          OutputType libc_result,
                                          double ulp_tolerance,
                                          RoundingMode rounding) {
  unsigned int precision = get_precision<InputType>(ulp_tolerance);
  MPFRNumber mpfr_result = ternary_operation_one_output(
      op, input.x, input.y, input.z, precision, rounding);
  double ulp = mpfr_result.ulp(libc_result);

  return (ulp <= ulp_tolerance);
}

template bool compare_ternary_operation_one_output(Operation,
                                                   const TernaryInput<float> &,
                                                   float, double, RoundingMode);
template bool compare_ternary_operation_one_output(Operation,
                                                   const TernaryInput<double> &,
                                                   float, double, RoundingMode);
template bool compare_ternary_operation_one_output(Operation,
                                                   const TernaryInput<double> &,
                                                   double, double,
````
- **L721 EN**: Comment explains nearby logic, invariants, or intent: `defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)`.
  **L721 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)`。
- **L722 EN**: Introduces template parameters or specialization context: `template <typename InputType, typename OutputType>`.
  **L722 CN**: 为后续声明引入模板参数或特化上下文：`template <typename InputType, typename OutputType>`。
- **L723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool compare_ternary_operation_one_output(Operation op,`.
  **L723 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool compare_ternary_operation_one_output(Operation op,`。
- **L724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<InputType> &input,`.
  **L724 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<InputType> &input,`。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputType libc_result,`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputType libc_result,`。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double ulp_tolerance,`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`double ulp_tolerance,`。
- **L727 EN**: Continues the surrounding expression or declaration: `RoundingMode rounding) {`.
  **L727 CN**: 继续构造周围的表达式或声明：`RoundingMode rounding) {`。
- **L728 EN**: Initializes variable `precision` from the right-hand expression.
  **L728 CN**: 使用右侧表达式初始化变量 `precision`。
- **L729 EN**: Continues logic associated with callable symbol `ternary_operation_one_output`.
  **L729 CN**: 继续与可调用符号 `ternary_operation_one_output` 相关的逻辑。
- **L730 EN**: Executes a standalone statement or declaration: `op, input.x, input.y, input.z, precision, rounding);`.
  **L730 CN**: 执行一条独立语句或声明：`op, input.x, input.y, input.z, precision, rounding);`。
- **L731 EN**: Initializes variable `ulp` from the right-hand expression.
  **L731 CN**: 使用右侧表达式初始化变量 `ulp`。
- **L732 EN**: Blank line separating nearby declarations or logic blocks.
  **L732 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L733 EN**: Returns from the current function with `(ulp <= ulp_tolerance)`.
  **L733 CN**: 以 `(ulp <= ulp_tolerance)` 从当前函数返回。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L736 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(Operation,`.
  **L736 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(Operation,`。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<float> &,`.
  **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<float> &,`。
- **L738 EN**: Executes a standalone statement or declaration: `float, double, RoundingMode);`.
  **L738 CN**: 执行一条独立语句或声明：`float, double, RoundingMode);`。
- **L739 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(Operation,`.
  **L739 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(Operation,`。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<double> &,`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<double> &,`。
- **L741 EN**: Executes a standalone statement or declaration: `float, double, RoundingMode);`.
  **L741 CN**: 执行一条独立语句或声明：`float, double, RoundingMode);`。
- **L742 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(Operation,`.
  **L742 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(Operation,`。
- **L743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<double> &,`.
  **L743 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<double> &,`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `double, double,`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`double, double,`。

### Lines 745-768

````cpp
                                                   RoundingMode);
template bool compare_ternary_operation_one_output(
    Operation, const TernaryInput<long double> &, float, double, RoundingMode);
template bool compare_ternary_operation_one_output(
    Operation, const TernaryInput<long double> &, double, double, RoundingMode);
template bool
compare_ternary_operation_one_output(Operation,
                                     const TernaryInput<long double> &,
                                     long double, double, RoundingMode);

#ifdef LIBC_TYPES_HAS_FLOAT16
template bool
compare_ternary_operation_one_output(Operation, const TernaryInput<float16> &,
                                     float16, double, RoundingMode);
template bool compare_ternary_operation_one_output(Operation,
                                                   const TernaryInput<float> &,
                                                   float16, double,
                                                   RoundingMode);
template bool compare_ternary_operation_one_output(Operation,
                                                   const TernaryInput<double> &,
                                                   float16, double,
                                                   RoundingMode);
template bool
compare_ternary_operation_one_output(Operation,
````
- **L745 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L745 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L746 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(`.
  **L746 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(`。
- **L747 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<long double> &, float, double, RoundingMode);`.
  **L747 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<long double> &, float, double, RoundingMode);`。
- **L748 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(`.
  **L748 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(`。
- **L749 EN**: Executes a standalone statement or declaration: `Operation, const TernaryInput<long double> &, double, double, RoundingMode);`.
  **L749 CN**: 执行一条独立语句或声明：`Operation, const TernaryInput<long double> &, double, double, RoundingMode);`。
- **L750 EN**: Introduces template parameters or specialization context: `template bool`.
  **L750 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_ternary_operation_one_output(Operation,`.
  **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_ternary_operation_one_output(Operation,`。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<long double> &,`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<long double> &,`。
- **L753 EN**: Executes a standalone statement or declaration: `long double, double, RoundingMode);`.
  **L753 CN**: 执行一条独立语句或声明：`long double, double, RoundingMode);`。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L755 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L755 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L756 EN**: Introduces template parameters or specialization context: `template bool`.
  **L756 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_ternary_operation_one_output(Operation, const TernaryInput<float16> &,`.
  **L757 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_ternary_operation_one_output(Operation, const TernaryInput<float16> &,`。
- **L758 EN**: Executes a standalone statement or declaration: `float16, double, RoundingMode);`.
  **L758 CN**: 执行一条独立语句或声明：`float16, double, RoundingMode);`。
- **L759 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(Operation,`.
  **L759 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(Operation,`。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<float> &,`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<float> &,`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L762 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L762 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L763 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(Operation,`.
  **L763 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(Operation,`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<double> &,`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<double> &,`。
- **L765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float16, double,`.
  **L765 CN**: 继续一个多行参数列表、初始化器或聚合项：`float16, double,`。
- **L766 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L766 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L767 EN**: Introduces template parameters or specialization context: `template bool`.
  **L767 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_ternary_operation_one_output(Operation,`.
  **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_ternary_operation_one_output(Operation,`。

### Lines 769-792

````cpp
                                     const TernaryInput<long double> &, float16,
                                     double, RoundingMode);
#endif

template bool
compare_ternary_operation_one_output(Operation, const TernaryInput<bfloat16> &,
                                     bfloat16, double, RoundingMode);
template bool compare_ternary_operation_one_output(Operation,
                                                   const TernaryInput<float> &,
                                                   bfloat16, double,
                                                   RoundingMode);
template bool compare_ternary_operation_one_output(Operation,
                                                   const TernaryInput<double> &,
                                                   bfloat16, double,
                                                   RoundingMode);
template bool
compare_ternary_operation_one_output(Operation,
                                     const TernaryInput<long double> &,
                                     bfloat16, double, RoundingMode);

#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \
    defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)
template bool
compare_ternary_operation_one_output(Operation, const TernaryInput<float128> &,
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<long double> &, float16,`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<long double> &, float16,`。
- **L770 EN**: Executes a standalone statement or declaration: `double, RoundingMode);`.
  **L770 CN**: 执行一条独立语句或声明：`double, RoundingMode);`。
- **L771 EN**: Closes the current preprocessor conditional block.
  **L771 CN**: 结束当前的预处理条件块。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L773 EN**: Introduces template parameters or specialization context: `template bool`.
  **L773 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_ternary_operation_one_output(Operation, const TernaryInput<bfloat16> &,`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_ternary_operation_one_output(Operation, const TernaryInput<bfloat16> &,`。
- **L775 EN**: Executes a standalone statement or declaration: `bfloat16, double, RoundingMode);`.
  **L775 CN**: 执行一条独立语句或声明：`bfloat16, double, RoundingMode);`。
- **L776 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(Operation,`.
  **L776 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(Operation,`。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<float> &,`.
  **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<float> &,`。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bfloat16, double,`.
  **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`bfloat16, double,`。
- **L779 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L779 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L780 EN**: Introduces template parameters or specialization context: `template bool compare_ternary_operation_one_output(Operation,`.
  **L780 CN**: 为后续声明引入模板参数或特化上下文：`template bool compare_ternary_operation_one_output(Operation,`。
- **L781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<double> &,`.
  **L781 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<double> &,`。
- **L782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bfloat16, double,`.
  **L782 CN**: 继续一个多行参数列表、初始化器或聚合项：`bfloat16, double,`。
- **L783 EN**: Executes a standalone statement or declaration: `RoundingMode);`.
  **L783 CN**: 执行一条独立语句或声明：`RoundingMode);`。
- **L784 EN**: Introduces template parameters or specialization context: `template bool`.
  **L784 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_ternary_operation_one_output(Operation,`.
  **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_ternary_operation_one_output(Operation,`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TernaryInput<long double> &,`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TernaryInput<long double> &,`。
- **L787 EN**: Executes a standalone statement or declaration: `bfloat16, double, RoundingMode);`.
  **L787 CN**: 执行一条独立语句或声明：`bfloat16, double, RoundingMode);`。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L789 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`.
  **L789 CN**: 开始一个预处理条件块：`#if defined(LIBC_TYPES_HAS_FLOAT128) &&                                        \`。
- **L790 EN**: Continues logic associated with callable symbol `defined`.
  **L790 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L791 EN**: Introduces template parameters or specialization context: `template bool`.
  **L791 CN**: 为后续声明引入模板参数或特化上下文：`template bool`。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `compare_ternary_operation_one_output(Operation, const TernaryInput<float128> &,`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`compare_ternary_operation_one_output(Operation, const TernaryInput<float128> &,`。

### Lines 793-816

````cpp
                                     bfloat16, double, RoundingMode);
#endif // defined(LIBC_TYPES_HAS_FLOAT128) &&
       // defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)

} // namespace internal

template <typename T> bool round_to_long(T x, long &result) {
  MPFRNumber mpfr(x);
  return mpfr.round_to_long(result);
}

template bool round_to_long<float>(float, long &);
template bool round_to_long<double>(double, long &);
template bool round_to_long<long double>(long double, long &);

#ifdef LIBC_TYPES_HAS_FLOAT16
template bool round_to_long<float16>(float16, long &);
#endif // LIBC_TYPES_HAS_FLOAT16

#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
template bool round_to_long<float128>(float128, long &);
#endif // LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE

template bool round_to_long<bfloat16>(bfloat16, long &);
````
- **L793 EN**: Executes a standalone statement or declaration: `bfloat16, double, RoundingMode);`.
  **L793 CN**: 执行一条独立语句或声明：`bfloat16, double, RoundingMode);`。
- **L794 EN**: Closes the current preprocessor conditional block.
  **L794 CN**: 结束当前的预处理条件块。
- **L795 EN**: Comment explains nearby logic, invariants, or intent: `defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)`.
  **L795 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`defined(LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE)`。
- **L796 EN**: Blank line separating nearby declarations or logic blocks.
  **L796 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L797 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace internal`.
  **L797 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace internal`。
- **L798 EN**: Blank line separating nearby declarations or logic blocks.
  **L798 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L799 EN**: Introduces template parameters or specialization context: `template <typename T> bool round_to_long(T x, long &result) {`.
  **L799 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool round_to_long(T x, long &result) {`。
- **L800 EN**: Constructs or initializes local object `mpfr` with parenthesized arguments.
  **L800 CN**: 使用带括号的参数构造或初始化局部对象 `mpfr`。
- **L801 EN**: Returns from the current function with `mpfr.round_to_long(result)`.
  **L801 CN**: 以 `mpfr.round_to_long(result)` 从当前函数返回。
- **L802 EN**: Closes the current lexical scope or compound statement.
  **L802 CN**: 结束当前词法作用域或复合语句块。
- **L803 EN**: Blank line separating nearby declarations or logic blocks.
  **L803 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L804 EN**: Introduces template parameters or specialization context: `template bool round_to_long<float>(float, long &);`.
  **L804 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<float>(float, long &);`。
- **L805 EN**: Introduces template parameters or specialization context: `template bool round_to_long<double>(double, long &);`.
  **L805 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<double>(double, long &);`。
- **L806 EN**: Introduces template parameters or specialization context: `template bool round_to_long<long double>(long double, long &);`.
  **L806 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<long double>(long double, long &);`。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L808 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L808 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L809 EN**: Introduces template parameters or specialization context: `template bool round_to_long<float16>(float16, long &);`.
  **L809 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<float16>(float16, long &);`。
- **L810 EN**: Closes the current preprocessor conditional block.
  **L810 CN**: 结束当前的预处理条件块。
- **L811 EN**: Blank line separating nearby declarations or logic blocks.
  **L811 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L812 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`.
  **L812 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`。
- **L813 EN**: Introduces template parameters or specialization context: `template bool round_to_long<float128>(float128, long &);`.
  **L813 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<float128>(float128, long &);`。
- **L814 EN**: Closes the current preprocessor conditional block.
  **L814 CN**: 结束当前的预处理条件块。
- **L815 EN**: Blank line separating nearby declarations or logic blocks.
  **L815 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L816 EN**: Introduces template parameters or specialization context: `template bool round_to_long<bfloat16>(bfloat16, long &);`.
  **L816 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<bfloat16>(bfloat16, long &);`。

### Lines 817-840

````cpp

template <typename T> bool round_to_long(T x, RoundingMode mode, long &result) {
  MPFRNumber mpfr(x);
  return mpfr.round_to_long(get_mpfr_rounding_mode(mode), result);
}

template bool round_to_long<float>(float, RoundingMode, long &);
template bool round_to_long<double>(double, RoundingMode, long &);
template bool round_to_long<long double>(long double, RoundingMode, long &);

#ifdef LIBC_TYPES_HAS_FLOAT16
template bool round_to_long<float16>(float16, RoundingMode, long &);
#endif // LIBC_TYPES_HAS_FLOAT16

#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
template bool round_to_long<float128>(float128, RoundingMode, long &);
#endif // LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE

template bool round_to_long<bfloat16>(bfloat16, RoundingMode, long &);

template <typename T> T round(T x, RoundingMode mode) {
  MPFRNumber mpfr(x);
  MPFRNumber result = mpfr.rint(get_mpfr_rounding_mode(mode));
  return result.as<T>();
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L818 EN**: Introduces template parameters or specialization context: `template <typename T> bool round_to_long(T x, RoundingMode mode, long &result) {`.
  **L818 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool round_to_long(T x, RoundingMode mode, long &result) {`。
- **L819 EN**: Constructs or initializes local object `mpfr` with parenthesized arguments.
  **L819 CN**: 使用带括号的参数构造或初始化局部对象 `mpfr`。
- **L820 EN**: Returns from the current function with `mpfr.round_to_long(get_mpfr_rounding_mode(mode), result)`.
  **L820 CN**: 以 `mpfr.round_to_long(get_mpfr_rounding_mode(mode), result)` 从当前函数返回。
- **L821 EN**: Closes the current lexical scope or compound statement.
  **L821 CN**: 结束当前词法作用域或复合语句块。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L823 EN**: Introduces template parameters or specialization context: `template bool round_to_long<float>(float, RoundingMode, long &);`.
  **L823 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<float>(float, RoundingMode, long &);`。
- **L824 EN**: Introduces template parameters or specialization context: `template bool round_to_long<double>(double, RoundingMode, long &);`.
  **L824 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<double>(double, RoundingMode, long &);`。
- **L825 EN**: Introduces template parameters or specialization context: `template bool round_to_long<long double>(long double, RoundingMode, long &);`.
  **L825 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<long double>(long double, RoundingMode, long &);`。
- **L826 EN**: Blank line separating nearby declarations or logic blocks.
  **L826 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L827 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L827 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L828 EN**: Introduces template parameters or specialization context: `template bool round_to_long<float16>(float16, RoundingMode, long &);`.
  **L828 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<float16>(float16, RoundingMode, long &);`。
- **L829 EN**: Closes the current preprocessor conditional block.
  **L829 CN**: 结束当前的预处理条件块。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L831 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`.
  **L831 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`。
- **L832 EN**: Introduces template parameters or specialization context: `template bool round_to_long<float128>(float128, RoundingMode, long &);`.
  **L832 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<float128>(float128, RoundingMode, long &);`。
- **L833 EN**: Closes the current preprocessor conditional block.
  **L833 CN**: 结束当前的预处理条件块。
- **L834 EN**: Blank line separating nearby declarations or logic blocks.
  **L834 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L835 EN**: Introduces template parameters or specialization context: `template bool round_to_long<bfloat16>(bfloat16, RoundingMode, long &);`.
  **L835 CN**: 为后续声明引入模板参数或特化上下文：`template bool round_to_long<bfloat16>(bfloat16, RoundingMode, long &);`。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L837 EN**: Introduces template parameters or specialization context: `template <typename T> T round(T x, RoundingMode mode) {`.
  **L837 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> T round(T x, RoundingMode mode) {`。
- **L838 EN**: Constructs or initializes local object `mpfr` with parenthesized arguments.
  **L838 CN**: 使用带括号的参数构造或初始化局部对象 `mpfr`。
- **L839 EN**: Initializes variable `result` from the right-hand expression.
  **L839 CN**: 使用右侧表达式初始化变量 `result`。
- **L840 EN**: Returns from the current function with `result.as<T>()`.
  **L840 CN**: 以 `result.as<T>()` 从当前函数返回。

### Lines 841-859

````cpp
}

template float round<float>(float, RoundingMode);
template double round<double>(double, RoundingMode);
template long double round<long double>(long double, RoundingMode);

#ifdef LIBC_TYPES_HAS_FLOAT16
template float16 round<float16>(float16, RoundingMode);
#endif // LIBC_TYPES_HAS_FLOAT16

#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE
template float128 round<float128>(float128, RoundingMode);
#endif // LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE

template bfloat16 round<bfloat16>(bfloat16, RoundingMode);

} // namespace mpfr
} // namespace testing
} // namespace LIBC_NAMESPACE_DECL
````
- **L841 EN**: Closes the current lexical scope or compound statement.
  **L841 CN**: 结束当前词法作用域或复合语句块。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L843 EN**: Introduces template parameters or specialization context: `template float round<float>(float, RoundingMode);`.
  **L843 CN**: 为后续声明引入模板参数或特化上下文：`template float round<float>(float, RoundingMode);`。
- **L844 EN**: Introduces template parameters or specialization context: `template double round<double>(double, RoundingMode);`.
  **L844 CN**: 为后续声明引入模板参数或特化上下文：`template double round<double>(double, RoundingMode);`。
- **L845 EN**: Introduces template parameters or specialization context: `template long double round<long double>(long double, RoundingMode);`.
  **L845 CN**: 为后续声明引入模板参数或特化上下文：`template long double round<long double>(long double, RoundingMode);`。
- **L846 EN**: Blank line separating nearby declarations or logic blocks.
  **L846 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L847 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_HAS_FLOAT16`.
  **L847 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_HAS_FLOAT16`。
- **L848 EN**: Introduces template parameters or specialization context: `template float16 round<float16>(float16, RoundingMode);`.
  **L848 CN**: 为后续声明引入模板参数或特化上下文：`template float16 round<float16>(float16, RoundingMode);`。
- **L849 EN**: Closes the current preprocessor conditional block.
  **L849 CN**: 结束当前的预处理条件块。
- **L850 EN**: Blank line separating nearby declarations or logic blocks.
  **L850 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L851 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`.
  **L851 CN**: 开始一个预处理条件块：`#ifdef LIBC_TYPES_FLOAT128_IS_NOT_LONG_DOUBLE`。
- **L852 EN**: Introduces template parameters or specialization context: `template float128 round<float128>(float128, RoundingMode);`.
  **L852 CN**: 为后续声明引入模板参数或特化上下文：`template float128 round<float128>(float128, RoundingMode);`。
- **L853 EN**: Closes the current preprocessor conditional block.
  **L853 CN**: 结束当前的预处理条件块。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L855 EN**: Introduces template parameters or specialization context: `template bfloat16 round<bfloat16>(bfloat16, RoundingMode);`.
  **L855 CN**: 为后续声明引入模板参数或特化上下文：`template bfloat16 round<bfloat16>(bfloat16, RoundingMode);`。
- **L856 EN**: Blank line separating nearby declarations or logic blocks.
  **L856 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L857 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace mpfr`.
  **L857 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace mpfr`。
- **L858 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace testing`.
  **L858 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace testing`。
- **L859 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace LIBC_NAMESPACE_DECL`.
  **L859 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace LIBC_NAMESPACE_DECL`。

## Key Concepts / 关键概念

- **Multiprecision reference utilities / 多精度参考工具**:
  - **EN**: Uses external multiprecision libraries to provide high-accuracy reference computations and validation helpers.
  - **CN**: 使用外部多精度库提供高精度参考计算与验证辅助逻辑。
- **Namespace isolation / 命名空间隔离**:
  - **EN**: Uses the LLVM libc namespace macro so implementation symbols remain isolated from the public ABI namespace.
  - **CN**: 使用 LLVM libc 命名空间宏，使实现符号与公共 ABI 命名空间隔离。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `MPFRUtils.h`, `MPCommon.h`, `src/__support/CPP/array.h`, `src/__support/CPP/stringstream.h`, `src/__support/FPUtil/bfloat16.h`, `src/__support/FPUtil/fpbits_str.h`, `src/__support/macros/config.h`, `src/__support/macros/properties/types.h`
- **Dependency categories / 依赖类别**: LLVM libc internal support utilities / LLVM libc 内部支撑工具 (4), nearby helper declarations / 附近的辅助声明 (2), LLVM libc internal C++ support utilities / LLVM libc 内部 C++ 支撑工具 (2)

- **EN**: `MPFRUtils.h` provides nearby helper declarations.
  - **CN**: `MPFRUtils.h` 提供的内容是：附近的辅助声明。
- **EN**: `MPCommon.h` provides nearby helper declarations.
  - **CN**: `MPCommon.h` 提供的内容是：附近的辅助声明。
- **EN**: `src/__support/CPP/array.h` provides LLVM libc internal C++ support utilities.
  - **CN**: `src/__support/CPP/array.h` 提供的内容是：LLVM libc 内部 C++ 支撑工具。
- **EN**: `src/__support/CPP/stringstream.h` provides LLVM libc internal C++ support utilities.
  - **CN**: `src/__support/CPP/stringstream.h` 提供的内容是：LLVM libc 内部 C++ 支撑工具。
- **EN**: `src/__support/FPUtil/bfloat16.h` provides LLVM libc internal support utilities.
  - **CN**: `src/__support/FPUtil/bfloat16.h` 提供的内容是：LLVM libc 内部支撑工具。
- **EN**: `src/__support/FPUtil/fpbits_str.h` provides LLVM libc internal support utilities.
  - **CN**: `src/__support/FPUtil/fpbits_str.h` 提供的内容是：LLVM libc 内部支撑工具。
- **EN**: `src/__support/macros/config.h` provides LLVM libc internal support utilities.
  - **CN**: `src/__support/macros/config.h` 提供的内容是：LLVM libc 内部支撑工具。
- **EN**: `src/__support/macros/properties/types.h` provides LLVM libc internal support utilities.
  - **CN**: `src/__support/macros/properties/types.h` 提供的内容是：LLVM libc 内部支撑工具。
