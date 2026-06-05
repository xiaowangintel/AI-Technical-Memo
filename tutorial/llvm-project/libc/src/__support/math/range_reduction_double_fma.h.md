# range_reduction_double_fma.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `libc/src/__support/math/range_reduction_double_fma.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Range reduction for double precision sin/cos/tan w/ FMA.
  - **CN**: 声明 LLVM libc 各入口复用的内部浮点内核、归约辅助逻辑以及按类型特化的数学支持组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- Range reduction for double precision sin/cos/tan w/ FMA -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_FMA_H
#define LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_FMA_H

#include "src/__support/FPUtil/FPBits.h"
#include "src/__support/FPUtil/double_double.h"
#include "src/__support/FPUtil/multiply_add.h"
#include "src/__support/FPUtil/nearest_integer.h"
#include "src/__support/macros/config.h"
#include "src/__support/math/range_reduction_double_common.h"

namespace LIBC_NAMESPACE_DECL {

namespace math {

namespace range_reduction_double_internal {

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
- **L9 EN**: Starts a header guard condition: `#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_FMA_H`.
  **L9 CN**: 开始头文件保护条件：`#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_FMA_H`。
- **L10 EN**: Defines macro `LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_FMA_H` for compile-time constants, aliases, or dispatch control.
  **L10 CN**: 定义宏 `LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_FMA_H`，用于编译期常量、别名或分发控制。
- **L11 EN**: Blank line separating nearby declarations or logic.
  **L11 CN**: 空行，用于分隔相邻声明或逻辑。
- **L12 EN**: Includes "src/__support/FPUtil/FPBits.h" to access LLVM libc floating-point utility helpers.
  **L12 CN**: 引入 "src/__support/FPUtil/FPBits.h" 以使用LLVM libc 浮点工具辅助组件。
- **L13 EN**: Includes "src/__support/FPUtil/double_double.h" to access LLVM libc floating-point utility helpers.
  **L13 CN**: 引入 "src/__support/FPUtil/double_double.h" 以使用LLVM libc 浮点工具辅助组件。
- **L14 EN**: Includes "src/__support/FPUtil/multiply_add.h" to access LLVM libc floating-point utility helpers.
  **L14 CN**: 引入 "src/__support/FPUtil/multiply_add.h" 以使用LLVM libc 浮点工具辅助组件。
- **L15 EN**: Includes "src/__support/FPUtil/nearest_integer.h" to access LLVM libc floating-point utility helpers.
  **L15 CN**: 引入 "src/__support/FPUtil/nearest_integer.h" 以使用LLVM libc 浮点工具辅助组件。
- **L16 EN**: Includes "src/__support/macros/config.h" to access LLVM libc configuration and attribute macros.
  **L16 CN**: 引入 "src/__support/macros/config.h" 以使用LLVM libc 配置与属性宏。
- **L17 EN**: Includes "src/__support/math/range_reduction_double_common.h" to access LLVM libc internal support utilities.
  **L17 CN**: 引入 "src/__support/math/range_reduction_double_common.h" 以使用LLVM libc 内部支撑工具。
- **L18 EN**: Blank line separating nearby declarations or logic.
  **L18 CN**: 空行，用于分隔相邻声明或逻辑。
- **L19 EN**: Opens namespace scope `LIBC_NAMESPACE_DECL`.
  **L19 CN**: 打开命名空间作用域 `LIBC_NAMESPACE_DECL`。
- **L20 EN**: Blank line separating nearby declarations or logic.
  **L20 CN**: 空行，用于分隔相邻声明或逻辑。
- **L21 EN**: Opens namespace scope `math`.
  **L21 CN**: 打开命名空间作用域 `math`。
- **L22 EN**: Blank line separating nearby declarations or logic.
  **L22 CN**: 空行，用于分隔相邻声明或逻辑。
- **L23 EN**: Opens namespace scope `range_reduction_double_internal`.
  **L23 CN**: 打开命名空间作用域 `range_reduction_double_internal`。
- **L24 EN**: Blank line separating nearby declarations or logic.
  **L24 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 25-48

````cpp
using LIBC_NAMESPACE::fputil::DoubleDouble;

LIBC_INLINE unsigned LargeRangeReduction::fast(double x, DoubleDouble &u) {
  using FPBits = typename fputil::FPBits<double>;
  FPBits xbits(x);

  int x_e_m62 = xbits.get_biased_exponent() - (FPBits::EXP_BIAS + 62);
  idx = static_cast<unsigned>((x_e_m62 >> 4) + 3);
  // Scale x down by 2^(-(16 * (idx - 3))
  xbits.set_biased_exponent((x_e_m62 & 15) + FPBits::EXP_BIAS + 62);
  // 2^62 <= |x_reduced| < 2^(62 + 16) = 2^78
  x_reduced = xbits.get_val();
  // x * c_hi = ph.hi + ph.lo exactly.
  DoubleDouble ph = fputil::exact_mult<double, SPLIT>(
      x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][0]);
  // x * c_mid = pm.hi + pm.lo exactly.
  DoubleDouble pm = fputil::exact_mult<double, SPLIT>(
      x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][1]);
  // x * c_lo = pl.hi + pl.lo exactly.
  DoubleDouble pl = fputil::exact_mult<double, SPLIT>(
      x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][2]);
  // Extract integral parts and fractional parts of (ph.lo + pm.hi).
  double sum_hi = ph.lo + pm.hi;
  double kd = fputil::nearest_integer(sum_hi);
````
- **L25 EN**: Introduces a using declaration or alias: `using LIBC_NAMESPACE::fputil::DoubleDouble;`.
  **L25 CN**: 引入一条 using 声明或别名：`using LIBC_NAMESPACE::fputil::DoubleDouble;`。
- **L26 EN**: Blank line separating nearby declarations or logic.
  **L26 CN**: 空行，用于分隔相邻声明或逻辑。
- **L27 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L27 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L28 EN**: Defines alias `FPBits` to simplify later code.
  **L28 CN**: 定义别名 `FPBits` 以简化后续代码。
- **L29 EN**: Executes a call or declaration centered on `xbits`.
  **L29 CN**: 执行以 `xbits` 为核心的调用或声明。
- **L30 EN**: Blank line separating nearby declarations or logic.
  **L30 CN**: 空行，用于分隔相邻声明或逻辑。
- **L31 EN**: Initializes variable `x_e_m62` from the right-hand expression.
  **L31 CN**: 使用右侧表达式初始化变量 `x_e_m62`。
- **L32 EN**: Executes a call or declaration centered on `static_cast<unsigned>`.
  **L32 CN**: 执行以 `static_cast<unsigned>` 为核心的调用或声明。
- **L33 EN**: Comment documents nearby intent or constraints: `Scale x down by 2^(-(16 * (idx - 3))`.
  **L33 CN**: 注释说明附近代码的意图或约束：`Scale x down by 2^(-(16 * (idx - 3))`。
- **L34 EN**: Executes a call or declaration centered on `xbits.set_biased_exponent`.
  **L34 CN**: 执行以 `xbits.set_biased_exponent` 为核心的调用或声明。
- **L35 EN**: Comment documents nearby intent or constraints: `2^62 <= \|x_reduced\| < 2^(62 + 16) = 2^78`.
  **L35 CN**: 注释说明附近代码的意图或约束：`2^62 <= \|x_reduced\| < 2^(62 + 16) = 2^78`。
- **L36 EN**: Executes a call or declaration centered on `xbits.get_val`.
  **L36 CN**: 执行以 `xbits.get_val` 为核心的调用或声明。
- **L37 EN**: Comment documents nearby intent or constraints: `x * c_hi = ph.hi + ph.lo exactly.`.
  **L37 CN**: 注释说明附近代码的意图或约束：`x * c_hi = ph.hi + ph.lo exactly.`。
- **L38 EN**: Continues logic associated with callable symbol `SPLIT>`.
  **L38 CN**: 继续与可调用符号 `SPLIT>` 相关的逻辑。
- **L39 EN**: Executes a standalone statement or declaration: `x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][0]);`.
  **L39 CN**: 执行一条独立语句或声明：`x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][0]);`。
- **L40 EN**: Comment documents nearby intent or constraints: `x * c_mid = pm.hi + pm.lo exactly.`.
  **L40 CN**: 注释说明附近代码的意图或约束：`x * c_mid = pm.hi + pm.lo exactly.`。
- **L41 EN**: Continues logic associated with callable symbol `SPLIT>`.
  **L41 CN**: 继续与可调用符号 `SPLIT>` 相关的逻辑。
- **L42 EN**: Executes a standalone statement or declaration: `x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][1]);`.
  **L42 CN**: 执行一条独立语句或声明：`x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][1]);`。
- **L43 EN**: Comment documents nearby intent or constraints: `x * c_lo = pl.hi + pl.lo exactly.`.
  **L43 CN**: 注释说明附近代码的意图或约束：`x * c_lo = pl.hi + pl.lo exactly.`。
- **L44 EN**: Continues logic associated with callable symbol `SPLIT>`.
  **L44 CN**: 继续与可调用符号 `SPLIT>` 相关的逻辑。
- **L45 EN**: Executes a standalone statement or declaration: `x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][2]);`.
  **L45 CN**: 执行一条独立语句或声明：`x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][2]);`。
- **L46 EN**: Comment documents nearby intent or constraints: `Extract integral parts and fractional parts of (ph.lo + pm.hi).`.
  **L46 CN**: 注释说明附近代码的意图或约束：`Extract integral parts and fractional parts of (ph.lo + pm.hi).`。
- **L47 EN**: Initializes variable `sum_hi` from the right-hand expression.
  **L47 CN**: 使用右侧表达式初始化变量 `sum_hi`。
- **L48 EN**: Initializes variable `kd` from the right-hand expression.
  **L48 CN**: 使用右侧表达式初始化变量 `kd`。

### Lines 49-72

````cpp

  // x * 128/pi mod 1 ~ y_hi + y_mid + y_lo
  y_hi = (ph.lo - kd) + pm.hi; // Exact
  y_mid = fputil::exact_add(pm.lo, pl.hi);
  y_lo = pl.lo;

  // y_l = x * c_lo_2 + pl.lo
  double y_l =
      fputil::multiply_add(x_reduced, ONE_TWENTY_EIGHT_OVER_PI[idx][3], y_lo);
  DoubleDouble y = fputil::exact_add(y_hi, y_mid.hi);
  y.lo += (y_mid.lo + y_l);

  // Digits of pi/128, generated by Sollya with:
  // > a = round(pi/128, D, RN);
  // > b = round(pi/128 - a, D, RN);
  constexpr DoubleDouble PI_OVER_128_DD = {0x1.1a62633145c07p-60,
                                           0x1.921fb54442d18p-6};

