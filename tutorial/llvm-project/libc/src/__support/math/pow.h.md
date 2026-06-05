# pow.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `libc/src/__support/math/pow.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares the internal LLVM libc interface for `pow`.
  - **CN**: 声明 `pow` 的 LLVM libc 内部接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- Implementation header for pow ---------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_POW_H
#define LLVM_LIBC_SRC___SUPPORT_MATH_POW_H

#include "common_constants.h" // Lookup tables EXP_M1 and EXP_M2.
#include "exp_constants.h"    // Lookup tables EXP_M1 and EXP_M2.
#include "hdr/errno_macros.h"
#include "hdr/fenv_macros.h"
#include "src/__support/CPP/bit.h"
#include "src/__support/FPUtil/FEnvImpl.h"
#include "src/__support/FPUtil/FPBits.h"
#include "src/__support/FPUtil/PolyEval.h"
#include "src/__support/FPUtil/double_double.h"
#include "src/__support/FPUtil/multiply_add.h"
#include "src/__support/FPUtil/nearest_integer.h"
#include "src/__support/FPUtil/sqrt.h" // Speedup for pow(x, 1/2) = sqrt(x)
#include "src/__support/common.h"
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
- **L9 EN**: Starts a header guard condition: `#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_POW_H`.
  **L9 CN**: 开始头文件保护条件：`#ifndef LLVM_LIBC_SRC___SUPPORT_MATH_POW_H`。
- **L10 EN**: Defines macro `LLVM_LIBC_SRC___SUPPORT_MATH_POW_H` for compile-time constants, aliases, or dispatch control.
  **L10 CN**: 定义宏 `LLVM_LIBC_SRC___SUPPORT_MATH_POW_H`，用于编译期常量、别名或分发控制。
- **L11 EN**: Blank line separating nearby declarations or logic.
  **L11 CN**: 空行，用于分隔相邻声明或逻辑。
- **L12 EN**: Includes "common_constants.h" to access nearby local declarations.
  **L12 CN**: 引入 "common_constants.h" 以使用附近的本地声明。
- **L13 EN**: Includes "exp_constants.h" to access nearby local declarations.
  **L13 CN**: 引入 "exp_constants.h" 以使用附近的本地声明。
- **L14 EN**: Includes "hdr/errno_macros.h" to access ABI-facing generated header declarations.
  **L14 CN**: 引入 "hdr/errno_macros.h" 以使用面向 ABI 的生成头声明。
- **L15 EN**: Includes "hdr/fenv_macros.h" to access ABI-facing generated header declarations.
  **L15 CN**: 引入 "hdr/fenv_macros.h" 以使用面向 ABI 的生成头声明。
- **L16 EN**: Includes "src/__support/CPP/bit.h" to access LLVM libc C++ support utilities.
  **L16 CN**: 引入 "src/__support/CPP/bit.h" 以使用LLVM libc C++ 支撑工具。
- **L17 EN**: Includes "src/__support/FPUtil/FEnvImpl.h" to access LLVM libc floating-point utility helpers.
  **L17 CN**: 引入 "src/__support/FPUtil/FEnvImpl.h" 以使用LLVM libc 浮点工具辅助组件。
- **L18 EN**: Includes "src/__support/FPUtil/FPBits.h" to access LLVM libc floating-point utility helpers.
  **L18 CN**: 引入 "src/__support/FPUtil/FPBits.h" 以使用LLVM libc 浮点工具辅助组件。
- **L19 EN**: Includes "src/__support/FPUtil/PolyEval.h" to access LLVM libc floating-point utility helpers.
  **L19 CN**: 引入 "src/__support/FPUtil/PolyEval.h" 以使用LLVM libc 浮点工具辅助组件。
- **L20 EN**: Includes "src/__support/FPUtil/double_double.h" to access LLVM libc floating-point utility helpers.
  **L20 CN**: 引入 "src/__support/FPUtil/double_double.h" 以使用LLVM libc 浮点工具辅助组件。
- **L21 EN**: Includes "src/__support/FPUtil/multiply_add.h" to access LLVM libc floating-point utility helpers.
  **L21 CN**: 引入 "src/__support/FPUtil/multiply_add.h" 以使用LLVM libc 浮点工具辅助组件。
- **L22 EN**: Includes "src/__support/FPUtil/nearest_integer.h" to access LLVM libc floating-point utility helpers.
  **L22 CN**: 引入 "src/__support/FPUtil/nearest_integer.h" 以使用LLVM libc 浮点工具辅助组件。
- **L23 EN**: Includes "src/__support/FPUtil/sqrt.h" to access LLVM libc floating-point utility helpers.
  **L23 CN**: 引入 "src/__support/FPUtil/sqrt.h" 以使用LLVM libc 浮点工具辅助组件。
- **L24 EN**: Includes "src/__support/common.h" to access LLVM libc internal support utilities.
  **L24 CN**: 引入 "src/__support/common.h" 以使用LLVM libc 内部支撑工具。

### Lines 25-48

````cpp
#include "src/__support/macros/config.h"
#include "src/__support/macros/optimization.h" // LIBC_UNLIKELY

namespace LIBC_NAMESPACE_DECL {

namespace math {

namespace pow_internal {

using fputil::DoubleDouble;

using namespace common_constants_internal;

// Constants for log2(x) range reduction, generated by Sollya with:
// > for i from 0 to 127 do {
//     r = 2^-8 * ceil( 2^8 * (1 - 2^(-8)) / (1 + i*2^-7) );
//     b = nearestint(log2(r) * 2^41) * 2^-41;
//     c = round(log2(r) - b, D, RN);
//     print("{", -c, ",", -b, "},");
//   };
// This is the same as -log2(RD[i]), with the least significant bits of the
// high part set to be 2^-41, so that the sum of high parts + e_x is exact in
// double precision.
// We also replace the first and the last ones to be 0.
````
- **L25 EN**: Includes "src/__support/macros/config.h" to access LLVM libc configuration and attribute macros.
  **L25 CN**: 引入 "src/__support/macros/config.h" 以使用LLVM libc 配置与属性宏。
- **L26 EN**: Includes "src/__support/macros/optimization.h" to access LLVM libc configuration and attribute macros.
  **L26 CN**: 引入 "src/__support/macros/optimization.h" 以使用LLVM libc 配置与属性宏。
- **L27 EN**: Blank line separating nearby declarations or logic.
  **L27 CN**: 空行，用于分隔相邻声明或逻辑。
- **L28 EN**: Opens namespace scope `LIBC_NAMESPACE_DECL`.
  **L28 CN**: 打开命名空间作用域 `LIBC_NAMESPACE_DECL`。
- **L29 EN**: Blank line separating nearby declarations or logic.
  **L29 CN**: 空行，用于分隔相邻声明或逻辑。
- **L30 EN**: Opens namespace scope `math`.
  **L30 CN**: 打开命名空间作用域 `math`。
- **L31 EN**: Blank line separating nearby declarations or logic.
  **L31 CN**: 空行，用于分隔相邻声明或逻辑。
- **L32 EN**: Opens namespace scope `pow_internal`.
  **L32 CN**: 打开命名空间作用域 `pow_internal`。
- **L33 EN**: Blank line separating nearby declarations or logic.
  **L33 CN**: 空行，用于分隔相邻声明或逻辑。
- **L34 EN**: Introduces a using declaration or alias: `using fputil::DoubleDouble;`.
  **L34 CN**: 引入一条 using 声明或别名：`using fputil::DoubleDouble;`。
- **L35 EN**: Blank line separating nearby declarations or logic.
  **L35 CN**: 空行，用于分隔相邻声明或逻辑。
- **L36 EN**: Brings namespace `common_constants_internal` into the local scope.
  **L36 CN**: 将命名空间 `common_constants_internal` 引入当前作用域。
- **L37 EN**: Blank line separating nearby declarations or logic.
  **L37 CN**: 空行，用于分隔相邻声明或逻辑。
- **L38 EN**: Comment documents nearby intent or constraints: `Constants for log2(x) range reduction, generated by Sollya with:`.
  **L38 CN**: 注释说明附近代码的意图或约束：`Constants for log2(x) range reduction, generated by Sollya with:`。
- **L39 EN**: Comment documents nearby intent or constraints: `> for i from 0 to 127 do {`.
  **L39 CN**: 注释说明附近代码的意图或约束：`> for i from 0 to 127 do {`。
- **L40 EN**: Comment documents nearby intent or constraints: `r = 2^-8 * ceil( 2^8 * (1 - 2^(-8)) / (1 + i*2^-7) );`.
  **L40 CN**: 注释说明附近代码的意图或约束：`r = 2^-8 * ceil( 2^8 * (1 - 2^(-8)) / (1 + i*2^-7) );`。
- **L41 EN**: Comment documents nearby intent or constraints: `b = nearestint(log2(r) * 2^41) * 2^-41;`.
  **L41 CN**: 注释说明附近代码的意图或约束：`b = nearestint(log2(r) * 2^41) * 2^-41;`。
- **L42 EN**: Comment documents nearby intent or constraints: `c = round(log2(r) - b, D, RN);`.
  **L42 CN**: 注释说明附近代码的意图或约束：`c = round(log2(r) - b, D, RN);`。
- **L43 EN**: Comment documents nearby intent or constraints: `print("{", -c, ",", -b, "},");`.
  **L43 CN**: 注释说明附近代码的意图或约束：`print("{", -c, ",", -b, "},");`。
- **L44 EN**: Comment documents nearby intent or constraints: `};`.
  **L44 CN**: 注释说明附近代码的意图或约束：`};`。
- **L45 EN**: Comment documents nearby intent or constraints: `This is the same as -log2(RD[i]), with the least significant bits of the`.
  **L45 CN**: 注释说明附近代码的意图或约束：`This is the same as -log2(RD[i]), with the least significant bits of the`。
- **L46 EN**: Comment documents nearby intent or constraints: `high part set to be 2^-41, so that the sum of high parts + e_x is exact in`.
  **L46 CN**: 注释说明附近代码的意图或约束：`high part set to be 2^-41, so that the sum of high parts + e_x is exact in`。
- **L47 EN**: Comment documents nearby intent or constraints: `double precision.`.
  **L47 CN**: 注释说明附近代码的意图或约束：`double precision.`。
- **L48 EN**: Comment documents nearby intent or constraints: `We also replace the first and the last ones to be 0.`.
  **L48 CN**: 注释说明附近代码的意图或约束：`We also replace the first and the last ones to be 0.`。

### Lines 49-72

````cpp
LIBC_INLINE_VAR constexpr DoubleDouble LOG2_R_DD[128] = {
    {0.0, 0.0},
    {-0x1.19b14945cf6bap-44, 0x1.72c7ba21p-7},
    {-0x1.95539356f93dcp-43, 0x1.743ee862p-6},
    {0x1.abe0a48f83604p-43, 0x1.184b8e4c5p-5},
    {0x1.635577970e04p-43, 0x1.77394c9d9p-5},
    {-0x1.401fbaaa67e3cp-45, 0x1.d6ebd1f2p-5},
    {-0x1.5b1799ceaeb51p-43, 0x1.1bb32a6008p-4},
    {0x1.7c407050799bfp-43, 0x1.4c560fe688p-4},
    {0x1.da6339da288fcp-43, 0x1.7d60496cf8p-4},
    {0x1.be4f6f22dbbadp-43, 0x1.960caf9ab8p-4},
    {-0x1.c760bc9b188c4p-45, 0x1.c7b528b71p-4},
    {0x1.164e932b2d51cp-44, 0x1.f9c95dc1dp-4},
    {0x1.924ae921f7ecap-45, 0x1.097e38ce6p-3},
    {-0x1.6d25a5b8a19b2p-44, 0x1.22dadc2ab4p-3},
    {0x1.e50a1644ac794p-43, 0x1.3c6fb650ccp-3},
    {0x1.f34baa74a7942p-43, 0x1.494f863b8cp-3},
    {-0x1.8f7aac147fdc1p-46, 0x1.633a8bf438p-3},
    {0x1.f84be19cb9578p-43, 0x1.7046031c78p-3},
    {-0x1.66cccab240e9p-46, 0x1.8a8980abfcp-3},
    {-0x1.3f7a55cd2af4cp-47, 0x1.97c1cb13c8p-3},
    {0x1.3458cde69308cp-43, 0x1.b2602497d4p-3},
    {-0x1.667f21fa8423fp-44, 0x1.bfc67a8p-3},
    {0x1.d2fe4574e09b9p-47, 0x1.dac22d3e44p-3},
````
- **L49 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L49 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L50 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0, 0.0},`.
  **L50 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0, 0.0},`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.19b14945cf6bap-44, 0x1.72c7ba21p-7},`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.19b14945cf6bap-44, 0x1.72c7ba21p-7},`。
- **L52 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.95539356f93dcp-43, 0x1.743ee862p-6},`.
  **L52 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.95539356f93dcp-43, 0x1.743ee862p-6},`。
- **L53 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.abe0a48f83604p-43, 0x1.184b8e4c5p-5},`.
  **L53 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.abe0a48f83604p-43, 0x1.184b8e4c5p-5},`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.635577970e04p-43, 0x1.77394c9d9p-5},`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.635577970e04p-43, 0x1.77394c9d9p-5},`。
- **L55 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.401fbaaa67e3cp-45, 0x1.d6ebd1f2p-5},`.
  **L55 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.401fbaaa67e3cp-45, 0x1.d6ebd1f2p-5},`。
- **L56 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5b1799ceaeb51p-43, 0x1.1bb32a6008p-4},`.
  **L56 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5b1799ceaeb51p-43, 0x1.1bb32a6008p-4},`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.7c407050799bfp-43, 0x1.4c560fe688p-4},`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.7c407050799bfp-43, 0x1.4c560fe688p-4},`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.da6339da288fcp-43, 0x1.7d60496cf8p-4},`.
  **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.da6339da288fcp-43, 0x1.7d60496cf8p-4},`。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.be4f6f22dbbadp-43, 0x1.960caf9ab8p-4},`.
  **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.be4f6f22dbbadp-43, 0x1.960caf9ab8p-4},`。
- **L60 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.c760bc9b188c4p-45, 0x1.c7b528b71p-4},`.
  **L60 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.c760bc9b188c4p-45, 0x1.c7b528b71p-4},`。
- **L61 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.164e932b2d51cp-44, 0x1.f9c95dc1dp-4},`.
  **L61 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.164e932b2d51cp-44, 0x1.f9c95dc1dp-4},`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.924ae921f7ecap-45, 0x1.097e38ce6p-3},`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.924ae921f7ecap-45, 0x1.097e38ce6p-3},`。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6d25a5b8a19b2p-44, 0x1.22dadc2ab4p-3},`.
  **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6d25a5b8a19b2p-44, 0x1.22dadc2ab4p-3},`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e50a1644ac794p-43, 0x1.3c6fb650ccp-3},`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e50a1644ac794p-43, 0x1.3c6fb650ccp-3},`。
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.f34baa74a7942p-43, 0x1.494f863b8cp-3},`.
  **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.f34baa74a7942p-43, 0x1.494f863b8cp-3},`。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.8f7aac147fdc1p-46, 0x1.633a8bf438p-3},`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.8f7aac147fdc1p-46, 0x1.633a8bf438p-3},`。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.f84be19cb9578p-43, 0x1.7046031c78p-3},`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.f84be19cb9578p-43, 0x1.7046031c78p-3},`。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.66cccab240e9p-46, 0x1.8a8980abfcp-3},`.
  **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.66cccab240e9p-46, 0x1.8a8980abfcp-3},`。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.3f7a55cd2af4cp-47, 0x1.97c1cb13c8p-3},`.
  **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.3f7a55cd2af4cp-47, 0x1.97c1cb13c8p-3},`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3458cde69308cp-43, 0x1.b2602497d4p-3},`.
  **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3458cde69308cp-43, 0x1.b2602497d4p-3},`。
