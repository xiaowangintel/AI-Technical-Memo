# range_reduction_double_common.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `libc/src/__support/math/range_reduction_double_common.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Range reduction for double precision sin/cos/tan -*- C++.
  - **CN**: 声明 LLVM libc 各入口复用的内部浮点内核、归约辅助逻辑以及按类型特化的数学支持组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- Range reduction for double precision sin/cos/tan -*- C++ --------*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_COMMON_H
#define LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_COMMON_H

#include "src/__support/FPUtil/double_double.h"
#include "src/__support/FPUtil/dyadic_float.h"
#include "src/__support/FPUtil/multiply_add.h"
#include "src/__support/FPUtil/nearest_integer.h"
#include "src/__support/common.h"
#include "src/__support/integer_literals.h"
#include "src/__support/macros/config.h"
#include "src/__support/macros/optimization.h"

namespace LIBC_NAMESPACE_DECL {

namespace math {

````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 分隔注释，用于视觉分组。
- **L3 EN**: Comment documents nearby intent or constraints: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的意图或约束：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment documents nearby intent or constraints: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的意图或约束：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents nearby intent or constraints: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的意图或约束：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 分隔注释，用于视觉分组。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic.
  **L8 CN**: 空行，用于分隔相邻声明或逻辑。
- **L9 EN**: Starts a header guard condition: `#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_COMMON_H`.
  **L9 CN**: 开始头文件保护条件：`#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_COMMON_H`。
- **L10 EN**: Defines macro `LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_COMMON_H` for compile-time constants, aliases, or dispatch control.
  **L10 CN**: 定义宏 `LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_COMMON_H`，用于编译期常量、别名或分发控制。
- **L11 EN**: Blank line separating nearby declarations or logic.
  **L11 CN**: 空行，用于分隔相邻声明或逻辑。
- **L12 EN**: Includes "src/__support/FPUtil/double_double.h" to access LLVM libc floating-point utility helpers.
  **L12 CN**: 引入 "src/__support/FPUtil/double_double.h" 以使用LLVM libc 浮点工具辅助组件。
- **L13 EN**: Includes "src/__support/FPUtil/dyadic_float.h" to access LLVM libc floating-point utility helpers.
  **L13 CN**: 引入 "src/__support/FPUtil/dyadic_float.h" 以使用LLVM libc 浮点工具辅助组件。
- **L14 EN**: Includes "src/__support/FPUtil/multiply_add.h" to access LLVM libc floating-point utility helpers.
  **L14 CN**: 引入 "src/__support/FPUtil/multiply_add.h" 以使用LLVM libc 浮点工具辅助组件。
- **L15 EN**: Includes "src/__support/FPUtil/nearest_integer.h" to access LLVM libc floating-point utility helpers.
  **L15 CN**: 引入 "src/__support/FPUtil/nearest_integer.h" 以使用LLVM libc 浮点工具辅助组件。
- **L16 EN**: Includes "src/__support/common.h" to access LLVM libc internal support utilities.
  **L16 CN**: 引入 "src/__support/common.h" 以使用LLVM libc 内部支撑工具。
- **L17 EN**: Includes "src/__support/integer_literals.h" to access LLVM libc internal support utilities.
  **L17 CN**: 引入 "src/__support/integer_literals.h" 以使用LLVM libc 内部支撑工具。
- **L18 EN**: Includes "src/__support/macros/config.h" to access LLVM libc configuration and attribute macros.
  **L18 CN**: 引入 "src/__support/macros/config.h" 以使用LLVM libc 配置与属性宏。
- **L19 EN**: Includes "src/__support/macros/optimization.h" to access LLVM libc configuration and attribute macros.
  **L19 CN**: 引入 "src/__support/macros/optimization.h" 以使用LLVM libc 配置与属性宏。
- **L20 EN**: Blank line separating nearby declarations or logic.
  **L20 CN**: 空行，用于分隔相邻声明或逻辑。
- **L21 EN**: Opens namespace scope `LIBC_NAMESPACE_DECL`.
  **L21 CN**: 打开命名空间作用域 `LIBC_NAMESPACE_DECL`。
- **L22 EN**: Blank line separating nearby declarations or logic.
  **L22 CN**: 空行，用于分隔相邻声明或逻辑。
- **L23 EN**: Opens namespace scope `math`.
  **L23 CN**: 打开命名空间作用域 `math`。
- **L24 EN**: Blank line separating nearby declarations or logic.
  **L24 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 25-48

````cpp
namespace range_reduction_double_internal {

#ifdef LIBC_TARGET_CPU_HAS_FMA_DOUBLE
LIBC_INLINE_VAR constexpr unsigned SPLIT = fputil::DefaultSplit<double>::VALUE;
#else
// When there is no-FMA instructions, in order to have exact product of 2 double
// precision with directional roundings, we need to lower the precision of the
// constants by at least 1 bit, and use a different splitting constant.
LIBC_INLINE_VAR constexpr unsigned SPLIT = 28;
#endif // LIBC_TARGET_CPU_HAS_FMA_DOUBLE

using LIBC_NAMESPACE::fputil::DoubleDouble;
using Float128 = LIBC_NAMESPACE::fputil::DyadicFloat<128>;

#define FAST_PASS_EXPONENT 16

// For 2^-7 < |x| < 2^16, return k and u such that:
//   k = round(x * 128/pi)
//   x mod pi/128 = x - k * pi/128 ~ u.hi + u.lo
// Error bound:
//   |(x - k * pi/128) - (u_hi + u_lo)| <= max(ulp(ulp(u_hi)), 2^-119)
//                                      <= 2^-111.
LIBC_INLINE unsigned range_reduction_small(double x, DoubleDouble &u) {
  // Values of -pi/128 used for inputs with absolute value <= 2^16.
````
- **L25 EN**: Opens namespace scope `range_reduction_double_internal`.
  **L25 CN**: 打开命名空间作用域 `range_reduction_double_internal`。
- **L26 EN**: Blank line separating nearby declarations or logic.
  **L26 CN**: 空行，用于分隔相邻声明或逻辑。
- **L27 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TARGET_CPU_HAS_FMA_DOUBLE`.
  **L27 CN**: 开始一个预处理条件块：`#ifdef LIBC_TARGET_CPU_HAS_FMA_DOUBLE`。
- **L28 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L28 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L29 EN**: Continues the current preprocessor branch selection.
  **L29 CN**: 继续当前的预处理分支选择。
- **L30 EN**: Comment documents nearby intent or constraints: `When there is no-FMA instructions, in order to have exact product of 2 double`.
  **L30 CN**: 注释说明附近代码的意图或约束：`When there is no-FMA instructions, in order to have exact product of 2 double`。
- **L31 EN**: Comment documents nearby intent or constraints: `precision with directional roundings, we need to lower the precision of the`.
  **L31 CN**: 注释说明附近代码的意图或约束：`precision with directional roundings, we need to lower the precision of the`。
- **L32 EN**: Comment documents nearby intent or constraints: `constants by at least 1 bit, and use a different splitting constant.`.
  **L32 CN**: 注释说明附近代码的意图或约束：`constants by at least 1 bit, and use a different splitting constant.`。
- **L33 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L33 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L34 EN**: Closes the current preprocessor conditional block or header guard.
  **L34 CN**: 结束当前预处理条件块或头文件保护。
- **L35 EN**: Blank line separating nearby declarations or logic.
  **L35 CN**: 空行，用于分隔相邻声明或逻辑。
- **L36 EN**: Introduces a using declaration or alias: `using LIBC_NAMESPACE::fputil::DoubleDouble;`.
  **L36 CN**: 引入一条 using 声明或别名：`using LIBC_NAMESPACE::fputil::DoubleDouble;`。
- **L37 EN**: Defines alias `Float128` to simplify later code.
  **L37 CN**: 定义别名 `Float128` 以简化后续代码。
- **L38 EN**: Blank line separating nearby declarations or logic.
  **L38 CN**: 空行，用于分隔相邻声明或逻辑。
- **L39 EN**: Defines macro `FAST_PASS_EXPONENT` for compile-time constants, aliases, or dispatch control.
  **L39 CN**: 定义宏 `FAST_PASS_EXPONENT`，用于编译期常量、别名或分发控制。
- **L40 EN**: Blank line separating nearby declarations or logic.
  **L40 CN**: 空行，用于分隔相邻声明或逻辑。
- **L41 EN**: Comment documents nearby intent or constraints: `For 2^-7 < \|x\| < 2^16, return k and u such that:`.
  **L41 CN**: 注释说明附近代码的意图或约束：`For 2^-7 < \|x\| < 2^16, return k and u such that:`。
- **L42 EN**: Comment documents nearby intent or constraints: `k = round(x * 128/pi)`.
  **L42 CN**: 注释说明附近代码的意图或约束：`k = round(x * 128/pi)`。
- **L43 EN**: Comment documents nearby intent or constraints: `x mod pi/128 = x - k * pi/128 ~ u.hi + u.lo`.
  **L43 CN**: 注释说明附近代码的意图或约束：`x mod pi/128 = x - k * pi/128 ~ u.hi + u.lo`。
- **L44 EN**: Comment documents nearby intent or constraints: `Error bound:`.
  **L44 CN**: 注释说明附近代码的意图或约束：`Error bound:`。
- **L45 EN**: Comment documents nearby intent or constraints: `\|(x - k * pi/128) - (u_hi + u_lo)\| <= max(ulp(ulp(u_hi)), 2^-119)`.
  **L45 CN**: 注释说明附近代码的意图或约束：`\|(x - k * pi/128) - (u_hi + u_lo)\| <= max(ulp(ulp(u_hi)), 2^-119)`。
- **L46 EN**: Comment documents nearby intent or constraints: `<= 2^-111.`.
  **L46 CN**: 注释说明附近代码的意图或约束：`<= 2^-111.`。
- **L47 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L47 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L48 EN**: Comment documents nearby intent or constraints: `Values of -pi/128 used for inputs with absolute value <= 2^16.`.
  **L48 CN**: 注释说明附近代码的意图或约束：`Values of -pi/128 used for inputs with absolute value <= 2^16.`。

### Lines 49-72

````cpp
  // The first 3 parts are generated with (53 - 21 = 32)-bit precision, so that
  // the product k * MPI_OVER_128[i] is exact.
  // Generated by Sollya with:
  // > display = hexadecimal!;
  // > a = round(pi/128, 32, RN);
  // > b = round(pi/128 - a, 32, RN);
  // > c = round(pi/128 - a - b, D, RN);
  // > print(-a, ",", -b, ",", -c);
  constexpr double MPI_OVER_128[3] = {-0x1.921fb544p-6, -0x1.0b4611a6p-40,
                                      -0x1.3198a2e037073p-75};
  constexpr double ONE_TWENTY_EIGHT_OVER_PI_D = 0x1.45f306dc9c883p5;
  double prod_hi = x * ONE_TWENTY_EIGHT_OVER_PI_D;
  double kd = fputil::nearest_integer(prod_hi);

  // Let y = x - k * (pi/128)
  // Then |y| < pi / 256
  // With extra rounding errors, we can bound |y| < 1.6 * 2^-7.
  double y_hi = fputil::multiply_add(kd, MPI_OVER_128[0], x); // Exact
  // |u.hi| < 1.6*2^-7
  u.hi = fputil::multiply_add(kd, MPI_OVER_128[1], y_hi);
  double u0 = y_hi - u.hi; // Exact
  // |u.lo| <= max(ulp(u.hi), |kd * MPI_OVER_128[2]|)
  double u1 = fputil::multiply_add(kd, MPI_OVER_128[1], u0); // Exact
  u.lo = fputil::multiply_add(kd, MPI_OVER_128[2], u1);
````
- **L49 EN**: Comment documents nearby intent or constraints: `The first 3 parts are generated with (53 - 21 = 32)-bit precision, so that`.
  **L49 CN**: 注释说明附近代码的意图或约束：`The first 3 parts are generated with (53 - 21 = 32)-bit precision, so that`。
- **L50 EN**: Comment documents nearby intent or constraints: `the product k * MPI_OVER_128[i] is exact.`.
  **L50 CN**: 注释说明附近代码的意图或约束：`the product k * MPI_OVER_128[i] is exact.`。
- **L51 EN**: Comment documents nearby intent or constraints: `Generated by Sollya with:`.
  **L51 CN**: 注释说明附近代码的意图或约束：`Generated by Sollya with:`。
- **L52 EN**: Comment documents nearby intent or constraints: `> display = hexadecimal!;`.
  **L52 CN**: 注释说明附近代码的意图或约束：`> display = hexadecimal!;`。
- **L53 EN**: Comment documents nearby intent or constraints: `> a = round(pi/128, 32, RN);`.
  **L53 CN**: 注释说明附近代码的意图或约束：`> a = round(pi/128, 32, RN);`。
- **L54 EN**: Comment documents nearby intent or constraints: `> b = round(pi/128 - a, 32, RN);`.
  **L54 CN**: 注释说明附近代码的意图或约束：`> b = round(pi/128 - a, 32, RN);`。
- **L55 EN**: Comment documents nearby intent or constraints: `> c = round(pi/128 - a - b, D, RN);`.
  **L55 CN**: 注释说明附近代码的意图或约束：`> c = round(pi/128 - a - b, D, RN);`。
- **L56 EN**: Comment documents nearby intent or constraints: `> print(-a, ",", -b, ",", -c);`.
  **L56 CN**: 注释说明附近代码的意图或约束：`> print(-a, ",", -b, ",", -c);`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `constexpr double MPI_OVER_128[3] = {-0x1.921fb544p-6, -0x1.0b4611a6p-40,`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`constexpr double MPI_OVER_128[3] = {-0x1.921fb544p-6, -0x1.0b4611a6p-40,`。
- **L58 EN**: Executes a standalone statement or declaration: `-0x1.3198a2e037073p-75};`.
  **L58 CN**: 执行一条独立语句或声明：`-0x1.3198a2e037073p-75};`。
- **L59 EN**: Initializes variable `ONE_TWENTY_EIGHT_OVER_PI_D` from the right-hand expression.
  **L59 CN**: 使用右侧表达式初始化变量 `ONE_TWENTY_EIGHT_OVER_PI_D`。
- **L60 EN**: Initializes variable `prod_hi` from the right-hand expression.
  **L60 CN**: 使用右侧表达式初始化变量 `prod_hi`。
- **L61 EN**: Initializes variable `kd` from the right-hand expression.
  **L61 CN**: 使用右侧表达式初始化变量 `kd`。
- **L62 EN**: Blank line separating nearby declarations or logic.
  **L62 CN**: 空行，用于分隔相邻声明或逻辑。
- **L63 EN**: Comment documents nearby intent or constraints: `Let y = x - k * (pi/128)`.
  **L63 CN**: 注释说明附近代码的意图或约束：`Let y = x - k * (pi/128)`。
- **L64 EN**: Comment documents nearby intent or constraints: `Then \|y\| < pi / 256`.
  **L64 CN**: 注释说明附近代码的意图或约束：`Then \|y\| < pi / 256`。
- **L65 EN**: Comment documents nearby intent or constraints: `With extra rounding errors, we can bound \|y\| < 1.6 * 2^-7.`.
  **L65 CN**: 注释说明附近代码的意图或约束：`With extra rounding errors, we can bound \|y\| < 1.6 * 2^-7.`。
- **L66 EN**: Continues logic associated with callable symbol `multiply_add`.
  **L66 CN**: 继续与可调用符号 `multiply_add` 相关的逻辑。
- **L67 EN**: Comment documents nearby intent or constraints: `\|u.hi\| < 1.6*2^-7`.
  **L67 CN**: 注释说明附近代码的意图或约束：`\|u.hi\| < 1.6*2^-7`。
- **L68 EN**: Executes a call or declaration centered on `fputil::multiply_add`.
  **L68 CN**: 执行以 `fputil::multiply_add` 为核心的调用或声明。
- **L69 EN**: Continues the surrounding expression or declaration: `double u0 = y_hi - u.hi; // Exact`.
  **L69 CN**: 继续构造周围的表达式或声明：`double u0 = y_hi - u.hi; // Exact`。
- **L70 EN**: Comment documents nearby intent or constraints: `\|u.lo\| <= max(ulp(u.hi), \|kd * MPI_OVER_128[2]\|)`.
  **L70 CN**: 注释说明附近代码的意图或约束：`\|u.lo\| <= max(ulp(u.hi), \|kd * MPI_OVER_128[2]\|)`。
- **L71 EN**: Continues logic associated with callable symbol `multiply_add`.
  **L71 CN**: 继续与可调用符号 `multiply_add` 相关的逻辑。
- **L72 EN**: Executes a call or declaration centered on `fputil::multiply_add`.
  **L72 CN**: 执行以 `fputil::multiply_add` 为核心的调用或声明。

### Lines 73-96

````cpp
  // Error bound:
  // |x - k * pi/128| - (u.hi + u.lo) <= ulp(u.lo)
  //                                  <= ulp(max(ulp(u.hi), kd*MPI_OVER_128[2]))
  //                                  <= 2^(-7 - 104) = 2^-111.

  return static_cast<unsigned>(static_cast<int64_t>(kd));
}

// Digits of 2^(16*i) / pi, generated by Sollya with:
// > procedure ulp(x, n) { return 2^(floor(log2(abs(x))) - n); };
// > for i from 0 to 63 do {
//     if i < 3 then { pi_inv = 0.25 + 2^(16*(i - 3)) / pi; }
//     else { pi_inv = 2^(16*(i-3)) / pi; };
//     pn = nearestint(pi_inv);
//     pi_frac = pi_inv - pn;
//     a = round(pi_frac, 51, RN);
//     b = round(pi_frac - a, 51, RN);
//     c = round(pi_frac - a - b, 51, RN);
//     d = round(pi_frac - a - b - c, D, RN);
//     print("{", 2^7 * a, ",", 2^7 * b, ",", 2^7 * c, ",", 2^7 * d, "},");
// };
//
// Notice that for [0..2] the leading bit of 2^(16*(i - 3)) / pi is very small,
// so we add 0.25 so that the conditions for the algorithms are still satisfied,
````
- **L73 EN**: Comment documents nearby intent or constraints: `Error bound:`.
  **L73 CN**: 注释说明附近代码的意图或约束：`Error bound:`。
- **L74 EN**: Comment documents nearby intent or constraints: `\|x - k * pi/128\| - (u.hi + u.lo) <= ulp(u.lo)`.
  **L74 CN**: 注释说明附近代码的意图或约束：`\|x - k * pi/128\| - (u.hi + u.lo) <= ulp(u.lo)`。
- **L75 EN**: Comment documents nearby intent or constraints: `<= ulp(max(ulp(u.hi), kd*MPI_OVER_128[2]))`.
  **L75 CN**: 注释说明附近代码的意图或约束：`<= ulp(max(ulp(u.hi), kd*MPI_OVER_128[2]))`。
- **L76 EN**: Comment documents nearby intent or constraints: `<= 2^(-7 - 104) = 2^-111.`.
  **L76 CN**: 注释说明附近代码的意图或约束：`<= 2^(-7 - 104) = 2^-111.`。
- **L77 EN**: Blank line separating nearby declarations or logic.
  **L77 CN**: 空行，用于分隔相邻声明或逻辑。
- **L78 EN**: Returns from the current function with `static_cast<unsigned>(static_cast<int64_t>(kd))`.
  **L78 CN**: 以 `static_cast<unsigned>(static_cast<int64_t>(kd))` 从当前函数返回。
- **L79 EN**: Closes the current lexical scope or compound statement.
  **L79 CN**: 结束当前词法作用域或复合语句块。
- **L80 EN**: Blank line separating nearby declarations or logic.
  **L80 CN**: 空行，用于分隔相邻声明或逻辑。
- **L81 EN**: Comment documents nearby intent or constraints: `Digits of 2^(16*i) / pi, generated by Sollya with:`.
  **L81 CN**: 注释说明附近代码的意图或约束：`Digits of 2^(16*i) / pi, generated by Sollya with:`。
- **L82 EN**: Comment documents nearby intent or constraints: `> procedure ulp(x, n) { return 2^(floor(log2(abs(x))) - n); };`.
  **L82 CN**: 注释说明附近代码的意图或约束：`> procedure ulp(x, n) { return 2^(floor(log2(abs(x))) - n); };`。
- **L83 EN**: Comment documents nearby intent or constraints: `> for i from 0 to 63 do {`.
  **L83 CN**: 注释说明附近代码的意图或约束：`> for i from 0 to 63 do {`。
- **L84 EN**: Comment documents nearby intent or constraints: `if i < 3 then { pi_inv = 0.25 + 2^(16*(i - 3)) / pi; }`.
  **L84 CN**: 注释说明附近代码的意图或约束：`if i < 3 then { pi_inv = 0.25 + 2^(16*(i - 3)) / pi; }`。
- **L85 EN**: Comment documents nearby intent or constraints: `else { pi_inv = 2^(16*(i-3)) / pi; };`.
  **L85 CN**: 注释说明附近代码的意图或约束：`else { pi_inv = 2^(16*(i-3)) / pi; };`。
- **L86 EN**: Comment documents nearby intent or constraints: `pn = nearestint(pi_inv);`.
  **L86 CN**: 注释说明附近代码的意图或约束：`pn = nearestint(pi_inv);`。
- **L87 EN**: Comment documents nearby intent or constraints: `pi_frac = pi_inv - pn;`.
  **L87 CN**: 注释说明附近代码的意图或约束：`pi_frac = pi_inv - pn;`。
- **L88 EN**: Comment documents nearby intent or constraints: `a = round(pi_frac, 51, RN);`.
  **L88 CN**: 注释说明附近代码的意图或约束：`a = round(pi_frac, 51, RN);`。
- **L89 EN**: Comment documents nearby intent or constraints: `b = round(pi_frac - a, 51, RN);`.
  **L89 CN**: 注释说明附近代码的意图或约束：`b = round(pi_frac - a, 51, RN);`。
- **L90 EN**: Comment documents nearby intent or constraints: `c = round(pi_frac - a - b, 51, RN);`.
  **L90 CN**: 注释说明附近代码的意图或约束：`c = round(pi_frac - a - b, 51, RN);`。
- **L91 EN**: Comment documents nearby intent or constraints: `d = round(pi_frac - a - b - c, D, RN);`.
  **L91 CN**: 注释说明附近代码的意图或约束：`d = round(pi_frac - a - b - c, D, RN);`。
- **L92 EN**: Comment documents nearby intent or constraints: `print("{", 2^7 * a, ",", 2^7 * b, ",", 2^7 * c, ",", 2^7 * d, "},");`.
  **L92 CN**: 注释说明附近代码的意图或约束：`print("{", 2^7 * a, ",", 2^7 * b, ",", 2^7 * c, ",", 2^7 * d, "},");`。
- **L93 EN**: Comment documents nearby intent or constraints: `};`.
  **L93 CN**: 注释说明附近代码的意图或约束：`};`。
- **L94 EN**: Separator comment used for visual grouping.
  **L94 CN**: 分隔注释，用于视觉分组。
- **L95 EN**: Comment documents nearby intent or constraints: `Notice that for [0..2] the leading bit of 2^(16*(i - 3)) / pi is very small,`.
  **L95 CN**: 注释说明附近代码的意图或约束：`Notice that for [0..2] the leading bit of 2^(16*(i - 3)) / pi is very small,`。
- **L96 EN**: Comment documents nearby intent or constraints: `so we add 0.25 so that the conditions for the algorithms are still satisfied,`.
  **L96 CN**: 注释说明附近代码的意图或约束：`so we add 0.25 so that the conditions for the algorithms are still satisfied,`。

### Lines 97-120

````cpp
// and one of those conditions guarantees that ulp(0.25 * x_reduced) >= 2, and
// will safely be discarded.

LIBC_INLINE_VAR constexpr double ONE_TWENTY_EIGHT_OVER_PI[64][4] = {
    {0x1.0000000000014p5, 0x1.7cc1b727220a8p-49, 0x1.4fe13abe8fa9cp-101,
     -0x1.911f924eb5336p-153},
    {0x1.0000000145f3p5, 0x1.b727220a94fep-49, 0x1.3abe8fa9a6eep-101,
     0x1.b6c52b3278872p-155},
    {0x1.000145f306dc8p5, 0x1.c882a53f84ebp-47, -0x1.70565911f925p-101,
     0x1.4acc9e21c821p-153},
    {0x1.45f306dc9c884p5, -0x1.5ac07b1505c14p-47, -0x1.96447e493ad4cp-99,
     -0x1.b0ef1bef806bap-152},
    {-0x1.f246c6efab58p4, -0x1.ec5417056591p-49, -0x1.f924eb53361ep-101,
     0x1.c820ff28b1d5fp-153},
    {0x1.391054a7f09d4p4, 0x1.f47d4d377036cp-48, 0x1.8a5664f10e41p-100,
     0x1.fe5163abdebbcp-154},
    {0x1.529fc2757d1f4p2, 0x1.34ddc0db62958p-50, 0x1.93c439041fe5p-102,
     0x1.63abdebbc561bp-154},
    {-0x1.ec5417056591p-1, -0x1.f924eb53361ep-53, 0x1.c820ff28b1d6p-105,
     -0x1.0a21d4f246dc9p-157},
    {-0x1.505c1596447e4p5, -0x1.275a99b0ef1cp-48, 0x1.07f9458eaf7bp-100,
     -0x1.0ea79236e4717p-152},
    {-0x1.596447e493ad4p1, -0x1.9b0ef1bef806cp-52, 0x1.63abdebbc561cp-106,
     -0x1.1b7238b7b645ap-159},
````
- **L97 EN**: Comment documents nearby intent or constraints: `and one of those conditions guarantees that ulp(0.25 * x_reduced) >= 2, and`.
  **L97 CN**: 注释说明附近代码的意图或约束：`and one of those conditions guarantees that ulp(0.25 * x_reduced) >= 2, and`。
- **L98 EN**: Comment documents nearby intent or constraints: `will safely be discarded.`.
  **L98 CN**: 注释说明附近代码的意图或约束：`will safely be discarded.`。
- **L99 EN**: Blank line separating nearby declarations or logic.
  **L99 CN**: 空行，用于分隔相邻声明或逻辑。
- **L100 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L100 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.0000000000014p5, 0x1.7cc1b727220a8p-49, 0x1.4fe13abe8fa9cp-101,`.
  **L101 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.0000000000014p5, 0x1.7cc1b727220a8p-49, 0x1.4fe13abe8fa9cp-101,`。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.911f924eb5336p-153},`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.911f924eb5336p-153},`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.0000000145f3p5, 0x1.b727220a94fep-49, 0x1.3abe8fa9a6eep-101,`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.0000000145f3p5, 0x1.b727220a94fep-49, 0x1.3abe8fa9a6eep-101,`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.b6c52b3278872p-155},`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.b6c52b3278872p-155},`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.000145f306dc8p5, 0x1.c882a53f84ebp-47, -0x1.70565911f925p-101,`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.000145f306dc8p5, 0x1.c882a53f84ebp-47, -0x1.70565911f925p-101,`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.4acc9e21c821p-153},`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.4acc9e21c821p-153},`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.45f306dc9c884p5, -0x1.5ac07b1505c14p-47, -0x1.96447e493ad4cp-99,`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.45f306dc9c884p5, -0x1.5ac07b1505c14p-47, -0x1.96447e493ad4cp-99,`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.b0ef1bef806bap-152},`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.b0ef1bef806bap-152},`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.f246c6efab58p4, -0x1.ec5417056591p-49, -0x1.f924eb53361ep-101,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.f246c6efab58p4, -0x1.ec5417056591p-49, -0x1.f924eb53361ep-101,`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.c820ff28b1d5fp-153},`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.c820ff28b1d5fp-153},`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.391054a7f09d4p4, 0x1.f47d4d377036cp-48, 0x1.8a5664f10e41p-100,`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.391054a7f09d4p4, 0x1.f47d4d377036cp-48, 0x1.8a5664f10e41p-100,`。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.fe5163abdebbcp-154},`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.fe5163abdebbcp-154},`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.529fc2757d1f4p2, 0x1.34ddc0db62958p-50, 0x1.93c439041fe5p-102,`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.529fc2757d1f4p2, 0x1.34ddc0db62958p-50, 0x1.93c439041fe5p-102,`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.63abdebbc561bp-154},`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.63abdebbc561bp-154},`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ec5417056591p-1, -0x1.f924eb53361ep-53, 0x1.c820ff28b1d6p-105,`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ec5417056591p-1, -0x1.f924eb53361ep-53, 0x1.c820ff28b1d6p-105,`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.0a21d4f246dc9p-157},`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.0a21d4f246dc9p-157},`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.505c1596447e4p5, -0x1.275a99b0ef1cp-48, 0x1.07f9458eaf7bp-100,`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.505c1596447e4p5, -0x1.275a99b0ef1cp-48, 0x1.07f9458eaf7bp-100,`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.0ea79236e4717p-152},`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.0ea79236e4717p-152},`。
- **L119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.596447e493ad4p1, -0x1.9b0ef1bef806cp-52, 0x1.63abdebbc561cp-106,`.
  **L119 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.596447e493ad4p1, -0x1.9b0ef1bef806cp-52, 0x1.63abdebbc561cp-106,`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.1b7238b7b645ap-159},`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.1b7238b7b645ap-159},`。

### Lines 121-144

````cpp
    {0x1.bb81b6c52b328p5, -0x1.de37df00d74e4p-49, 0x1.5ef5de2b0db94p-101,
     -0x1.c8e2ded9169p-153},
    {0x1.b6c52b3278874p5, -0x1.f7c035d38a844p-47, 0x1.778ac36e48dc8p-99,
     -0x1.6f6c8b47fe6dbp-152},
    {0x1.2b3278872084p5, -0x1.ae9c5421443a8p-50, -0x1.e48db91c5bdb4p-102,
     0x1.d2e006492eea1p-154},
    {-0x1.8778df7c035d4p5, 0x1.d5ef5de2b0db8p-49, 0x1.2371d2126e97p-101,
     0x1.924bba8274648p-160},
    {-0x1.bef806ba71508p4, -0x1.443a9e48db91cp-50, -0x1.6f6c8b47fe6dcp-104,
     0x1.77504e8c90e7fp-157},
    {-0x1.ae9c5421443a8p-2, -0x1.e48db91c5bdb4p-54, 0x1.d2e006492eeap-106,
     0x1.3a32439fc3bd6p-159},
    {-0x1.38a84288753c8p5, -0x1.1b7238b7b645cp-47, 0x1.c00c925dd413cp-99,
     -0x1.cdbc603c429c7p-151},
    {-0x1.0a21d4f246dc8p3, -0x1.c5bdb22d1ff9cp-50, 0x1.25dd413a32438p-103,
     0x1.fc3bd63962535p-155},
    {-0x1.d4f246dc8e2ep3, 0x1.26e9700324978p-49, -0x1.5f62e6de301e4p-102,
     0x1.eb1cb129a73efp-154},
    {-0x1.236e4716f6c8cp4, 0x1.700324977505p-49, -0x1.736f180f10a7p-101,
     -0x1.a76b2c608bbeep-153},
    {0x1.b8e909374b8p4, 0x1.924bba8274648p-48, 0x1.cfe1deb1cb128p-102,
     0x1.a73ee88235f53p-154},
    {0x1.09374b801924cp4, -0x1.15f62e6de302p-50, 0x1.deb1cb129a74p-102,
     -0x1.177dca0ad144cp-154},
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.bb81b6c52b328p5, -0x1.de37df00d74e4p-49, 0x1.5ef5de2b0db94p-101,`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.bb81b6c52b328p5, -0x1.de37df00d74e4p-49, 0x1.5ef5de2b0db94p-101,`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.c8e2ded9169p-153},`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.c8e2ded9169p-153},`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.b6c52b3278874p5, -0x1.f7c035d38a844p-47, 0x1.778ac36e48dc8p-99,`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.b6c52b3278874p5, -0x1.f7c035d38a844p-47, 0x1.778ac36e48dc8p-99,`。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.6f6c8b47fe6dbp-152},`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.6f6c8b47fe6dbp-152},`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2b3278872084p5, -0x1.ae9c5421443a8p-50, -0x1.e48db91c5bdb4p-102,`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2b3278872084p5, -0x1.ae9c5421443a8p-50, -0x1.e48db91c5bdb4p-102,`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.d2e006492eea1p-154},`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.d2e006492eea1p-154},`。
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.8778df7c035d4p5, 0x1.d5ef5de2b0db8p-49, 0x1.2371d2126e97p-101,`.
  **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.8778df7c035d4p5, 0x1.d5ef5de2b0db8p-49, 0x1.2371d2126e97p-101,`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.924bba8274648p-160},`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.924bba8274648p-160},`。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.bef806ba71508p4, -0x1.443a9e48db91cp-50, -0x1.6f6c8b47fe6dcp-104,`.
  **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.bef806ba71508p4, -0x1.443a9e48db91cp-50, -0x1.6f6c8b47fe6dcp-104,`。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.77504e8c90e7fp-157},`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.77504e8c90e7fp-157},`。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ae9c5421443a8p-2, -0x1.e48db91c5bdb4p-54, 0x1.d2e006492eeap-106,`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ae9c5421443a8p-2, -0x1.e48db91c5bdb4p-54, 0x1.d2e006492eeap-106,`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.3a32439fc3bd6p-159},`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.3a32439fc3bd6p-159},`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.38a84288753c8p5, -0x1.1b7238b7b645cp-47, 0x1.c00c925dd413cp-99,`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.38a84288753c8p5, -0x1.1b7238b7b645cp-47, 0x1.c00c925dd413cp-99,`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.cdbc603c429c7p-151},`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.cdbc603c429c7p-151},`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0a21d4f246dc8p3, -0x1.c5bdb22d1ff9cp-50, 0x1.25dd413a32438p-103,`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0a21d4f246dc8p3, -0x1.c5bdb22d1ff9cp-50, 0x1.25dd413a32438p-103,`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.fc3bd63962535p-155},`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.fc3bd63962535p-155},`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.d4f246dc8e2ep3, 0x1.26e9700324978p-49, -0x1.5f62e6de301e4p-102,`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.d4f246dc8e2ep3, 0x1.26e9700324978p-49, -0x1.5f62e6de301e4p-102,`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.eb1cb129a73efp-154},`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.eb1cb129a73efp-154},`。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.236e4716f6c8cp4, 0x1.700324977505p-49, -0x1.736f180f10a7p-101,`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.236e4716f6c8cp4, 0x1.700324977505p-49, -0x1.736f180f10a7p-101,`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.a76b2c608bbeep-153},`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.a76b2c608bbeep-153},`。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.b8e909374b8p4, 0x1.924bba8274648p-48, 0x1.cfe1deb1cb128p-102,`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.b8e909374b8p4, 0x1.924bba8274648p-48, 0x1.cfe1deb1cb128p-102,`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.a73ee88235f53p-154},`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.a73ee88235f53p-154},`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.09374b801924cp4, -0x1.15f62e6de302p-50, 0x1.deb1cb129a74p-102,`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.09374b801924cp4, -0x1.15f62e6de302p-50, 0x1.deb1cb129a74p-102,`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.177dca0ad144cp-154},`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.177dca0ad144cp-154},`。

### Lines 145-168

````cpp
    {-0x1.68ffcdb688afcp3, 0x1.d1921cfe1debp-50, 0x1.cb129a73ee884p-102,
     -0x1.ca0ad144bb7b1p-154},
    {0x1.924bba8274648p0, 0x1.cfe1deb1cb128p-54, 0x1.a73ee88235f54p-106,
     -0x1.144bb7b16639p-158},
    {-0x1.a22bec5cdbc6p5, -0x1.e214e34ed658cp-50, -0x1.177dca0ad144cp-106,
     0x1.213a671c09ad1p-160},
    {0x1.3a32439fc3bd8p1, -0x1.c69dacb1822fp-51, 0x1.1afa975da2428p-105,
     -0x1.6638fd94ba082p-158},
    {-0x1.b78c0788538d4p4, 0x1.29a73ee88236p-50, -0x1.5a28976f62cc8p-103,
     0x1.c09ad17df904ep-156},
    {0x1.fc3bd63962534p5, 0x1.cfba208d7d4bcp-48, -0x1.12edec598e3f8p-100,
     0x1.ad17df904e647p-152},
    {-0x1.4e34ed658c118p2, 0x1.046bea5d7689p-51, 0x1.3a671c09ad17cp-104,
     0x1.f904e64758e61p-156},
    {0x1.62534e7dd1048p5, -0x1.415a28976f62cp-47, -0x1.8e3f652e8207p-100,
     0x1.3991d63983534p-154},
    {-0x1.63045df7282b4p4, -0x1.44bb7b16638fcp-50, -0x1.94ba081bec67p-102,
     0x1.d639835339f4ap-154},
    {0x1.d1046bea5d768p5, 0x1.213a671c09adp-48, 0x1.7df904e64759p-100,
     -0x1.9f2b3182d8defp-152},
    {0x1.afa975da24274p3, 0x1.9c7026b45f7e4p-50, 0x1.3991d63983534p-106,
     -0x1.82d8dee81d108p-160},
    {-0x1.a28976f62cc7p5, -0x1.fb29741037d8cp-47, -0x1.b8a719f2b3184p-100,
     0x1.272117e2ef7e5p-152},
````
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.68ffcdb688afcp3, 0x1.d1921cfe1debp-50, 0x1.cb129a73ee884p-102,`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.68ffcdb688afcp3, 0x1.d1921cfe1debp-50, 0x1.cb129a73ee884p-102,`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.ca0ad144bb7b1p-154},`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.ca0ad144bb7b1p-154},`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.924bba8274648p0, 0x1.cfe1deb1cb128p-54, 0x1.a73ee88235f54p-106,`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.924bba8274648p0, 0x1.cfe1deb1cb128p-54, 0x1.a73ee88235f54p-106,`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.144bb7b16639p-158},`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.144bb7b16639p-158},`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a22bec5cdbc6p5, -0x1.e214e34ed658cp-50, -0x1.177dca0ad144cp-106,`.
  **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a22bec5cdbc6p5, -0x1.e214e34ed658cp-50, -0x1.177dca0ad144cp-106,`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.213a671c09ad1p-160},`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.213a671c09ad1p-160},`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3a32439fc3bd8p1, -0x1.c69dacb1822fp-51, 0x1.1afa975da2428p-105,`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3a32439fc3bd8p1, -0x1.c69dacb1822fp-51, 0x1.1afa975da2428p-105,`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.6638fd94ba082p-158},`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.6638fd94ba082p-158},`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.b78c0788538d4p4, 0x1.29a73ee88236p-50, -0x1.5a28976f62cc8p-103,`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.b78c0788538d4p4, 0x1.29a73ee88236p-50, -0x1.5a28976f62cc8p-103,`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.c09ad17df904ep-156},`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.c09ad17df904ep-156},`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.fc3bd63962534p5, 0x1.cfba208d7d4bcp-48, -0x1.12edec598e3f8p-100,`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.fc3bd63962534p5, 0x1.cfba208d7d4bcp-48, -0x1.12edec598e3f8p-100,`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.ad17df904e647p-152},`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.ad17df904e647p-152},`。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.4e34ed658c118p2, 0x1.046bea5d7689p-51, 0x1.3a671c09ad17cp-104,`.
  **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.4e34ed658c118p2, 0x1.046bea5d7689p-51, 0x1.3a671c09ad17cp-104,`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.f904e64758e61p-156},`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.f904e64758e61p-156},`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.62534e7dd1048p5, -0x1.415a28976f62cp-47, -0x1.8e3f652e8207p-100,`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.62534e7dd1048p5, -0x1.415a28976f62cp-47, -0x1.8e3f652e8207p-100,`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.3991d63983534p-154},`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.3991d63983534p-154},`。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.63045df7282b4p4, -0x1.44bb7b16638fcp-50, -0x1.94ba081bec67p-102,`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.63045df7282b4p4, -0x1.44bb7b16638fcp-50, -0x1.94ba081bec67p-102,`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.d639835339f4ap-154},`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.d639835339f4ap-154},`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.d1046bea5d768p5, 0x1.213a671c09adp-48, 0x1.7df904e64759p-100,`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.d1046bea5d768p5, 0x1.213a671c09adp-48, 0x1.7df904e64759p-100,`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.9f2b3182d8defp-152},`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.9f2b3182d8defp-152},`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.afa975da24274p3, 0x1.9c7026b45f7e4p-50, 0x1.3991d63983534p-106,`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.afa975da24274p3, 0x1.9c7026b45f7e4p-50, 0x1.3991d63983534p-106,`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.82d8dee81d108p-160},`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.82d8dee81d108p-160},`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a28976f62cc7p5, -0x1.fb29741037d8cp-47, -0x1.b8a719f2b3184p-100,`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a28976f62cc7p5, -0x1.fb29741037d8cp-47, -0x1.b8a719f2b3184p-100,`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.272117e2ef7e5p-152},`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.272117e2ef7e5p-152},`。

### Lines 169-192

````cpp
    {-0x1.76f62cc71fb28p5, -0x1.741037d8cdc54p-47, 0x1.cc1a99cfa4e44p-101,
     -0x1.d03a21036be27p-153},
    {0x1.d338e04d68bfp5, -0x1.bec66e29c67ccp-50, 0x1.339f49c845f8cp-102,
     -0x1.081b5f13801dap-156},
    {0x1.c09ad17df905p4, -0x1.9b8a719f2b318p-48, -0x1.6c6f740e8840cp-103,
     -0x1.af89c00ed0004p-155},
    {0x1.68befc827323cp5, -0x1.38cf9598c16c8p-47, 0x1.08bf177bf2508p-99,
     -0x1.3801da00087eap-152},
    {-0x1.037d8cdc538dp5, 0x1.a99cfa4e422fcp-49, 0x1.77bf250763ffp-103,
     0x1.2fffbc0b301fep-155},
    {-0x1.8cdc538cf9598p5, -0x1.82d8dee81d108p-48, -0x1.b5f13801dap-104,
     -0x1.0fd33f8086877p-157},
    {-0x1.4e33e566305bp3, -0x1.bdd03a21036cp-49, 0x1.d8ffc4bffef04p-101,
     -0x1.33f80868773a5p-153},
    {-0x1.f2b3182d8dee8p4, -0x1.d1081b5f138p-52, -0x1.da00087e99fcp-104,
     -0x1.0d0ee74a5f593p-158},
    {-0x1.8c16c6f740e88p5, -0x1.036be27003b4p-49, -0x1.0fd33f8086878p-109,
     0x1.8b5a0a6d1f6d3p-162},
    {0x1.3908bf177bf24p5, 0x1.0763ff12fffbcp-47, 0x1.6603fbcbc462cp-104,
     0x1.6829b47db4dap-156},
    {0x1.7e2ef7e4a0ec8p4, -0x1.da00087e99fcp-56, -0x1.0d0ee74a5f594p-110,
     0x1.1f6d367ecf27dp-162},
    {-0x1.081b5f13801dcp4, 0x1.fff7816603fbcp-48, 0x1.788c5ad05369p-101,
     -0x1.25930261b069fp-155},
````
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.76f62cc71fb28p5, -0x1.741037d8cdc54p-47, 0x1.cc1a99cfa4e44p-101,`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.76f62cc71fb28p5, -0x1.741037d8cdc54p-47, 0x1.cc1a99cfa4e44p-101,`。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.d03a21036be27p-153},`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.d03a21036be27p-153},`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.d338e04d68bfp5, -0x1.bec66e29c67ccp-50, 0x1.339f49c845f8cp-102,`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.d338e04d68bfp5, -0x1.bec66e29c67ccp-50, 0x1.339f49c845f8cp-102,`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.081b5f13801dap-156},`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.081b5f13801dap-156},`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.c09ad17df905p4, -0x1.9b8a719f2b318p-48, -0x1.6c6f740e8840cp-103,`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.c09ad17df905p4, -0x1.9b8a719f2b318p-48, -0x1.6c6f740e8840cp-103,`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.af89c00ed0004p-155},`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.af89c00ed0004p-155},`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.68befc827323cp5, -0x1.38cf9598c16c8p-47, 0x1.08bf177bf2508p-99,`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.68befc827323cp5, -0x1.38cf9598c16c8p-47, 0x1.08bf177bf2508p-99,`。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.3801da00087eap-152},`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.3801da00087eap-152},`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.037d8cdc538dp5, 0x1.a99cfa4e422fcp-49, 0x1.77bf250763ffp-103,`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.037d8cdc538dp5, 0x1.a99cfa4e422fcp-49, 0x1.77bf250763ffp-103,`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.2fffbc0b301fep-155},`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.2fffbc0b301fep-155},`。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.8cdc538cf9598p5, -0x1.82d8dee81d108p-48, -0x1.b5f13801dap-104,`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.8cdc538cf9598p5, -0x1.82d8dee81d108p-48, -0x1.b5f13801dap-104,`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.0fd33f8086877p-157},`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.0fd33f8086877p-157},`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.4e33e566305bp3, -0x1.bdd03a21036cp-49, 0x1.d8ffc4bffef04p-101,`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.4e33e566305bp3, -0x1.bdd03a21036cp-49, 0x1.d8ffc4bffef04p-101,`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.33f80868773a5p-153},`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.33f80868773a5p-153},`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.f2b3182d8dee8p4, -0x1.d1081b5f138p-52, -0x1.da00087e99fcp-104,`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.f2b3182d8dee8p4, -0x1.d1081b5f138p-52, -0x1.da00087e99fcp-104,`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.0d0ee74a5f593p-158},`.
  **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.0d0ee74a5f593p-158},`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.8c16c6f740e88p5, -0x1.036be27003b4p-49, -0x1.0fd33f8086878p-109,`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.8c16c6f740e88p5, -0x1.036be27003b4p-49, -0x1.0fd33f8086878p-109,`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.8b5a0a6d1f6d3p-162},`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.8b5a0a6d1f6d3p-162},`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3908bf177bf24p5, 0x1.0763ff12fffbcp-47, 0x1.6603fbcbc462cp-104,`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3908bf177bf24p5, 0x1.0763ff12fffbcp-47, 0x1.6603fbcbc462cp-104,`。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.6829b47db4dap-156},`.
  **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.6829b47db4dap-156},`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.7e2ef7e4a0ec8p4, -0x1.da00087e99fcp-56, -0x1.0d0ee74a5f594p-110,`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.7e2ef7e4a0ec8p4, -0x1.da00087e99fcp-56, -0x1.0d0ee74a5f594p-110,`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.1f6d367ecf27dp-162},`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.1f6d367ecf27dp-162},`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.081b5f13801dcp4, 0x1.fff7816603fbcp-48, 0x1.788c5ad05369p-101,`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.081b5f13801dcp4, 0x1.fff7816603fbcp-48, 0x1.788c5ad05369p-101,`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.25930261b069fp-155},`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.25930261b069fp-155},`。

### Lines 193-216

````cpp
    {-0x1.af89c00ed0004p5, -0x1.fa67f010d0ee8p-50, 0x1.6b414da3eda6cp-103,
     0x1.fb3c9f2c26dd4p-156},
    {-0x1.c00ed00043f4cp5, -0x1.fc04343b9d298p-48, 0x1.4da3eda6cfdap-103,
     -0x1.b069ec9161738p-155},
    {0x1.2fffbc0b301fcp5, 0x1.e5e2316b414dcp-47, -0x1.c125930261b08p-99,
     0x1.6136e9e8c7ecdp-151},
    {-0x1.0fd33f8086878p3, 0x1.8b5a0a6d1f6d4p-50, -0x1.30261b069ec9p-103,
     -0x1.61738132c3403p-155},
    {-0x1.9fc04343b9d28p4, -0x1.7d64b824b2604p-48, -0x1.86c1a7b24585cp-101,
     -0x1.c09961a015d29p-154},
    {-0x1.0d0ee74a5f594p2, 0x1.1f6d367ecf27cp-50, 0x1.6136e9e8c7eccp-103,
     0x1.3cbfd45aea4f7p-155},
    {-0x1.dce94beb25c14p5, 0x1.a6cfd9e4f9614p-47, -0x1.22c2e70265868p-100,
     -0x1.5d28ad8453814p-158},
    {-0x1.4beb25c12593p5, -0x1.30d834f648b0cp-50, 0x1.8fd9a797fa8b4p-104,
     0x1.d49eeb1faf97cp-156},
    {0x1.b47db4d9fb3c8p4, 0x1.f2c26dd3d18fcp-48, 0x1.9a797fa8b5d48p-100,
     0x1.eeb1faf97c5edp-152},
    {-0x1.25930261b06ap5, 0x1.36e9e8c7ecd3cp-47, 0x1.7fa8b5d49eebp-100,
     0x1.faf97c5ecf41dp-152},
    {0x1.fb3c9f2c26dd4p4, -0x1.738132c3402bcp-51, 0x1.aea4f758fd7ccp-103,
     -0x1.d0985f18c10ebp-159},
    {-0x1.b069ec9161738p5, -0x1.32c3402ba515cp-51, 0x1.eeb1faf97c5ecp-104,
     0x1.e839cfbc52949p-157},
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.af89c00ed0004p5, -0x1.fa67f010d0ee8p-50, 0x1.6b414da3eda6cp-103,`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.af89c00ed0004p5, -0x1.fa67f010d0ee8p-50, 0x1.6b414da3eda6cp-103,`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.fb3c9f2c26dd4p-156},`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.fb3c9f2c26dd4p-156},`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.c00ed00043f4cp5, -0x1.fc04343b9d298p-48, 0x1.4da3eda6cfdap-103,`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.c00ed00043f4cp5, -0x1.fc04343b9d298p-48, 0x1.4da3eda6cfdap-103,`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.b069ec9161738p-155},`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.b069ec9161738p-155},`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2fffbc0b301fcp5, 0x1.e5e2316b414dcp-47, -0x1.c125930261b08p-99,`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2fffbc0b301fcp5, 0x1.e5e2316b414dcp-47, -0x1.c125930261b08p-99,`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.6136e9e8c7ecdp-151},`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.6136e9e8c7ecdp-151},`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0fd33f8086878p3, 0x1.8b5a0a6d1f6d4p-50, -0x1.30261b069ec9p-103,`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0fd33f8086878p3, 0x1.8b5a0a6d1f6d4p-50, -0x1.30261b069ec9p-103,`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.61738132c3403p-155},`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.61738132c3403p-155},`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.9fc04343b9d28p4, -0x1.7d64b824b2604p-48, -0x1.86c1a7b24585cp-101,`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.9fc04343b9d28p4, -0x1.7d64b824b2604p-48, -0x1.86c1a7b24585cp-101,`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.c09961a015d29p-154},`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.c09961a015d29p-154},`。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0d0ee74a5f594p2, 0x1.1f6d367ecf27cp-50, 0x1.6136e9e8c7eccp-103,`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0d0ee74a5f594p2, 0x1.1f6d367ecf27cp-50, 0x1.6136e9e8c7eccp-103,`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.3cbfd45aea4f7p-155},`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.3cbfd45aea4f7p-155},`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.dce94beb25c14p5, 0x1.a6cfd9e4f9614p-47, -0x1.22c2e70265868p-100,`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.dce94beb25c14p5, 0x1.a6cfd9e4f9614p-47, -0x1.22c2e70265868p-100,`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.5d28ad8453814p-158},`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.5d28ad8453814p-158},`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.4beb25c12593p5, -0x1.30d834f648b0cp-50, 0x1.8fd9a797fa8b4p-104,`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.4beb25c12593p5, -0x1.30d834f648b0cp-50, 0x1.8fd9a797fa8b4p-104,`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.d49eeb1faf97cp-156},`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.d49eeb1faf97cp-156},`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.b47db4d9fb3c8p4, 0x1.f2c26dd3d18fcp-48, 0x1.9a797fa8b5d48p-100,`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.b47db4d9fb3c8p4, 0x1.f2c26dd3d18fcp-48, 0x1.9a797fa8b5d48p-100,`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.eeb1faf97c5edp-152},`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.eeb1faf97c5edp-152},`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.25930261b06ap5, 0x1.36e9e8c7ecd3cp-47, 0x1.7fa8b5d49eebp-100,`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.25930261b06ap5, 0x1.36e9e8c7ecd3cp-47, 0x1.7fa8b5d49eebp-100,`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.faf97c5ecf41dp-152},`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.faf97c5ecf41dp-152},`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.fb3c9f2c26dd4p4, -0x1.738132c3402bcp-51, 0x1.aea4f758fd7ccp-103,`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.fb3c9f2c26dd4p4, -0x1.738132c3402bcp-51, 0x1.aea4f758fd7ccp-103,`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.d0985f18c10ebp-159},`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.d0985f18c10ebp-159},`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.b069ec9161738p5, -0x1.32c3402ba515cp-51, 0x1.eeb1faf97c5ecp-104,`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.b069ec9161738p5, -0x1.32c3402ba515cp-51, 0x1.eeb1faf97c5ecp-104,`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.e839cfbc52949p-157},`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.e839cfbc52949p-157},`。

### Lines 217-240

````cpp
    {-0x1.ec9161738132cp5, -0x1.a015d28ad8454p-50, 0x1.faf97c5ecf41cp-104,
     0x1.cfbc529497536p-157},
    {-0x1.61738132c3404p5, 0x1.45aea4f758fd8p-47, -0x1.a0e84c2f8c608p-102,
     -0x1.d6b5b45650128p-156},
    {0x1.fb34f2ff516bcp3, -0x1.6c229c0a0d074p-49, -0x1.30be31821d6b4p-104,
     -0x1.b4565012813b8p-156},
    {0x1.3cbfd45aea4f8p5, -0x1.4e050683a130cp-48, 0x1.ce7de294a4ba8p-104,
     0x1.afed7ec47e357p-156},
    {-0x1.5d28ad8453814p2, -0x1.a0e84c2f8c608p-54, -0x1.d6b5b45650128p-108,
     -0x1.3b81ca8bdea7fp-164},
    {-0x1.15b08a702834p5, -0x1.d0985f18c10ecp-47, 0x1.4a4ba9afed7ecp-100,
     0x1.1f8d5d0856033p-154},
};

// For large range |x| >= 2^16, we perform the range reduction computations as:
//   u = x - k * pi/128 = (pi/128) * (x * (128/pi) - k).
// We use the exponent of x to find 4 double-chunks of 128/pi:
// c_hi, c_mid, c_lo, c_lo_2 such that:
//   1) ulp(round(x * c_hi, D, RN)) >= 2^8 = 256,
//   2) If x * c_hi = ph_hi + ph_lo and x * c_mid = pm_hi + pm_lo, then
//        min(ulp(ph_lo), ulp(pm_hi)) >= 2^-53.
// This will allow us to drop the high part ph_hi and the addition:
//   (ph_lo + pm_hi) mod 1
// can be exactly representable in a double precision.
````
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ec9161738132cp5, -0x1.a015d28ad8454p-50, 0x1.faf97c5ecf41cp-104,`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ec9161738132cp5, -0x1.a015d28ad8454p-50, 0x1.faf97c5ecf41cp-104,`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.cfbc529497536p-157},`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.cfbc529497536p-157},`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.61738132c3404p5, 0x1.45aea4f758fd8p-47, -0x1.a0e84c2f8c608p-102,`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.61738132c3404p5, 0x1.45aea4f758fd8p-47, -0x1.a0e84c2f8c608p-102,`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.d6b5b45650128p-156},`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.d6b5b45650128p-156},`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.fb34f2ff516bcp3, -0x1.6c229c0a0d074p-49, -0x1.30be31821d6b4p-104,`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.fb34f2ff516bcp3, -0x1.6c229c0a0d074p-49, -0x1.30be31821d6b4p-104,`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.b4565012813b8p-156},`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.b4565012813b8p-156},`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3cbfd45aea4f8p5, -0x1.4e050683a130cp-48, 0x1.ce7de294a4ba8p-104,`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3cbfd45aea4f8p5, -0x1.4e050683a130cp-48, 0x1.ce7de294a4ba8p-104,`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.afed7ec47e357p-156},`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.afed7ec47e357p-156},`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5d28ad8453814p2, -0x1.a0e84c2f8c608p-54, -0x1.d6b5b45650128p-108,`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5d28ad8453814p2, -0x1.a0e84c2f8c608p-54, -0x1.d6b5b45650128p-108,`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-0x1.3b81ca8bdea7fp-164},`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`-0x1.3b81ca8bdea7fp-164},`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.15b08a702834p5, -0x1.d0985f18c10ecp-47, 0x1.4a4ba9afed7ecp-100,`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.15b08a702834p5, -0x1.d0985f18c10ecp-47, 0x1.4a4ba9afed7ecp-100,`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.1f8d5d0856033p-154},`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.1f8d5d0856033p-154},`。
- **L229 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L229 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L230 EN**: Blank line separating nearby declarations or logic.
  **L230 CN**: 空行，用于分隔相邻声明或逻辑。