  // Error bound: with {a} denote the fractional part of a, i.e.:
  //   {a} = a - round(a)
  // Then,
  //   | {x * 128/pi} - (y_hi + y_lo) | <=  ulp(ulp(y_hi)) <= 2^-105
  //   | {x mod pi/128} - (u.hi + u.lo) | < 2 * 2^-6 * 2^-105 = 2^-110
  u = fputil::quick_mult<SPLIT>(y, PI_OVER_128_DD);
````
- **L49 EN**: Blank line separating nearby declarations or logic.
  **L49 CN**: 空行，用于分隔相邻声明或逻辑。
- **L50 EN**: Comment documents nearby intent or constraints: `x * 128/pi mod 1 ~ y_hi + y_mid + y_lo`.
  **L50 CN**: 注释说明附近代码的意图或约束：`x * 128/pi mod 1 ~ y_hi + y_mid + y_lo`。
- **L51 EN**: Continues the surrounding expression or declaration: `y_hi = (ph.lo - kd) + pm.hi; // Exact`.
  **L51 CN**: 继续构造周围的表达式或声明：`y_hi = (ph.lo - kd) + pm.hi; // Exact`。
- **L52 EN**: Executes a call or declaration centered on `fputil::exact_add`.
  **L52 CN**: 执行以 `fputil::exact_add` 为核心的调用或声明。
- **L53 EN**: Executes a standalone statement or declaration: `y_lo = pl.lo;`.
  **L53 CN**: 执行一条独立语句或声明：`y_lo = pl.lo;`。
- **L54 EN**: Blank line separating nearby declarations or logic.
  **L54 CN**: 空行，用于分隔相邻声明或逻辑。
- **L55 EN**: Comment documents nearby intent or constraints: `y_l = x * c_lo_2 + pl.lo`.
  **L55 CN**: 注释说明附近代码的意图或约束：`y_l = x * c_lo_2 + pl.lo`。
- **L56 EN**: Continues the surrounding expression or declaration: `double y_l =`.
  **L56 CN**: 继续构造周围的表达式或声明：`double y_l =`。
- **L57 EN**: Executes a call or declaration centered on `fputil::multiply_add`.
  **L57 CN**: 执行以 `fputil::multiply_add` 为核心的调用或声明。
- **L58 EN**: Initializes variable `y` from the right-hand expression.
  **L58 CN**: 使用右侧表达式初始化变量 `y`。
- **L59 EN**: Executes a call or declaration centered on `+=`.
  **L59 CN**: 执行以 `+=` 为核心的调用或声明。
- **L60 EN**: Blank line separating nearby declarations or logic.
  **L60 CN**: 空行，用于分隔相邻声明或逻辑。
- **L61 EN**: Comment documents nearby intent or constraints: `Digits of pi/128, generated by Sollya with:`.
  **L61 CN**: 注释说明附近代码的意图或约束：`Digits of pi/128, generated by Sollya with:`。
- **L62 EN**: Comment documents nearby intent or constraints: `> a = round(pi/128, D, RN);`.
  **L62 CN**: 注释说明附近代码的意图或约束：`> a = round(pi/128, D, RN);`。
- **L63 EN**: Comment documents nearby intent or constraints: `> b = round(pi/128 - a, D, RN);`.
  **L63 CN**: 注释说明附近代码的意图或约束：`> b = round(pi/128 - a, D, RN);`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `constexpr DoubleDouble PI_OVER_128_DD = {0x1.1a62633145c07p-60,`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`constexpr DoubleDouble PI_OVER_128_DD = {0x1.1a62633145c07p-60,`。
- **L65 EN**: Executes a standalone statement or declaration: `0x1.921fb54442d18p-6};`.
  **L65 CN**: 执行一条独立语句或声明：`0x1.921fb54442d18p-6};`。
- **L66 EN**: Blank line separating nearby declarations or logic.
  **L66 CN**: 空行，用于分隔相邻声明或逻辑。
- **L67 EN**: Comment documents nearby intent or constraints: `Error bound: with {a} denote the fractional part of a, i.e.:`.
  **L67 CN**: 注释说明附近代码的意图或约束：`Error bound: with {a} denote the fractional part of a, i.e.:`。
- **L68 EN**: Comment documents nearby intent or constraints: `{a} = a - round(a)`.
  **L68 CN**: 注释说明附近代码的意图或约束：`{a} = a - round(a)`。
- **L69 EN**: Comment documents nearby intent or constraints: `Then,`.
  **L69 CN**: 注释说明附近代码的意图或约束：`Then,`。
- **L70 EN**: Comment documents nearby intent or constraints: `\| {x * 128/pi} - (y_hi + y_lo) \| <=  ulp(ulp(y_hi)) <= 2^-105`.
  **L70 CN**: 注释说明附近代码的意图或约束：`\| {x * 128/pi} - (y_hi + y_lo) \| <=  ulp(ulp(y_hi)) <= 2^-105`。
- **L71 EN**: Comment documents nearby intent or constraints: `\| {x mod pi/128} - (u.hi + u.lo) \| < 2 * 2^-6 * 2^-105 = 2^-110`.
  **L71 CN**: 注释说明附近代码的意图或约束：`\| {x mod pi/128} - (u.hi + u.lo) \| < 2 * 2^-6 * 2^-105 = 2^-110`。
- **L72 EN**: Executes a call or declaration centered on `fputil::quick_mult<SPLIT>`.
  **L72 CN**: 执行以 `fputil::quick_mult<SPLIT>` 为核心的调用或声明。

### Lines 73-96

````cpp

  return static_cast<unsigned>(static_cast<int64_t>(kd));
}

// Lookup table for sin(k * pi / 128) with k = 0, ..., 255.
// Table is generated with Sollya as follow:
// > display = hexadecimal;
// > for k from 0 to 255 do {
//     a = D(sin(k * pi/128)); };
//     b = D(sin(k * pi/128) - a);
//     print("{", b, ",", a, "},");
//   };
LIBC_INLINE_VAR constexpr DoubleDouble SIN_K_PI_OVER_128[] = {
    {0, 0},
    {-0x1.b1d63091a013p-64, 0x1.92155f7a3667ep-6},
    {-0x1.912bd0d569a9p-61, 0x1.91f65f10dd814p-5},
    {-0x1.9a088a8bf6b2cp-59, 0x1.2d52092ce19f6p-4},
    {-0x1.e2718d26ed688p-60, 0x1.917a6bc29b42cp-4},
    {0x1.a2704729ae56dp-59, 0x1.f564e56a9730ep-4},
    {0x1.13000a89a11ep-58, 0x1.2c8106e8e613ap-3},
    {0x1.531ff779ddac6p-57, 0x1.5e214448b3fc6p-3},
    {-0x1.26d19b9ff8d82p-57, 0x1.8f8b83c69a60bp-3},
    {-0x1.af1439e521935p-62, 0x1.c0b826a7e4f63p-3},
    {-0x1.42deef11da2c4p-57, 0x1.f19f97b215f1bp-3},
````
- **L73 EN**: Blank line separating nearby declarations or logic.
  **L73 CN**: 空行，用于分隔相邻声明或逻辑。
- **L74 EN**: Returns from the current function with `static_cast<unsigned>(static_cast<int64_t>(kd))`.
  **L74 CN**: 以 `static_cast<unsigned>(static_cast<int64_t>(kd))` 从当前函数返回。
- **L75 EN**: Closes the current lexical scope or compound statement.
  **L75 CN**: 结束当前词法作用域或复合语句块。
- **L76 EN**: Blank line separating nearby declarations or logic.
  **L76 CN**: 空行，用于分隔相邻声明或逻辑。
- **L77 EN**: Comment documents nearby intent or constraints: `Lookup table for sin(k * pi / 128) with k = 0, ..., 255.`.
  **L77 CN**: 注释说明附近代码的意图或约束：`Lookup table for sin(k * pi / 128) with k = 0, ..., 255.`。
- **L78 EN**: Comment documents nearby intent or constraints: `Table is generated with Sollya as follow:`.
  **L78 CN**: 注释说明附近代码的意图或约束：`Table is generated with Sollya as follow:`。
- **L79 EN**: Comment documents nearby intent or constraints: `> display = hexadecimal;`.
  **L79 CN**: 注释说明附近代码的意图或约束：`> display = hexadecimal;`。
- **L80 EN**: Comment documents nearby intent or constraints: `> for k from 0 to 255 do {`.
  **L80 CN**: 注释说明附近代码的意图或约束：`> for k from 0 to 255 do {`。
- **L81 EN**: Comment documents nearby intent or constraints: `a = D(sin(k * pi/128)); };`.
  **L81 CN**: 注释说明附近代码的意图或约束：`a = D(sin(k * pi/128)); };`。
- **L82 EN**: Comment documents nearby intent or constraints: `b = D(sin(k * pi/128) - a);`.
  **L82 CN**: 注释说明附近代码的意图或约束：`b = D(sin(k * pi/128) - a);`。
- **L83 EN**: Comment documents nearby intent or constraints: `print("{", b, ",", a, "},");`.
  **L83 CN**: 注释说明附近代码的意图或约束：`print("{", b, ",", a, "},");`。
- **L84 EN**: Comment documents nearby intent or constraints: `};`.
  **L84 CN**: 注释说明附近代码的意图或约束：`};`。
- **L85 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L85 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L86 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 0},`.
  **L86 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 0},`。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.b1d63091a013p-64, 0x1.92155f7a3667ep-6},`.
  **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.b1d63091a013p-64, 0x1.92155f7a3667ep-6},`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.912bd0d569a9p-61, 0x1.91f65f10dd814p-5},`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.912bd0d569a9p-61, 0x1.91f65f10dd814p-5},`。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.9a088a8bf6b2cp-59, 0x1.2d52092ce19f6p-4},`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.9a088a8bf6b2cp-59, 0x1.2d52092ce19f6p-4},`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e2718d26ed688p-60, 0x1.917a6bc29b42cp-4},`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e2718d26ed688p-60, 0x1.917a6bc29b42cp-4},`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.a2704729ae56dp-59, 0x1.f564e56a9730ep-4},`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.a2704729ae56dp-59, 0x1.f564e56a9730ep-4},`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.13000a89a11ep-58, 0x1.2c8106e8e613ap-3},`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.13000a89a11ep-58, 0x1.2c8106e8e613ap-3},`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.531ff779ddac6p-57, 0x1.5e214448b3fc6p-3},`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.531ff779ddac6p-57, 0x1.5e214448b3fc6p-3},`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.26d19b9ff8d82p-57, 0x1.8f8b83c69a60bp-3},`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.26d19b9ff8d82p-57, 0x1.8f8b83c69a60bp-3},`。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.af1439e521935p-62, 0x1.c0b826a7e4f63p-3},`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.af1439e521935p-62, 0x1.c0b826a7e4f63p-3},`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.42deef11da2c4p-57, 0x1.f19f97b215f1bp-3},`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.42deef11da2c4p-57, 0x1.f19f97b215f1bp-3},`。

### Lines 97-120