- **L71 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.667f21fa8423fp-44, 0x1.bfc67a8p-3},`.
  **L71 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.667f21fa8423fp-44, 0x1.bfc67a8p-3},`。
- **L72 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.d2fe4574e09b9p-47, 0x1.dac22d3e44p-3},`.
  **L72 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.d2fe4574e09b9p-47, 0x1.dac22d3e44p-3},`。

### Lines 73-96

````cpp
    {0x1.367bde40c5e6dp-43, 0x1.e857d3d36p-3},
    {0x1.d45da26510033p-46, 0x1.01d9bbcfa6p-2},
    {-0x1.7204f55bbf90dp-44, 0x1.08bce0d96p-2},
    {-0x1.d4f1b95e0ff45p-43, 0x1.169c05364p-2},
    {0x1.c20d74c0211bfp-44, 0x1.1d982c9d52p-2},
    {0x1.ad89a083e072ap-43, 0x1.249cd2b13cp-2},
    {0x1.cd0cb4492f1bcp-43, 0x1.32bfee370ep-2},
    {-0x1.2101a9685c779p-47, 0x1.39de8e155ap-2},
    {0x1.9451cd394fe8dp-43, 0x1.4106017c3ep-2},
    {0x1.661e393a16b95p-44, 0x1.4f6fbb2cecp-2},
    {-0x1.c6d8d86531d56p-44, 0x1.56b22e6b58p-2},
    {0x1.c1c885adb21d3p-43, 0x1.5dfdcf1eeap-2},
    {0x1.3bb5921006679p-45, 0x1.6552b49986p-2},
    {0x1.1d406db502403p-43, 0x1.6cb0f6865cp-2},
    {0x1.55a63e278bad5p-43, 0x1.7b89f02cf2p-2},
    {-0x1.66ae2a7ada553p-49, 0x1.8304d90c12p-2},
    {-0x1.66cccab240e9p-45, 0x1.8a8980abfcp-2},
    {-0x1.62404772a151dp-45, 0x1.921800924ep-2},
    {0x1.ac9bca36fd02ep-44, 0x1.99b072a96cp-2},
    {0x1.4bc302ffa76fbp-43, 0x1.a8ff97181p-2},
    {0x1.01fea1ec47c71p-43, 0x1.b0b67f4f46p-2},
    {-0x1.f20203b3186a6p-43, 0x1.b877c57b1cp-2},
    {-0x1.2642415d47384p-45, 0x1.c043859e3p-2},
    {-0x1.bc76a2753b99bp-50, 0x1.c819dc2d46p-2},
````
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.367bde40c5e6dp-43, 0x1.e857d3d36p-3},`.
  **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.367bde40c5e6dp-43, 0x1.e857d3d36p-3},`。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.d45da26510033p-46, 0x1.01d9bbcfa6p-2},`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.d45da26510033p-46, 0x1.01d9bbcfa6p-2},`。
- **L75 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.7204f55bbf90dp-44, 0x1.08bce0d96p-2},`.
  **L75 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.7204f55bbf90dp-44, 0x1.08bce0d96p-2},`。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.d4f1b95e0ff45p-43, 0x1.169c05364p-2},`.
  **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.d4f1b95e0ff45p-43, 0x1.169c05364p-2},`。
- **L77 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.c20d74c0211bfp-44, 0x1.1d982c9d52p-2},`.
  **L77 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.c20d74c0211bfp-44, 0x1.1d982c9d52p-2},`。
- **L78 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.ad89a083e072ap-43, 0x1.249cd2b13cp-2},`.
  **L78 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.ad89a083e072ap-43, 0x1.249cd2b13cp-2},`。
- **L79 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.cd0cb4492f1bcp-43, 0x1.32bfee370ep-2},`.
  **L79 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.cd0cb4492f1bcp-43, 0x1.32bfee370ep-2},`。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2101a9685c779p-47, 0x1.39de8e155ap-2},`.
  **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2101a9685c779p-47, 0x1.39de8e155ap-2},`。
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.9451cd394fe8dp-43, 0x1.4106017c3ep-2},`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.9451cd394fe8dp-43, 0x1.4106017c3ep-2},`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.661e393a16b95p-44, 0x1.4f6fbb2cecp-2},`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.661e393a16b95p-44, 0x1.4f6fbb2cecp-2},`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.c6d8d86531d56p-44, 0x1.56b22e6b58p-2},`.
  **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.c6d8d86531d56p-44, 0x1.56b22e6b58p-2},`。
- **L84 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.c1c885adb21d3p-43, 0x1.5dfdcf1eeap-2},`.
  **L84 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.c1c885adb21d3p-43, 0x1.5dfdcf1eeap-2},`。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3bb5921006679p-45, 0x1.6552b49986p-2},`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3bb5921006679p-45, 0x1.6552b49986p-2},`。
- **L86 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1d406db502403p-43, 0x1.6cb0f6865cp-2},`.
  **L86 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1d406db502403p-43, 0x1.6cb0f6865cp-2},`。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.55a63e278bad5p-43, 0x1.7b89f02cf2p-2},`.
  **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.55a63e278bad5p-43, 0x1.7b89f02cf2p-2},`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.66ae2a7ada553p-49, 0x1.8304d90c12p-2},`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.66ae2a7ada553p-49, 0x1.8304d90c12p-2},`。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.66cccab240e9p-45, 0x1.8a8980abfcp-2},`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.66cccab240e9p-45, 0x1.8a8980abfcp-2},`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.62404772a151dp-45, 0x1.921800924ep-2},`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.62404772a151dp-45, 0x1.921800924ep-2},`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.ac9bca36fd02ep-44, 0x1.99b072a96cp-2},`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.ac9bca36fd02ep-44, 0x1.99b072a96cp-2},`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.4bc302ffa76fbp-43, 0x1.a8ff97181p-2},`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.4bc302ffa76fbp-43, 0x1.a8ff97181p-2},`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.01fea1ec47c71p-43, 0x1.b0b67f4f46p-2},`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.01fea1ec47c71p-43, 0x1.b0b67f4f46p-2},`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.f20203b3186a6p-43, 0x1.b877c57b1cp-2},`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.f20203b3186a6p-43, 0x1.b877c57b1cp-2},`。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2642415d47384p-45, 0x1.c043859e3p-2},`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2642415d47384p-45, 0x1.c043859e3p-2},`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.bc76a2753b99bp-50, 0x1.c819dc2d46p-2},`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.bc76a2753b99bp-50, 0x1.c819dc2d46p-2},`。

### Lines 97-120

````cpp
    {-0x1.da93ae3a5f451p-43, 0x1.cffae611aep-2},
    {-0x1.50e785694a8c6p-43, 0x1.d7e6c0abc4p-2},
    {0x1.c56138c894641p-43, 0x1.dfdd89d586p-2},
    {0x1.5669df6a2b592p-43, 0x1.e7df5fe538p-2},
    {-0x1.ea92d9e0e8ac2p-48, 0x1.efec61b012p-2},
    {0x1.a0331af2e6feap-43, 0x1.f804ae8d0cp-2},
    {0x1.9518ce032f41dp-48, 0x1.0014332bep-1},
    {-0x1.b3b3864c60011p-44, 0x1.042bd4b9a8p-1},
    {-0x1.103e8f00d41c8p-45, 0x1.08494c66b9p-1},
    {0x1.65be75cc3da17p-43, 0x1.0c6caaf0c5p-1},
    {0x1.3676289cd3dd4p-43, 0x1.1096015deep-1},
    {-0x1.41dfc7d7c3321p-43, 0x1.14c560fe69p-1},
    {0x1.e0cda8bd74461p-44, 0x1.18fadb6e2dp-1},
    {0x1.2a606046ad444p-44, 0x1.1d368296b5p-1},
    {0x1.f9ea977a639cp-43, 0x1.217868b0c3p-1},
    {-0x1.50520a377c7ecp-45, 0x1.25c0a0463cp-1},
    {0x1.6e3cb71b554e7p-47, 0x1.2a0f3c3407p-1},
    {-0x1.4275f1035e5e8p-48, 0x1.2e644fac05p-1},
    {-0x1.4275f1035e5e8p-48, 0x1.2e644fac05p-1},
    {-0x1.979a5db68721dp-45, 0x1.32bfee370fp-1},
    {0x1.1ee969a95f529p-43, 0x1.37222bb707p-1},
    {0x1.bb4b69336b66ep-43, 0x1.3b8b1c68fap-1},
    {0x1.d5e6a8a4fb059p-45, 0x1.3ffad4e74fp-1},
    {0x1.3106e404cabb7p-44, 0x1.44716a2c08p-1},
````
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.da93ae3a5f451p-43, 0x1.cffae611aep-2},`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.da93ae3a5f451p-43, 0x1.cffae611aep-2},`。
- **L98 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.50e785694a8c6p-43, 0x1.d7e6c0abc4p-2},`.
  **L98 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.50e785694a8c6p-43, 0x1.d7e6c0abc4p-2},`。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.c56138c894641p-43, 0x1.dfdd89d586p-2},`.
  **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.c56138c894641p-43, 0x1.dfdd89d586p-2},`。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.5669df6a2b592p-43, 0x1.e7df5fe538p-2},`.
  **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.5669df6a2b592p-43, 0x1.e7df5fe538p-2},`。
- **L101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ea92d9e0e8ac2p-48, 0x1.efec61b012p-2},`.
  **L101 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ea92d9e0e8ac2p-48, 0x1.efec61b012p-2},`。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.a0331af2e6feap-43, 0x1.f804ae8d0cp-2},`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.a0331af2e6feap-43, 0x1.f804ae8d0cp-2},`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.9518ce032f41dp-48, 0x1.0014332bep-1},`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.9518ce032f41dp-48, 0x1.0014332bep-1},`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.b3b3864c60011p-44, 0x1.042bd4b9a8p-1},`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.b3b3864c60011p-44, 0x1.042bd4b9a8p-1},`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.103e8f00d41c8p-45, 0x1.08494c66b9p-1},`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.103e8f00d41c8p-45, 0x1.08494c66b9p-1},`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.65be75cc3da17p-43, 0x1.0c6caaf0c5p-1},`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.65be75cc3da17p-43, 0x1.0c6caaf0c5p-1},`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3676289cd3dd4p-43, 0x1.1096015deep-1},`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3676289cd3dd4p-43, 0x1.1096015deep-1},`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.41dfc7d7c3321p-43, 0x1.14c560fe69p-1},`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.41dfc7d7c3321p-43, 0x1.14c560fe69p-1},`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e0cda8bd74461p-44, 0x1.18fadb6e2dp-1},`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e0cda8bd74461p-44, 0x1.18fadb6e2dp-1},`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2a606046ad444p-44, 0x1.1d368296b5p-1},`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2a606046ad444p-44, 0x1.1d368296b5p-1},`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.f9ea977a639cp-43, 0x1.217868b0c3p-1},`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.f9ea977a639cp-43, 0x1.217868b0c3p-1},`。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.50520a377c7ecp-45, 0x1.25c0a0463cp-1},`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.50520a377c7ecp-45, 0x1.25c0a0463cp-1},`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.6e3cb71b554e7p-47, 0x1.2a0f3c3407p-1},`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.6e3cb71b554e7p-47, 0x1.2a0f3c3407p-1},`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.4275f1035e5e8p-48, 0x1.2e644fac05p-1},`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.4275f1035e5e8p-48, 0x1.2e644fac05p-1},`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.4275f1035e5e8p-48, 0x1.2e644fac05p-1},`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.4275f1035e5e8p-48, 0x1.2e644fac05p-1},`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.979a5db68721dp-45, 0x1.32bfee370fp-1},`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.979a5db68721dp-45, 0x1.32bfee370fp-1},`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1ee969a95f529p-43, 0x1.37222bb707p-1},`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1ee969a95f529p-43, 0x1.37222bb707p-1},`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.bb4b69336b66ep-43, 0x1.3b8b1c68fap-1},`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.bb4b69336b66ep-43, 0x1.3b8b1c68fap-1},`。
- **L119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.d5e6a8a4fb059p-45, 0x1.3ffad4e74fp-1},`.
  **L119 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.d5e6a8a4fb059p-45, 0x1.3ffad4e74fp-1},`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3106e404cabb7p-44, 0x1.44716a2c08p-1},`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3106e404cabb7p-44, 0x1.44716a2c08p-1},`。

### Lines 121-144

````cpp
    {0x1.3106e404cabb7p-44, 0x1.44716a2c08p-1},
    {-0x1.9bcaf1aa4168ap-43, 0x1.48eef19318p-1},
    {0x1.1646b761c48dep-44, 0x1.4d7380dcc4p-1},
    {0x1.2f0c0bfe9dbecp-43, 0x1.51ff2e3021p-1},
    {0x1.29904613e33cp-43, 0x1.5692101d9bp-1},
    {0x1.1d406db502403p-44, 0x1.5b2c3da197p-1},
    {0x1.1d406db502403p-44, 0x1.5b2c3da197p-1},
    {-0x1.125d6cbcd1095p-44, 0x1.5fcdce2728p-1},
    {-0x1.bd9b32266d92cp-43, 0x1.6476d98adap-1},
    {0x1.54243b21709cep-44, 0x1.6927781d93p-1},
    {0x1.54243b21709cep-44, 0x1.6927781d93p-1},
    {-0x1.ce60916e52e91p-44, 0x1.6ddfc2a79p-1},
    {0x1.f1f5ae718f241p-43, 0x1.729fd26b7p-1},
    {-0x1.6eb9612e0b4f3p-43, 0x1.7767c12968p-1},
    {-0x1.6eb9612e0b4f3p-43, 0x1.7767c12968p-1},
    {0x1.fed21f9cb2cc5p-43, 0x1.7c37a9227ep-1},
    {0x1.7f5dc57266758p-43, 0x1.810fa51bf6p-1},
    {0x1.7f5dc57266758p-43, 0x1.810fa51bf6p-1},
    {0x1.5b338360c2ae2p-43, 0x1.85efd062c6p-1},
    {-0x1.96fc8f4b56502p-43, 0x1.8ad846cf37p-1},
    {-0x1.96fc8f4b56502p-43, 0x1.8ad846cf37p-1},
    {-0x1.bdc81c4db3134p-44, 0x1.8fc924c89bp-1},
    {0x1.36c101ee1344p-43, 0x1.94c287492cp-1},
    {0x1.36c101ee1344p-43, 0x1.94c287492cp-1},
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3106e404cabb7p-44, 0x1.44716a2c08p-1},`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3106e404cabb7p-44, 0x1.44716a2c08p-1},`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.9bcaf1aa4168ap-43, 0x1.48eef19318p-1},`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.9bcaf1aa4168ap-43, 0x1.48eef19318p-1},`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1646b761c48dep-44, 0x1.4d7380dcc4p-1},`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1646b761c48dep-44, 0x1.4d7380dcc4p-1},`。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.2f0c0bfe9dbecp-43, 0x1.51ff2e3021p-1},`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.2f0c0bfe9dbecp-43, 0x1.51ff2e3021p-1},`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.29904613e33cp-43, 0x1.5692101d9bp-1},`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.29904613e33cp-43, 0x1.5692101d9bp-1},`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1d406db502403p-44, 0x1.5b2c3da197p-1},`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1d406db502403p-44, 0x1.5b2c3da197p-1},`。
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1d406db502403p-44, 0x1.5b2c3da197p-1},`.
  **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1d406db502403p-44, 0x1.5b2c3da197p-1},`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.125d6cbcd1095p-44, 0x1.5fcdce2728p-1},`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.125d6cbcd1095p-44, 0x1.5fcdce2728p-1},`。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.bd9b32266d92cp-43, 0x1.6476d98adap-1},`.
  **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.bd9b32266d92cp-43, 0x1.6476d98adap-1},`。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.54243b21709cep-44, 0x1.6927781d93p-1},`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.54243b21709cep-44, 0x1.6927781d93p-1},`。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.54243b21709cep-44, 0x1.6927781d93p-1},`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.54243b21709cep-44, 0x1.6927781d93p-1},`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.ce60916e52e91p-44, 0x1.6ddfc2a79p-1},`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.ce60916e52e91p-44, 0x1.6ddfc2a79p-1},`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.f1f5ae718f241p-43, 0x1.729fd26b7p-1},`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.f1f5ae718f241p-43, 0x1.729fd26b7p-1},`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6eb9612e0b4f3p-43, 0x1.7767c12968p-1},`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6eb9612e0b4f3p-43, 0x1.7767c12968p-1},`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6eb9612e0b4f3p-43, 0x1.7767c12968p-1},`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6eb9612e0b4f3p-43, 0x1.7767c12968p-1},`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.fed21f9cb2cc5p-43, 0x1.7c37a9227ep-1},`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.fed21f9cb2cc5p-43, 0x1.7c37a9227ep-1},`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.7f5dc57266758p-43, 0x1.810fa51bf6p-1},`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.7f5dc57266758p-43, 0x1.810fa51bf6p-1},`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.7f5dc57266758p-43, 0x1.810fa51bf6p-1},`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.7f5dc57266758p-43, 0x1.810fa51bf6p-1},`。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.5b338360c2ae2p-43, 0x1.85efd062c6p-1},`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.5b338360c2ae2p-43, 0x1.85efd062c6p-1},`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.96fc8f4b56502p-43, 0x1.8ad846cf37p-1},`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.96fc8f4b56502p-43, 0x1.8ad846cf37p-1},`。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.96fc8f4b56502p-43, 0x1.8ad846cf37p-1},`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.96fc8f4b56502p-43, 0x1.8ad846cf37p-1},`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.bdc81c4db3134p-44, 0x1.8fc924c89bp-1},`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.bdc81c4db3134p-44, 0x1.8fc924c89bp-1},`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.36c101ee1344p-43, 0x1.94c287492cp-1},`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.36c101ee1344p-43, 0x1.94c287492cp-1},`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.36c101ee1344p-43, 0x1.94c287492cp-1},`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.36c101ee1344p-43, 0x1.94c287492cp-1},`。