- **L231 EN**: Comment documents nearby intent or constraints: `For large range \|x\| >= 2^16, we perform the range reduction computations as:`.
  **L231 CN**: 注释说明附近代码的意图或约束：`For large range \|x\| >= 2^16, we perform the range reduction computations as:`。
- **L232 EN**: Comment documents nearby intent or constraints: `u = x - k * pi/128 = (pi/128) * (x * (128/pi) - k).`.
  **L232 CN**: 注释说明附近代码的意图或约束：`u = x - k * pi/128 = (pi/128) * (x * (128/pi) - k).`。
- **L233 EN**: Comment documents nearby intent or constraints: `We use the exponent of x to find 4 double-chunks of 128/pi:`.
  **L233 CN**: 注释说明附近代码的意图或约束：`We use the exponent of x to find 4 double-chunks of 128/pi:`。
- **L234 EN**: Comment documents nearby intent or constraints: `c_hi, c_mid, c_lo, c_lo_2 such that:`.
  **L234 CN**: 注释说明附近代码的意图或约束：`c_hi, c_mid, c_lo, c_lo_2 such that:`。
- **L235 EN**: Comment documents nearby intent or constraints: `1) ulp(round(x * c_hi, D, RN)) >= 2^8 = 256,`.
  **L235 CN**: 注释说明附近代码的意图或约束：`1) ulp(round(x * c_hi, D, RN)) >= 2^8 = 256,`。