````cpp
    {0x1.824c20ab7aa9ap-56, 0x1.111d262b1f677p-2},
    {-0x1.5d28da2c4612dp-56, 0x1.294062ed59f06p-2},
    {0x1.0c97c4afa2518p-56, 0x1.4135c94176601p-2},
    {-0x1.efdc0d58cf62p-62, 0x1.58f9a75ab1fddp-2},
    {-0x1.44b19e0864c5dp-56, 0x1.7088530fa459fp-2},
    {-0x1.72cedd3d5a61p-57, 0x1.87de2a6aea963p-2},
    {0x1.6da81290bdbabp-57, 0x1.9ef7943a8ed8ap-2},
    {0x1.5b362cb974183p-57, 0x1.b5d1009e15ccp-2},
    {0x1.6850e59c37f8fp-58, 0x1.cc66e9931c45ep-2},
    {0x1.e0d891d3c6841p-58, 0x1.e2b5d3806f63bp-2},
    {-0x1.2ec1fc1b776b8p-60, 0x1.f8ba4dbf89abap-2},
    {-0x1.a5a014347406cp-55, 0x1.073879922ffeep-1},
    {-0x1.ef23b69abe4f1p-55, 0x1.11eb3541b4b23p-1},
    {0x1.b25dd267f66p-55, 0x1.1c73b39ae68c8p-1},
    {-0x1.5da743ef3770cp-55, 0x1.26d054cdd12dfp-1},
    {-0x1.efcc626f74a6fp-57, 0x1.30ff7fce17035p-1},
    {0x1.e3e25e3954964p-56, 0x1.3affa292050b9p-1},
    {0x1.8076a2cfdc6b3p-57, 0x1.44cf325091dd6p-1},
    {0x1.3c293edceb327p-57, 0x1.4e6cabbe3e5e9p-1},
    {-0x1.75720992bfbb2p-55, 0x1.57d69348cecap-1},
    {-0x1.251b352ff2a37p-56, 0x1.610b7551d2cdfp-1},
    {-0x1.bdd3413b26456p-55, 0x1.6a09e667f3bcdp-1},
    {0x1.0d4ef0f1d915cp-55, 0x1.72d0837efff96p-1},
    {-0x1.0f537acdf0ad7p-56, 0x1.7b5df226aafafp-1},
````
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.824c20ab7aa9ap-56, 0x1.111d262b1f677p-2},`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.824c20ab7aa9ap-56, 0x1.111d262b1f677p-2},`。
- **L98 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5d28da2c4612dp-56, 0x1.294062ed59f06p-2},`.
  **L98 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5d28da2c4612dp-56, 0x1.294062ed59f06p-2},`。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.0c97c4afa2518p-56, 0x1.4135c94176601p-2},`.
  **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.0c97c4afa2518p-56, 0x1.4135c94176601p-2},`。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.efdc0d58cf62p-62, 0x1.58f9a75ab1fddp-2},`.
  **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.efdc0d58cf62p-62, 0x1.58f9a75ab1fddp-2},`。
- **L101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.44b19e0864c5dp-56, 0x1.7088530fa459fp-2},`.
  **L101 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.44b19e0864c5dp-56, 0x1.7088530fa459fp-2},`。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.72cedd3d5a61p-57, 0x1.87de2a6aea963p-2},`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.72cedd3d5a61p-57, 0x1.87de2a6aea963p-2},`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6da81290bdbabp-57, 0x1.9ef7943a8ed8ap-2},`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6da81290bdbabp-57, 0x1.9ef7943a8ed8ap-2},`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.5b362cb974183p-57, 0x1.b5d1009e15ccp-2},`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.5b362cb974183p-57, 0x1.b5d1009e15ccp-2},`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6850e59c37f8fp-58, 0x1.cc66e9931c45ep-2},`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6850e59c37f8fp-58, 0x1.cc66e9931c45ep-2},`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e0d891d3c6841p-58, 0x1.e2b5d3806f63bp-2},`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e0d891d3c6841p-58, 0x1.e2b5d3806f63bp-2},`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2ec1fc1b776b8p-60, 0x1.f8ba4dbf89abap-2},`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2ec1fc1b776b8p-60, 0x1.f8ba4dbf89abap-2},`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a5a014347406cp-55, 0x1.073879922ffeep-1},`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a5a014347406cp-55, 0x1.073879922ffeep-1},`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ef23b69abe4f1p-55, 0x1.11eb3541b4b23p-1},`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ef23b69abe4f1p-55, 0x1.11eb3541b4b23p-1},`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.b25dd267f66p-55, 0x1.1c73b39ae68c8p-1},`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.b25dd267f66p-55, 0x1.1c73b39ae68c8p-1},`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5da743ef3770cp-55, 0x1.26d054cdd12dfp-1},`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5da743ef3770cp-55, 0x1.26d054cdd12dfp-1},`。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.efcc626f74a6fp-57, 0x1.30ff7fce17035p-1},`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.efcc626f74a6fp-57, 0x1.30ff7fce17035p-1},`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e3e25e3954964p-56, 0x1.3affa292050b9p-1},`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e3e25e3954964p-56, 0x1.3affa292050b9p-1},`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.8076a2cfdc6b3p-57, 0x1.44cf325091dd6p-1},`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.8076a2cfdc6b3p-57, 0x1.44cf325091dd6p-1},`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3c293edceb327p-57, 0x1.4e6cabbe3e5e9p-1},`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3c293edceb327p-57, 0x1.4e6cabbe3e5e9p-1},`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.75720992bfbb2p-55, 0x1.57d69348cecap-1},`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.75720992bfbb2p-55, 0x1.57d69348cecap-1},`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.251b352ff2a37p-56, 0x1.610b7551d2cdfp-1},`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.251b352ff2a37p-56, 0x1.610b7551d2cdfp-1},`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.bdd3413b26456p-55, 0x1.6a09e667f3bcdp-1},`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.bdd3413b26456p-55, 0x1.6a09e667f3bcdp-1},`。
- **L119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.0d4ef0f1d915cp-55, 0x1.72d0837efff96p-1},`.
  **L119 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.0d4ef0f1d915cp-55, 0x1.72d0837efff96p-1},`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0f537acdf0ad7p-56, 0x1.7b5df226aafafp-1},`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0f537acdf0ad7p-56, 0x1.7b5df226aafafp-1},`。

### Lines 121-144

````cpp
    {-0x1.6f420f8ea3475p-56, 0x1.83b0e0bff976ep-1},
    {-0x1.2c5e12ed1336dp-55, 0x1.8bc806b151741p-1},
    {0x1.3d419a920df0bp-55, 0x1.93a22499263fbp-1},
    {-0x1.30ee286712474p-55, 0x1.9b3e047f38741p-1},
    {-0x1.128bb015df175p-56, 0x1.a29a7a0462782p-1},
    {0x1.9f630e8b6dac8p-60, 0x1.a9b66290ea1a3p-1},
    {-0x1.926da300ffccep-55, 0x1.b090a581502p-1},
    {-0x1.bc69f324e6d61p-55, 0x1.b728345196e3ep-1},
    {-0x1.825a732ac700ap-55, 0x1.bd7c0ac6f952ap-1},
    {-0x1.6e0b1757c8d07p-56, 0x1.c38b2f180bdb1p-1},
    {-0x1.2fb761e946603p-58, 0x1.c954b213411f5p-1},
    {-0x1.e7b6bb5ab58aep-58, 0x1.ced7af43cc773p-1},
    {-0x1.4ef5295d25af2p-55, 0x1.d4134d14dc93ap-1},
    {0x1.457e610231ac2p-56, 0x1.d906bcf328d46p-1},
    {0x1.83c37c6107db3p-55, 0x1.ddb13b6ccc23cp-1},
    {-0x1.014c76c126527p-55, 0x1.e212104f686e5p-1},
    {-0x1.16b56f2847754p-57, 0x1.e6288ec48e112p-1},
    {0x1.760b1e2e3f81ep-55, 0x1.e9f4156c62ddap-1},
    {0x1.e82c791f59cc2p-56, 0x1.ed740e7684963p-1},
    {0x1.52c7adc6b4989p-56, 0x1.f0a7efb9230d7p-1},
    {-0x1.d7bafb51f72e6p-56, 0x1.f38f3ac64e589p-1},
    {0x1.562172a361fd3p-56, 0x1.f6297cff75cbp-1},
    {0x1.ab256778ffcb6p-56, 0x1.f8764fa714ba9p-1},
    {-0x1.7a0a8ca13571fp-55, 0x1.fa7557f08a517p-1},
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6f420f8ea3475p-56, 0x1.83b0e0bff976ep-1},`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6f420f8ea3475p-56, 0x1.83b0e0bff976ep-1},`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2c5e12ed1336dp-55, 0x1.8bc806b151741p-1},`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2c5e12ed1336dp-55, 0x1.8bc806b151741p-1},`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3d419a920df0bp-55, 0x1.93a22499263fbp-1},`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3d419a920df0bp-55, 0x1.93a22499263fbp-1},`。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.30ee286712474p-55, 0x1.9b3e047f38741p-1},`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.30ee286712474p-55, 0x1.9b3e047f38741p-1},`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.128bb015df175p-56, 0x1.a29a7a0462782p-1},`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.128bb015df175p-56, 0x1.a29a7a0462782p-1},`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.9f630e8b6dac8p-60, 0x1.a9b66290ea1a3p-1},`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.9f630e8b6dac8p-60, 0x1.a9b66290ea1a3p-1},`。
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.926da300ffccep-55, 0x1.b090a581502p-1},`.
  **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.926da300ffccep-55, 0x1.b090a581502p-1},`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.bc69f324e6d61p-55, 0x1.b728345196e3ep-1},`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.bc69f324e6d61p-55, 0x1.b728345196e3ep-1},`。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.825a732ac700ap-55, 0x1.bd7c0ac6f952ap-1},`.
  **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.825a732ac700ap-55, 0x1.bd7c0ac6f952ap-1},`。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6e0b1757c8d07p-56, 0x1.c38b2f180bdb1p-1},`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6e0b1757c8d07p-56, 0x1.c38b2f180bdb1p-1},`。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2fb761e946603p-58, 0x1.c954b213411f5p-1},`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2fb761e946603p-58, 0x1.c954b213411f5p-1},`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e7b6bb5ab58aep-58, 0x1.ced7af43cc773p-1},`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e7b6bb5ab58aep-58, 0x1.ced7af43cc773p-1},`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.4ef5295d25af2p-55, 0x1.d4134d14dc93ap-1},`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.4ef5295d25af2p-55, 0x1.d4134d14dc93ap-1},`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.457e610231ac2p-56, 0x1.d906bcf328d46p-1},`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.457e610231ac2p-56, 0x1.d906bcf328d46p-1},`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.83c37c6107db3p-55, 0x1.ddb13b6ccc23cp-1},`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.83c37c6107db3p-55, 0x1.ddb13b6ccc23cp-1},`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.014c76c126527p-55, 0x1.e212104f686e5p-1},`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.014c76c126527p-55, 0x1.e212104f686e5p-1},`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.16b56f2847754p-57, 0x1.e6288ec48e112p-1},`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.16b56f2847754p-57, 0x1.e6288ec48e112p-1},`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.760b1e2e3f81ep-55, 0x1.e9f4156c62ddap-1},`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.760b1e2e3f81ep-55, 0x1.e9f4156c62ddap-1},`。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e82c791f59cc2p-56, 0x1.ed740e7684963p-1},`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e82c791f59cc2p-56, 0x1.ed740e7684963p-1},`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.52c7adc6b4989p-56, 0x1.f0a7efb9230d7p-1},`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.52c7adc6b4989p-56, 0x1.f0a7efb9230d7p-1},`。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.d7bafb51f72e6p-56, 0x1.f38f3ac64e589p-1},`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.d7bafb51f72e6p-56, 0x1.f38f3ac64e589p-1},`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.562172a361fd3p-56, 0x1.f6297cff75cbp-1},`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.562172a361fd3p-56, 0x1.f6297cff75cbp-1},`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.ab256778ffcb6p-56, 0x1.f8764fa714ba9p-1},`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.ab256778ffcb6p-56, 0x1.f8764fa714ba9p-1},`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.7a0a8ca13571fp-55, 0x1.fa7557f08a517p-1},`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.7a0a8ca13571fp-55, 0x1.fa7557f08a517p-1},`。