### Lines 145-168

````cpp
    {0x1.e41fa0a62e6aep-44, 0x1.99c48be206p-1},
    {-0x1.d97ee9124773bp-46, 0x1.9ecf50bf44p-1},
    {-0x1.d97ee9124773bp-46, 0x1.9ecf50bf44p-1},
    {-0x1.3f94e00e7d6bcp-46, 0x1.a3e2f4ac44p-1},
    {-0x1.6879fa00b120ap-43, 0x1.a8ff971811p-1},
    {-0x1.6879fa00b120ap-43, 0x1.a8ff971811p-1},
    {0x1.1659d8e2d7d38p-44, 0x1.ae255819fp-1},
    {0x1.1e5e0ae0d3f8ap-43, 0x1.b35458761dp-1},
    {0x1.1e5e0ae0d3f8ap-43, 0x1.b35458761dp-1},
    {0x1.484a15babcf88p-43, 0x1.b88cb9a2abp-1},
    {0x1.484a15babcf88p-43, 0x1.b88cb9a2abp-1},
    {0x1.871a7610e40bdp-45, 0x1.bdce9dcc96p-1},
    {-0x1.2d90e5edaeceep-43, 0x1.c31a27dd01p-1},
    {-0x1.2d90e5edaeceep-43, 0x1.c31a27dd01p-1},
    {-0x1.5dd31d962d373p-43, 0x1.c86f7b7ea5p-1},
    {-0x1.5dd31d962d373p-43, 0x1.c86f7b7ea5p-1},
    {-0x1.9ad57391924a7p-43, 0x1.cdcebd2374p-1},
    {-0x1.3167ccc538261p-44, 0x1.d338120a6ep-1},
    {-0x1.3167ccc538261p-44, 0x1.d338120a6ep-1},
    {0x1.c7a4ff65ddbc9p-45, 0x1.d8aba045bp-1},
    {0x1.c7a4ff65ddbc9p-45, 0x1.d8aba045bp-1},
    {-0x1.f9ab3cf74babap-44, 0x1.de298ec0bbp-1},
    {-0x1.f9ab3cf74babap-44, 0x1.de298ec0bbp-1},
    {0x1.52842c1c1e586p-43, 0x1.e3b20546f5p-1},
````
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.e41fa0a62e6aep-44, 0x1.99c48be206p-1},`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.e41fa0a62e6aep-44, 0x1.99c48be206p-1},`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.d97ee9124773bp-46, 0x1.9ecf50bf44p-1},`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.d97ee9124773bp-46, 0x1.9ecf50bf44p-1},`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.d97ee9124773bp-46, 0x1.9ecf50bf44p-1},`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.d97ee9124773bp-46, 0x1.9ecf50bf44p-1},`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.3f94e00e7d6bcp-46, 0x1.a3e2f4ac44p-1},`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.3f94e00e7d6bcp-46, 0x1.a3e2f4ac44p-1},`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6879fa00b120ap-43, 0x1.a8ff971811p-1},`.
  **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6879fa00b120ap-43, 0x1.a8ff971811p-1},`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.6879fa00b120ap-43, 0x1.a8ff971811p-1},`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.6879fa00b120ap-43, 0x1.a8ff971811p-1},`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1659d8e2d7d38p-44, 0x1.ae255819fp-1},`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1659d8e2d7d38p-44, 0x1.ae255819fp-1},`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1e5e0ae0d3f8ap-43, 0x1.b35458761dp-1},`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1e5e0ae0d3f8ap-43, 0x1.b35458761dp-1},`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.1e5e0ae0d3f8ap-43, 0x1.b35458761dp-1},`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.1e5e0ae0d3f8ap-43, 0x1.b35458761dp-1},`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.484a15babcf88p-43, 0x1.b88cb9a2abp-1},`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.484a15babcf88p-43, 0x1.b88cb9a2abp-1},`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.484a15babcf88p-43, 0x1.b88cb9a2abp-1},`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.484a15babcf88p-43, 0x1.b88cb9a2abp-1},`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.871a7610e40bdp-45, 0x1.bdce9dcc96p-1},`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.871a7610e40bdp-45, 0x1.bdce9dcc96p-1},`。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2d90e5edaeceep-43, 0x1.c31a27dd01p-1},`.
  **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2d90e5edaeceep-43, 0x1.c31a27dd01p-1},`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.2d90e5edaeceep-43, 0x1.c31a27dd01p-1},`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.2d90e5edaeceep-43, 0x1.c31a27dd01p-1},`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5dd31d962d373p-43, 0x1.c86f7b7ea5p-1},`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5dd31d962d373p-43, 0x1.c86f7b7ea5p-1},`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.5dd31d962d373p-43, 0x1.c86f7b7ea5p-1},`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.5dd31d962d373p-43, 0x1.c86f7b7ea5p-1},`。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.9ad57391924a7p-43, 0x1.cdcebd2374p-1},`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.9ad57391924a7p-43, 0x1.cdcebd2374p-1},`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.3167ccc538261p-44, 0x1.d338120a6ep-1},`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.3167ccc538261p-44, 0x1.d338120a6ep-1},`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.3167ccc538261p-44, 0x1.d338120a6ep-1},`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.3167ccc538261p-44, 0x1.d338120a6ep-1},`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.c7a4ff65ddbc9p-45, 0x1.d8aba045bp-1},`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.c7a4ff65ddbc9p-45, 0x1.d8aba045bp-1},`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.c7a4ff65ddbc9p-45, 0x1.d8aba045bp-1},`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.c7a4ff65ddbc9p-45, 0x1.d8aba045bp-1},`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.f9ab3cf74babap-44, 0x1.de298ec0bbp-1},`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.f9ab3cf74babap-44, 0x1.de298ec0bbp-1},`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.f9ab3cf74babap-44, 0x1.de298ec0bbp-1},`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.f9ab3cf74babap-44, 0x1.de298ec0bbp-1},`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.52842c1c1e586p-43, 0x1.e3b20546f5p-1},`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.52842c1c1e586p-43, 0x1.e3b20546f5p-1},`。

### Lines 169-192

````cpp
    {0x1.52842c1c1e586p-43, 0x1.e3b20546f5p-1},
    {0x1.3c6764fc87b4ap-48, 0x1.e9452c8a71p-1},
    {0x1.3c6764fc87b4ap-48, 0x1.e9452c8a71p-1},
    {-0x1.a0976c0a2827dp-44, 0x1.eee32e2aedp-1},
    {-0x1.a0976c0a2827dp-44, 0x1.eee32e2aedp-1},
    {-0x1.a45314dc4fc42p-43, 0x1.f48c34bd1fp-1},
    {-0x1.a45314dc4fc42p-43, 0x1.f48c34bd1fp-1},
    {0x1.ef5d00e390ap-44, 0x1.fa406bd244p-1},
    {0.0, 1.0},
};

LIBC_INLINE constexpr bool is_odd_integer(double x) {
  using FPBits = fputil::FPBits<double>;
  FPBits xbits(x);
  uint64_t x_u = xbits.uintval();
  unsigned x_e = static_cast<unsigned>(xbits.get_biased_exponent());
  unsigned lsb =
      static_cast<unsigned>(cpp::countr_zero(x_u | FPBits::EXP_MASK));
  constexpr unsigned UNIT_EXPONENT =
      static_cast<unsigned>(FPBits::EXP_BIAS + FPBits::FRACTION_LEN);
  return (x_e + lsb == UNIT_EXPONENT);
}

LIBC_INLINE constexpr bool is_integer(double x) {
````
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.52842c1c1e586p-43, 0x1.e3b20546f5p-1},`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.52842c1c1e586p-43, 0x1.e3b20546f5p-1},`。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3c6764fc87b4ap-48, 0x1.e9452c8a71p-1},`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3c6764fc87b4ap-48, 0x1.e9452c8a71p-1},`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.3c6764fc87b4ap-48, 0x1.e9452c8a71p-1},`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.3c6764fc87b4ap-48, 0x1.e9452c8a71p-1},`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a0976c0a2827dp-44, 0x1.eee32e2aedp-1},`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a0976c0a2827dp-44, 0x1.eee32e2aedp-1},`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a0976c0a2827dp-44, 0x1.eee32e2aedp-1},`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a0976c0a2827dp-44, 0x1.eee32e2aedp-1},`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a45314dc4fc42p-43, 0x1.f48c34bd1fp-1},`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a45314dc4fc42p-43, 0x1.f48c34bd1fp-1},`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{-0x1.a45314dc4fc42p-43, 0x1.f48c34bd1fp-1},`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`{-0x1.a45314dc4fc42p-43, 0x1.f48c34bd1fp-1},`。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0x1.ef5d00e390ap-44, 0x1.fa406bd244p-1},`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0x1.ef5d00e390ap-44, 0x1.fa406bd244p-1},`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0, 1.0},`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0, 1.0},`。
- **L178 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L178 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L179 EN**: Blank line separating nearby declarations or logic.
  **L179 CN**: 空行，用于分隔相邻声明或逻辑。
- **L180 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L180 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L181 EN**: Defines alias `FPBits` to simplify later code.
  **L181 CN**: 定义别名 `FPBits` 以简化后续代码。
- **L182 EN**: Executes a call or declaration centered on `xbits`.
  **L182 CN**: 执行以 `xbits` 为核心的调用或声明。
- **L183 EN**: Initializes variable `x_u` from the right-hand expression.
  **L183 CN**: 使用右侧表达式初始化变量 `x_u`。
- **L184 EN**: Initializes variable `x_e` from the right-hand expression.
  **L184 CN**: 使用右侧表达式初始化变量 `x_e`。
- **L185 EN**: Continues the surrounding expression or declaration: `unsigned lsb =`.
  **L185 CN**: 继续构造周围的表达式或声明：`unsigned lsb =`。
- **L186 EN**: Executes a call or declaration centered on `static_cast<unsigned>`.
  **L186 CN**: 执行以 `static_cast<unsigned>` 为核心的调用或声明。
- **L187 EN**: Continues the surrounding expression or declaration: `constexpr unsigned UNIT_EXPONENT =`.
  **L187 CN**: 继续构造周围的表达式或声明：`constexpr unsigned UNIT_EXPONENT =`。
- **L188 EN**: Executes a call or declaration centered on `static_cast<unsigned>`.
  **L188 CN**: 执行以 `static_cast<unsigned>` 为核心的调用或声明。
- **L189 EN**: Returns from the current function with `(x_e + lsb == UNIT_EXPONENT)`.
  **L189 CN**: 以 `(x_e + lsb == UNIT_EXPONENT)` 从当前函数返回。
- **L190 EN**: Closes the current lexical scope or compound statement.
  **L190 CN**: 结束当前词法作用域或复合语句块。
- **L191 EN**: Blank line separating nearby declarations or logic.
  **L191 CN**: 空行，用于分隔相邻声明或逻辑。
- **L192 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L192 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。

### Lines 193-216

````cpp
  using FPBits = fputil::FPBits<double>;
  FPBits xbits(x);
  uint64_t x_u = xbits.uintval();
  unsigned x_e = static_cast<unsigned>(xbits.get_biased_exponent());
  unsigned lsb =
      static_cast<unsigned>(cpp::countr_zero(x_u | FPBits::EXP_MASK));
  constexpr unsigned UNIT_EXPONENT =
      static_cast<unsigned>(FPBits::EXP_BIAS + FPBits::FRACTION_LEN);
  return (x_e + lsb >= UNIT_EXPONENT);
}

} // namespace pow_internal

LIBC_INLINE double pow(double x, double y) {
  using namespace pow_internal;
  using FPBits = fputil::FPBits<double>;

  FPBits xbits(x), ybits(y);

  bool x_sign = xbits.sign() == Sign::NEG;
  bool y_sign = ybits.sign() == Sign::NEG;

  FPBits x_abs = xbits.abs();
  FPBits y_abs = ybits.abs();
````
- **L193 EN**: Defines alias `FPBits` to simplify later code.
  **L193 CN**: 定义别名 `FPBits` 以简化后续代码。
- **L194 EN**: Executes a call or declaration centered on `xbits`.
  **L194 CN**: 执行以 `xbits` 为核心的调用或声明。
- **L195 EN**: Initializes variable `x_u` from the right-hand expression.
  **L195 CN**: 使用右侧表达式初始化变量 `x_u`。
- **L196 EN**: Initializes variable `x_e` from the right-hand expression.
  **L196 CN**: 使用右侧表达式初始化变量 `x_e`。
- **L197 EN**: Continues the surrounding expression or declaration: `unsigned lsb =`.
  **L197 CN**: 继续构造周围的表达式或声明：`unsigned lsb =`。
- **L198 EN**: Executes a call or declaration centered on `static_cast<unsigned>`.
  **L198 CN**: 执行以 `static_cast<unsigned>` 为核心的调用或声明。
- **L199 EN**: Continues the surrounding expression or declaration: `constexpr unsigned UNIT_EXPONENT =`.
  **L199 CN**: 继续构造周围的表达式或声明：`constexpr unsigned UNIT_EXPONENT =`。
- **L200 EN**: Executes a call or declaration centered on `static_cast<unsigned>`.
  **L200 CN**: 执行以 `static_cast<unsigned>` 为核心的调用或声明。
- **L201 EN**: Returns from the current function with `(x_e + lsb >= UNIT_EXPONENT)`.
  **L201 CN**: 以 `(x_e + lsb >= UNIT_EXPONENT)` 从当前函数返回。
- **L202 EN**: Closes the current lexical scope or compound statement.
  **L202 CN**: 结束当前词法作用域或复合语句块。
- **L203 EN**: Blank line separating nearby declarations or logic.
  **L203 CN**: 空行，用于分隔相邻声明或逻辑。
- **L204 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace pow_internal`.
  **L204 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace pow_internal`。
- **L205 EN**: Blank line separating nearby declarations or logic.
  **L205 CN**: 空行，用于分隔相邻声明或逻辑。
- **L206 EN**: Marks the declaration with LLVM libc inlining or linkage attributes.
  **L206 CN**: 使用 LLVM libc 的内联或链接属性标注该声明。
- **L207 EN**: Brings namespace `pow_internal` into the local scope.
  **L207 CN**: 将命名空间 `pow_internal` 引入当前作用域。
- **L208 EN**: Defines alias `FPBits` to simplify later code.
  **L208 CN**: 定义别名 `FPBits` 以简化后续代码。
- **L209 EN**: Blank line separating nearby declarations or logic.
  **L209 CN**: 空行，用于分隔相邻声明或逻辑。
- **L210 EN**: Executes a call or declaration centered on `xbits`.
  **L210 CN**: 执行以 `xbits` 为核心的调用或声明。
- **L211 EN**: Blank line separating nearby declarations or logic.
  **L211 CN**: 空行，用于分隔相邻声明或逻辑。
- **L212 EN**: Initializes variable `x_sign` from the right-hand expression.
  **L212 CN**: 使用右侧表达式初始化变量 `x_sign`。
- **L213 EN**: Initializes variable `y_sign` from the right-hand expression.
  **L213 CN**: 使用右侧表达式初始化变量 `y_sign`。
- **L214 EN**: Blank line separating nearby declarations or logic.
  **L214 CN**: 空行，用于分隔相邻声明或逻辑。
- **L215 EN**: Initializes variable `x_abs` from the right-hand expression.
  **L215 CN**: 使用右侧表达式初始化变量 `x_abs`。
- **L216 EN**: Initializes variable `y_abs` from the right-hand expression.
  **L216 CN**: 使用右侧表达式初始化变量 `y_abs`。

### Lines 217-240

````cpp

  uint64_t x_mant = xbits.get_mantissa();
  uint64_t y_mant = ybits.get_mantissa();
  uint64_t x_u = xbits.uintval();
  uint64_t x_a = x_abs.uintval();
  uint64_t y_a = y_abs.uintval();

  double e_x = static_cast<double>(xbits.get_exponent());
  uint64_t sign = 0;

  ///////// BEGIN - Check exceptional cases ////////////////////////////////////
  // If x or y is signaling NaN
  if (x_abs.is_signaling_nan() || y_abs.is_signaling_nan()) {
    fputil::raise_except_if_required(FE_INVALID);
    return FPBits::quiet_nan().get_val();
  }

  // The double precision number that is closest to 1 is (1 - 2^-53), which has
  //   log2(1 - 2^-53) ~ -1.715...p-53.
  // So if |y| > |1075 / log2(1 - 2^-53)|, and x is finite:
  //   |y * log2(x)| = 0 or > 1075.
  // Hence x^y will either overflow or underflow if x is not zero.
  if (LIBC_UNLIKELY(y_mant == 0 || y_a > 0x43d7'4910'd52d'3052 ||
                    x_u == FPBits::one().uintval() ||
````
- **L217 EN**: Blank line separating nearby declarations or logic.
  **L217 CN**: 空行，用于分隔相邻声明或逻辑。
- **L218 EN**: Initializes variable `x_mant` from the right-hand expression.
  **L218 CN**: 使用右侧表达式初始化变量 `x_mant`。
- **L219 EN**: Initializes variable `y_mant` from the right-hand expression.
  **L219 CN**: 使用右侧表达式初始化变量 `y_mant`。
- **L220 EN**: Initializes variable `x_u` from the right-hand expression.
  **L220 CN**: 使用右侧表达式初始化变量 `x_u`。
- **L221 EN**: Initializes variable `x_a` from the right-hand expression.
  **L221 CN**: 使用右侧表达式初始化变量 `x_a`。
- **L222 EN**: Initializes variable `y_a` from the right-hand expression.
  **L222 CN**: 使用右侧表达式初始化变量 `y_a`。
- **L223 EN**: Blank line separating nearby declarations or logic.
  **L223 CN**: 空行，用于分隔相邻声明或逻辑。
- **L224 EN**: Initializes variable `e_x` from the right-hand expression.
  **L224 CN**: 使用右侧表达式初始化变量 `e_x`。
- **L225 EN**: Initializes variable `sign` from the right-hand expression.
  **L225 CN**: 使用右侧表达式初始化变量 `sign`。
- **L226 EN**: Blank line separating nearby declarations or logic.
  **L226 CN**: 空行，用于分隔相邻声明或逻辑。
- **L227 EN**: Comment documents nearby intent or constraints: `BEGIN - Check exceptional cases ////////////////////////////////////`.
  **L227 CN**: 注释说明附近代码的意图或约束：`BEGIN - Check exceptional cases ////////////////////////////////////`。