- **L236 EN**: Comment documents nearby intent or constraints: `2) If x * c_hi = ph_hi + ph_lo and x * c_mid = pm_hi + pm_lo, then`.
  **L236 CN**: 注释说明附近代码的意图或约束：`2) If x * c_hi = ph_hi + ph_lo and x * c_mid = pm_hi + pm_lo, then`。
- **L237 EN**: Comment documents nearby intent or constraints: `min(ulp(ph_lo), ulp(pm_hi)) >= 2^-53.`.
  **L237 CN**: 注释说明附近代码的意图或约束：`min(ulp(ph_lo), ulp(pm_hi)) >= 2^-53.`。
- **L238 EN**: Comment documents nearby intent or constraints: `This will allow us to drop the high part ph_hi and the addition:`.
  **L238 CN**: 注释说明附近代码的意图或约束：`This will allow us to drop the high part ph_hi and the addition:`。
- **L239 EN**: Comment documents nearby intent or constraints: `(ph_lo + pm_hi) mod 1`.
  **L239 CN**: 注释说明附近代码的意图或约束：`(ph_lo + pm_hi) mod 1`。
- **L240 EN**: Comment documents nearby intent or constraints: `can be exactly representable in a double precision.`.
  **L240 CN**: 注释说明附近代码的意图或约束：`can be exactly representable in a double precision.`。