### Lines 145-168

````cpp
    {0x1.1ec8668ecaceep-55, 0x1.fc26470e19fd3p-1},
    {-0x1.87df6378811c7p-55, 0x1.fd88da3d12526p-1},
    {0x1.521ecd0c67e35p-57, 0x1.fe9cdad01883ap-1},
    {-0x1.c57bc2e24aa15p-57, 0x1.ff621e3796d7ep-1},
    {-0x1.1354d4556e4cbp-55, 0x1.ffd886084cd0dp-1},
    {0, 1},
#ifndef LIBC_MATH_HAS_SMALL_TABLES
    {-0x1.1354d4556e4cbp-55, 0x1.ffd886084cd0dp-1},
    {-0x1.c57bc2e24aa15p-57, 0x1.ff621e3796d7ep-1},
    {0x1.521ecd0c67e35p-57, 0x1.fe9cdad01883ap-1},
    {-0x1.87df6378811c7p-55, 0x1.fd88da3d12526p-1},
    {0x1.1ec8668ecaceep-55, 0x1.fc26470e19fd3p-1},
    {-0x1.7a0a8ca13571fp-55, 0x1.fa7557f08a517p-1},
    {0x1.ab256778ffcb6p-56, 0x1.f8764fa714ba9p-1},
    {0x1.562172a361fd3p-56, 0x1.f6297cff75cbp-1},
    {-0x1.d7bafb51f72e6p-56, 0x1.f38f3ac64e589p-1},
    {0x1.52c7adc6b4989p-56, 0x1.f0a7efb9230d7p-1},
    {0x1.e82c791f59cc2p-56, 0x1.ed740e7684963p-1},
    {0x1.760b1e2e3f81ep-55, 0x1.e9f4156c62ddap-1},
    {-0x1.16b56f2847754p-57, 0x1.e6288ec48e112p-1},
    {-0x1.014c76c126527p-55, 0x1.e212104f686e5p-1},
    {0x1.83c37c6107db3p-55, 0x1.ddb13b6ccc23cp-1},
    {0x1.457e610231ac2p-56, 0x1.d906bcf328d46p-1},
    {-0x1.4ef5295d25af2p-55, 0x1.d4134d14dc93ap-1},
````
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1ec8668ecaceep-55, 0x1.fc26470e19fd3p-1},`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1ec8668ecaceep-55, 0x1.fc26470e19fd3p-1},`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.87df6378811c7p-55, 0x1.fd88da3d12526p-1},`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.87df6378811c7p-55, 0x1.fd88da3d12526p-1},`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.521ecd0c67e35p-57, 0x1.fe9cdad01883ap-1},`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.521ecd0c67e35p-57, 0x1.fe9cdad01883ap-1},`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.c57bc2e24aa15p-57, 0x1.ff621e3796d7ep-1},`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.c57bc2e24aa15p-57, 0x1.ff621e3796d7ep-1},`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.1354d4556e4cbp-55, 0x1.ffd886084cd0dp-1},`.
  **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.1354d4556e4cbp-55, 0x1.ffd886084cd0dp-1},`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 1},`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 1},`。
- **L151 EN**: Starts a header guard condition: `#ifndef LIBC_MATH_HAS_SMALL_TABLES`.
  **L151 CN**: 开始头文件保护条件：`#ifndef LIBC_MATH_HAS_SMALL_TABLES`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.1354d4556e4cbp-55, 0x1.ffd886084cd0dp-1},`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.1354d4556e4cbp-55, 0x1.ffd886084cd0dp-1},`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.c57bc2e24aa15p-57, 0x1.ff621e3796d7ep-1},`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.c57bc2e24aa15p-57, 0x1.ff621e3796d7ep-1},`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.521ecd0c67e35p-57, 0x1.fe9cdad01883ap-1},`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.521ecd0c67e35p-57, 0x1.fe9cdad01883ap-1},`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.87df6378811c7p-55, 0x1.fd88da3d12526p-1},`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.87df6378811c7p-55, 0x1.fd88da3d12526p-1},`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1ec8668ecaceep-55, 0x1.fc26470e19fd3p-1},`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1ec8668ecaceep-55, 0x1.fc26470e19fd3p-1},`。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.7a0a8ca13571fp-55, 0x1.fa7557f08a517p-1},`.
  **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.7a0a8ca13571fp-55, 0x1.fa7557f08a517p-1},`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.ab256778ffcb6p-56, 0x1.f8764fa714ba9p-1},`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.ab256778ffcb6p-56, 0x1.f8764fa714ba9p-1},`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.562172a361fd3p-56, 0x1.f6297cff75cbp-1},`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.562172a361fd3p-56, 0x1.f6297cff75cbp-1},`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.d7bafb51f72e6p-56, 0x1.f38f3ac64e589p-1},`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.d7bafb51f72e6p-56, 0x1.f38f3ac64e589p-1},`。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.52c7adc6b4989p-56, 0x1.f0a7efb9230d7p-1},`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.52c7adc6b4989p-56, 0x1.f0a7efb9230d7p-1},`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e82c791f59cc2p-56, 0x1.ed740e7684963p-1},`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e82c791f59cc2p-56, 0x1.ed740e7684963p-1},`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.760b1e2e3f81ep-55, 0x1.e9f4156c62ddap-1},`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.760b1e2e3f81ep-55, 0x1.e9f4156c62ddap-1},`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.16b56f2847754p-57, 0x1.e6288ec48e112p-1},`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.16b56f2847754p-57, 0x1.e6288ec48e112p-1},`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.014c76c126527p-55, 0x1.e212104f686e5p-1},`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.014c76c126527p-55, 0x1.e212104f686e5p-1},`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.83c37c6107db3p-55, 0x1.ddb13b6ccc23cp-1},`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.83c37c6107db3p-55, 0x1.ddb13b6ccc23cp-1},`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.457e610231ac2p-56, 0x1.d906bcf328d46p-1},`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.457e610231ac2p-56, 0x1.d906bcf328d46p-1},`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.4ef5295d25af2p-55, 0x1.d4134d14dc93ap-1},`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.4ef5295d25af2p-55, 0x1.d4134d14dc93ap-1},`。

### Lines 169-192

````cpp
    {-0x1.e7b6bb5ab58aep-58, 0x1.ced7af43cc773p-1},
    {-0x1.2fb761e946603p-58, 0x1.c954b213411f5p-1},
    {-0x1.6e0b1757c8d07p-56, 0x1.c38b2f180bdb1p-1},
    {-0x1.825a732ac700ap-55, 0x1.bd7c0ac6f952ap-1},
    {-0x1.bc69f324e6d61p-55, 0x1.b728345196e3ep-1},
    {-0x1.926da300ffccep-55, 0x1.b090a581502p-1},
    {0x1.9f630e8b6dac8p-60, 0x1.a9b66290ea1a3p-1},
    {-0x1.128bb015df175p-56, 0x1.a29a7a0462782p-1},
    {-0x1.30ee286712474p-55, 0x1.9b3e047f38741p-1},
    {0x1.3d419a920df0bp-55, 0x1.93a22499263fbp-1},
    {-0x1.2c5e12ed1336dp-55, 0x1.8bc806b151741p-1},
    {-0x1.6f420f8ea3475p-56, 0x1.83b0e0bff976ep-1},
    {-0x1.0f537acdf0ad7p-56, 0x1.7b5df226aafafp-1},
    {0x1.0d4ef0f1d915cp-55, 0x1.72d0837efff96p-1},
    {-0x1.bdd3413b26456p-55, 0x1.6a09e667f3bcdp-1},
    {-0x1.251b352ff2a37p-56, 0x1.610b7551d2cdfp-1},
    {-0x1.75720992bfbb2p-55, 0x1.57d69348cecap-1},
    {0x1.3c293edceb327p-57, 0x1.4e6cabbe3e5e9p-1},
    {0x1.8076a2cfdc6b3p-57, 0x1.44cf325091dd6p-1},
    {0x1.e3e25e3954964p-56, 0x1.3affa292050b9p-1},
    {-0x1.efcc626f74a6fp-57, 0x1.30ff7fce17035p-1},
    {-0x1.5da743ef3770cp-55, 0x1.26d054cdd12dfp-1},
    {0x1.b25dd267f66p-55, 0x1.1c73b39ae68c8p-1},
    {-0x1.ef23b69abe4f1p-55, 0x1.11eb3541b4b23p-1},