- **L228 EN**: Comment documents nearby intent or constraints: `If x or y is signaling NaN`.
  **L228 CN**: 注释说明附近代码的意图或约束：`If x or y is signaling NaN`。
- **L229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L230 EN**: Executes a call or declaration centered on `fputil::raise_except_if_required`.
  **L230 CN**: 执行以 `fputil::raise_except_if_required` 为核心的调用或声明。
- **L231 EN**: Returns from the current function with `FPBits::quiet_nan().get_val()`.
  **L231 CN**: 以 `FPBits::quiet_nan().get_val()` 从当前函数返回。
- **L232 EN**: Closes the current lexical scope or compound statement.
  **L232 CN**: 结束当前词法作用域或复合语句块。
- **L233 EN**: Blank line separating nearby declarations or logic.
  **L233 CN**: 空行，用于分隔相邻声明或逻辑。
- **L234 EN**: Comment documents nearby intent or constraints: `The double precision number that is closest to 1 is (1 - 2^-53), which has`.
  **L234 CN**: 注释说明附近代码的意图或约束：`The double precision number that is closest to 1 is (1 - 2^-53), which has`。
- **L235 EN**: Comment documents nearby intent or constraints: `log2(1 - 2^-53) ~ -1.715...p-53.`.
  **L235 CN**: 注释说明附近代码的意图或约束：`log2(1 - 2^-53) ~ -1.715...p-53.`。
- **L236 EN**: Comment documents nearby intent or constraints: `So if \|y\| > \|1075 / log2(1 - 2^-53)\|, and x is finite:`.
  **L236 CN**: 注释说明附近代码的意图或约束：`So if \|y\| > \|1075 / log2(1 - 2^-53)\|, and x is finite:`。
- **L237 EN**: Comment documents nearby intent or constraints: `\|y * log2(x)\| = 0 or > 1075.`.
  **L237 CN**: 注释说明附近代码的意图或约束：`\|y * log2(x)\| = 0 or > 1075.`。
- **L238 EN**: Comment documents nearby intent or constraints: `Hence x^y will either overflow or underflow if x is not zero.`.
  **L238 CN**: 注释说明附近代码的意图或约束：`Hence x^y will either overflow or underflow if x is not zero.`。
- **L239 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L239 CN**: 开始 `if` 控制流语句并计算其条件。
- **L240 EN**: Continues logic associated with callable symbol `one`.
  **L240 CN**: 继续与可调用符号 `one` 相关的逻辑。

### Lines 241-264

````cpp
                    x_u >= FPBits::inf().uintval() ||
                    x_u < FPBits::min_normal().uintval())) {
    // Exceptional exponents.
    if (y == 0.0)
      return 1.0;

    switch (y_a) {
    case 0x3fe0'0000'0000'0000: { // y = +-0.5
      // TODO: speed up x^(-1/2) with rsqrt(x) when available.
      if (LIBC_UNLIKELY(
              (x == 0.0 || x_u == FPBits::inf(Sign::NEG).uintval()))) {
        // pow(-0, 1/2) = +0
        // pow(-inf, 1/2) = +inf
        // Make sure it works correctly for FTZ/DAZ.
        return y_sign ? 1.0 / (x * x) : (x * x);
      }
      return y_sign ? (1.0 / fputil::sqrt<double>(x)) : fputil::sqrt<double>(x);
    }
    case 0x3ff0'0000'0000'0000: // y = +-1.0
      return y_sign ? (1.0 / x) : x;
    case 0x4000'0000'0000'0000: // y = +-2.0;
      return y_sign ? (1.0 / (x * x)) : (x * x);
    }

````
- **L241 EN**: Continues logic associated with callable symbol `inf`.
  **L241 CN**: 继续与可调用符号 `inf` 相关的逻辑。
- **L242 EN**: Starts a function, method, lambda, or structured scope: `x_u < FPBits::min_normal().uintval())) {`.
  **L242 CN**: 开始一个函数、方法、lambda 或结构化作用域：`x_u < FPBits::min_normal().uintval())) {`。
- **L243 EN**: Comment documents nearby intent or constraints: `Exceptional exponents.`.
  **L243 CN**: 注释说明附近代码的意图或约束：`Exceptional exponents.`。
- **L244 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L244 CN**: 开始 `if` 控制流语句并计算其条件。
- **L245 EN**: Returns from the current function with `1.0`.
  **L245 CN**: 以 `1.0` 从当前函数返回。
- **L246 EN**: Blank line separating nearby declarations or logic.
  **L246 CN**: 空行，用于分隔相邻声明或逻辑。
- **L247 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L247 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L248 EN**: Introduces a switch dispatch label: `case 0x3fe0'0000'0000'0000: { // y = +-0.5`.
  **L248 CN**: 引入一个 switch 分发标签：`case 0x3fe0'0000'0000'0000: { // y = +-0.5`。
- **L249 EN**: Comment records a pending task or caution: `TODO: speed up x^(-1/2) with rsqrt(x) when available.`.
  **L249 CN**: 注释记录待办事项或注意点：`TODO: speed up x^(-1/2) with rsqrt(x) when available.`。