### Lines 241-264

````cpp
// This will allow us to do split the computations as:
//   (x * 256/pi) ~ x * (c_hi + c_mid + c_lo + c_lo_2)    (mod 256)
//                ~ (ph_lo + pm_hi) + (pm_lo + x * c_lo) + x * c_lo_2.
// Then,
//   round(x * 128/pi) = round(ph_lo + pm_hi)    (mod 256)
// And the high part of fractional part of (x * 128/pi) can simply be:
//   {x * 128/pi}_hi = {ph_lo + pm_hi}.
// To prevent overflow when x is very large, we simply scale up
// (c_hi, c_mid, c_lo, c_lo_2) by a fixed power of 2 (based on the index) and
// scale down x by the same amount.

struct LargeRangeReduction {

  // To be implemented in range_reduction_double_fma.h and
  // range_reduction_double_nofma.h.
  unsigned fast(double x, DoubleDouble &u);

#ifndef LIBC_MATH_HAS_SKIP_ACCURATE_PASS
  LIBC_INLINE Float128 accurate() const {
    constexpr Float128 PI_OVER_128_F128 = {
        Sign::POS, -133, 0xc90f'daa2'2168'c234'c4c6'628b'80dc'1cd1_u128};

    // y_lo = x * c_lo + pm.lo
    Float128 y_lo_0(x_reduced * ONE_TWENTY_EIGHT_OVER_PI[idx][3]);
````
- **L241 EN**: Comment documents nearby intent or constraints: `This will allow us to do split the computations as:`.
  **L241 CN**: 注释说明附近代码的意图或约束：`This will allow us to do split the computations as:`。
- **L242 EN**: Comment documents nearby intent or constraints: `(x * 256/pi) ~ x * (c_hi + c_mid + c_lo + c_lo_2)    (mod 256)`.
  **L242 CN**: 注释说明附近代码的意图或约束：`(x * 256/pi) ~ x * (c_hi + c_mid + c_lo + c_lo_2)    (mod 256)`。
- **L243 EN**: Comment documents nearby intent or constraints: `~ (ph_lo + pm_hi) + (pm_lo + x * c_lo) + x * c_lo_2.`.
  **L243 CN**: 注释说明附近代码的意图或约束：`~ (ph_lo + pm_hi) + (pm_lo + x * c_lo) + x * c_lo_2.`。
- **L244 EN**: Comment documents nearby intent or constraints: `Then,`.
  **L244 CN**: 注释说明附近代码的意图或约束：`Then,`。
- **L245 EN**: Comment documents nearby intent or constraints: `round(x * 128/pi) = round(ph_lo + pm_hi)    (mod 256)`.
  **L245 CN**: 注释说明附近代码的意图或约束：`round(x * 128/pi) = round(ph_lo + pm_hi)    (mod 256)`。
- **L246 EN**: Comment documents nearby intent or constraints: `And the high part of fractional part of (x * 128/pi) can simply be:`.
  **L246 CN**: 注释说明附近代码的意图或约束：`And the high part of fractional part of (x * 128/pi) can simply be:`。
- **L247 EN**: Comment documents nearby intent or constraints: `{x * 128/pi}_hi = {ph_lo + pm_hi}.`.
  **L247 CN**: 注释说明附近代码的意图或约束：`{x * 128/pi}_hi = {ph_lo + pm_hi}.`。
- **L248 EN**: Comment documents nearby intent or constraints: `To prevent overflow when x is very large, we simply scale up`.
  **L248 CN**: 注释说明附近代码的意图或约束：`To prevent overflow when x is very large, we simply scale up`。
- **L249 EN**: Comment documents nearby intent or constraints: `(c_hi, c_mid, c_lo, c_lo_2) by a fixed power of 2 (based on the index) and`.
  **L249 CN**: 注释说明附近代码的意图或约束：`(c_hi, c_mid, c_lo, c_lo_2) by a fixed power of 2 (based on the index) and`。
- **L250 EN**: Comment documents nearby intent or constraints: `scale down x by the same amount.`.
  **L250 CN**: 注释说明附近代码的意图或约束：`scale down x by the same amount.`。
- **L251 EN**: Blank line separating nearby declarations or logic.
  **L251 CN**: 空行，用于分隔相邻声明或逻辑。
- **L252 EN**: Declares struct `LargeRangeReduction`.
  **L252 CN**: 声明 struct `LargeRangeReduction`。
- **L253 EN**: Blank line separating nearby declarations or logic.
  **L253 CN**: 空行，用于分隔相邻声明或逻辑。
- **L254 EN**: Comment documents nearby intent or constraints: `To be implemented in range_reduction_double_fma.h and`.
  **L254 CN**: 注释说明附近代码的意图或约束：`To be implemented in range_reduction_double_fma.h and`。
- **L255 EN**: Comment documents nearby intent or constraints: `range_reduction_double_nofma.h.`.
  **L255 CN**: 注释说明附近代码的意图或约束：`range_reduction_double_nofma.h.`。
- **L256 EN**: Executes a call or declaration centered on `fast`.
  **L256 CN**: 执行以 `fast` 为核心的调用或声明。
- **L257 EN**: Blank line separating nearby declarations or logic.
  **L257 CN**: 空行，用于分隔相邻声明或逻辑。
- **L258 EN**: Starts a header guard condition: `#ifndef LIBC_MATH_HAS_SKIP_ACCURATE_PASS`.
  **L258 CN**: 开始头文件保护条件：`#ifndef LIBC_MATH_HAS_SKIP_ACCURATE_PASS`。
- **L259 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L259 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L260 EN**: Continues the surrounding expression or declaration: `constexpr Float128 PI_OVER_128_F128 = {`.
  **L260 CN**: 继续构造周围的表达式或声明：`constexpr Float128 PI_OVER_128_F128 = {`。
- **L261 EN**: Executes a standalone statement or declaration: `Sign::POS, -133, 0xc90f'daa2'2168'c234'c4c6'628b'80dc'1cd1_u128};`.
  **L261 CN**: 执行一条独立语句或声明：`Sign::POS, -133, 0xc90f'daa2'2168'c234'c4c6'628b'80dc'1cd1_u128};`。
- **L262 EN**: Blank line separating nearby declarations or logic.
  **L262 CN**: 空行，用于分隔相邻声明或逻辑。
- **L263 EN**: Comment documents nearby intent or constraints: `y_lo = x * c_lo + pm.lo`.
  **L263 CN**: 注释说明附近代码的意图或约束：`y_lo = x * c_lo + pm.lo`。
- **L264 EN**: Executes a call or declaration centered on `y_lo_0`.
  **L264 CN**: 执行以 `y_lo_0` 为核心的调用或声明。

### Lines 265-288

````cpp
    Float128 y_lo_1 = fputil::quick_add(Float128(y_lo), y_lo_0);
    Float128 y_mid_f128 = fputil::quick_add(Float128(y_mid.lo), y_lo_1);
    Float128 y_hi_f128 = fputil::quick_add(Float128(y_hi), Float128(y_mid.hi));
    Float128 y = fputil::quick_add(y_hi_f128, y_mid_f128);

    return fputil::quick_mul(y, PI_OVER_128_F128);
  }
#endif // !LIBC_MATH_HAS_SKIP_ACCURATE_PASS

  LIBC_INLINE LargeRangeReduction() = default;

private:
  // Index of x in the look-up table ONE_TWENTY_EIGHT_OVER_PI.
  unsigned idx = 0;
  // x scaled down by 2^(-16 *(idx - 3))).
  double x_reduced = 0;
  // Parts of (x * 128/pi) mod 1.
  double y_hi = 0, y_lo = 0;
  DoubleDouble y_mid;
};

#ifndef LIBC_MATH_HAS_SKIP_ACCURATE_PASS
LIBC_INLINE Float128 range_reduction_small_f128(double x) {
  constexpr Float128 PI_OVER_128_F128 = {
````
- **L265 EN**: Initializes variable `y_lo_1` from the right-hand expression.
  **L265 CN**: 使用右侧表达式初始化变量 `y_lo_1`。
- **L266 EN**: Initializes variable `y_mid_f128` from the right-hand expression.
  **L266 CN**: 使用右侧表达式初始化变量 `y_mid_f128`。
- **L267 EN**: Initializes variable `y_hi_f128` from the right-hand expression.
  **L267 CN**: 使用右侧表达式初始化变量 `y_hi_f128`。
- **L268 EN**: Initializes variable `y` from the right-hand expression.
  **L268 CN**: 使用右侧表达式初始化变量 `y`。
- **L269 EN**: Blank line separating nearby declarations or logic.
  **L269 CN**: 空行，用于分隔相邻声明或逻辑。
- **L270 EN**: Returns from the current function with `fputil::quick_mul(y, PI_OVER_128_F128)`.
  **L270 CN**: 以 `fputil::quick_mul(y, PI_OVER_128_F128)` 从当前函数返回。
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Closes the current preprocessor conditional block or header guard.
  **L272 CN**: 结束当前预处理条件块或头文件保护。
- **L273 EN**: Blank line separating nearby declarations or logic.
  **L273 CN**: 空行，用于分隔相邻声明或逻辑。
- **L274 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L274 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L275 EN**: Blank line separating nearby declarations or logic.
  **L275 CN**: 空行，用于分隔相邻声明或逻辑。
- **L276 EN**: Sets the following members to `private` access.
  **L276 CN**: 将后续成员的访问级别设为 `private`。
- **L277 EN**: Comment documents nearby intent or constraints: `Index of x in the look-up table ONE_TWENTY_EIGHT_OVER_PI.`.
  **L277 CN**: 注释说明附近代码的意图或约束：`Index of x in the look-up table ONE_TWENTY_EIGHT_OVER_PI.`。
- **L278 EN**: Initializes variable `idx` from the right-hand expression.
  **L278 CN**: 使用右侧表达式初始化变量 `idx`。
- **L279 EN**: Comment documents nearby intent or constraints: `x scaled down by 2^(-16 *(idx - 3))).`.
  **L279 CN**: 注释说明附近代码的意图或约束：`x scaled down by 2^(-16 *(idx - 3))).`。
- **L280 EN**: Initializes variable `x_reduced` from the right-hand expression.
  **L280 CN**: 使用右侧表达式初始化变量 `x_reduced`。
- **L281 EN**: Comment documents nearby intent or constraints: `Parts of (x * 128/pi) mod 1.`.
  **L281 CN**: 注释说明附近代码的意图或约束：`Parts of (x * 128/pi) mod 1.`。
- **L282 EN**: Initializes variable `y_hi` from the right-hand expression.
  **L282 CN**: 使用右侧表达式初始化变量 `y_hi`。
- **L283 EN**: Executes a standalone statement or declaration: `DoubleDouble y_mid;`.
  **L283 CN**: 执行一条独立语句或声明：`DoubleDouble y_mid;`。
- **L284 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L284 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L285 EN**: Blank line separating nearby declarations or logic.
  **L285 CN**: 空行，用于分隔相邻声明或逻辑。
- **L286 EN**: Starts a header guard condition: `#ifndef LIBC_MATH_HAS_SKIP_ACCURATE_PASS`.
  **L286 CN**: 开始头文件保护条件：`#ifndef LIBC_MATH_HAS_SKIP_ACCURATE_PASS`。
- **L287 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L287 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L288 EN**: Continues the surrounding expression or declaration: `constexpr Float128 PI_OVER_128_F128 = {`.
  **L288 CN**: 继续构造周围的表达式或声明：`constexpr Float128 PI_OVER_128_F128 = {`。

### Lines 289-312

````cpp
      Sign::POS, -133, 0xc90f'daa2'2168'c234'c4c6'628b'80dc'1cd1_u128};
  constexpr double ONE_TWENTY_EIGHT_OVER_PI_D = 0x1.45f306dc9c883p5;
  double prod_hi = x * ONE_TWENTY_EIGHT_OVER_PI_D;
  double kd = fputil::nearest_integer(prod_hi);

  Float128 mk_f128(-kd);
  Float128 x_f128(x);
  Float128 p_hi =
      fputil::quick_mul(x_f128, Float128(ONE_TWENTY_EIGHT_OVER_PI[3][0]));
  Float128 p_mid =
      fputil::quick_mul(x_f128, Float128(ONE_TWENTY_EIGHT_OVER_PI[3][1]));
  Float128 p_lo =
      fputil::quick_mul(x_f128, Float128(ONE_TWENTY_EIGHT_OVER_PI[3][2]));
  Float128 s_hi = fputil::quick_add(p_hi, mk_f128);
  Float128 s_lo = fputil::quick_add(p_mid, p_lo);
  Float128 y = fputil::quick_add(s_hi, s_lo);

  return fputil::quick_mul(y, PI_OVER_128_F128);
}

LIBC_INLINE_VAR constexpr Float128 SIN_K_PI_OVER_128_F128[65] = {
    {Sign::POS, 0, 0},
    {Sign::POS, -133, 0xc90a'afbd'1b33'efc9'c539'edcb'fda0'cf2c_u128},
    {Sign::POS, -132, 0xc8fb'2f88'6ec0'9f37'6a17'954b'2b7c'5171_u128},
````
- **L289 EN**: Executes a standalone statement or declaration: `Sign::POS, -133, 0xc90f'daa2'2168'c234'c4c6'628b'80dc'1cd1_u128};`.
  **L289 CN**: 执行一条独立语句或声明：`Sign::POS, -133, 0xc90f'daa2'2168'c234'c4c6'628b'80dc'1cd1_u128};`。
- **L290 EN**: Initializes variable `ONE_TWENTY_EIGHT_OVER_PI_D` from the right-hand expression.
  **L290 CN**: 使用右侧表达式初始化变量 `ONE_TWENTY_EIGHT_OVER_PI_D`。
- **L291 EN**: Initializes variable `prod_hi` from the right-hand expression.
  **L291 CN**: 使用右侧表达式初始化变量 `prod_hi`。
- **L292 EN**: Initializes variable `kd` from the right-hand expression.
  **L292 CN**: 使用右侧表达式初始化变量 `kd`。
- **L293 EN**: Blank line separating nearby declarations or logic.
  **L293 CN**: 空行，用于分隔相邻声明或逻辑。
- **L294 EN**: Executes a call or declaration centered on `mk_f128`.
  **L294 CN**: 执行以 `mk_f128` 为核心的调用或声明。
- **L295 EN**: Executes a call or declaration centered on `x_f128`.
  **L295 CN**: 执行以 `x_f128` 为核心的调用或声明。
- **L296 EN**: Continues the surrounding expression or declaration: `Float128 p_hi =`.
  **L296 CN**: 继续构造周围的表达式或声明：`Float128 p_hi =`。
- **L297 EN**: Executes a call or declaration centered on `fputil::quick_mul`.
  **L297 CN**: 执行以 `fputil::quick_mul` 为核心的调用或声明。
- **L298 EN**: Continues the surrounding expression or declaration: `Float128 p_mid =`.
  **L298 CN**: 继续构造周围的表达式或声明：`Float128 p_mid =`。
- **L299 EN**: Executes a call or declaration centered on `fputil::quick_mul`.
  **L299 CN**: 执行以 `fputil::quick_mul` 为核心的调用或声明。
- **L300 EN**: Continues the surrounding expression or declaration: `Float128 p_lo =`.
  **L300 CN**: 继续构造周围的表达式或声明：`Float128 p_lo =`。
- **L301 EN**: Executes a call or declaration centered on `fputil::quick_mul`.
  **L301 CN**: 执行以 `fputil::quick_mul` 为核心的调用或声明。
- **L302 EN**: Initializes variable `s_hi` from the right-hand expression.
  **L302 CN**: 使用右侧表达式初始化变量 `s_hi`。
- **L303 EN**: Initializes variable `s_lo` from the right-hand expression.
  **L303 CN**: 使用右侧表达式初始化变量 `s_lo`。
- **L304 EN**: Initializes variable `y` from the right-hand expression.
  **L304 CN**: 使用右侧表达式初始化变量 `y`。
- **L305 EN**: Blank line separating nearby declarations or logic.
  **L305 CN**: 空行，用于分隔相邻声明或逻辑。
- **L306 EN**: Returns from the current function with `fputil::quick_mul(y, PI_OVER_128_F128)`.
  **L306 CN**: 以 `fputil::quick_mul(y, PI_OVER_128_F128)` 从当前函数返回。
- **L307 EN**: Closes the current lexical scope or compound statement.
  **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Blank line separating nearby declarations or logic.
  **L308 CN**: 空行，用于分隔相邻声明或逻辑。
- **L309 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L309 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, 0, 0},`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, 0, 0},`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -133, 0xc90a'afbd'1b33'efc9'c539'edcb'fda0'cf2c_u128},`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -133, 0xc90a'afbd'1b33'efc9'c539'edcb'fda0'cf2c_u128},`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -132, 0xc8fb'2f88'6ec0'9f37'6a17'954b'2b7c'5171_u128},`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -132, 0xc8fb'2f88'6ec0'9f37'6a17'954b'2b7c'5171_u128},`。

### Lines 313-336

````cpp
    {Sign::POS, -131, 0x96a9'0496'70cf'ae65'f775'7409'4d3c'35c4_u128},
    {Sign::POS, -131, 0xc8bd'35e1'4da1'5f0e'c739'6c89'4bbf'7389_u128},
    {Sign::POS, -131, 0xfab2'72b5'4b98'71a2'7047'29ae'56d7'8a37_u128},
    {Sign::POS, -130, 0x9640'8374'7309'd113'000a'89a1'1e07'c1fe_u128},
    {Sign::POS, -130, 0xaf10'a224'59fe'32a6'3fee'f3bb'58b1'f10d_u128},
    {Sign::POS, -130, 0xc7c5'c1e3'4d30'55b2'5cc8'c00e'4fcc'd850_u128},
    {Sign::POS, -130, 0xe05c'1353'f27b'17e5'0ebc'61ad'e6ca'83cd_u128},
    {Sign::POS, -130, 0xf8cf'cbd9'0af8'd57a'4221'dc4b'a772'598d_u128},
    {Sign::POS, -129, 0x888e'9315'8fb3'bb04'9841'56f5'5334'4306_u128},
    {Sign::POS, -129, 0x94a0'3176'acf8'2d45'ae4b'a773'da6b'f754_u128},
    {Sign::POS, -129, 0xa09a'e4a0'bb30'0a19'2f89'5f44'a303'cc0b_u128},
    {Sign::POS, -129, 0xac7c'd3ad'58fe'e7f0'811f'9539'84ef'f83e_u128},
    {Sign::POS, -129, 0xb844'2987'd22c'f576'9cc3'ef36'746d'e3b8_u128},
    {Sign::POS, -129, 0xc3ef'1535'754b'168d'3122'c2a5'9efd'dc37_u128},
    {Sign::POS, -129, 0xcf7b'ca1d'476c'516d'a812'90bd'baad'62e4_u128},
    {Sign::POS, -129, 0xdae8'804f'0ae6'015b'362c'b974'182e'3030_u128},
    {Sign::POS, -129, 0xe633'74c9'8e22'f0b4'2872'ce1b'fc7a'd1cd_u128},
    {Sign::POS, -129, 0xf15a'e9c0'37b1'd8f0'6c48'e9e3'420b'0f1e_u128},
    {Sign::POS, -129, 0xfc5d'26df'c4d5'cfda'27c0'7c91'1290'b8d1_u128},
    {Sign::POS, -128, 0x839c'3cc9'17ff'6cb4'bfd7'9717'f288'0abf_u128},
    {Sign::POS, -128, 0x88f5'9aa0'da59'1421'b892'ca83'61d8'c84c_u128},
    {Sign::POS, -128, 0x8e39'd9cd'7346'4364'bba4'cfec'bff5'4867_u128},
    {Sign::POS, -128, 0x9368'2a66'e896'f544'b178'2191'1e71'c16e_u128},
    {Sign::POS, -128, 0x987f'bfe7'0b81'a708'19ce'c845'ac87'a5c6_u128},
````
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -131, 0x96a9'0496'70cf'ae65'f775'7409'4d3c'35c4_u128},`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -131, 0x96a9'0496'70cf'ae65'f775'7409'4d3c'35c4_u128},`。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -131, 0xc8bd'35e1'4da1'5f0e'c739'6c89'4bbf'7389_u128},`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -131, 0xc8bd'35e1'4da1'5f0e'c739'6c89'4bbf'7389_u128},`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -131, 0xfab2'72b5'4b98'71a2'7047'29ae'56d7'8a37_u128},`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -131, 0xfab2'72b5'4b98'71a2'7047'29ae'56d7'8a37_u128},`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -130, 0x9640'8374'7309'd113'000a'89a1'1e07'c1fe_u128},`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -130, 0x9640'8374'7309'd113'000a'89a1'1e07'c1fe_u128},`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -130, 0xaf10'a224'59fe'32a6'3fee'f3bb'58b1'f10d_u128},`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -130, 0xaf10'a224'59fe'32a6'3fee'f3bb'58b1'f10d_u128},`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -130, 0xc7c5'c1e3'4d30'55b2'5cc8'c00e'4fcc'd850_u128},`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -130, 0xc7c5'c1e3'4d30'55b2'5cc8'c00e'4fcc'd850_u128},`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -130, 0xe05c'1353'f27b'17e5'0ebc'61ad'e6ca'83cd_u128},`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -130, 0xe05c'1353'f27b'17e5'0ebc'61ad'e6ca'83cd_u128},`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -130, 0xf8cf'cbd9'0af8'd57a'4221'dc4b'a772'598d_u128},`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -130, 0xf8cf'cbd9'0af8'd57a'4221'dc4b'a772'598d_u128},`。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0x888e'9315'8fb3'bb04'9841'56f5'5334'4306_u128},`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0x888e'9315'8fb3'bb04'9841'56f5'5334'4306_u128},`。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0x94a0'3176'acf8'2d45'ae4b'a773'da6b'f754_u128},`.
  **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0x94a0'3176'acf8'2d45'ae4b'a773'da6b'f754_u128},`。
- **L323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xa09a'e4a0'bb30'0a19'2f89'5f44'a303'cc0b_u128},`.
  **L323 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xa09a'e4a0'bb30'0a19'2f89'5f44'a303'cc0b_u128},`。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xac7c'd3ad'58fe'e7f0'811f'9539'84ef'f83e_u128},`.
  **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xac7c'd3ad'58fe'e7f0'811f'9539'84ef'f83e_u128},`。
- **L325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xb844'2987'd22c'f576'9cc3'ef36'746d'e3b8_u128},`.
  **L325 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xb844'2987'd22c'f576'9cc3'ef36'746d'e3b8_u128},`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xc3ef'1535'754b'168d'3122'c2a5'9efd'dc37_u128},`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xc3ef'1535'754b'168d'3122'c2a5'9efd'dc37_u128},`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xcf7b'ca1d'476c'516d'a812'90bd'baad'62e4_u128},`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xcf7b'ca1d'476c'516d'a812'90bd'baad'62e4_u128},`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xdae8'804f'0ae6'015b'362c'b974'182e'3030_u128},`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xdae8'804f'0ae6'015b'362c'b974'182e'3030_u128},`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xe633'74c9'8e22'f0b4'2872'ce1b'fc7a'd1cd_u128},`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xe633'74c9'8e22'f0b4'2872'ce1b'fc7a'd1cd_u128},`。
- **L330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xf15a'e9c0'37b1'd8f0'6c48'e9e3'420b'0f1e_u128},`.
  **L330 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xf15a'e9c0'37b1'd8f0'6c48'e9e3'420b'0f1e_u128},`。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -129, 0xfc5d'26df'c4d5'cfda'27c0'7c91'1290'b8d1_u128},`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -129, 0xfc5d'26df'c4d5'cfda'27c0'7c91'1290'b8d1_u128},`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0x839c'3cc9'17ff'6cb4'bfd7'9717'f288'0abf_u128},`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0x839c'3cc9'17ff'6cb4'bfd7'9717'f288'0abf_u128},`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0x88f5'9aa0'da59'1421'b892'ca83'61d8'c84c_u128},`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0x88f5'9aa0'da59'1421'b892'ca83'61d8'c84c_u128},`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0x8e39'd9cd'7346'4364'bba4'cfec'bff5'4867_u128},`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0x8e39'd9cd'7346'4364'bba4'cfec'bff5'4867_u128},`。
- **L335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0x9368'2a66'e896'f544'b178'2191'1e71'c16e_u128},`.
  **L335 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0x9368'2a66'e896'f544'b178'2191'1e71'c16e_u128},`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0x987f'bfe7'0b81'a708'19ce'c845'ac87'a5c6_u128},`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0x987f'bfe7'0b81'a708'19ce'c845'ac87'a5c6_u128},`。

### Lines 337-360

````cpp
    {Sign::POS, -128, 0x9d7f'd149'0285'c9e3'e25e'3954'9638'ae68_u128},
    {Sign::POS, -128, 0xa267'9928'48ee'b0c0'3b51'67ee'359a'234e_u128},
    {Sign::POS, -128, 0xa736'55df'1f2f'489e'149f'6e75'9934'68a3_u128},
    {Sign::POS, -128, 0xabeb'49a4'6764'fd15'1bec'da80'89c1'a94c_u128},
    {Sign::POS, -128, 0xb085'baa8'e966'f6da'e4ca'd00d'5c94'bcd2_u128},
    {Sign::POS, -128, 0xb504'f333'f9de'6484'597d'89b3'754a'be9f_u128},
    {Sign::POS, -128, 0xb968'41bf'7ffc'b21a'9de1'e3b2'2b8b'f4db_u128},
    {Sign::POS, -128, 0xbdae'f913'557d'76f0'ac85'320f'528d'6d5d_u128},
    {Sign::POS, -128, 0xc1d8'705f'fcbb'6e90'bdf0'715c'b8b2'0bd7_u128},
    {Sign::POS, -128, 0xc5e4'0358'a8ba'05a7'43da'25d9'9267'326b_u128},
    {Sign::POS, -128, 0xc9d1'124c'931f'da7a'8335'241b'e169'3225_u128},
    {Sign::POS, -128, 0xcd9f'023f'9c3a'059e'23af'31db'7179'a4aa_u128},
    {Sign::POS, -128, 0xd14d'3d02'313c'0eed'744f'ea20'e8ab'ef92_u128},
    {Sign::POS, -128, 0xd4db'3148'750d'1819'f630'e8b6'dac8'3e69_u128},
    {Sign::POS, -128, 0xd848'52c0'a80f'fcdb'24b9'fe00'6635'74a4_u128},
    {Sign::POS, -128, 0xdb94'1a28'cb71'ec87'2c19'b632'53da'43fc_u128},
    {Sign::POS, -128, 0xdebe'0563'7ca9'4cfb'4b19'aa71'fec3'ae6d_u128},
    {Sign::POS, -128, 0xe1c5'978c'05ed'8691'f4e8'a837'2f8c'5810_u128},
    {Sign::POS, -128, 0xe4aa'5909'a08f'a7b4'1227'85ae'67f5'515d_u128},
    {Sign::POS, -128, 0xe76b'd7a1'e63b'9786'1251'2952'9d48'a92f_u128},
    {Sign::POS, -128, 0xea09'a68a'6e49'cd62'15ad'45b4'a1b5'e823_u128},
    {Sign::POS, -128, 0xec83'5e79'946a'3145'7e61'0231'ac1d'6181_u128},
    {Sign::POS, -128, 0xeed8'9db6'6611'e307'86f8'c20f'b664'b01b_u128},
    {Sign::POS, -128, 0xf109'0827'b437'25fd'6712'7db3'5b28'7316_u128},
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0x9d7f'd149'0285'c9e3'e25e'3954'9638'ae68_u128},`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0x9d7f'd149'0285'c9e3'e25e'3954'9638'ae68_u128},`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xa267'9928'48ee'b0c0'3b51'67ee'359a'234e_u128},`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xa267'9928'48ee'b0c0'3b51'67ee'359a'234e_u128},`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xa736'55df'1f2f'489e'149f'6e75'9934'68a3_u128},`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xa736'55df'1f2f'489e'149f'6e75'9934'68a3_u128},`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xabeb'49a4'6764'fd15'1bec'da80'89c1'a94c_u128},`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xabeb'49a4'6764'fd15'1bec'da80'89c1'a94c_u128},`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xb085'baa8'e966'f6da'e4ca'd00d'5c94'bcd2_u128},`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xb085'baa8'e966'f6da'e4ca'd00d'5c94'bcd2_u128},`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xb504'f333'f9de'6484'597d'89b3'754a'be9f_u128},`.
  **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xb504'f333'f9de'6484'597d'89b3'754a'be9f_u128},`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xb968'41bf'7ffc'b21a'9de1'e3b2'2b8b'f4db_u128},`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xb968'41bf'7ffc'b21a'9de1'e3b2'2b8b'f4db_u128},`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xbdae'f913'557d'76f0'ac85'320f'528d'6d5d_u128},`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xbdae'f913'557d'76f0'ac85'320f'528d'6d5d_u128},`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xc1d8'705f'fcbb'6e90'bdf0'715c'b8b2'0bd7_u128},`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xc1d8'705f'fcbb'6e90'bdf0'715c'b8b2'0bd7_u128},`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xc5e4'0358'a8ba'05a7'43da'25d9'9267'326b_u128},`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xc5e4'0358'a8ba'05a7'43da'25d9'9267'326b_u128},`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xc9d1'124c'931f'da7a'8335'241b'e169'3225_u128},`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xc9d1'124c'931f'da7a'8335'241b'e169'3225_u128},`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xcd9f'023f'9c3a'059e'23af'31db'7179'a4aa_u128},`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xcd9f'023f'9c3a'059e'23af'31db'7179'a4aa_u128},`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xd14d'3d02'313c'0eed'744f'ea20'e8ab'ef92_u128},`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xd14d'3d02'313c'0eed'744f'ea20'e8ab'ef92_u128},`。
- **L350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xd4db'3148'750d'1819'f630'e8b6'dac8'3e69_u128},`.
  **L350 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xd4db'3148'750d'1819'f630'e8b6'dac8'3e69_u128},`。
- **L351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xd848'52c0'a80f'fcdb'24b9'fe00'6635'74a4_u128},`.
  **L351 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xd848'52c0'a80f'fcdb'24b9'fe00'6635'74a4_u128},`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xdb94'1a28'cb71'ec87'2c19'b632'53da'43fc_u128},`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xdb94'1a28'cb71'ec87'2c19'b632'53da'43fc_u128},`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xdebe'0563'7ca9'4cfb'4b19'aa71'fec3'ae6d_u128},`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xdebe'0563'7ca9'4cfb'4b19'aa71'fec3'ae6d_u128},`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xe1c5'978c'05ed'8691'f4e8'a837'2f8c'5810_u128},`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xe1c5'978c'05ed'8691'f4e8'a837'2f8c'5810_u128},`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xe4aa'5909'a08f'a7b4'1227'85ae'67f5'515d_u128},`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xe4aa'5909'a08f'a7b4'1227'85ae'67f5'515d_u128},`。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xe76b'd7a1'e63b'9786'1251'2952'9d48'a92f_u128},`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xe76b'd7a1'e63b'9786'1251'2952'9d48'a92f_u128},`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xea09'a68a'6e49'cd62'15ad'45b4'a1b5'e823_u128},`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xea09'a68a'6e49'cd62'15ad'45b4'a1b5'e823_u128},`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xec83'5e79'946a'3145'7e61'0231'ac1d'6181_u128},`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xec83'5e79'946a'3145'7e61'0231'ac1d'6181_u128},`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xeed8'9db6'6611'e307'86f8'c20f'b664'b01b_u128},`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xeed8'9db6'6611'e307'86f8'c20f'b664'b01b_u128},`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xf109'0827'b437'25fd'6712'7db3'5b28'7316_u128},`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xf109'0827'b437'25fd'6712'7db3'5b28'7316_u128},`。

### Lines 361-384

````cpp
    {Sign::POS, -128, 0xf314'4762'4708'8f74'a548'6bdc'455d'56a2_u128},
    {Sign::POS, -128, 0xf4fa'0ab6'316e'd2ec'163c'5c7f'03b7'18c5_u128},
    {Sign::POS, -128, 0xf6ba'073b'424b'19e8'2c79'1f59'cc1f'fc23_u128},
    {Sign::POS, -128, 0xf853'f7dc'9186'b952'c7ad'c6b4'9888'91bb_u128},
    {Sign::POS, -128, 0xf9c7'9d63'272c'4628'4504'ae08'd19b'2980_u128},
    {Sign::POS, -128, 0xfb14'be7f'bae5'8156'2172'a361'fd2a'722f_u128},
    {Sign::POS, -128, 0xfc3b'27d3'8a5d'49ab'2567'78ff'cb5c'1769_u128},
    {Sign::POS, -128, 0xfd3a'abf8'4528'b50b'eae6'bd95'1c1d'abbe_u128},
    {Sign::POS, -128, 0xfe13'2387'0cfe'9a3d'90cd'1d95'9db6'74ef_u128},
    {Sign::POS, -128, 0xfec4'6d1e'8929'2cf0'4139'0efd'c726'e9ef_u128},
    {Sign::POS, -128, 0xff4e'6d68'0c41'd0a9'0f66'8633'f1ab'858a_u128},
    {Sign::POS, -128, 0xffb1'0f1b'cb6b'ef1d'421e'8eda'af59'453e_u128},
    {Sign::POS, -128, 0xffec'4304'2668'65d9'5657'5523'6696'1732_u128},
    {Sign::POS, 0, 1},
};
#endif // !LIBC_MATH_HAS_SKIP_ACCURATE_PASS

} // namespace range_reduction_double_internal

} // namespace math

} // namespace LIBC_NAMESPACE_DECL

#endif // LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_COMMON_H
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xf314'4762'4708'8f74'a548'6bdc'455d'56a2_u128},`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xf314'4762'4708'8f74'a548'6bdc'455d'56a2_u128},`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xf4fa'0ab6'316e'd2ec'163c'5c7f'03b7'18c5_u128},`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xf4fa'0ab6'316e'd2ec'163c'5c7f'03b7'18c5_u128},`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xf6ba'073b'424b'19e8'2c79'1f59'cc1f'fc23_u128},`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xf6ba'073b'424b'19e8'2c79'1f59'cc1f'fc23_u128},`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xf853'f7dc'9186'b952'c7ad'c6b4'9888'91bb_u128},`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xf853'f7dc'9186'b952'c7ad'c6b4'9888'91bb_u128},`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xf9c7'9d63'272c'4628'4504'ae08'd19b'2980_u128},`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xf9c7'9d63'272c'4628'4504'ae08'd19b'2980_u128},`。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xfb14'be7f'bae5'8156'2172'a361'fd2a'722f_u128},`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xfb14'be7f'bae5'8156'2172'a361'fd2a'722f_u128},`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xfc3b'27d3'8a5d'49ab'2567'78ff'cb5c'1769_u128},`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xfc3b'27d3'8a5d'49ab'2567'78ff'cb5c'1769_u128},`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xfd3a'abf8'4528'b50b'eae6'bd95'1c1d'abbe_u128},`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xfd3a'abf8'4528'b50b'eae6'bd95'1c1d'abbe_u128},`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xfe13'2387'0cfe'9a3d'90cd'1d95'9db6'74ef_u128},`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xfe13'2387'0cfe'9a3d'90cd'1d95'9db6'74ef_u128},`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xfec4'6d1e'8929'2cf0'4139'0efd'c726'e9ef_u128},`.
  **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xfec4'6d1e'8929'2cf0'4139'0efd'c726'e9ef_u128},`。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xff4e'6d68'0c41'd0a9'0f66'8633'f1ab'858a_u128},`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xff4e'6d68'0c41'd0a9'0f66'8633'f1ab'858a_u128},`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xffb1'0f1b'cb6b'ef1d'421e'8eda'af59'453e_u128},`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xffb1'0f1b'cb6b'ef1d'421e'8eda'af59'453e_u128},`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, -128, 0xffec'4304'2668'65d9'5657'5523'6696'1732_u128},`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, -128, 0xffec'4304'2668'65d9'5657'5523'6696'1732_u128},`。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{Sign::POS, 0, 1},`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`{Sign::POS, 0, 1},`。
- **L375 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L375 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L376 EN**: Closes the current preprocessor conditional block or header guard.
  **L376 CN**: 结束当前预处理条件块或头文件保护。
- **L377 EN**: Blank line separating nearby declarations or logic.
  **L377 CN**: 空行，用于分隔相邻声明或逻辑。
- **L378 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace range_reduction_double_internal`.
  **L378 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace range_reduction_double_internal`。
- **L379 EN**: Blank line separating nearby declarations or logic.
  **L379 CN**: 空行，用于分隔相邻声明或逻辑。
- **L380 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace math`.
  **L380 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace math`。
- **L381 EN**: Blank line separating nearby declarations or logic.
  **L381 CN**: 空行，用于分隔相邻声明或逻辑。
- **L382 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace LIBC_NAMESPACE_DECL`.
  **L382 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace LIBC_NAMESPACE_DECL`。
- **L383 EN**: Blank line separating nearby declarations or logic.
  **L383 CN**: 空行，用于分隔相邻声明或逻辑。
- **L384 EN**: Closes the current preprocessor conditional block or header guard.
  **L384 CN**: 结束当前预处理条件块或头文件保护。

## Key Concepts / 关键概念

- **Floating-point support kernels / 浮点支撑内核**: Provides reusable math internals such as argument reduction, approximation helpers, and type-specific wrappers. / 提供可复用的数学内部组件，例如自变量归约、近似辅助逻辑以及按类型区分的包装层。
- **Argument reduction tables / 自变量归约表**: Stores constants and helper logic used to reduce arguments for transcendental functions. / 存储超越函数自变量归约使用的常量与辅助逻辑。
- **Floating-point bit manipulation / 浮点位级操作**: Uses helper types to inspect exponents, mantissas, special values, and sign bits directly. / 使用辅助类型直接检查指数、尾数、特殊值和符号位。
- **Numeric text conversion / 数字文本转换**: Builds or consumes textual numeric representations while tracking buffer sizes and edge cases. / 在跟踪缓冲区大小和边界情况的同时，构建或消费数字文本表示。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `src/__support/FPUtil/double_double.h`, `src/__support/FPUtil/dyadic_float.h`, `src/__support/FPUtil/multiply_add.h`, `src/__support/FPUtil/nearest_integer.h`, `src/__support/common.h`, `src/__support/integer_literals.h`, `src/__support/macros/config.h`, `src/__support/macros/optimization.h`
- **Dependency categories / 依赖类别**: LLVM libc floating-point utility helpers / LLVM libc 浮点工具辅助组件 (4), LLVM libc internal support utilities / LLVM libc 内部支撑工具 (2), LLVM libc configuration and attribute macros / LLVM libc 配置与属性宏 (2)

- `src/__support/FPUtil/double_double.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/dyadic_float.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/multiply_add.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/nearest_integer.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/common.h`: Provides LLVM libc internal support utilities. / 提供LLVM libc 内部支撑工具。
- `src/__support/integer_literals.h`: Provides LLVM libc internal support utilities. / 提供LLVM libc 内部支撑工具。
- `src/__support/macros/config.h`: Provides LLVM libc configuration and attribute macros. / 提供LLVM libc 配置与属性宏。
- `src/__support/macros/optimization.h`: Provides LLVM libc configuration and attribute macros. / 提供LLVM libc 配置与属性宏。