````
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e7b6bb5ab58aep-58, 0x1.ced7af43cc773p-1},`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e7b6bb5ab58aep-58, 0x1.ced7af43cc773p-1},`。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2fb761e946603p-58, 0x1.c954b213411f5p-1},`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2fb761e946603p-58, 0x1.c954b213411f5p-1},`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6e0b1757c8d07p-56, 0x1.c38b2f180bdb1p-1},`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6e0b1757c8d07p-56, 0x1.c38b2f180bdb1p-1},`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.825a732ac700ap-55, 0x1.bd7c0ac6f952ap-1},`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.825a732ac700ap-55, 0x1.bd7c0ac6f952ap-1},`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.bc69f324e6d61p-55, 0x1.b728345196e3ep-1},`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.bc69f324e6d61p-55, 0x1.b728345196e3ep-1},`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.926da300ffccep-55, 0x1.b090a581502p-1},`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.926da300ffccep-55, 0x1.b090a581502p-1},`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.9f630e8b6dac8p-60, 0x1.a9b66290ea1a3p-1},`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.9f630e8b6dac8p-60, 0x1.a9b66290ea1a3p-1},`。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.128bb015df175p-56, 0x1.a29a7a0462782p-1},`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.128bb015df175p-56, 0x1.a29a7a0462782p-1},`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.30ee286712474p-55, 0x1.9b3e047f38741p-1},`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.30ee286712474p-55, 0x1.9b3e047f38741p-1},`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3d419a920df0bp-55, 0x1.93a22499263fbp-1},`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3d419a920df0bp-55, 0x1.93a22499263fbp-1},`。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2c5e12ed1336dp-55, 0x1.8bc806b151741p-1},`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2c5e12ed1336dp-55, 0x1.8bc806b151741p-1},`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6f420f8ea3475p-56, 0x1.83b0e0bff976ep-1},`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6f420f8ea3475p-56, 0x1.83b0e0bff976ep-1},`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0f537acdf0ad7p-56, 0x1.7b5df226aafafp-1},`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0f537acdf0ad7p-56, 0x1.7b5df226aafafp-1},`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.0d4ef0f1d915cp-55, 0x1.72d0837efff96p-1},`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.0d4ef0f1d915cp-55, 0x1.72d0837efff96p-1},`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.bdd3413b26456p-55, 0x1.6a09e667f3bcdp-1},`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.bdd3413b26456p-55, 0x1.6a09e667f3bcdp-1},`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.251b352ff2a37p-56, 0x1.610b7551d2cdfp-1},`.
  **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.251b352ff2a37p-56, 0x1.610b7551d2cdfp-1},`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.75720992bfbb2p-55, 0x1.57d69348cecap-1},`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.75720992bfbb2p-55, 0x1.57d69348cecap-1},`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3c293edceb327p-57, 0x1.4e6cabbe3e5e9p-1},`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3c293edceb327p-57, 0x1.4e6cabbe3e5e9p-1},`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.8076a2cfdc6b3p-57, 0x1.44cf325091dd6p-1},`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.8076a2cfdc6b3p-57, 0x1.44cf325091dd6p-1},`。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e3e25e3954964p-56, 0x1.3affa292050b9p-1},`.
  **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e3e25e3954964p-56, 0x1.3affa292050b9p-1},`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.efcc626f74a6fp-57, 0x1.30ff7fce17035p-1},`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.efcc626f74a6fp-57, 0x1.30ff7fce17035p-1},`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5da743ef3770cp-55, 0x1.26d054cdd12dfp-1},`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5da743ef3770cp-55, 0x1.26d054cdd12dfp-1},`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.b25dd267f66p-55, 0x1.1c73b39ae68c8p-1},`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.b25dd267f66p-55, 0x1.1c73b39ae68c8p-1},`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ef23b69abe4f1p-55, 0x1.11eb3541b4b23p-1},`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ef23b69abe4f1p-55, 0x1.11eb3541b4b23p-1},`。

### Lines 193-216

````cpp
    {-0x1.a5a014347406cp-55, 0x1.073879922ffeep-1},
    {-0x1.2ec1fc1b776b8p-60, 0x1.f8ba4dbf89abap-2},
    {0x1.e0d891d3c6841p-58, 0x1.e2b5d3806f63bp-2},
    {0x1.6850e59c37f8fp-58, 0x1.cc66e9931c45ep-2},
    {0x1.5b362cb974183p-57, 0x1.b5d1009e15ccp-2},
    {0x1.6da81290bdbabp-57, 0x1.9ef7943a8ed8ap-2},
    {-0x1.72cedd3d5a61p-57, 0x1.87de2a6aea963p-2},
    {-0x1.44b19e0864c5dp-56, 0x1.7088530fa459fp-2},
    {-0x1.efdc0d58cf62p-62, 0x1.58f9a75ab1fddp-2},
    {0x1.0c97c4afa2518p-56, 0x1.4135c94176601p-2},
    {-0x1.5d28da2c4612dp-56, 0x1.294062ed59f06p-2},
    {0x1.824c20ab7aa9ap-56, 0x1.111d262b1f677p-2},
    {-0x1.42deef11da2c4p-57, 0x1.f19f97b215f1bp-3},
    {-0x1.af1439e521935p-62, 0x1.c0b826a7e4f63p-3},
    {-0x1.26d19b9ff8d82p-57, 0x1.8f8b83c69a60bp-3},
    {0x1.531ff779ddac6p-57, 0x1.5e214448b3fc6p-3},
    {0x1.13000a89a11ep-58, 0x1.2c8106e8e613ap-3},
    {0x1.a2704729ae56dp-59, 0x1.f564e56a9730ep-4},
    {-0x1.e2718d26ed688p-60, 0x1.917a6bc29b42cp-4},
    {-0x1.9a088a8bf6b2cp-59, 0x1.2d52092ce19f6p-4},
    {-0x1.912bd0d569a9p-61, 0x1.91f65f10dd814p-5},
    {-0x1.b1d63091a013p-64, 0x1.92155f7a3667ep-6},
    {0, 0},
    {0x1.b1d63091a013p-64, -0x1.92155f7a3667ep-6},
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a5a014347406cp-55, 0x1.073879922ffeep-1},`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a5a014347406cp-55, 0x1.073879922ffeep-1},`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2ec1fc1b776b8p-60, 0x1.f8ba4dbf89abap-2},`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2ec1fc1b776b8p-60, 0x1.f8ba4dbf89abap-2},`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e0d891d3c6841p-58, 0x1.e2b5d3806f63bp-2},`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e0d891d3c6841p-58, 0x1.e2b5d3806f63bp-2},`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6850e59c37f8fp-58, 0x1.cc66e9931c45ep-2},`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6850e59c37f8fp-58, 0x1.cc66e9931c45ep-2},`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.5b362cb974183p-57, 0x1.b5d1009e15ccp-2},`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.5b362cb974183p-57, 0x1.b5d1009e15ccp-2},`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6da81290bdbabp-57, 0x1.9ef7943a8ed8ap-2},`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6da81290bdbabp-57, 0x1.9ef7943a8ed8ap-2},`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.72cedd3d5a61p-57, 0x1.87de2a6aea963p-2},`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.72cedd3d5a61p-57, 0x1.87de2a6aea963p-2},`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.44b19e0864c5dp-56, 0x1.7088530fa459fp-2},`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.44b19e0864c5dp-56, 0x1.7088530fa459fp-2},`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.efdc0d58cf62p-62, 0x1.58f9a75ab1fddp-2},`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.efdc0d58cf62p-62, 0x1.58f9a75ab1fddp-2},`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.0c97c4afa2518p-56, 0x1.4135c94176601p-2},`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.0c97c4afa2518p-56, 0x1.4135c94176601p-2},`。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5d28da2c4612dp-56, 0x1.294062ed59f06p-2},`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5d28da2c4612dp-56, 0x1.294062ed59f06p-2},`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.824c20ab7aa9ap-56, 0x1.111d262b1f677p-2},`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.824c20ab7aa9ap-56, 0x1.111d262b1f677p-2},`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.42deef11da2c4p-57, 0x1.f19f97b215f1bp-3},`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.42deef11da2c4p-57, 0x1.f19f97b215f1bp-3},`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.af1439e521935p-62, 0x1.c0b826a7e4f63p-3},`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.af1439e521935p-62, 0x1.c0b826a7e4f63p-3},`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.26d19b9ff8d82p-57, 0x1.8f8b83c69a60bp-3},`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.26d19b9ff8d82p-57, 0x1.8f8b83c69a60bp-3},`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.531ff779ddac6p-57, 0x1.5e214448b3fc6p-3},`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.531ff779ddac6p-57, 0x1.5e214448b3fc6p-3},`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.13000a89a11ep-58, 0x1.2c8106e8e613ap-3},`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.13000a89a11ep-58, 0x1.2c8106e8e613ap-3},`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.a2704729ae56dp-59, 0x1.f564e56a9730ep-4},`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.a2704729ae56dp-59, 0x1.f564e56a9730ep-4},`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e2718d26ed688p-60, 0x1.917a6bc29b42cp-4},`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e2718d26ed688p-60, 0x1.917a6bc29b42cp-4},`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.9a088a8bf6b2cp-59, 0x1.2d52092ce19f6p-4},`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.9a088a8bf6b2cp-59, 0x1.2d52092ce19f6p-4},`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.912bd0d569a9p-61, 0x1.91f65f10dd814p-5},`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.912bd0d569a9p-61, 0x1.91f65f10dd814p-5},`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.b1d63091a013p-64, 0x1.92155f7a3667ep-6},`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.b1d63091a013p-64, 0x1.92155f7a3667ep-6},`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, 0},`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, 0},`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.b1d63091a013p-64, -0x1.92155f7a3667ep-6},`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.b1d63091a013p-64, -0x1.92155f7a3667ep-6},`。

### Lines 217-240

````cpp
    {0x1.912bd0d569a9p-61, -0x1.91f65f10dd814p-5},
    {0x1.9a088a8bf6b2cp-59, -0x1.2d52092ce19f6p-4},
    {0x1.e2718d26ed688p-60, -0x1.917a6bc29b42cp-4},
    {-0x1.a2704729ae56dp-59, -0x1.f564e56a9730ep-4},
    {-0x1.13000a89a11ep-58, -0x1.2c8106e8e613ap-3},
    {-0x1.531ff779ddac6p-57, -0x1.5e214448b3fc6p-3},
    {0x1.26d19b9ff8d82p-57, -0x1.8f8b83c69a60bp-3},
    {0x1.af1439e521935p-62, -0x1.c0b826a7e4f63p-3},
    {0x1.42deef11da2c4p-57, -0x1.f19f97b215f1bp-3},
    {-0x1.824c20ab7aa9ap-56, -0x1.111d262b1f677p-2},
    {0x1.5d28da2c4612dp-56, -0x1.294062ed59f06p-2},
    {-0x1.0c97c4afa2518p-56, -0x1.4135c94176601p-2},
    {0x1.efdc0d58cf62p-62, -0x1.58f9a75ab1fddp-2},
    {0x1.44b19e0864c5dp-56, -0x1.7088530fa459fp-2},
    {0x1.72cedd3d5a61p-57, -0x1.87de2a6aea963p-2},
    {-0x1.6da81290bdbabp-57, -0x1.9ef7943a8ed8ap-2},
    {-0x1.5b362cb974183p-57, -0x1.b5d1009e15ccp-2},
    {-0x1.6850e59c37f8fp-58, -0x1.cc66e9931c45ep-2},
    {-0x1.e0d891d3c6841p-58, -0x1.e2b5d3806f63bp-2},
    {0x1.2ec1fc1b776b8p-60, -0x1.f8ba4dbf89abap-2},
    {0x1.a5a014347406cp-55, -0x1.073879922ffeep-1},
    {0x1.ef23b69abe4f1p-55, -0x1.11eb3541b4b23p-1},
    {-0x1.b25dd267f66p-55, -0x1.1c73b39ae68c8p-1},
    {0x1.5da743ef3770cp-55, -0x1.26d054cdd12dfp-1},