- **L250 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L250 CN**: 开始 `if` 控制流语句并计算其条件。
- **L251 EN**: Starts a function, method, lambda, or structured scope: `(x == 0.0 \|\| x_u == FPBits::inf(Sign::NEG).uintval()))) {`.
  **L251 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(x == 0.0 \|\| x_u == FPBits::inf(Sign::NEG).uintval()))) {`。
- **L252 EN**: Comment documents nearby intent or constraints: `pow(-0, 1/2) = +0`.
  **L252 CN**: 注释说明附近代码的意图或约束：`pow(-0, 1/2) = +0`。
- **L253 EN**: Comment documents nearby intent or constraints: `pow(-inf, 1/2) = +inf`.
  **L253 CN**: 注释说明附近代码的意图或约束：`pow(-inf, 1/2) = +inf`。
- **L254 EN**: Comment documents nearby intent or constraints: `Make sure it works correctly for FTZ/DAZ.`.
  **L254 CN**: 注释说明附近代码的意图或约束：`Make sure it works correctly for FTZ/DAZ.`。
- **L255 EN**: Returns from the current function with `y_sign ? 1.0 / (x * x) : (x * x)`.
  **L255 CN**: 以 `y_sign ? 1.0 / (x * x) : (x * x)` 从当前函数返回。
- **L256 EN**: Closes the current lexical scope or compound statement.
  **L256 CN**: 结束当前词法作用域或复合语句块。
- **L257 EN**: Returns from the current function with `y_sign ? (1.0 / fputil::sqrt<double>(x)) : fputil::sqrt<double>(x)`.
  **L257 CN**: 以 `y_sign ? (1.0 / fputil::sqrt<double>(x)) : fputil::sqrt<double>(x)` 从当前函数返回。
- **L258 EN**: Closes the current lexical scope or compound statement.
  **L258 CN**: 结束当前词法作用域或复合语句块。
- **L259 EN**: Introduces a switch dispatch label: `case 0x3ff0'0000'0000'0000: // y = +-1.0`.
  **L259 CN**: 引入一个 switch 分发标签：`case 0x3ff0'0000'0000'0000: // y = +-1.0`。
- **L260 EN**: Returns from the current function with `y_sign ? (1.0 / x) : x`.
  **L260 CN**: 以 `y_sign ? (1.0 / x) : x` 从当前函数返回。
- **L261 EN**: Introduces a switch dispatch label: `case 0x4000'0000'0000'0000: // y = +-2.0;`.
  **L261 CN**: 引入一个 switch 分发标签：`case 0x4000'0000'0000'0000: // y = +-2.0;`。
- **L262 EN**: Returns from the current function with `y_sign ? (1.0 / (x * x)) : (x * x)`.
  **L262 CN**: 以 `y_sign ? (1.0 / (x * x)) : (x * x)` 从当前函数返回。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Blank line separating nearby declarations or logic.
  **L264 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 265-288

````cpp
    // |y| > |1075 / log2(1 - 2^-53)|.
    if (y_a > 0x43d7'4910'd52d'3052) {
      if (y_a >= 0x7ff0'0000'0000'0000) {
        // y is inf or nan
        if (y_mant != 0) {
          // y is NaN
          // pow(1, NaN) = 1
          // pow(x, NaN) = NaN
          return (x_u == FPBits::one().uintval()) ? 1.0 : y;
        }

        // Now y is +-Inf
        if (x_abs.is_nan()) {
          // pow(NaN, +-Inf) = NaN
          return x;
        }

        if (x_a == 0x3ff0'0000'0000'0000) {
          // pow(+-1, +-Inf) = 1.0
          return 1.0;
        }

        if (x == 0.0 && y_sign) {
          // pow(+-0, -Inf) = +inf and raise FE_DIVBYZERO
````
- **L265 EN**: Comment documents nearby intent or constraints: `\|y\| > \|1075 / log2(1 - 2^-53)\|.`.
  **L265 CN**: 注释说明附近代码的意图或约束：`\|y\| > \|1075 / log2(1 - 2^-53)\|.`。
- **L266 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L266 CN**: 开始 `if` 控制流语句并计算其条件。
- **L267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L268 EN**: Comment documents nearby intent or constraints: `y is inf or nan`.
  **L268 CN**: 注释说明附近代码的意图或约束：`y is inf or nan`。
- **L269 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L269 CN**: 开始 `if` 控制流语句并计算其条件。
- **L270 EN**: Comment documents nearby intent or constraints: `y is NaN`.
  **L270 CN**: 注释说明附近代码的意图或约束：`y is NaN`。
- **L271 EN**: Comment documents nearby intent or constraints: `pow(1, NaN) = 1`.
  **L271 CN**: 注释说明附近代码的意图或约束：`pow(1, NaN) = 1`。
- **L272 EN**: Comment documents nearby intent or constraints: `pow(x, NaN) = NaN`.
  **L272 CN**: 注释说明附近代码的意图或约束：`pow(x, NaN) = NaN`。
- **L273 EN**: Returns from the current function with `(x_u == FPBits::one().uintval()) ? 1.0 : y`.
  **L273 CN**: 以 `(x_u == FPBits::one().uintval()) ? 1.0 : y` 从当前函数返回。
- **L274 EN**: Closes the current lexical scope or compound statement.
  **L274 CN**: 结束当前词法作用域或复合语句块。
- **L275 EN**: Blank line separating nearby declarations or logic.
  **L275 CN**: 空行，用于分隔相邻声明或逻辑。
- **L276 EN**: Comment documents nearby intent or constraints: `Now y is +-Inf`.
  **L276 CN**: 注释说明附近代码的意图或约束：`Now y is +-Inf`。
- **L277 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L277 CN**: 开始 `if` 控制流语句并计算其条件。
- **L278 EN**: Comment documents nearby intent or constraints: `pow(NaN, +-Inf) = NaN`.
  **L278 CN**: 注释说明附近代码的意图或约束：`pow(NaN, +-Inf) = NaN`。
- **L279 EN**: Returns from the current function with `x`.
  **L279 CN**: 以 `x` 从当前函数返回。
- **L280 EN**: Closes the current lexical scope or compound statement.
  **L280 CN**: 结束当前词法作用域或复合语句块。
- **L281 EN**: Blank line separating nearby declarations or logic.
  **L281 CN**: 空行，用于分隔相邻声明或逻辑。
- **L282 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L282 CN**: 开始 `if` 控制流语句并计算其条件。
- **L283 EN**: Comment documents nearby intent or constraints: `pow(+-1, +-Inf) = 1.0`.
  **L283 CN**: 注释说明附近代码的意图或约束：`pow(+-1, +-Inf) = 1.0`。
- **L284 EN**: Returns from the current function with `1.0`.
  **L284 CN**: 以 `1.0` 从当前函数返回。
- **L285 EN**: Closes the current lexical scope or compound statement.
  **L285 CN**: 结束当前词法作用域或复合语句块。
- **L286 EN**: Blank line separating nearby declarations or logic.
  **L286 CN**: 空行，用于分隔相邻声明或逻辑。
- **L287 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L287 CN**: 开始 `if` 控制流语句并计算其条件。
- **L288 EN**: Comment documents nearby intent or constraints: `pow(+-0, -Inf) = +inf and raise FE_DIVBYZERO`.
  **L288 CN**: 注释说明附近代码的意图或约束：`pow(+-0, -Inf) = +inf and raise FE_DIVBYZERO`。

### Lines 289-312

````cpp
          fputil::set_errno_if_required(EDOM);
          fputil::raise_except_if_required(FE_DIVBYZERO);
          return FPBits::inf().get_val();
        }
        // pow (|x| < 1, -inf) = +inf
        // pow (|x| < 1, +inf) = 0.0
        // pow (|x| > 1, -inf) = 0.0
        // pow (|x| > 1, +inf) = +inf
        return ((x_a < FPBits::one().uintval()) == y_sign)
                   ? FPBits::inf().get_val()
                   : 0.0;
      }
      // x^y will overflow / underflow in double precision.  Set y to a
      // large enough exponent but not too large, so that the computations
      // won't overflow in double precision.
      y = y_sign ? -0x1.0p100 : 0x1.0p100;
    }

    // y is finite and non-zero.

    if (x_u == FPBits::one().uintval()) {
      // pow(1, y) = 1
      return 1.0;
    }
````
- **L289 EN**: Executes a call or declaration centered on `fputil::set_errno_if_required`.
  **L289 CN**: 执行以 `fputil::set_errno_if_required` 为核心的调用或声明。
- **L290 EN**: Executes a call or declaration centered on `fputil::raise_except_if_required`.
  **L290 CN**: 执行以 `fputil::raise_except_if_required` 为核心的调用或声明。
- **L291 EN**: Returns from the current function with `FPBits::inf().get_val()`.
  **L291 CN**: 以 `FPBits::inf().get_val()` 从当前函数返回。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Comment documents nearby intent or constraints: `pow (\|x\| < 1, -inf) = +inf`.
  **L293 CN**: 注释说明附近代码的意图或约束：`pow (\|x\| < 1, -inf) = +inf`。
- **L294 EN**: Comment documents nearby intent or constraints: `pow (\|x\| < 1, +inf) = 0.0`.
  **L294 CN**: 注释说明附近代码的意图或约束：`pow (\|x\| < 1, +inf) = 0.0`。
- **L295 EN**: Comment documents nearby intent or constraints: `pow (\|x\| > 1, -inf) = 0.0`.
  **L295 CN**: 注释说明附近代码的意图或约束：`pow (\|x\| > 1, -inf) = 0.0`。
- **L296 EN**: Comment documents nearby intent or constraints: `pow (\|x\| > 1, +inf) = +inf`.
  **L296 CN**: 注释说明附近代码的意图或约束：`pow (\|x\| > 1, +inf) = +inf`。
- **L297 EN**: Returns from the current function with `((x_a < FPBits::one().uintval()) == y_sign)`.
  **L297 CN**: 以 `((x_a < FPBits::one().uintval()) == y_sign)` 从当前函数返回。
- **L298 EN**: Continues logic associated with callable symbol `inf`.
  **L298 CN**: 继续与可调用符号 `inf` 相关的逻辑。
- **L299 EN**: Executes a standalone statement or declaration: `: 0.0;`.
  **L299 CN**: 执行一条独立语句或声明：`: 0.0;`。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Comment documents nearby intent or constraints: `x^y will overflow / underflow in double precision.  Set y to a`.
  **L301 CN**: 注释说明附近代码的意图或约束：`x^y will overflow / underflow in double precision.  Set y to a`。
- **L302 EN**: Comment documents nearby intent or constraints: `large enough exponent but not too large, so that the computations`.
  **L302 CN**: 注释说明附近代码的意图或约束：`large enough exponent but not too large, so that the computations`。
- **L303 EN**: Comment documents nearby intent or constraints: `won't overflow in double precision.`.
  **L303 CN**: 注释说明附近代码的意图或约束：`won't overflow in double precision.`。
- **L304 EN**: Executes a standalone statement or declaration: `y = y_sign ? -0x1.0p100 : 0x1.0p100;`.
  **L304 CN**: 执行一条独立语句或声明：`y = y_sign ? -0x1.0p100 : 0x1.0p100;`。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Blank line separating nearby declarations or logic.
  **L306 CN**: 空行，用于分隔相邻声明或逻辑。
- **L307 EN**: Comment documents nearby intent or constraints: `y is finite and non-zero.`.
  **L307 CN**: 注释说明附近代码的意图或约束：`y is finite and non-zero.`。
- **L308 EN**: Blank line separating nearby declarations or logic.
  **L308 CN**: 空行，用于分隔相邻声明或逻辑。
- **L309 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L309 CN**: 开始 `if` 控制流语句并计算其条件。
- **L310 EN**: Comment documents nearby intent or constraints: `pow(1, y) = 1`.
  **L310 CN**: 注释说明附近代码的意图或约束：`pow(1, y) = 1`。
- **L311 EN**: Returns from the current function with `1.0`.
  **L311 CN**: 以 `1.0` 从当前函数返回。
- **L312 EN**: Closes the current lexical scope or compound statement.
  **L312 CN**: 结束当前词法作用域或复合语句块。

### Lines 313-336

````cpp

    // TODO: Speed things up with pow(2, y) = exp2(y) and pow(10, y) = exp10(y).

    if (x == 0.0) {
      bool out_is_neg = x_sign && is_odd_integer(y);
      if (y_sign) {
        // pow(0, negative number) = inf
        fputil::set_errno_if_required(EDOM);
        fputil::raise_except_if_required(FE_DIVBYZERO);
        return FPBits::inf(out_is_neg ? Sign::NEG : Sign::POS).get_val();
      }
      // pow(0, positive number) = 0
      return out_is_neg ? -0.0 : 0.0;
    }

    if (x_a == FPBits::inf().uintval()) {
      bool out_is_neg = x_sign && is_odd_integer(y);
      if (y_sign)
        return out_is_neg ? -0.0 : 0.0;
      return FPBits::inf(out_is_neg ? Sign::NEG : Sign::POS).get_val();
    }

    if (x_a > FPBits::inf().uintval()) {
      // x is NaN.
````
- **L313 EN**: Blank line separating nearby declarations or logic.
  **L313 CN**: 空行，用于分隔相邻声明或逻辑。
- **L314 EN**: Comment records a pending task or caution: `TODO: Speed things up with pow(2, y) = exp2(y) and pow(10, y) = exp10(y).`.
  **L314 CN**: 注释记录待办事项或注意点：`TODO: Speed things up with pow(2, y) = exp2(y) and pow(10, y) = exp10(y).`。
- **L315 EN**: Blank line separating nearby declarations or logic.
  **L315 CN**: 空行，用于分隔相邻声明或逻辑。
- **L316 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L316 CN**: 开始 `if` 控制流语句并计算其条件。
- **L317 EN**: Initializes variable `out_is_neg` from the right-hand expression.
  **L317 CN**: 使用右侧表达式初始化变量 `out_is_neg`。
- **L318 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L318 CN**: 开始 `if` 控制流语句并计算其条件。
- **L319 EN**: Comment documents nearby intent or constraints: `pow(0, negative number) = inf`.
  **L319 CN**: 注释说明附近代码的意图或约束：`pow(0, negative number) = inf`。
- **L320 EN**: Executes a call or declaration centered on `fputil::set_errno_if_required`.
  **L320 CN**: 执行以 `fputil::set_errno_if_required` 为核心的调用或声明。
- **L321 EN**: Executes a call or declaration centered on `fputil::raise_except_if_required`.
  **L321 CN**: 执行以 `fputil::raise_except_if_required` 为核心的调用或声明。
- **L322 EN**: Returns from the current function with `FPBits::inf(out_is_neg ? Sign::NEG : Sign::POS).get_val()`.
  **L322 CN**: 以 `FPBits::inf(out_is_neg ? Sign::NEG : Sign::POS).get_val()` 从当前函数返回。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Comment documents nearby intent or constraints: `pow(0, positive number) = 0`.
  **L324 CN**: 注释说明附近代码的意图或约束：`pow(0, positive number) = 0`。
- **L325 EN**: Returns from the current function with `out_is_neg ? -0.0 : 0.0`.
  **L325 CN**: 以 `out_is_neg ? -0.0 : 0.0` 从当前函数返回。
- **L326 EN**: Closes the current lexical scope or compound statement.
  **L326 CN**: 结束当前词法作用域或复合语句块。
- **L327 EN**: Blank line separating nearby declarations or logic.
  **L327 CN**: 空行，用于分隔相邻声明或逻辑。
- **L328 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L328 CN**: 开始 `if` 控制流语句并计算其条件。
- **L329 EN**: Initializes variable `out_is_neg` from the right-hand expression.
  **L329 CN**: 使用右侧表达式初始化变量 `out_is_neg`。
- **L330 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L330 CN**: 开始 `if` 控制流语句并计算其条件。
- **L331 EN**: Returns from the current function with `out_is_neg ? -0.0 : 0.0`.
  **L331 CN**: 以 `out_is_neg ? -0.0 : 0.0` 从当前函数返回。
- **L332 EN**: Returns from the current function with `FPBits::inf(out_is_neg ? Sign::NEG : Sign::POS).get_val()`.
  **L332 CN**: 以 `FPBits::inf(out_is_neg ? Sign::NEG : Sign::POS).get_val()` 从当前函数返回。
- **L333 EN**: Closes the current lexical scope or compound statement.
  **L333 CN**: 结束当前词法作用域或复合语句块。
- **L334 EN**: Blank line separating nearby declarations or logic.
  **L334 CN**: 空行，用于分隔相邻声明或逻辑。
- **L335 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L335 CN**: 开始 `if` 控制流语句并计算其条件。
- **L336 EN**: Comment documents nearby intent or constraints: `x is NaN.`.
  **L336 CN**: 注释说明附近代码的意图或约束：`x is NaN.`。

### Lines 337-360

````cpp
      // pow (aNaN, 0) is already taken care above.
      return x;
    }

    // Normalize denormal inputs.
    if (x_a < FPBits::min_normal().uintval()) {
      FPBits x_norm(x * 0x1.0p64);
      e_x = static_cast<double>(x_norm.get_exponent()) - 64.0;
      x_mant = x_norm.get_mantissa();
    }

    // x is finite and negative, and y is a finite integer.
    if (x_sign) {
      if (is_integer(y)) {
        x = -x;
        if (is_odd_integer(y))
          // sign = -1.0;
          sign = 0x8000'0000'0000'0000;
      } else {
        // pow( negative, non-integer ) = NaN
        fputil::set_errno_if_required(EDOM);
        fputil::raise_except_if_required(FE_INVALID);
        return FPBits::quiet_nan().get_val();
      }
````
- **L337 EN**: Comment documents nearby intent or constraints: `pow (aNaN, 0) is already taken care above.`.
  **L337 CN**: 注释说明附近代码的意图或约束：`pow (aNaN, 0) is already taken care above.`。
- **L338 EN**: Returns from the current function with `x`.
  **L338 CN**: 以 `x` 从当前函数返回。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Blank line separating nearby declarations or logic.
  **L340 CN**: 空行，用于分隔相邻声明或逻辑。
- **L341 EN**: Comment documents nearby intent or constraints: `Normalize denormal inputs.`.
  **L341 CN**: 注释说明附近代码的意图或约束：`Normalize denormal inputs.`。
- **L342 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L342 CN**: 开始 `if` 控制流语句并计算其条件。
- **L343 EN**: Executes a call or declaration centered on `x_norm`.
  **L343 CN**: 执行以 `x_norm` 为核心的调用或声明。
- **L344 EN**: Executes a call or declaration centered on `static_cast<double>`.
  **L344 CN**: 执行以 `static_cast<double>` 为核心的调用或声明。
- **L345 EN**: Executes a call or declaration centered on `x_norm.get_mantissa`.
  **L345 CN**: 执行以 `x_norm.get_mantissa` 为核心的调用或声明。
- **L346 EN**: Closes the current lexical scope or compound statement.
  **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Blank line separating nearby declarations or logic.
  **L347 CN**: 空行，用于分隔相邻声明或逻辑。
- **L348 EN**: Comment documents nearby intent or constraints: `x is finite and negative, and y is a finite integer.`.
  **L348 CN**: 注释说明附近代码的意图或约束：`x is finite and negative, and y is a finite integer.`。
- **L349 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L349 CN**: 开始 `if` 控制流语句并计算其条件。
- **L350 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L350 CN**: 开始 `if` 控制流语句并计算其条件。
- **L351 EN**: Executes a standalone statement or declaration: `x = -x;`.
  **L351 CN**: 执行一条独立语句或声明：`x = -x;`。
- **L352 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L352 CN**: 开始 `if` 控制流语句并计算其条件。
- **L353 EN**: Comment documents nearby intent or constraints: `sign = -1.0;`.
  **L353 CN**: 注释说明附近代码的意图或约束：`sign = -1.0;`。
- **L354 EN**: Executes a standalone statement or declaration: `sign = 0x8000'0000'0000'0000;`.
  **L354 CN**: 执行一条独立语句或声明：`sign = 0x8000'0000'0000'0000;`。
- **L355 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L355 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L356 EN**: Comment documents nearby intent or constraints: `pow( negative, non-integer ) = NaN`.
  **L356 CN**: 注释说明附近代码的意图或约束：`pow( negative, non-integer ) = NaN`。
- **L357 EN**: Executes a call or declaration centered on `fputil::set_errno_if_required`.
  **L357 CN**: 执行以 `fputil::set_errno_if_required` 为核心的调用或声明。
- **L358 EN**: Executes a call or declaration centered on `fputil::raise_except_if_required`.
  **L358 CN**: 执行以 `fputil::raise_except_if_required` 为核心的调用或声明。
- **L359 EN**: Returns from the current function with `FPBits::quiet_nan().get_val()`.
  **L359 CN**: 以 `FPBits::quiet_nan().get_val()` 从当前函数返回。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384

````cpp
    }
  }

  ///////// END - Check exceptional cases //////////////////////////////////////

  // x^y = 2^( y * log2(x) )
  //     = 2^( y * ( e_x + log2(m_x) ) )
  // First we compute log2(x) = e_x + log2(m_x)

  // Extract exponent field of x.

  // Use the highest 7 fractional bits of m_x as the index for look up tables.
  unsigned idx_x = static_cast<unsigned>(x_mant >> (FPBits::FRACTION_LEN - 7));
  // Add the hidden bit to the mantissa.
  // 1 <= m_x < 2
  FPBits m_x = FPBits(x_mant | 0x3ff0'0000'0000'0000);

  // Reduced argument for log2(m_x):
  //   dx = r * m_x - 1.
  // The computation is exact, and -2^-8 <= dx < 2^-7.
  // Then m_x = (1 + dx) / r, and
  //   log2(m_x) = log2( (1 + dx) / r )
  //             = log2(1 + dx) - log2(r).

````
- **L361 EN**: Closes the current lexical scope or compound statement.
  **L361 CN**: 结束当前词法作用域或复合语句块。
- **L362 EN**: Closes the current lexical scope or compound statement.
  **L362 CN**: 结束当前词法作用域或复合语句块。
- **L363 EN**: Blank line separating nearby declarations or logic.
  **L363 CN**: 空行，用于分隔相邻声明或逻辑。
- **L364 EN**: Comment documents nearby intent or constraints: `END - Check exceptional cases //////////////////////////////////////`.
  **L364 CN**: 注释说明附近代码的意图或约束：`END - Check exceptional cases //////////////////////////////////////`。
- **L365 EN**: Blank line separating nearby declarations or logic.
  **L365 CN**: 空行，用于分隔相邻声明或逻辑。
- **L366 EN**: Comment documents nearby intent or constraints: `x^y = 2^( y * log2(x) )`.
  **L366 CN**: 注释说明附近代码的意图或约束：`x^y = 2^( y * log2(x) )`。
- **L367 EN**: Comment documents nearby intent or constraints: `= 2^( y * ( e_x + log2(m_x) ) )`.
  **L367 CN**: 注释说明附近代码的意图或约束：`= 2^( y * ( e_x + log2(m_x) ) )`。
- **L368 EN**: Comment documents nearby intent or constraints: `First we compute log2(x) = e_x + log2(m_x)`.
  **L368 CN**: 注释说明附近代码的意图或约束：`First we compute log2(x) = e_x + log2(m_x)`。
- **L369 EN**: Blank line separating nearby declarations or logic.
  **L369 CN**: 空行，用于分隔相邻声明或逻辑。
- **L370 EN**: Comment documents nearby intent or constraints: `Extract exponent field of x.`.
  **L370 CN**: 注释说明附近代码的意图或约束：`Extract exponent field of x.`。
- **L371 EN**: Blank line separating nearby declarations or logic.
  **L371 CN**: 空行，用于分隔相邻声明或逻辑。
- **L372 EN**: Comment documents nearby intent or constraints: `Use the highest 7 fractional bits of m_x as the index for look up tables.`.
  **L372 CN**: 注释说明附近代码的意图或约束：`Use the highest 7 fractional bits of m_x as the index for look up tables.`。
- **L373 EN**: Initializes variable `idx_x` from the right-hand expression.
  **L373 CN**: 使用右侧表达式初始化变量 `idx_x`。
- **L374 EN**: Comment documents nearby intent or constraints: `Add the hidden bit to the mantissa.`.
  **L374 CN**: 注释说明附近代码的意图或约束：`Add the hidden bit to the mantissa.`。
- **L375 EN**: Comment documents nearby intent or constraints: `1 <= m_x < 2`.
  **L375 CN**: 注释说明附近代码的意图或约束：`1 <= m_x < 2`。
- **L376 EN**: Initializes variable `m_x` from the right-hand expression.
  **L376 CN**: 使用右侧表达式初始化变量 `m_x`。
- **L377 EN**: Blank line separating nearby declarations or logic.
  **L377 CN**: 空行，用于分隔相邻声明或逻辑。
- **L378 EN**: Comment documents nearby intent or constraints: `Reduced argument for log2(m_x):`.
  **L378 CN**: 注释说明附近代码的意图或约束：`Reduced argument for log2(m_x):`。
- **L379 EN**: Comment documents nearby intent or constraints: `dx = r * m_x - 1.`.
  **L379 CN**: 注释说明附近代码的意图或约束：`dx = r * m_x - 1.`。
- **L380 EN**: Comment documents nearby intent or constraints: `The computation is exact, and -2^-8 <= dx < 2^-7.`.
  **L380 CN**: 注释说明附近代码的意图或约束：`The computation is exact, and -2^-8 <= dx < 2^-7.`。
- **L381 EN**: Comment documents nearby intent or constraints: `Then m_x = (1 + dx) / r, and`.
  **L381 CN**: 注释说明附近代码的意图或约束：`Then m_x = (1 + dx) / r, and`。
- **L382 EN**: Comment documents nearby intent or constraints: `log2(m_x) = log2( (1 + dx) / r )`.
  **L382 CN**: 注释说明附近代码的意图或约束：`log2(m_x) = log2( (1 + dx) / r )`。
- **L383 EN**: Comment documents nearby intent or constraints: `= log2(1 + dx) - log2(r).`.
  **L383 CN**: 注释说明附近代码的意图或约束：`= log2(1 + dx) - log2(r).`。
- **L384 EN**: Blank line separating nearby declarations or logic.
  **L384 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 385-408

````cpp
  // In order for the overall computations x^y = 2^(y * log2(x)) to have the
  // relative errors < 2^-52 (1ULP), we will need to evaluate the exponent part
  // y * log2(x) with absolute errors < 2^-52 (or better, 2^-53).  Since the
  // whole exponent range for double precision is bounded by
  // |y * log2(x)| < 1076 ~ 2^10, we need to evaluate log2(x) with absolute
  // errors < 2^-53 * 2^-10 = 2^-63.

  // With that requirement, we use the following degree-6 polynomial
  // approximation:
  //   P(dx) ~ log2(1 + dx) / dx
  // Generated by Sollya with:
  // > P = fpminimax(log2(1 + x)/x, 6, [|D...|], [-2^-8, 2^-7]); P;
  // > dirtyinfnorm(log2(1 + x) - x*P, [-2^-8, 2^-7]);
  //   0x1.d03cc...p-66
  constexpr double COEFFS[] = {0x1.71547652b82fep0,  -0x1.71547652b82e7p-1,
                               0x1.ec709dc3b1fd5p-2, -0x1.7154766124215p-2,
                               0x1.2776bd90259d8p-2, -0x1.ec586c6f3d311p-3,
                               0x1.9c4775eccf524p-3};
  // Error: ulp(dx^2) <= (2^-7)^2 * 2^-52 = 2^-66
  // Extra errors from various computations and rounding directions, the overall
  // errors we can be bounded by 2^-65.

  DoubleDouble dx_c0;

````
- **L385 EN**: Comment documents nearby intent or constraints: `In order for the overall computations x^y = 2^(y * log2(x)) to have the`.
  **L385 CN**: 注释说明附近代码的意图或约束：`In order for the overall computations x^y = 2^(y * log2(x)) to have the`。
- **L386 EN**: Comment documents nearby intent or constraints: `relative errors < 2^-52 (1ULP), we will need to evaluate the exponent part`.
  **L386 CN**: 注释说明附近代码的意图或约束：`relative errors < 2^-52 (1ULP), we will need to evaluate the exponent part`。
- **L387 EN**: Comment documents nearby intent or constraints: `y * log2(x) with absolute errors < 2^-52 (or better, 2^-53).  Since the`.
  **L387 CN**: 注释说明附近代码的意图或约束：`y * log2(x) with absolute errors < 2^-52 (or better, 2^-53).  Since the`。
- **L388 EN**: Comment documents nearby intent or constraints: `whole exponent range for double precision is bounded by`.
  **L388 CN**: 注释说明附近代码的意图或约束：`whole exponent range for double precision is bounded by`。
- **L389 EN**: Comment documents nearby intent or constraints: `\|y * log2(x)\| < 1076 ~ 2^10, we need to evaluate log2(x) with absolute`.
  **L389 CN**: 注释说明附近代码的意图或约束：`\|y * log2(x)\| < 1076 ~ 2^10, we need to evaluate log2(x) with absolute`。
- **L390 EN**: Comment documents nearby intent or constraints: `errors < 2^-53 * 2^-10 = 2^-63.`.
  **L390 CN**: 注释说明附近代码的意图或约束：`errors < 2^-53 * 2^-10 = 2^-63.`。
- **L391 EN**: Blank line separating nearby declarations or logic.
  **L391 CN**: 空行，用于分隔相邻声明或逻辑。
- **L392 EN**: Comment documents nearby intent or constraints: `With that requirement, we use the following degree-6 polynomial`.
  **L392 CN**: 注释说明附近代码的意图或约束：`With that requirement, we use the following degree-6 polynomial`。
- **L393 EN**: Comment documents nearby intent or constraints: `approximation:`.
  **L393 CN**: 注释说明附近代码的意图或约束：`approximation:`。
- **L394 EN**: Comment documents nearby intent or constraints: `P(dx) ~ log2(1 + dx) / dx`.
  **L394 CN**: 注释说明附近代码的意图或约束：`P(dx) ~ log2(1 + dx) / dx`。
- **L395 EN**: Comment documents nearby intent or constraints: `Generated by Sollya with:`.
  **L395 CN**: 注释说明附近代码的意图或约束：`Generated by Sollya with:`。
- **L396 EN**: Comment documents nearby intent or constraints: `> P = fpminimax(log2(1 + x)/x, 6, [\|D...\|], [-2^-8, 2^-7]); P;`.
  **L396 CN**: 注释说明附近代码的意图或约束：`> P = fpminimax(log2(1 + x)/x, 6, [\|D...\|], [-2^-8, 2^-7]); P;`。
- **L397 EN**: Comment documents nearby intent or constraints: `> dirtyinfnorm(log2(1 + x) - x*P, [-2^-8, 2^-7]);`.
  **L397 CN**: 注释说明附近代码的意图或约束：`> dirtyinfnorm(log2(1 + x) - x*P, [-2^-8, 2^-7]);`。
- **L398 EN**: Comment documents nearby intent or constraints: `0x1.d03cc...p-66`.
  **L398 CN**: 注释说明附近代码的意图或约束：`0x1.d03cc...p-66`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `constexpr double COEFFS[] = {0x1.71547652b82fep0,  -0x1.71547652b82e7p-1,`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`constexpr double COEFFS[] = {0x1.71547652b82fep0,  -0x1.71547652b82e7p-1,`。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.ec709dc3b1fd5p-2, -0x1.7154766124215p-2,`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.ec709dc3b1fd5p-2, -0x1.7154766124215p-2,`。
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.2776bd90259d8p-2, -0x1.ec586c6f3d311p-3,`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.2776bd90259d8p-2, -0x1.ec586c6f3d311p-3,`。
- **L402 EN**: Executes a standalone statement or declaration: `0x1.9c4775eccf524p-3};`.
  **L402 CN**: 执行一条独立语句或声明：`0x1.9c4775eccf524p-3};`。
- **L403 EN**: Comment documents nearby intent or constraints: `Error: ulp(dx^2) <= (2^-7)^2 * 2^-52 = 2^-66`.
  **L403 CN**: 注释说明附近代码的意图或约束：`Error: ulp(dx^2) <= (2^-7)^2 * 2^-52 = 2^-66`。
- **L404 EN**: Comment documents nearby intent or constraints: `Extra errors from various computations and rounding directions, the overall`.
  **L404 CN**: 注释说明附近代码的意图或约束：`Extra errors from various computations and rounding directions, the overall`。
- **L405 EN**: Comment documents nearby intent or constraints: `errors we can be bounded by 2^-65.`.
  **L405 CN**: 注释说明附近代码的意图或约束：`errors we can be bounded by 2^-65.`。
- **L406 EN**: Blank line separating nearby declarations or logic.
  **L406 CN**: 空行，用于分隔相邻声明或逻辑。
- **L407 EN**: Executes a standalone statement or declaration: `DoubleDouble dx_c0;`.
  **L407 CN**: 执行一条独立语句或声明：`DoubleDouble dx_c0;`。
- **L408 EN**: Blank line separating nearby declarations or logic.
  **L408 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 409-432

````cpp
  // Perform exact range reduction and exact product dx * c0.
#ifdef LIBC_TARGET_CPU_HAS_FMA_DOUBLE
  double dx = fputil::multiply_add(RD[idx_x], m_x.get_val(), -1.0); // Exact
  dx_c0 = fputil::exact_mult(COEFFS[0], dx);
#else
  double c = FPBits(m_x.uintval() & 0x3fff'e000'0000'0000).get_val();
  double dx =
      fputil::multiply_add(RD[idx_x], m_x.get_val() - c, CD[idx_x]); // Exact
  dx_c0 = fputil::exact_mult<double, 28>(dx, COEFFS[0]);             // Exact
#endif // LIBC_TARGET_CPU_HAS_FMA_DOUBLE

  double dx2 = dx * dx;
  double c0 = fputil::multiply_add(dx, COEFFS[2], COEFFS[1]);
  double c1 = fputil::multiply_add(dx, COEFFS[4], COEFFS[3]);
  double c2 = fputil::multiply_add(dx, COEFFS[6], COEFFS[5]);

  double p = fputil::polyeval(dx2, c0, c1, c2);

  // s = e_x - log2(r) + dx * P(dx)
  // Absolute error bound:
  //   |log2(x) - log2_x.hi - log2_x.lo| < 2^-65.

  // Notice that e_x - log2(r).hi is exact, so we perform an exact sum of
  // e_x - log2(r).hi and the high part of the product dx * c0:
````
- **L409 EN**: Comment documents nearby intent or constraints: `Perform exact range reduction and exact product dx * c0.`.
  **L409 CN**: 注释说明附近代码的意图或约束：`Perform exact range reduction and exact product dx * c0.`。
- **L410 EN**: Starts a preprocessor conditional block: `#ifdef LIBC_TARGET_CPU_HAS_FMA_DOUBLE`.
  **L410 CN**: 开始一个预处理条件块：`#ifdef LIBC_TARGET_CPU_HAS_FMA_DOUBLE`。
- **L411 EN**: Continues logic associated with callable symbol `multiply_add`.
  **L411 CN**: 继续与可调用符号 `multiply_add` 相关的逻辑。
- **L412 EN**: Executes a call or declaration centered on `fputil::exact_mult`.
  **L412 CN**: 执行以 `fputil::exact_mult` 为核心的调用或声明。
- **L413 EN**: Continues the current preprocessor branch selection.
  **L413 CN**: 继续当前的预处理分支选择。
- **L414 EN**: Initializes variable `c` from the right-hand expression.
  **L414 CN**: 使用右侧表达式初始化变量 `c`。
- **L415 EN**: Continues the surrounding expression or declaration: `double dx =`.
  **L415 CN**: 继续构造周围的表达式或声明：`double dx =`。
- **L416 EN**: Continues logic associated with callable symbol `multiply_add`.
  **L416 CN**: 继续与可调用符号 `multiply_add` 相关的逻辑。
- **L417 EN**: Continues the surrounding expression or declaration: `dx_c0 = fputil::exact_mult<double, 28>(dx, COEFFS[0]);             // Exact`.
  **L417 CN**: 继续构造周围的表达式或声明：`dx_c0 = fputil::exact_mult<double, 28>(dx, COEFFS[0]);             // Exact`。
- **L418 EN**: Closes the current preprocessor conditional block or header guard.
  **L418 CN**: 结束当前预处理条件块或头文件保护。
- **L419 EN**: Blank line separating nearby declarations or logic.
  **L419 CN**: 空行，用于分隔相邻声明或逻辑。
- **L420 EN**: Initializes variable `dx2` from the right-hand expression.
  **L420 CN**: 使用右侧表达式初始化变量 `dx2`。
- **L421 EN**: Initializes variable `c0` from the right-hand expression.
  **L421 CN**: 使用右侧表达式初始化变量 `c0`。
- **L422 EN**: Initializes variable `c1` from the right-hand expression.
  **L422 CN**: 使用右侧表达式初始化变量 `c1`。
- **L423 EN**: Initializes variable `c2` from the right-hand expression.
  **L423 CN**: 使用右侧表达式初始化变量 `c2`。
- **L424 EN**: Blank line separating nearby declarations or logic.
  **L424 CN**: 空行，用于分隔相邻声明或逻辑。
- **L425 EN**: Initializes variable `p` from the right-hand expression.
  **L425 CN**: 使用右侧表达式初始化变量 `p`。
- **L426 EN**: Blank line separating nearby declarations or logic.
  **L426 CN**: 空行，用于分隔相邻声明或逻辑。
- **L427 EN**: Comment documents nearby intent or constraints: `s = e_x - log2(r) + dx * P(dx)`.
  **L427 CN**: 注释说明附近代码的意图或约束：`s = e_x - log2(r) + dx * P(dx)`。
- **L428 EN**: Comment documents nearby intent or constraints: `Absolute error bound:`.
  **L428 CN**: 注释说明附近代码的意图或约束：`Absolute error bound:`。
- **L429 EN**: Comment documents nearby intent or constraints: `\|log2(x) - log2_x.hi - log2_x.lo\| < 2^-65.`.
  **L429 CN**: 注释说明附近代码的意图或约束：`\|log2(x) - log2_x.hi - log2_x.lo\| < 2^-65.`。
- **L430 EN**: Blank line separating nearby declarations or logic.
  **L430 CN**: 空行，用于分隔相邻声明或逻辑。
- **L431 EN**: Comment documents nearby intent or constraints: `Notice that e_x - log2(r).hi is exact, so we perform an exact sum of`.
  **L431 CN**: 注释说明附近代码的意图或约束：`Notice that e_x - log2(r).hi is exact, so we perform an exact sum of`。
- **L432 EN**: Comment documents nearby intent or constraints: `e_x - log2(r).hi and the high part of the product dx * c0:`.
  **L432 CN**: 注释说明附近代码的意图或约束：`e_x - log2(r).hi and the high part of the product dx * c0:`。

### Lines 433-456

````cpp
  //   log2_x_hi.hi + log2_x_hi.lo = e_x - log2(r).hi + (dx * c0).hi
  DoubleDouble log2_x_hi =
      fputil::exact_add(e_x + LOG2_R_DD[idx_x].hi, dx_c0.hi);
  // The low part is dx^2 * p + low part of (dx * c0) + low part of -log2(r).
  double log2_x_lo =
      fputil::multiply_add(dx2, p, dx_c0.lo + LOG2_R_DD[idx_x].lo);
  // Perform accurate sums.
  DoubleDouble log2_x = fputil::exact_add(log2_x_hi.hi, log2_x_lo);
  log2_x.lo += log2_x_hi.lo;

  // To compute 2^(y * log2(x)), we break the exponent into 3 parts:
  //   y * log(2) = hi + mid + lo, where
  //   hi is an integer
  //   mid * 2^6 is an integer
  //   |lo| <= 2^-7
  // Then:
  //   x^y = 2^(y * log2(x)) = 2^hi * 2^mid * 2^lo,
  // In which 2^mid is obtained from a look-up table of size 2^6 = 64 elements,
  // and 2^lo ~ 1 + lo * P(lo).
  // Thus, we have:
  //   hi + mid = 2^-6 * round( 2^6 * y * log2(x) )
  // If we restrict the output such that |hi| < 150, (hi + mid) uses (8 + 6)
  // bits, hence, if we use double precision to perform
  //   round( 2^6 * y * log2(x))
````
- **L433 EN**: Comment documents nearby intent or constraints: `log2_x_hi.hi + log2_x_hi.lo = e_x - log2(r).hi + (dx * c0).hi`.
  **L433 CN**: 注释说明附近代码的意图或约束：`log2_x_hi.hi + log2_x_hi.lo = e_x - log2(r).hi + (dx * c0).hi`。
- **L434 EN**: Continues the surrounding expression or declaration: `DoubleDouble log2_x_hi =`.
  **L434 CN**: 继续构造周围的表达式或声明：`DoubleDouble log2_x_hi =`。
- **L435 EN**: Executes a call or declaration centered on `fputil::exact_add`.
  **L435 CN**: 执行以 `fputil::exact_add` 为核心的调用或声明。
- **L436 EN**: Comment documents nearby intent or constraints: `The low part is dx^2 * p + low part of (dx * c0) + low part of -log2(r).`.
  **L436 CN**: 注释说明附近代码的意图或约束：`The low part is dx^2 * p + low part of (dx * c0) + low part of -log2(r).`。
- **L437 EN**: Continues the surrounding expression or declaration: `double log2_x_lo =`.
  **L437 CN**: 继续构造周围的表达式或声明：`double log2_x_lo =`。
- **L438 EN**: Executes a call or declaration centered on `fputil::multiply_add`.
  **L438 CN**: 执行以 `fputil::multiply_add` 为核心的调用或声明。
- **L439 EN**: Comment documents nearby intent or constraints: `Perform accurate sums.`.
  **L439 CN**: 注释说明附近代码的意图或约束：`Perform accurate sums.`。
- **L440 EN**: Initializes variable `log2_x` from the right-hand expression.
  **L440 CN**: 使用右侧表达式初始化变量 `log2_x`。
- **L441 EN**: Executes a standalone statement or declaration: `log2_x.lo += log2_x_hi.lo;`.
  **L441 CN**: 执行一条独立语句或声明：`log2_x.lo += log2_x_hi.lo;`。
- **L442 EN**: Blank line separating nearby declarations or logic.
  **L442 CN**: 空行，用于分隔相邻声明或逻辑。
- **L443 EN**: Comment documents nearby intent or constraints: `To compute 2^(y * log2(x)), we break the exponent into 3 parts:`.
  **L443 CN**: 注释说明附近代码的意图或约束：`To compute 2^(y * log2(x)), we break the exponent into 3 parts:`。
- **L444 EN**: Comment documents nearby intent or constraints: `y * log(2) = hi + mid + lo, where`.
  **L444 CN**: 注释说明附近代码的意图或约束：`y * log(2) = hi + mid + lo, where`。
- **L445 EN**: Comment documents nearby intent or constraints: `hi is an integer`.
  **L445 CN**: 注释说明附近代码的意图或约束：`hi is an integer`。
- **L446 EN**: Comment documents nearby intent or constraints: `mid * 2^6 is an integer`.
  **L446 CN**: 注释说明附近代码的意图或约束：`mid * 2^6 is an integer`。
- **L447 EN**: Comment documents nearby intent or constraints: `\|lo\| <= 2^-7`.
  **L447 CN**: 注释说明附近代码的意图或约束：`\|lo\| <= 2^-7`。
- **L448 EN**: Comment documents nearby intent or constraints: `Then:`.
  **L448 CN**: 注释说明附近代码的意图或约束：`Then:`。
- **L449 EN**: Comment documents nearby intent or constraints: `x^y = 2^(y * log2(x)) = 2^hi * 2^mid * 2^lo,`.
  **L449 CN**: 注释说明附近代码的意图或约束：`x^y = 2^(y * log2(x)) = 2^hi * 2^mid * 2^lo,`。
- **L450 EN**: Comment documents nearby intent or constraints: `In which 2^mid is obtained from a look-up table of size 2^6 = 64 elements,`.
  **L450 CN**: 注释说明附近代码的意图或约束：`In which 2^mid is obtained from a look-up table of size 2^6 = 64 elements,`。
- **L451 EN**: Comment documents nearby intent or constraints: `and 2^lo ~ 1 + lo * P(lo).`.
  **L451 CN**: 注释说明附近代码的意图或约束：`and 2^lo ~ 1 + lo * P(lo).`。
- **L452 EN**: Comment documents nearby intent or constraints: `Thus, we have:`.
  **L452 CN**: 注释说明附近代码的意图或约束：`Thus, we have:`。
- **L453 EN**: Comment documents nearby intent or constraints: `hi + mid = 2^-6 * round( 2^6 * y * log2(x) )`.
  **L453 CN**: 注释说明附近代码的意图或约束：`hi + mid = 2^-6 * round( 2^6 * y * log2(x) )`。
- **L454 EN**: Comment documents nearby intent or constraints: `If we restrict the output such that \|hi\| < 150, (hi + mid) uses (8 + 6)`.
  **L454 CN**: 注释说明附近代码的意图或约束：`If we restrict the output such that \|hi\| < 150, (hi + mid) uses (8 + 6)`。
- **L455 EN**: Comment documents nearby intent or constraints: `bits, hence, if we use double precision to perform`.
  **L455 CN**: 注释说明附近代码的意图或约束：`bits, hence, if we use double precision to perform`。
- **L456 EN**: Comment documents nearby intent or constraints: `round( 2^6 * y * log2(x))`.
  **L456 CN**: 注释说明附近代码的意图或约束：`round( 2^6 * y * log2(x))`。

### Lines 457-480

````cpp
  // the lo part is bounded by 2^-7 + 2^(-(52 - 14)) = 2^-7 + 2^-38

  // In the following computations:
  //   y6  = 2^6 * y
  //   hm  = 2^6 * (hi + mid) = round(2^6 * y * log2(x)) ~ round(y6 * s)
  //   lo6 = 2^6 * lo = 2^6 * (y - (hi + mid)) = y6 * log2(x) - hm.
  double y6 = y * 0x1.0p6; // Exact.

  DoubleDouble y6_log2_x = fputil::exact_mult(y6, log2_x.hi);
  y6_log2_x.lo = fputil::multiply_add(y6, log2_x.lo, y6_log2_x.lo);

  // Check overflow/underflow.
  double scale = 1.0;

  // |2^(hi + mid) - exp2_hi_mid| <= ulp(exp2_hi_mid) / 2
  // Clamp the exponent part into smaller range that fits double precision.
  // For those exponents that are out of range, the final conversion will round
  // them correctly to inf/max float or 0/min float accordingly.
  constexpr double UPPER_EXP_BOUND = 512.0 * 0x1.0p6;
  if (LIBC_UNLIKELY(FPBits(y6_log2_x.hi).abs().get_val() >= UPPER_EXP_BOUND)) {
    if (FPBits(y6_log2_x.hi).sign() == Sign::POS) {
      scale = 0x1.0p512;
      y6_log2_x.hi -= 512.0 * 64.0;
      if (y6_log2_x.hi > 513.0 * 64.0)
````
- **L457 EN**: Comment documents nearby intent or constraints: `the lo part is bounded by 2^-7 + 2^(-(52 - 14)) = 2^-7 + 2^-38`.
  **L457 CN**: 注释说明附近代码的意图或约束：`the lo part is bounded by 2^-7 + 2^(-(52 - 14)) = 2^-7 + 2^-38`。
- **L458 EN**: Blank line separating nearby declarations or logic.
  **L458 CN**: 空行，用于分隔相邻声明或逻辑。
- **L459 EN**: Comment documents nearby intent or constraints: `In the following computations:`.
  **L459 CN**: 注释说明附近代码的意图或约束：`In the following computations:`。
- **L460 EN**: Comment documents nearby intent or constraints: `y6  = 2^6 * y`.
  **L460 CN**: 注释说明附近代码的意图或约束：`y6  = 2^6 * y`。
- **L461 EN**: Comment documents nearby intent or constraints: `hm  = 2^6 * (hi + mid) = round(2^6 * y * log2(x)) ~ round(y6 * s)`.
  **L461 CN**: 注释说明附近代码的意图或约束：`hm  = 2^6 * (hi + mid) = round(2^6 * y * log2(x)) ~ round(y6 * s)`。
- **L462 EN**: Comment documents nearby intent or constraints: `lo6 = 2^6 * lo = 2^6 * (y - (hi + mid)) = y6 * log2(x) - hm.`.
  **L462 CN**: 注释说明附近代码的意图或约束：`lo6 = 2^6 * lo = 2^6 * (y - (hi + mid)) = y6 * log2(x) - hm.`。
- **L463 EN**: Continues the surrounding expression or declaration: `double y6 = y * 0x1.0p6; // Exact.`.
  **L463 CN**: 继续构造周围的表达式或声明：`double y6 = y * 0x1.0p6; // Exact.`。
- **L464 EN**: Blank line separating nearby declarations or logic.
  **L464 CN**: 空行，用于分隔相邻声明或逻辑。
- **L465 EN**: Initializes variable `y6_log2_x` from the right-hand expression.
  **L465 CN**: 使用右侧表达式初始化变量 `y6_log2_x`。
- **L466 EN**: Executes a call or declaration centered on `fputil::multiply_add`.
  **L466 CN**: 执行以 `fputil::multiply_add` 为核心的调用或声明。
- **L467 EN**: Blank line separating nearby declarations or logic.
  **L467 CN**: 空行，用于分隔相邻声明或逻辑。
- **L468 EN**: Comment documents nearby intent or constraints: `Check overflow/underflow.`.
  **L468 CN**: 注释说明附近代码的意图或约束：`Check overflow/underflow.`。
- **L469 EN**: Initializes variable `scale` from the right-hand expression.
  **L469 CN**: 使用右侧表达式初始化变量 `scale`。
- **L470 EN**: Blank line separating nearby declarations or logic.
  **L470 CN**: 空行，用于分隔相邻声明或逻辑。
- **L471 EN**: Comment documents nearby intent or constraints: `\|2^(hi + mid) - exp2_hi_mid\| <= ulp(exp2_hi_mid) / 2`.
  **L471 CN**: 注释说明附近代码的意图或约束：`\|2^(hi + mid) - exp2_hi_mid\| <= ulp(exp2_hi_mid) / 2`。
- **L472 EN**: Comment documents nearby intent or constraints: `Clamp the exponent part into smaller range that fits double precision.`.
  **L472 CN**: 注释说明附近代码的意图或约束：`Clamp the exponent part into smaller range that fits double precision.`。
- **L473 EN**: Comment documents nearby intent or constraints: `For those exponents that are out of range, the final conversion will round`.
  **L473 CN**: 注释说明附近代码的意图或约束：`For those exponents that are out of range, the final conversion will round`。
- **L474 EN**: Comment documents nearby intent or constraints: `them correctly to inf/max float or 0/min float accordingly.`.
  **L474 CN**: 注释说明附近代码的意图或约束：`them correctly to inf/max float or 0/min float accordingly.`。
- **L475 EN**: Initializes variable `UPPER_EXP_BOUND` from the right-hand expression.
  **L475 CN**: 使用右侧表达式初始化变量 `UPPER_EXP_BOUND`。
- **L476 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L476 CN**: 开始 `if` 控制流语句并计算其条件。
- **L477 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L477 CN**: 开始 `if` 控制流语句并计算其条件。
- **L478 EN**: Executes a standalone statement or declaration: `scale = 0x1.0p512;`.
  **L478 CN**: 执行一条独立语句或声明：`scale = 0x1.0p512;`。
- **L479 EN**: Executes a standalone statement or declaration: `y6_log2_x.hi -= 512.0 * 64.0;`.
  **L479 CN**: 执行一条独立语句或声明：`y6_log2_x.hi -= 512.0 * 64.0;`。
- **L480 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L480 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 481-504

````cpp
        y6_log2_x.hi = 513.0 * 64.0;
    } else {
      scale = 0x1.0p-512;
      y6_log2_x.hi += 512.0 * 64.0;
      if (y6_log2_x.hi < (-1076.0 + 512.0) * 64.0)
        y6_log2_x.hi = -564.0 * 64.0;
    }
  }

  double hm = fputil::nearest_integer(y6_log2_x.hi);

  // lo6 = 2^6 * lo.
  double lo6_hi = y6_log2_x.hi - hm;
  double lo6 = lo6_hi + y6_log2_x.lo;

  int hm_i = static_cast<int>(hm);
  unsigned idx_y = static_cast<unsigned>(hm_i) & 0x3f;

  // 2^hi
  int64_t exp2_hi_i = static_cast<int64_t>(
      static_cast<uint64_t>(static_cast<int64_t>(hm_i >> 6))
      << FPBits::FRACTION_LEN);
  // 2^mid
  int64_t exp2_mid_hi_i =
````
- **L481 EN**: Executes a standalone statement or declaration: `y6_log2_x.hi = 513.0 * 64.0;`.
  **L481 CN**: 执行一条独立语句或声明：`y6_log2_x.hi = 513.0 * 64.0;`。
- **L482 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L482 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L483 EN**: Executes a standalone statement or declaration: `scale = 0x1.0p-512;`.
  **L483 CN**: 执行一条独立语句或声明：`scale = 0x1.0p-512;`。
- **L484 EN**: Executes a standalone statement or declaration: `y6_log2_x.hi += 512.0 * 64.0;`.
  **L484 CN**: 执行一条独立语句或声明：`y6_log2_x.hi += 512.0 * 64.0;`。
- **L485 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L485 CN**: 开始 `if` 控制流语句并计算其条件。
- **L486 EN**: Executes a standalone statement or declaration: `y6_log2_x.hi = -564.0 * 64.0;`.
  **L486 CN**: 执行一条独立语句或声明：`y6_log2_x.hi = -564.0 * 64.0;`。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Closes the current lexical scope or compound statement.
  **L488 CN**: 结束当前词法作用域或复合语句块。
- **L489 EN**: Blank line separating nearby declarations or logic.
  **L489 CN**: 空行，用于分隔相邻声明或逻辑。
- **L490 EN**: Initializes variable `hm` from the right-hand expression.
  **L490 CN**: 使用右侧表达式初始化变量 `hm`。
- **L491 EN**: Blank line separating nearby declarations or logic.
  **L491 CN**: 空行，用于分隔相邻声明或逻辑。
- **L492 EN**: Comment documents nearby intent or constraints: `lo6 = 2^6 * lo.`.
  **L492 CN**: 注释说明附近代码的意图或约束：`lo6 = 2^6 * lo.`。
- **L493 EN**: Initializes variable `lo6_hi` from the right-hand expression.
  **L493 CN**: 使用右侧表达式初始化变量 `lo6_hi`。
- **L494 EN**: Initializes variable `lo6` from the right-hand expression.
  **L494 CN**: 使用右侧表达式初始化变量 `lo6`。
- **L495 EN**: Blank line separating nearby declarations or logic.
  **L495 CN**: 空行，用于分隔相邻声明或逻辑。
- **L496 EN**: Initializes variable `hm_i` from the right-hand expression.
  **L496 CN**: 使用右侧表达式初始化变量 `hm_i`。
- **L497 EN**: Initializes variable `idx_y` from the right-hand expression.
  **L497 CN**: 使用右侧表达式初始化变量 `idx_y`。
- **L498 EN**: Blank line separating nearby declarations or logic.
  **L498 CN**: 空行，用于分隔相邻声明或逻辑。
- **L499 EN**: Comment documents nearby intent or constraints: `2^hi`.
  **L499 CN**: 注释说明附近代码的意图或约束：`2^hi`。
- **L500 EN**: Continues logic associated with callable symbol `static_cast<int64_t>`.
  **L500 CN**: 继续与可调用符号 `static_cast<int64_t>` 相关的逻辑。
- **L501 EN**: Continues logic associated with callable symbol `static_cast<uint64_t>`.
  **L501 CN**: 继续与可调用符号 `static_cast<uint64_t>` 相关的逻辑。
- **L502 EN**: Executes a standalone statement or declaration: `<< FPBits::FRACTION_LEN);`.
  **L502 CN**: 执行一条独立语句或声明：`<< FPBits::FRACTION_LEN);`。
- **L503 EN**: Comment documents nearby intent or constraints: `2^mid`.
  **L503 CN**: 注释说明附近代码的意图或约束：`2^mid`。
- **L504 EN**: Continues the surrounding expression or declaration: `int64_t exp2_mid_hi_i =`.
  **L504 CN**: 继续构造周围的表达式或声明：`int64_t exp2_mid_hi_i =`。

### Lines 505-528

````cpp
      static_cast<int64_t>(FPBits(EXP2_MID1[idx_y].hi).uintval());
  int64_t exp2_mid_lo_i =
      static_cast<int64_t>(FPBits(EXP2_MID1[idx_y].mid).uintval());
  // (-1)^sign * 2^hi * 2^mid
  // Error <= 2^hi * 2^-53
  uint64_t exp2_hm_hi_i =
      static_cast<uint64_t>(exp2_hi_i + exp2_mid_hi_i) + sign;
  // The low part could be 0.
  uint64_t exp2_hm_lo_i =
      idx_y != 0 ? static_cast<uint64_t>(exp2_hi_i + exp2_mid_lo_i) + sign
                 : sign;
  double exp2_hm_hi = FPBits(exp2_hm_hi_i).get_val();
  double exp2_hm_lo = FPBits(exp2_hm_lo_i).get_val();

  // Degree-5 polynomial approximation P(lo6) ~ 2^(lo6 / 2^6) = 2^(lo).
  // Generated by Sollya with:
  // > P = fpminimax(2^(x/64), 5, [|1, D...|], [-2^-1, 2^-1]);
  // > dirtyinfnorm(2^(x/64) - P, [-0.5, 0.5]);
  // 0x1.a2b77e618f5c4c176fd11b7659016cde5de83cb72p-60
  constexpr double EXP2_COEFFS[] = {0x1p0,
                                    0x1.62e42fefa39efp-7,
                                    0x1.ebfbdff82a23ap-15,
                                    0x1.c6b08d7076268p-23,
                                    0x1.3b2ad33f8b48bp-31,
````
- **L505 EN**: Executes a call or declaration centered on `static_cast<int64_t>`.
  **L505 CN**: 执行以 `static_cast<int64_t>` 为核心的调用或声明。
- **L506 EN**: Continues the surrounding expression or declaration: `int64_t exp2_mid_lo_i =`.
  **L506 CN**: 继续构造周围的表达式或声明：`int64_t exp2_mid_lo_i =`。
- **L507 EN**: Executes a call or declaration centered on `static_cast<int64_t>`.
  **L507 CN**: 执行以 `static_cast<int64_t>` 为核心的调用或声明。
- **L508 EN**: Comment documents nearby intent or constraints: `(-1)^sign * 2^hi * 2^mid`.
  **L508 CN**: 注释说明附近代码的意图或约束：`(-1)^sign * 2^hi * 2^mid`。
- **L509 EN**: Comment documents nearby intent or constraints: `Error <= 2^hi * 2^-53`.
  **L509 CN**: 注释说明附近代码的意图或约束：`Error <= 2^hi * 2^-53`。
- **L510 EN**: Continues the surrounding expression or declaration: `uint64_t exp2_hm_hi_i =`.
  **L510 CN**: 继续构造周围的表达式或声明：`uint64_t exp2_hm_hi_i =`。
- **L511 EN**: Executes a call or declaration centered on `static_cast<uint64_t>`.
  **L511 CN**: 执行以 `static_cast<uint64_t>` 为核心的调用或声明。
- **L512 EN**: Comment documents nearby intent or constraints: `The low part could be 0.`.
  **L512 CN**: 注释说明附近代码的意图或约束：`The low part could be 0.`。
- **L513 EN**: Continues the surrounding expression or declaration: `uint64_t exp2_hm_lo_i =`.
  **L513 CN**: 继续构造周围的表达式或声明：`uint64_t exp2_hm_lo_i =`。
- **L514 EN**: Continues logic associated with callable symbol `static_cast<uint64_t>`.
  **L514 CN**: 继续与可调用符号 `static_cast<uint64_t>` 相关的逻辑。
- **L515 EN**: Executes a standalone statement or declaration: `: sign;`.
  **L515 CN**: 执行一条独立语句或声明：`: sign;`。
- **L516 EN**: Initializes variable `exp2_hm_hi` from the right-hand expression.
  **L516 CN**: 使用右侧表达式初始化变量 `exp2_hm_hi`。
- **L517 EN**: Initializes variable `exp2_hm_lo` from the right-hand expression.
  **L517 CN**: 使用右侧表达式初始化变量 `exp2_hm_lo`。
- **L518 EN**: Blank line separating nearby declarations or logic.
  **L518 CN**: 空行，用于分隔相邻声明或逻辑。
- **L519 EN**: Comment documents nearby intent or constraints: `Degree-5 polynomial approximation P(lo6) ~ 2^(lo6 / 2^6) = 2^(lo).`.
  **L519 CN**: 注释说明附近代码的意图或约束：`Degree-5 polynomial approximation P(lo6) ~ 2^(lo6 / 2^6) = 2^(lo).`。
- **L520 EN**: Comment documents nearby intent or constraints: `Generated by Sollya with:`.
  **L520 CN**: 注释说明附近代码的意图或约束：`Generated by Sollya with:`。
- **L521 EN**: Comment documents nearby intent or constraints: `> P = fpminimax(2^(x/64), 5, [\|1, D...\|], [-2^-1, 2^-1]);`.
  **L521 CN**: 注释说明附近代码的意图或约束：`> P = fpminimax(2^(x/64), 5, [\|1, D...\|], [-2^-1, 2^-1]);`。
- **L522 EN**: Comment documents nearby intent or constraints: `> dirtyinfnorm(2^(x/64) - P, [-0.5, 0.5]);`.
  **L522 CN**: 注释说明附近代码的意图或约束：`> dirtyinfnorm(2^(x/64) - P, [-0.5, 0.5]);`。
- **L523 EN**: Comment documents nearby intent or constraints: `0x1.a2b77e618f5c4c176fd11b7659016cde5de83cb72p-60`.
  **L523 CN**: 注释说明附近代码的意图或约束：`0x1.a2b77e618f5c4c176fd11b7659016cde5de83cb72p-60`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `constexpr double EXP2_COEFFS[] = {0x1p0,`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`constexpr double EXP2_COEFFS[] = {0x1p0,`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.62e42fefa39efp-7,`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.62e42fefa39efp-7,`。
- **L526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.ebfbdff82a23ap-15,`.
  **L526 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.ebfbdff82a23ap-15,`。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.c6b08d7076268p-23,`.
  **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.c6b08d7076268p-23,`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0x1.3b2ad33f8b48bp-31,`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`0x1.3b2ad33f8b48bp-31,`。

### Lines 529-546

````cpp
                                    0x1.5d870c4d84445p-40};

  double lo6_sqr = lo6 * lo6;

  double d0 = fputil::multiply_add(lo6, EXP2_COEFFS[2], EXP2_COEFFS[1]);
  double d1 = fputil::multiply_add(lo6, EXP2_COEFFS[4], EXP2_COEFFS[3]);
  double pp = fputil::polyeval(lo6_sqr, d0, d1, EXP2_COEFFS[5]);

  double r = fputil::multiply_add(exp2_hm_hi * lo6, pp, exp2_hm_lo);
  r += exp2_hm_hi;

  return r * scale;
}

} // namespace math
} // namespace LIBC_NAMESPACE_DECL

#endif // LLVM_LIBC_SRC___SUPPORT_MATH_POW_H
````
- **L529 EN**: Executes a standalone statement or declaration: `0x1.5d870c4d84445p-40};`.
  **L529 CN**: 执行一条独立语句或声明：`0x1.5d870c4d84445p-40};`。
- **L530 EN**: Blank line separating nearby declarations or logic.
  **L530 CN**: 空行，用于分隔相邻声明或逻辑。
- **L531 EN**: Initializes variable `lo6_sqr` from the right-hand expression.
  **L531 CN**: 使用右侧表达式初始化变量 `lo6_sqr`。
- **L532 EN**: Blank line separating nearby declarations or logic.
  **L532 CN**: 空行，用于分隔相邻声明或逻辑。
- **L533 EN**: Initializes variable `d0` from the right-hand expression.
  **L533 CN**: 使用右侧表达式初始化变量 `d0`。
- **L534 EN**: Initializes variable `d1` from the right-hand expression.
  **L534 CN**: 使用右侧表达式初始化变量 `d1`。
- **L535 EN**: Initializes variable `pp` from the right-hand expression.
  **L535 CN**: 使用右侧表达式初始化变量 `pp`。
- **L536 EN**: Blank line separating nearby declarations or logic.
  **L536 CN**: 空行，用于分隔相邻声明或逻辑。
- **L537 EN**: Initializes variable `r` from the right-hand expression.
  **L537 CN**: 使用右侧表达式初始化变量 `r`。
- **L538 EN**: Executes a standalone statement or declaration: `r += exp2_hm_hi;`.
  **L538 CN**: 执行一条独立语句或声明：`r += exp2_hm_hi;`。
- **L539 EN**: Blank line separating nearby declarations or logic.
  **L539 CN**: 空行，用于分隔相邻声明或逻辑。
- **L540 EN**: Returns from the current function with `r * scale`.
  **L540 CN**: 以 `r * scale` 从当前函数返回。
- **L541 EN**: Closes the current lexical scope or compound statement.
  **L541 CN**: 结束当前词法作用域或复合语句块。
- **L542 EN**: Blank line separating nearby declarations or logic.
  **L542 CN**: 空行，用于分隔相邻声明或逻辑。
- **L543 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace math`.
  **L543 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace math`。
- **L544 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace LIBC_NAMESPACE_DECL`.
  **L544 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace LIBC_NAMESPACE_DECL`。
- **L545 EN**: Blank line separating nearby declarations or logic.
  **L545 CN**: 空行，用于分隔相邻声明或逻辑。
- **L546 EN**: Closes the current preprocessor conditional block or header guard.
  **L546 CN**: 结束当前预处理条件块或头文件保护。

## Key Concepts / 关键概念

- **Floating-point support kernels / 浮点支撑内核**: Provides reusable math internals such as argument reduction, approximation helpers, and type-specific wrappers. / 提供可复用的数学内部组件，例如自变量归约、近似辅助逻辑以及按类型区分的包装层。
- **Exponentiation support / 幂函数支撑**: Combines logarithm/exponential style approximations or tables to compute powers accurately. / 组合对数/指数式近似或查表逻辑，以较高精度计算幂函数。
- **Floating-point bit manipulation / 浮点位级操作**: Uses helper types to inspect exponents, mantissas, special values, and sign bits directly. / 使用辅助类型直接检查指数、尾数、特殊值和符号位。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `common_constants.h`, `exp_constants.h`, `hdr/errno_macros.h`, `hdr/fenv_macros.h`, `src/__support/CPP/bit.h`, `src/__support/FPUtil/FEnvImpl.h`, `src/__support/FPUtil/FPBits.h`, `src/__support/FPUtil/PolyEval.h`, `src/__support/FPUtil/double_double.h`, `src/__support/FPUtil/multiply_add.h`, `src/__support/FPUtil/nearest_integer.h`, `src/__support/FPUtil/sqrt.h` ... (+3 more)
- **Dependency categories / 依赖类别**: LLVM libc floating-point utility helpers / LLVM libc 浮点工具辅助组件 (7), nearby local declarations / 附近的本地声明 (2), ABI-facing generated header declarations / 面向 ABI 的生成头声明 (2), LLVM libc configuration and attribute macros / LLVM libc 配置与属性宏 (2), LLVM libc C++ support utilities / LLVM libc C++ 支撑工具 (1), LLVM libc internal support utilities / LLVM libc 内部支撑工具 (1)

- `common_constants.h`: Provides nearby local declarations. / 提供附近的本地声明。
- `exp_constants.h`: Provides nearby local declarations. / 提供附近的本地声明。
- `hdr/errno_macros.h`: Provides ABI-facing generated header declarations. / 提供面向 ABI 的生成头声明。
- `hdr/fenv_macros.h`: Provides ABI-facing generated header declarations. / 提供面向 ABI 的生成头声明。
- `src/__support/CPP/bit.h`: Provides LLVM libc C++ support utilities. / 提供LLVM libc C++ 支撑工具。
- `src/__support/FPUtil/FEnvImpl.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/FPBits.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/PolyEval.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/double_double.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/multiply_add.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/nearest_integer.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/FPUtil/sqrt.h`: Provides LLVM libc floating-point utility helpers. / 提供LLVM libc 浮点工具辅助组件。
- `src/__support/common.h`: Provides LLVM libc internal support utilities. / 提供LLVM libc 内部支撑工具。
- `src/__support/macros/config.h`: Provides LLVM libc configuration and attribute macros. / 提供LLVM libc 配置与属性宏。
- `src/__support/macros/optimization.h`: Provides LLVM libc configuration and attribute macros. / 提供LLVM libc 配置与属性宏。