````
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.912bd0d569a9p-61, -0x1.91f65f10dd814p-5},`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.912bd0d569a9p-61, -0x1.91f65f10dd814p-5},`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.9a088a8bf6b2cp-59, -0x1.2d52092ce19f6p-4},`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.9a088a8bf6b2cp-59, -0x1.2d52092ce19f6p-4},`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e2718d26ed688p-60, -0x1.917a6bc29b42cp-4},`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e2718d26ed688p-60, -0x1.917a6bc29b42cp-4},`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a2704729ae56dp-59, -0x1.f564e56a9730ep-4},`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a2704729ae56dp-59, -0x1.f564e56a9730ep-4},`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.13000a89a11ep-58, -0x1.2c8106e8e613ap-3},`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.13000a89a11ep-58, -0x1.2c8106e8e613ap-3},`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.531ff779ddac6p-57, -0x1.5e214448b3fc6p-3},`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.531ff779ddac6p-57, -0x1.5e214448b3fc6p-3},`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.26d19b9ff8d82p-57, -0x1.8f8b83c69a60bp-3},`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.26d19b9ff8d82p-57, -0x1.8f8b83c69a60bp-3},`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.af1439e521935p-62, -0x1.c0b826a7e4f63p-3},`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.af1439e521935p-62, -0x1.c0b826a7e4f63p-3},`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.42deef11da2c4p-57, -0x1.f19f97b215f1bp-3},`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.42deef11da2c4p-57, -0x1.f19f97b215f1bp-3},`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.824c20ab7aa9ap-56, -0x1.111d262b1f677p-2},`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.824c20ab7aa9ap-56, -0x1.111d262b1f677p-2},`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.5d28da2c4612dp-56, -0x1.294062ed59f06p-2},`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.5d28da2c4612dp-56, -0x1.294062ed59f06p-2},`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0c97c4afa2518p-56, -0x1.4135c94176601p-2},`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0c97c4afa2518p-56, -0x1.4135c94176601p-2},`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.efdc0d58cf62p-62, -0x1.58f9a75ab1fddp-2},`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.efdc0d58cf62p-62, -0x1.58f9a75ab1fddp-2},`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.44b19e0864c5dp-56, -0x1.7088530fa459fp-2},`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.44b19e0864c5dp-56, -0x1.7088530fa459fp-2},`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.72cedd3d5a61p-57, -0x1.87de2a6aea963p-2},`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.72cedd3d5a61p-57, -0x1.87de2a6aea963p-2},`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6da81290bdbabp-57, -0x1.9ef7943a8ed8ap-2},`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6da81290bdbabp-57, -0x1.9ef7943a8ed8ap-2},`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5b362cb974183p-57, -0x1.b5d1009e15ccp-2},`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5b362cb974183p-57, -0x1.b5d1009e15ccp-2},`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6850e59c37f8fp-58, -0x1.cc66e9931c45ep-2},`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6850e59c37f8fp-58, -0x1.cc66e9931c45ep-2},`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e0d891d3c6841p-58, -0x1.e2b5d3806f63bp-2},`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e0d891d3c6841p-58, -0x1.e2b5d3806f63bp-2},`。
- **L236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2ec1fc1b776b8p-60, -0x1.f8ba4dbf89abap-2},`.
  **L236 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2ec1fc1b776b8p-60, -0x1.f8ba4dbf89abap-2},`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.a5a014347406cp-55, -0x1.073879922ffeep-1},`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.a5a014347406cp-55, -0x1.073879922ffeep-1},`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.ef23b69abe4f1p-55, -0x1.11eb3541b4b23p-1},`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.ef23b69abe4f1p-55, -0x1.11eb3541b4b23p-1},`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.b25dd267f66p-55, -0x1.1c73b39ae68c8p-1},`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.b25dd267f66p-55, -0x1.1c73b39ae68c8p-1},`。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.5da743ef3770cp-55, -0x1.26d054cdd12dfp-1},`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.5da743ef3770cp-55, -0x1.26d054cdd12dfp-1},`。

### Lines 241-264

````cpp
    {0x1.efcc626f74a6fp-57, -0x1.30ff7fce17035p-1},
    {-0x1.e3e25e3954964p-56, -0x1.3affa292050b9p-1},
    {-0x1.8076a2cfdc6b3p-57, -0x1.44cf325091dd6p-1},
    {-0x1.3c293edceb327p-57, -0x1.4e6cabbe3e5e9p-1},
    {0x1.75720992bfbb2p-55, -0x1.57d69348cecap-1},
    {0x1.251b352ff2a37p-56, -0x1.610b7551d2cdfp-1},
    {0x1.bdd3413b26456p-55, -0x1.6a09e667f3bcdp-1},
    {-0x1.0d4ef0f1d915cp-55, -0x1.72d0837efff96p-1},
    {0x1.0f537acdf0ad7p-56, -0x1.7b5df226aafafp-1},
    {0x1.6f420f8ea3475p-56, -0x1.83b0e0bff976ep-1},
    {0x1.2c5e12ed1336dp-55, -0x1.8bc806b151741p-1},
    {-0x1.3d419a920df0bp-55, -0x1.93a22499263fbp-1},
    {0x1.30ee286712474p-55, -0x1.9b3e047f38741p-1},
    {0x1.128bb015df175p-56, -0x1.a29a7a0462782p-1},
    {-0x1.9f630e8b6dac8p-60, -0x1.a9b66290ea1a3p-1},
    {0x1.926da300ffccep-55, -0x1.b090a581502p-1},
    {0x1.bc69f324e6d61p-55, -0x1.b728345196e3ep-1},
    {0x1.825a732ac700ap-55, -0x1.bd7c0ac6f952ap-1},
    {0x1.6e0b1757c8d07p-56, -0x1.c38b2f180bdb1p-1},
    {0x1.2fb761e946603p-58, -0x1.c954b213411f5p-1},
    {0x1.e7b6bb5ab58aep-58, -0x1.ced7af43cc773p-1},
    {0x1.4ef5295d25af2p-55, -0x1.d4134d14dc93ap-1},
    {-0x1.457e610231ac2p-56, -0x1.d906bcf328d46p-1},
    {-0x1.83c37c6107db3p-55, -0x1.ddb13b6ccc23cp-1},
````
- **L241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.efcc626f74a6fp-57, -0x1.30ff7fce17035p-1},`.
  **L241 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.efcc626f74a6fp-57, -0x1.30ff7fce17035p-1},`。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e3e25e3954964p-56, -0x1.3affa292050b9p-1},`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e3e25e3954964p-56, -0x1.3affa292050b9p-1},`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.8076a2cfdc6b3p-57, -0x1.44cf325091dd6p-1},`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.8076a2cfdc6b3p-57, -0x1.44cf325091dd6p-1},`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.3c293edceb327p-57, -0x1.4e6cabbe3e5e9p-1},`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.3c293edceb327p-57, -0x1.4e6cabbe3e5e9p-1},`。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.75720992bfbb2p-55, -0x1.57d69348cecap-1},`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.75720992bfbb2p-55, -0x1.57d69348cecap-1},`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.251b352ff2a37p-56, -0x1.610b7551d2cdfp-1},`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.251b352ff2a37p-56, -0x1.610b7551d2cdfp-1},`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.bdd3413b26456p-55, -0x1.6a09e667f3bcdp-1},`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.bdd3413b26456p-55, -0x1.6a09e667f3bcdp-1},`。
- **L248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0d4ef0f1d915cp-55, -0x1.72d0837efff96p-1},`.
  **L248 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0d4ef0f1d915cp-55, -0x1.72d0837efff96p-1},`。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.0f537acdf0ad7p-56, -0x1.7b5df226aafafp-1},`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.0f537acdf0ad7p-56, -0x1.7b5df226aafafp-1},`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6f420f8ea3475p-56, -0x1.83b0e0bff976ep-1},`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6f420f8ea3475p-56, -0x1.83b0e0bff976ep-1},`。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2c5e12ed1336dp-55, -0x1.8bc806b151741p-1},`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2c5e12ed1336dp-55, -0x1.8bc806b151741p-1},`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.3d419a920df0bp-55, -0x1.93a22499263fbp-1},`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.3d419a920df0bp-55, -0x1.93a22499263fbp-1},`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.30ee286712474p-55, -0x1.9b3e047f38741p-1},`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.30ee286712474p-55, -0x1.9b3e047f38741p-1},`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.128bb015df175p-56, -0x1.a29a7a0462782p-1},`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.128bb015df175p-56, -0x1.a29a7a0462782p-1},`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.9f630e8b6dac8p-60, -0x1.a9b66290ea1a3p-1},`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.9f630e8b6dac8p-60, -0x1.a9b66290ea1a3p-1},`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.926da300ffccep-55, -0x1.b090a581502p-1},`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.926da300ffccep-55, -0x1.b090a581502p-1},`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.bc69f324e6d61p-55, -0x1.b728345196e3ep-1},`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.bc69f324e6d61p-55, -0x1.b728345196e3ep-1},`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.825a732ac700ap-55, -0x1.bd7c0ac6f952ap-1},`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.825a732ac700ap-55, -0x1.bd7c0ac6f952ap-1},`。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6e0b1757c8d07p-56, -0x1.c38b2f180bdb1p-1},`.
  **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6e0b1757c8d07p-56, -0x1.c38b2f180bdb1p-1},`。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2fb761e946603p-58, -0x1.c954b213411f5p-1},`.
  **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2fb761e946603p-58, -0x1.c954b213411f5p-1},`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e7b6bb5ab58aep-58, -0x1.ced7af43cc773p-1},`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e7b6bb5ab58aep-58, -0x1.ced7af43cc773p-1},`。
- **L262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.4ef5295d25af2p-55, -0x1.d4134d14dc93ap-1},`.
  **L262 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.4ef5295d25af2p-55, -0x1.d4134d14dc93ap-1},`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.457e610231ac2p-56, -0x1.d906bcf328d46p-1},`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.457e610231ac2p-56, -0x1.d906bcf328d46p-1},`。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.83c37c6107db3p-55, -0x1.ddb13b6ccc23cp-1},`.
  **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.83c37c6107db3p-55, -0x1.ddb13b6ccc23cp-1},`。

### Lines 265-288

````cpp
    {0x1.014c76c126527p-55, -0x1.e212104f686e5p-1},
    {0x1.16b56f2847754p-57, -0x1.e6288ec48e112p-1},
    {-0x1.760b1e2e3f81ep-55, -0x1.e9f4156c62ddap-1},
    {-0x1.e82c791f59cc2p-56, -0x1.ed740e7684963p-1},
    {-0x1.52c7adc6b4989p-56, -0x1.f0a7efb9230d7p-1},
    {0x1.d7bafb51f72e6p-56, -0x1.f38f3ac64e589p-1},
    {-0x1.562172a361fd3p-56, -0x1.f6297cff75cbp-1},
    {-0x1.ab256778ffcb6p-56, -0x1.f8764fa714ba9p-1},
    {0x1.7a0a8ca13571fp-55, -0x1.fa7557f08a517p-1},
    {-0x1.1ec8668ecaceep-55, -0x1.fc26470e19fd3p-1},
    {0x1.87df6378811c7p-55, -0x1.fd88da3d12526p-1},
    {-0x1.521ecd0c67e35p-57, -0x1.fe9cdad01883ap-1},
    {0x1.c57bc2e24aa15p-57, -0x1.ff621e3796d7ep-1},
    {0x1.1354d4556e4cbp-55, -0x1.ffd886084cd0dp-1},
    {0, -1},
    {0x1.1354d4556e4cbp-55, -0x1.ffd886084cd0dp-1},
    {0x1.c57bc2e24aa15p-57, -0x1.ff621e3796d7ep-1},
    {-0x1.521ecd0c67e35p-57, -0x1.fe9cdad01883ap-1},
    {0x1.87df6378811c7p-55, -0x1.fd88da3d12526p-1},
    {-0x1.1ec8668ecaceep-55, -0x1.fc26470e19fd3p-1},
    {0x1.7a0a8ca13571fp-55, -0x1.fa7557f08a517p-1},
    {-0x1.ab256778ffcb6p-56, -0x1.f8764fa714ba9p-1},
    {-0x1.562172a361fd3p-56, -0x1.f6297cff75cbp-1},
    {0x1.d7bafb51f72e6p-56, -0x1.f38f3ac64e589p-1},
````
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.014c76c126527p-55, -0x1.e212104f686e5p-1},`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.014c76c126527p-55, -0x1.e212104f686e5p-1},`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.16b56f2847754p-57, -0x1.e6288ec48e112p-1},`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.16b56f2847754p-57, -0x1.e6288ec48e112p-1},`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.760b1e2e3f81ep-55, -0x1.e9f4156c62ddap-1},`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.760b1e2e3f81ep-55, -0x1.e9f4156c62ddap-1},`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e82c791f59cc2p-56, -0x1.ed740e7684963p-1},`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e82c791f59cc2p-56, -0x1.ed740e7684963p-1},`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.52c7adc6b4989p-56, -0x1.f0a7efb9230d7p-1},`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.52c7adc6b4989p-56, -0x1.f0a7efb9230d7p-1},`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.d7bafb51f72e6p-56, -0x1.f38f3ac64e589p-1},`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.d7bafb51f72e6p-56, -0x1.f38f3ac64e589p-1},`。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.562172a361fd3p-56, -0x1.f6297cff75cbp-1},`.
  **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.562172a361fd3p-56, -0x1.f6297cff75cbp-1},`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ab256778ffcb6p-56, -0x1.f8764fa714ba9p-1},`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ab256778ffcb6p-56, -0x1.f8764fa714ba9p-1},`。
- **L273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.7a0a8ca13571fp-55, -0x1.fa7557f08a517p-1},`.
  **L273 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.7a0a8ca13571fp-55, -0x1.fa7557f08a517p-1},`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.1ec8668ecaceep-55, -0x1.fc26470e19fd3p-1},`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.1ec8668ecaceep-55, -0x1.fc26470e19fd3p-1},`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.87df6378811c7p-55, -0x1.fd88da3d12526p-1},`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.87df6378811c7p-55, -0x1.fd88da3d12526p-1},`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.521ecd0c67e35p-57, -0x1.fe9cdad01883ap-1},`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.521ecd0c67e35p-57, -0x1.fe9cdad01883ap-1},`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.c57bc2e24aa15p-57, -0x1.ff621e3796d7ep-1},`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.c57bc2e24aa15p-57, -0x1.ff621e3796d7ep-1},`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1354d4556e4cbp-55, -0x1.ffd886084cd0dp-1},`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1354d4556e4cbp-55, -0x1.ffd886084cd0dp-1},`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0, -1},`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0, -1},`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1354d4556e4cbp-55, -0x1.ffd886084cd0dp-1},`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1354d4556e4cbp-55, -0x1.ffd886084cd0dp-1},`。
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.c57bc2e24aa15p-57, -0x1.ff621e3796d7ep-1},`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.c57bc2e24aa15p-57, -0x1.ff621e3796d7ep-1},`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.521ecd0c67e35p-57, -0x1.fe9cdad01883ap-1},`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.521ecd0c67e35p-57, -0x1.fe9cdad01883ap-1},`。
- **L283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.87df6378811c7p-55, -0x1.fd88da3d12526p-1},`.
  **L283 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.87df6378811c7p-55, -0x1.fd88da3d12526p-1},`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.1ec8668ecaceep-55, -0x1.fc26470e19fd3p-1},`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.1ec8668ecaceep-55, -0x1.fc26470e19fd3p-1},`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.7a0a8ca13571fp-55, -0x1.fa7557f08a517p-1},`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.7a0a8ca13571fp-55, -0x1.fa7557f08a517p-1},`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ab256778ffcb6p-56, -0x1.f8764fa714ba9p-1},`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ab256778ffcb6p-56, -0x1.f8764fa714ba9p-1},`。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.562172a361fd3p-56, -0x1.f6297cff75cbp-1},`.
  **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.562172a361fd3p-56, -0x1.f6297cff75cbp-1},`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.d7bafb51f72e6p-56, -0x1.f38f3ac64e589p-1},`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.d7bafb51f72e6p-56, -0x1.f38f3ac64e589p-1},`。

### Lines 289-312

````cpp
    {-0x1.52c7adc6b4989p-56, -0x1.f0a7efb9230d7p-1},
    {-0x1.e82c791f59cc2p-56, -0x1.ed740e7684963p-1},
    {-0x1.760b1e2e3f81ep-55, -0x1.e9f4156c62ddap-1},
    {0x1.16b56f2847754p-57, -0x1.e6288ec48e112p-1},
    {0x1.014c76c126527p-55, -0x1.e212104f686e5p-1},
    {-0x1.83c37c6107db3p-55, -0x1.ddb13b6ccc23cp-1},
    {-0x1.457e610231ac2p-56, -0x1.d906bcf328d46p-1},
    {0x1.4ef5295d25af2p-55, -0x1.d4134d14dc93ap-1},
    {0x1.e7b6bb5ab58aep-58, -0x1.ced7af43cc773p-1},
    {0x1.2fb761e946603p-58, -0x1.c954b213411f5p-1},
    {0x1.6e0b1757c8d07p-56, -0x1.c38b2f180bdb1p-1},
    {0x1.825a732ac700ap-55, -0x1.bd7c0ac6f952ap-1},
    {0x1.bc69f324e6d61p-55, -0x1.b728345196e3ep-1},
    {0x1.926da300ffccep-55, -0x1.b090a581502p-1},
    {-0x1.9f630e8b6dac8p-60, -0x1.a9b66290ea1a3p-1},
    {0x1.128bb015df175p-56, -0x1.a29a7a0462782p-1},
    {0x1.30ee286712474p-55, -0x1.9b3e047f38741p-1},
    {-0x1.3d419a920df0bp-55, -0x1.93a22499263fbp-1},
    {0x1.2c5e12ed1336dp-55, -0x1.8bc806b151741p-1},
    {0x1.6f420f8ea3475p-56, -0x1.83b0e0bff976ep-1},
    {0x1.0f537acdf0ad7p-56, -0x1.7b5df226aafafp-1},
    {-0x1.0d4ef0f1d915cp-55, -0x1.72d0837efff96p-1},
    {0x1.bdd3413b26456p-55, -0x1.6a09e667f3bcdp-1},
    {0x1.251b352ff2a37p-56, -0x1.610b7551d2cdfp-1},
````
- **L289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.52c7adc6b4989p-56, -0x1.f0a7efb9230d7p-1},`.
  **L289 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.52c7adc6b4989p-56, -0x1.f0a7efb9230d7p-1},`。
- **L290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e82c791f59cc2p-56, -0x1.ed740e7684963p-1},`.
  **L290 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e82c791f59cc2p-56, -0x1.ed740e7684963p-1},`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.760b1e2e3f81ep-55, -0x1.e9f4156c62ddap-1},`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.760b1e2e3f81ep-55, -0x1.e9f4156c62ddap-1},`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.16b56f2847754p-57, -0x1.e6288ec48e112p-1},`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.16b56f2847754p-57, -0x1.e6288ec48e112p-1},`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.014c76c126527p-55, -0x1.e212104f686e5p-1},`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.014c76c126527p-55, -0x1.e212104f686e5p-1},`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.83c37c6107db3p-55, -0x1.ddb13b6ccc23cp-1},`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.83c37c6107db3p-55, -0x1.ddb13b6ccc23cp-1},`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.457e610231ac2p-56, -0x1.d906bcf328d46p-1},`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.457e610231ac2p-56, -0x1.d906bcf328d46p-1},`。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.4ef5295d25af2p-55, -0x1.d4134d14dc93ap-1},`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.4ef5295d25af2p-55, -0x1.d4134d14dc93ap-1},`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e7b6bb5ab58aep-58, -0x1.ced7af43cc773p-1},`.
  **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e7b6bb5ab58aep-58, -0x1.ced7af43cc773p-1},`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2fb761e946603p-58, -0x1.c954b213411f5p-1},`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2fb761e946603p-58, -0x1.c954b213411f5p-1},`。
- **L299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6e0b1757c8d07p-56, -0x1.c38b2f180bdb1p-1},`.
  **L299 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6e0b1757c8d07p-56, -0x1.c38b2f180bdb1p-1},`。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.825a732ac700ap-55, -0x1.bd7c0ac6f952ap-1},`.
  **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.825a732ac700ap-55, -0x1.bd7c0ac6f952ap-1},`。
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.bc69f324e6d61p-55, -0x1.b728345196e3ep-1},`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.bc69f324e6d61p-55, -0x1.b728345196e3ep-1},`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.926da300ffccep-55, -0x1.b090a581502p-1},`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.926da300ffccep-55, -0x1.b090a581502p-1},`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.9f630e8b6dac8p-60, -0x1.a9b66290ea1a3p-1},`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.9f630e8b6dac8p-60, -0x1.a9b66290ea1a3p-1},`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.128bb015df175p-56, -0x1.a29a7a0462782p-1},`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.128bb015df175p-56, -0x1.a29a7a0462782p-1},`。
- **L305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.30ee286712474p-55, -0x1.9b3e047f38741p-1},`.
  **L305 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.30ee286712474p-55, -0x1.9b3e047f38741p-1},`。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.3d419a920df0bp-55, -0x1.93a22499263fbp-1},`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.3d419a920df0bp-55, -0x1.93a22499263fbp-1},`。
- **L307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2c5e12ed1336dp-55, -0x1.8bc806b151741p-1},`.
  **L307 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2c5e12ed1336dp-55, -0x1.8bc806b151741p-1},`。
- **L308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6f420f8ea3475p-56, -0x1.83b0e0bff976ep-1},`.
  **L308 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6f420f8ea3475p-56, -0x1.83b0e0bff976ep-1},`。
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.0f537acdf0ad7p-56, -0x1.7b5df226aafafp-1},`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.0f537acdf0ad7p-56, -0x1.7b5df226aafafp-1},`。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0d4ef0f1d915cp-55, -0x1.72d0837efff96p-1},`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0d4ef0f1d915cp-55, -0x1.72d0837efff96p-1},`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.bdd3413b26456p-55, -0x1.6a09e667f3bcdp-1},`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.bdd3413b26456p-55, -0x1.6a09e667f3bcdp-1},`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.251b352ff2a37p-56, -0x1.610b7551d2cdfp-1},`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.251b352ff2a37p-56, -0x1.610b7551d2cdfp-1},`。

### Lines 313-336

````cpp
    {0x1.75720992bfbb2p-55, -0x1.57d69348cecap-1},
    {-0x1.3c293edceb327p-57, -0x1.4e6cabbe3e5e9p-1},
    {-0x1.8076a2cfdc6b3p-57, -0x1.44cf325091dd6p-1},
    {-0x1.e3e25e3954964p-56, -0x1.3affa292050b9p-1},
    {0x1.efcc626f74a6fp-57, -0x1.30ff7fce17035p-1},
    {0x1.5da743ef3770cp-55, -0x1.26d054cdd12dfp-1},
    {-0x1.b25dd267f66p-55, -0x1.1c73b39ae68c8p-1},
    {0x1.ef23b69abe4f1p-55, -0x1.11eb3541b4b23p-1},
    {0x1.a5a014347406cp-55, -0x1.073879922ffeep-1},
    {0x1.2ec1fc1b776b8p-60, -0x1.f8ba4dbf89abap-2},
    {-0x1.e0d891d3c6841p-58, -0x1.e2b5d3806f63bp-2},
    {-0x1.6850e59c37f8fp-58, -0x1.cc66e9931c45ep-2},
    {-0x1.5b362cb974183p-57, -0x1.b5d1009e15ccp-2},
    {-0x1.6da81290bdbabp-57, -0x1.9ef7943a8ed8ap-2},
    {0x1.72cedd3d5a61p-57, -0x1.87de2a6aea963p-2},
    {0x1.44b19e0864c5dp-56, -0x1.7088530fa459fp-2},
    {0x1.efdc0d58cf62p-62, -0x1.58f9a75ab1fddp-2},
    {-0x1.0c97c4afa2518p-56, -0x1.4135c94176601p-2},
    {0x1.5d28da2c4612dp-56, -0x1.294062ed59f06p-2},
    {-0x1.824c20ab7aa9ap-56, -0x1.111d262b1f677p-2},
    {0x1.42deef11da2c4p-57, -0x1.f19f97b215f1bp-3},
    {0x1.af1439e521935p-62, -0x1.c0b826a7e4f63p-3},
    {0x1.26d19b9ff8d82p-57, -0x1.8f8b83c69a60bp-3},
    {-0x1.531ff779ddac6p-57, -0x1.5e214448b3fc6p-3},
````
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.75720992bfbb2p-55, -0x1.57d69348cecap-1},`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.75720992bfbb2p-55, -0x1.57d69348cecap-1},`。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.3c293edceb327p-57, -0x1.4e6cabbe3e5e9p-1},`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.3c293edceb327p-57, -0x1.4e6cabbe3e5e9p-1},`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.8076a2cfdc6b3p-57, -0x1.44cf325091dd6p-1},`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.8076a2cfdc6b3p-57, -0x1.44cf325091dd6p-1},`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e3e25e3954964p-56, -0x1.3affa292050b9p-1},`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e3e25e3954964p-56, -0x1.3affa292050b9p-1},`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.efcc626f74a6fp-57, -0x1.30ff7fce17035p-1},`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.efcc626f74a6fp-57, -0x1.30ff7fce17035p-1},`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.5da743ef3770cp-55, -0x1.26d054cdd12dfp-1},`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.5da743ef3770cp-55, -0x1.26d054cdd12dfp-1},`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.b25dd267f66p-55, -0x1.1c73b39ae68c8p-1},`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.b25dd267f66p-55, -0x1.1c73b39ae68c8p-1},`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.ef23b69abe4f1p-55, -0x1.11eb3541b4b23p-1},`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.ef23b69abe4f1p-55, -0x1.11eb3541b4b23p-1},`。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.a5a014347406cp-55, -0x1.073879922ffeep-1},`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.a5a014347406cp-55, -0x1.073879922ffeep-1},`。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2ec1fc1b776b8p-60, -0x1.f8ba4dbf89abap-2},`.
  **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2ec1fc1b776b8p-60, -0x1.f8ba4dbf89abap-2},`。
- **L323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.e0d891d3c6841p-58, -0x1.e2b5d3806f63bp-2},`.
  **L323 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.e0d891d3c6841p-58, -0x1.e2b5d3806f63bp-2},`。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6850e59c37f8fp-58, -0x1.cc66e9931c45ep-2},`.
  **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6850e59c37f8fp-58, -0x1.cc66e9931c45ep-2},`。
- **L325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5b362cb974183p-57, -0x1.b5d1009e15ccp-2},`.
  **L325 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5b362cb974183p-57, -0x1.b5d1009e15ccp-2},`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6da81290bdbabp-57, -0x1.9ef7943a8ed8ap-2},`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6da81290bdbabp-57, -0x1.9ef7943a8ed8ap-2},`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.72cedd3d5a61p-57, -0x1.87de2a6aea963p-2},`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.72cedd3d5a61p-57, -0x1.87de2a6aea963p-2},`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.44b19e0864c5dp-56, -0x1.7088530fa459fp-2},`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.44b19e0864c5dp-56, -0x1.7088530fa459fp-2},`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.efdc0d58cf62p-62, -0x1.58f9a75ab1fddp-2},`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.efdc0d58cf62p-62, -0x1.58f9a75ab1fddp-2},`。
- **L330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.0c97c4afa2518p-56, -0x1.4135c94176601p-2},`.
  **L330 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.0c97c4afa2518p-56, -0x1.4135c94176601p-2},`。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.5d28da2c4612dp-56, -0x1.294062ed59f06p-2},`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.5d28da2c4612dp-56, -0x1.294062ed59f06p-2},`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.824c20ab7aa9ap-56, -0x1.111d262b1f677p-2},`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.824c20ab7aa9ap-56, -0x1.111d262b1f677p-2},`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.42deef11da2c4p-57, -0x1.f19f97b215f1bp-3},`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.42deef11da2c4p-57, -0x1.f19f97b215f1bp-3},`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.af1439e521935p-62, -0x1.c0b826a7e4f63p-3},`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.af1439e521935p-62, -0x1.c0b826a7e4f63p-3},`。
- **L335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.26d19b9ff8d82p-57, -0x1.8f8b83c69a60bp-3},`.
  **L335 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.26d19b9ff8d82p-57, -0x1.8f8b83c69a60bp-3},`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.531ff779ddac6p-57, -0x1.5e214448b3fc6p-3},`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.531ff779ddac6p-57, -0x1.5e214448b3fc6p-3},`。

### Lines 337-352

````cpp
    {-0x1.13000a89a11ep-58, -0x1.2c8106e8e613ap-3},
    {-0x1.a2704729ae56dp-59, -0x1.f564e56a9730ep-4},
    {0x1.e2718d26ed688p-60, -0x1.917a6bc29b42cp-4},
    {0x1.9a088a8bf6b2cp-59, -0x1.2d52092ce19f6p-4},
    {0x1.912bd0d569a9p-61, -0x1.91f65f10dd814p-5},
    {0x1.b1d63091a013p-64, -0x1.92155f7a3667ep-6},
#endif // !LIBC_MATH_HAS_SMALL_TABLES
};

} // namespace range_reduction_double_internal

} // namespace math

} // namespace LIBC_NAMESPACE_DECL

#endif // LLVM_LIBC_SRC___SUPPORT_MATH_RANGE_REDUCTION_DOUBLE_FMA_H
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.13000a89a11ep-58, -0x1.2c8106e8e613ap-3},`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.13000a89a11ep-58, -0x1.2c8106e8e613ap-3},`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a2704729ae56dp-59, -0x1.f564e56a9730ep-4},`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a2704729ae56dp-59, -0x1.f564e56a9730ep-4},`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e2718d26ed688p-60, -0x1.917a6bc29b42cp-4},`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e2718d26ed688p-60, -0x1.917a6bc29b42cp-4},`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.9a088a8bf6b2cp-59, -0x1.2d52092ce19f6p-4},`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.9a088a8bf6b2cp-59, -0x1.2d52092ce19f6p-4},`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.912bd0d569a9p-61, -0x1.91f65f10dd814p-5},`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.912bd0d569a9p-61, -0x1.91f65f10dd814p-5},`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.b1d63091a013p-64, -0x1.92155f7a3667ep-6},`.
  **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.b1d63091a013p-64, -0x1.92155f7a3667ep-6},`。
- **L343 EN**: Closes the current preprocessor conditional block or header guard.
  **L343 CN**: 结束当前预处理条件块或头文件保护。
- **L344 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L344 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L345 EN**: Blank line separating nearby declarations or logic.
  **L345 CN**: 空行，用于分隔相邻声明或逻辑。
- **L346 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace range_reduction_double_internal`.
  **L346 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace range_reduction_double_internal`。
- **L347 EN**: Blank line separating nearby declarations or logic.
  **L347 CN**: 空行，用于分隔相邻声明或逻辑。
- **L348 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace math`.
  **L348 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace math`。
- **L349 EN**: Blank line separating nearby declarations or logic.
  **L349 CN**: 空行，用于分隔相邻声明或逻辑。
- **L350 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace LIBC_NAMESPACE_DECL`.
  **L350 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace LIBC_NAMESPACE_DECL`。
- **L351 EN**: Blank line separating nearby declarations or logic.
  **L351 CN**: 空行，用于分隔相邻声明或逻辑。
- **L352 EN**: Closes the current preprocessor conditional block or header guard.
  **L352 CN**: 结束当前预处理条件块或头文件保护。

## Key Concepts / 关键概念

- **Floating-point support kernels / 浮点支撑内核**: Provides reusable math internals such as argument reduction, approximation helpers, and type-specific wrappers. / 提供可复用的数学内部组件，例如自变量归约、近似辅助逻辑以及按类型区分的包装层。
- **Argument reduction tables / 自变量归约表**: Stores constants and helper logic used to reduce arguments for transcendental functions. / 存储超越函数自变量归约使用的常量与辅助逻辑。
- **Floating-point bit manipulation / 浮点位级操作**: Uses helper types to inspect exponents, mantissas, special values, and sign bits directly. / 使用辅助类型直接检查指数、尾数、特殊值和符号位。
- **Numeric text conversion / 数字文本转换**: Builds or consumes textual numeric representations while tracking buffer sizes and edge cases. / 在跟踪缓冲区大小和边界情况的同时，构建或消费数字文本表示。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `src/__support/FPUtil/FPBits.h`, `src/__support/FPUtil/double_double.h`, `src/__support/FPUtil/multiply_add.h`, `src/__support/FPUtil/nearest_integer.h`, `src/__support/macros/config.h`, `src/__support/math/range_reduction_double_common.h`
- **Dependency categories / 依赖类别**: LLVM libc floating-point utility helpers / LLVM libc 浮点工具辅助组件 (4), LLVM libc configuration and attribute macros / LLVM libc 配置与属性宏 (1), LLVM libc internal support utilities / LLVM libc 内部支撑工具 (1)

- `src/__support/FPUtil/FPBits.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/double_double.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/multiply_add.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/nearest_integer.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/macros/config.h`: Provides LLVM libc configuration and attribute macros. / 提供LLVM libc 配置与属性宏。
- `src/__support/math/range_reduction_double_common.h`: Provides LLVM libc internal support utilities. / 提供LLVM libc 内部支撑工具。
