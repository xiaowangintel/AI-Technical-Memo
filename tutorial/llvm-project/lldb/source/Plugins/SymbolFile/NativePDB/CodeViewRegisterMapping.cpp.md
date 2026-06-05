# CodeViewRegisterMapping.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/SymbolFile/NativePDB/CodeViewRegisterMapping.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: Implements LLDB logic for symbol-file parsing, debug-info loading, compile-unit discovery, and type extraction related to `CodeViewRegisterMapping` in the `SymbolFile` subsystem.
- **Purpose (CN)**: 该文件在 LLDB 的 `SymbolFile` 子系统中实现与 `CodeViewRegisterMapping` 相关的逻辑，重点覆盖符号文件解析、调试信息加载、编译单元发现与类型提取。对应英文说明：Implements LLDB logic for symbol-file parsing, debug-info loading, compile-unit discovery, and type extraction related to `CodeViewRegisterMapping` in the `SymbolFile` subsystem。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行

````cpp
//===-- CodeViewRegisterMapping.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "CodeViewRegisterMapping.h"

#include "lldb/lldb-defines.h"

#include "Plugins/Process/Utility/lldb-arm64-register-enums.h"
#include "Plugins/Process/Utility/lldb-x86-register-enums.h"

using namespace lldb_private;

static const uint32_t g_code_view_to_lldb_registers_arm64[] = {
    LLDB_INVALID_REGNUM, // NONE
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    gpr_w0_arm64,  // ARM64_W0, 10)
    gpr_w1_arm64,  // ARM64_W1, 11)
````
- **L1 EN**: Banner comment marks a file or section boundary.
  **L1 CN**: 横幅注释用于标记文件或章节边界。
- **L2 EN**: Separator comment visually groups nearby code.
  **L2 CN**: 分隔注释用于在视觉上分组附近代码。
- **L3 EN**: Comment explains surrounding design intent or invariants: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明周边设计意图或不变式：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains surrounding design intent or invariants: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明周边设计意图或不变式：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains surrounding design intent or invariants: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明周边设计意图或不变式：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment visually groups nearby code.
  **L6 CN**: 分隔注释用于在视觉上分组附近代码。
- **L7 EN**: Banner comment marks a file or section boundary.
  **L7 CN**: 横幅注释用于标记文件或章节边界。
- **L8 EN**: Blank line separates nearby declarations or logic blocks.
  **L8 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes `CodeViewRegisterMapping.h` so this header can use supporting declarations from another header.
  **L9 CN**: 引入 `CodeViewRegisterMapping.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L10 EN**: Blank line separates nearby declarations or logic blocks.
  **L10 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L11 EN**: Includes `lldb/lldb-defines.h` so this header can use umbrella LLDB declarations, enums, and type aliases.
  **L11 CN**: 引入 `lldb/lldb-defines.h`，使该头文件能够使用LLDB 总括声明、枚举与类型别名。
- **L12 EN**: Blank line separates nearby declarations or logic blocks.
  **L12 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes `Plugins/Process/Utility/lldb-arm64-register-enums.h` so this header can use supporting declarations from another header.
  **L13 CN**: 引入 `Plugins/Process/Utility/lldb-arm64-register-enums.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L14 EN**: Includes `Plugins/Process/Utility/lldb-x86-register-enums.h` so this header can use supporting declarations from another header.
  **L14 CN**: 引入 `Plugins/Process/Utility/lldb-x86-register-enums.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L15 EN**: Blank line separates nearby declarations or logic blocks.
  **L15 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L16 EN**: Imports namespace `lldb_private` into the current scope.
  **L16 CN**: 将命名空间 `lldb_private` 导入当前作用域。
- **L17 EN**: Blank line separates nearby declarations or logic blocks.
  **L17 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L18 EN**: Continues the surrounding declaration or expression: `static const uint32_t g_code_view_to_lldb_registers_arm64[] = {`.
  **L18 CN**: 继续构造周围的声明或表达式：`static const uint32_t g_code_view_to_lldb_registers_arm64[] = {`。
- **L19 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // NONE`.
  **L19 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // NONE`。
- **L20 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L20 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L21 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L21 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L22 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L22 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L23 EN**: Continues the surrounding declaration or expression: `gpr_w0_arm64,  // ARM64_W0, 10)`.
  **L23 CN**: 继续构造周围的声明或表达式：`gpr_w0_arm64,  // ARM64_W0, 10)`。
- **L24 EN**: Continues the surrounding declaration or expression: `gpr_w1_arm64,  // ARM64_W1, 11)`.
  **L24 CN**: 继续构造周围的声明或表达式：`gpr_w1_arm64,  // ARM64_W1, 11)`。

### Lines 25-48 / 第 25-48 行

````cpp
    gpr_w2_arm64,  // ARM64_W2, 12)
    gpr_w3_arm64,  // ARM64_W3, 13)
    gpr_w4_arm64,  // ARM64_W4, 14)
    gpr_w5_arm64,  // ARM64_W5, 15)
    gpr_w6_arm64,  // ARM64_W6, 16)
    gpr_w7_arm64,  // ARM64_W7, 17)
    gpr_w8_arm64,  // ARM64_W8, 18)
    gpr_w9_arm64,  // ARM64_W9, 19)
    gpr_w10_arm64, // ARM64_W10, 20)
    gpr_w11_arm64, // ARM64_W11, 21)
    gpr_w12_arm64, // ARM64_W12, 22)
    gpr_w13_arm64, // ARM64_W13, 23)
    gpr_w14_arm64, // ARM64_W14, 24)
    gpr_w15_arm64, // ARM64_W15, 25)
    gpr_w16_arm64, // ARM64_W16, 26)
    gpr_w17_arm64, // ARM64_W17, 27)
    gpr_w18_arm64, // ARM64_W18, 28)
    gpr_w19_arm64, // ARM64_W19, 29)
    gpr_w20_arm64, // ARM64_W20, 30)
    gpr_w21_arm64, // ARM64_W21, 31)
    gpr_w22_arm64, // ARM64_W22, 32)
    gpr_w23_arm64, // ARM64_W23, 33)
    gpr_w24_arm64, // ARM64_W24, 34)
    gpr_w25_arm64, // ARM64_W25, 35)
````
- **L25 EN**: Continues the surrounding declaration or expression: `gpr_w2_arm64,  // ARM64_W2, 12)`.
  **L25 CN**: 继续构造周围的声明或表达式：`gpr_w2_arm64,  // ARM64_W2, 12)`。
- **L26 EN**: Continues the surrounding declaration or expression: `gpr_w3_arm64,  // ARM64_W3, 13)`.
  **L26 CN**: 继续构造周围的声明或表达式：`gpr_w3_arm64,  // ARM64_W3, 13)`。
- **L27 EN**: Continues the surrounding declaration or expression: `gpr_w4_arm64,  // ARM64_W4, 14)`.
  **L27 CN**: 继续构造周围的声明或表达式：`gpr_w4_arm64,  // ARM64_W4, 14)`。
- **L28 EN**: Continues the surrounding declaration or expression: `gpr_w5_arm64,  // ARM64_W5, 15)`.
  **L28 CN**: 继续构造周围的声明或表达式：`gpr_w5_arm64,  // ARM64_W5, 15)`。
- **L29 EN**: Continues the surrounding declaration or expression: `gpr_w6_arm64,  // ARM64_W6, 16)`.
  **L29 CN**: 继续构造周围的声明或表达式：`gpr_w6_arm64,  // ARM64_W6, 16)`。
- **L30 EN**: Continues the surrounding declaration or expression: `gpr_w7_arm64,  // ARM64_W7, 17)`.
  **L30 CN**: 继续构造周围的声明或表达式：`gpr_w7_arm64,  // ARM64_W7, 17)`。
- **L31 EN**: Continues the surrounding declaration or expression: `gpr_w8_arm64,  // ARM64_W8, 18)`.
  **L31 CN**: 继续构造周围的声明或表达式：`gpr_w8_arm64,  // ARM64_W8, 18)`。
- **L32 EN**: Continues the surrounding declaration or expression: `gpr_w9_arm64,  // ARM64_W9, 19)`.
  **L32 CN**: 继续构造周围的声明或表达式：`gpr_w9_arm64,  // ARM64_W9, 19)`。
- **L33 EN**: Continues the surrounding declaration or expression: `gpr_w10_arm64, // ARM64_W10, 20)`.
  **L33 CN**: 继续构造周围的声明或表达式：`gpr_w10_arm64, // ARM64_W10, 20)`。
- **L34 EN**: Continues the surrounding declaration or expression: `gpr_w11_arm64, // ARM64_W11, 21)`.
  **L34 CN**: 继续构造周围的声明或表达式：`gpr_w11_arm64, // ARM64_W11, 21)`。
- **L35 EN**: Continues the surrounding declaration or expression: `gpr_w12_arm64, // ARM64_W12, 22)`.
  **L35 CN**: 继续构造周围的声明或表达式：`gpr_w12_arm64, // ARM64_W12, 22)`。
- **L36 EN**: Continues the surrounding declaration or expression: `gpr_w13_arm64, // ARM64_W13, 23)`.
  **L36 CN**: 继续构造周围的声明或表达式：`gpr_w13_arm64, // ARM64_W13, 23)`。
- **L37 EN**: Continues the surrounding declaration or expression: `gpr_w14_arm64, // ARM64_W14, 24)`.
  **L37 CN**: 继续构造周围的声明或表达式：`gpr_w14_arm64, // ARM64_W14, 24)`。
- **L38 EN**: Continues the surrounding declaration or expression: `gpr_w15_arm64, // ARM64_W15, 25)`.
  **L38 CN**: 继续构造周围的声明或表达式：`gpr_w15_arm64, // ARM64_W15, 25)`。
- **L39 EN**: Continues the surrounding declaration or expression: `gpr_w16_arm64, // ARM64_W16, 26)`.
  **L39 CN**: 继续构造周围的声明或表达式：`gpr_w16_arm64, // ARM64_W16, 26)`。
- **L40 EN**: Continues the surrounding declaration or expression: `gpr_w17_arm64, // ARM64_W17, 27)`.
  **L40 CN**: 继续构造周围的声明或表达式：`gpr_w17_arm64, // ARM64_W17, 27)`。
- **L41 EN**: Continues the surrounding declaration or expression: `gpr_w18_arm64, // ARM64_W18, 28)`.
  **L41 CN**: 继续构造周围的声明或表达式：`gpr_w18_arm64, // ARM64_W18, 28)`。
- **L42 EN**: Continues the surrounding declaration or expression: `gpr_w19_arm64, // ARM64_W19, 29)`.
  **L42 CN**: 继续构造周围的声明或表达式：`gpr_w19_arm64, // ARM64_W19, 29)`。
- **L43 EN**: Continues the surrounding declaration or expression: `gpr_w20_arm64, // ARM64_W20, 30)`.
  **L43 CN**: 继续构造周围的声明或表达式：`gpr_w20_arm64, // ARM64_W20, 30)`。
- **L44 EN**: Continues the surrounding declaration or expression: `gpr_w21_arm64, // ARM64_W21, 31)`.
  **L44 CN**: 继续构造周围的声明或表达式：`gpr_w21_arm64, // ARM64_W21, 31)`。
- **L45 EN**: Continues the surrounding declaration or expression: `gpr_w22_arm64, // ARM64_W22, 32)`.
  **L45 CN**: 继续构造周围的声明或表达式：`gpr_w22_arm64, // ARM64_W22, 32)`。
- **L46 EN**: Continues the surrounding declaration or expression: `gpr_w23_arm64, // ARM64_W23, 33)`.
  **L46 CN**: 继续构造周围的声明或表达式：`gpr_w23_arm64, // ARM64_W23, 33)`。
- **L47 EN**: Continues the surrounding declaration or expression: `gpr_w24_arm64, // ARM64_W24, 34)`.
  **L47 CN**: 继续构造周围的声明或表达式：`gpr_w24_arm64, // ARM64_W24, 34)`。
- **L48 EN**: Continues the surrounding declaration or expression: `gpr_w25_arm64, // ARM64_W25, 35)`.
  **L48 CN**: 继续构造周围的声明或表达式：`gpr_w25_arm64, // ARM64_W25, 35)`。

### Lines 49-72 / 第 49-72 行

````cpp
    gpr_w26_arm64, // ARM64_W26, 36)
    gpr_w27_arm64, // ARM64_W27, 37)
    gpr_w28_arm64, // ARM64_W28, 38)
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    gpr_x0_arm64,  // ARM64_X0, 50)
    gpr_x1_arm64,  // ARM64_X1, 51)
    gpr_x2_arm64,  // ARM64_X2, 52)
    gpr_x3_arm64,  // ARM64_X3, 53)
    gpr_x4_arm64,  // ARM64_X4, 54)
    gpr_x5_arm64,  // ARM64_X5, 55)
    gpr_x6_arm64,  // ARM64_X6, 56)
    gpr_x7_arm64,  // ARM64_X7, 57)
    gpr_x8_arm64,  // ARM64_X8, 58)
    gpr_x9_arm64,  // ARM64_X9, 59)
    gpr_x10_arm64, // ARM64_X10, 60)
    gpr_x11_arm64, // ARM64_X11, 61)
    gpr_x12_arm64, // ARM64_X12, 62)
    gpr_x13_arm64, // ARM64_X13, 63)
    gpr_x14_arm64, // ARM64_X14, 64)
    gpr_x15_arm64, // ARM64_X15, 65)
    gpr_x16_arm64, // ARM64_X16, 66)
````
- **L49 EN**: Continues the surrounding declaration or expression: `gpr_w26_arm64, // ARM64_W26, 36)`.
  **L49 CN**: 继续构造周围的声明或表达式：`gpr_w26_arm64, // ARM64_W26, 36)`。
- **L50 EN**: Continues the surrounding declaration or expression: `gpr_w27_arm64, // ARM64_W27, 37)`.
  **L50 CN**: 继续构造周围的声明或表达式：`gpr_w27_arm64, // ARM64_W27, 37)`。
- **L51 EN**: Continues the surrounding declaration or expression: `gpr_w28_arm64, // ARM64_W28, 38)`.
  **L51 CN**: 继续构造周围的声明或表达式：`gpr_w28_arm64, // ARM64_W28, 38)`。
- **L52 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L52 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L53 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L53 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L54 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L54 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L55 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L55 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L56 EN**: Continues the surrounding declaration or expression: `gpr_x0_arm64,  // ARM64_X0, 50)`.
  **L56 CN**: 继续构造周围的声明或表达式：`gpr_x0_arm64,  // ARM64_X0, 50)`。
- **L57 EN**: Continues the surrounding declaration or expression: `gpr_x1_arm64,  // ARM64_X1, 51)`.
  **L57 CN**: 继续构造周围的声明或表达式：`gpr_x1_arm64,  // ARM64_X1, 51)`。
- **L58 EN**: Continues the surrounding declaration or expression: `gpr_x2_arm64,  // ARM64_X2, 52)`.
  **L58 CN**: 继续构造周围的声明或表达式：`gpr_x2_arm64,  // ARM64_X2, 52)`。
- **L59 EN**: Continues the surrounding declaration or expression: `gpr_x3_arm64,  // ARM64_X3, 53)`.
  **L59 CN**: 继续构造周围的声明或表达式：`gpr_x3_arm64,  // ARM64_X3, 53)`。
- **L60 EN**: Continues the surrounding declaration or expression: `gpr_x4_arm64,  // ARM64_X4, 54)`.
  **L60 CN**: 继续构造周围的声明或表达式：`gpr_x4_arm64,  // ARM64_X4, 54)`。
- **L61 EN**: Continues the surrounding declaration or expression: `gpr_x5_arm64,  // ARM64_X5, 55)`.
  **L61 CN**: 继续构造周围的声明或表达式：`gpr_x5_arm64,  // ARM64_X5, 55)`。
- **L62 EN**: Continues the surrounding declaration or expression: `gpr_x6_arm64,  // ARM64_X6, 56)`.
  **L62 CN**: 继续构造周围的声明或表达式：`gpr_x6_arm64,  // ARM64_X6, 56)`。
- **L63 EN**: Continues the surrounding declaration or expression: `gpr_x7_arm64,  // ARM64_X7, 57)`.
  **L63 CN**: 继续构造周围的声明或表达式：`gpr_x7_arm64,  // ARM64_X7, 57)`。
- **L64 EN**: Continues the surrounding declaration or expression: `gpr_x8_arm64,  // ARM64_X8, 58)`.
  **L64 CN**: 继续构造周围的声明或表达式：`gpr_x8_arm64,  // ARM64_X8, 58)`。
- **L65 EN**: Continues the surrounding declaration or expression: `gpr_x9_arm64,  // ARM64_X9, 59)`.
  **L65 CN**: 继续构造周围的声明或表达式：`gpr_x9_arm64,  // ARM64_X9, 59)`。
- **L66 EN**: Continues the surrounding declaration or expression: `gpr_x10_arm64, // ARM64_X10, 60)`.
  **L66 CN**: 继续构造周围的声明或表达式：`gpr_x10_arm64, // ARM64_X10, 60)`。
- **L67 EN**: Continues the surrounding declaration or expression: `gpr_x11_arm64, // ARM64_X11, 61)`.
  **L67 CN**: 继续构造周围的声明或表达式：`gpr_x11_arm64, // ARM64_X11, 61)`。
- **L68 EN**: Continues the surrounding declaration or expression: `gpr_x12_arm64, // ARM64_X12, 62)`.
  **L68 CN**: 继续构造周围的声明或表达式：`gpr_x12_arm64, // ARM64_X12, 62)`。
- **L69 EN**: Continues the surrounding declaration or expression: `gpr_x13_arm64, // ARM64_X13, 63)`.
  **L69 CN**: 继续构造周围的声明或表达式：`gpr_x13_arm64, // ARM64_X13, 63)`。
- **L70 EN**: Continues the surrounding declaration or expression: `gpr_x14_arm64, // ARM64_X14, 64)`.
  **L70 CN**: 继续构造周围的声明或表达式：`gpr_x14_arm64, // ARM64_X14, 64)`。
- **L71 EN**: Continues the surrounding declaration or expression: `gpr_x15_arm64, // ARM64_X15, 65)`.
  **L71 CN**: 继续构造周围的声明或表达式：`gpr_x15_arm64, // ARM64_X15, 65)`。
- **L72 EN**: Continues the surrounding declaration or expression: `gpr_x16_arm64, // ARM64_X16, 66)`.
  **L72 CN**: 继续构造周围的声明或表达式：`gpr_x16_arm64, // ARM64_X16, 66)`。

### Lines 73-96 / 第 73-96 行

````cpp
    gpr_x17_arm64, // ARM64_X17, 67)
    gpr_x18_arm64, // ARM64_X18, 68)
    gpr_x19_arm64, // ARM64_X19, 69)
    gpr_x20_arm64, // ARM64_X20, 70)
    gpr_x21_arm64, // ARM64_X21, 71)
    gpr_x22_arm64, // ARM64_X22, 72)
    gpr_x23_arm64, // ARM64_X23, 73)
    gpr_x24_arm64, // ARM64_X24, 74)
    gpr_x25_arm64, // ARM64_X25, 75)
    gpr_x26_arm64, // ARM64_X26, 76)
    gpr_x27_arm64, // ARM64_X27, 77)
    gpr_x28_arm64, // ARM64_X28, 78)
    gpr_fp_arm64,  // ARM64_FP, 79)
    gpr_lr_arm64,  // ARM64_LR, 80)
    gpr_sp_arm64,  // ARM64_SP, 81)
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    gpr_cpsr_arm64, // ARM64_NZCV, 90)
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    fpu_s0_arm64,  // (ARM64_S0, 100)
    fpu_s1_arm64,  // (ARM64_S1, 101)
````
- **L73 EN**: Continues the surrounding declaration or expression: `gpr_x17_arm64, // ARM64_X17, 67)`.
  **L73 CN**: 继续构造周围的声明或表达式：`gpr_x17_arm64, // ARM64_X17, 67)`。
- **L74 EN**: Continues the surrounding declaration or expression: `gpr_x18_arm64, // ARM64_X18, 68)`.
  **L74 CN**: 继续构造周围的声明或表达式：`gpr_x18_arm64, // ARM64_X18, 68)`。
- **L75 EN**: Continues the surrounding declaration or expression: `gpr_x19_arm64, // ARM64_X19, 69)`.
  **L75 CN**: 继续构造周围的声明或表达式：`gpr_x19_arm64, // ARM64_X19, 69)`。
- **L76 EN**: Continues the surrounding declaration or expression: `gpr_x20_arm64, // ARM64_X20, 70)`.
  **L76 CN**: 继续构造周围的声明或表达式：`gpr_x20_arm64, // ARM64_X20, 70)`。
- **L77 EN**: Continues the surrounding declaration or expression: `gpr_x21_arm64, // ARM64_X21, 71)`.
  **L77 CN**: 继续构造周围的声明或表达式：`gpr_x21_arm64, // ARM64_X21, 71)`。
- **L78 EN**: Continues the surrounding declaration or expression: `gpr_x22_arm64, // ARM64_X22, 72)`.
  **L78 CN**: 继续构造周围的声明或表达式：`gpr_x22_arm64, // ARM64_X22, 72)`。
- **L79 EN**: Continues the surrounding declaration or expression: `gpr_x23_arm64, // ARM64_X23, 73)`.
  **L79 CN**: 继续构造周围的声明或表达式：`gpr_x23_arm64, // ARM64_X23, 73)`。
- **L80 EN**: Continues the surrounding declaration or expression: `gpr_x24_arm64, // ARM64_X24, 74)`.
  **L80 CN**: 继续构造周围的声明或表达式：`gpr_x24_arm64, // ARM64_X24, 74)`。
- **L81 EN**: Continues the surrounding declaration or expression: `gpr_x25_arm64, // ARM64_X25, 75)`.
  **L81 CN**: 继续构造周围的声明或表达式：`gpr_x25_arm64, // ARM64_X25, 75)`。
- **L82 EN**: Continues the surrounding declaration or expression: `gpr_x26_arm64, // ARM64_X26, 76)`.
  **L82 CN**: 继续构造周围的声明或表达式：`gpr_x26_arm64, // ARM64_X26, 76)`。
- **L83 EN**: Continues the surrounding declaration or expression: `gpr_x27_arm64, // ARM64_X27, 77)`.
  **L83 CN**: 继续构造周围的声明或表达式：`gpr_x27_arm64, // ARM64_X27, 77)`。
- **L84 EN**: Continues the surrounding declaration or expression: `gpr_x28_arm64, // ARM64_X28, 78)`.
  **L84 CN**: 继续构造周围的声明或表达式：`gpr_x28_arm64, // ARM64_X28, 78)`。
- **L85 EN**: Continues the surrounding declaration or expression: `gpr_fp_arm64,  // ARM64_FP, 79)`.
  **L85 CN**: 继续构造周围的声明或表达式：`gpr_fp_arm64,  // ARM64_FP, 79)`。
- **L86 EN**: Continues the surrounding declaration or expression: `gpr_lr_arm64,  // ARM64_LR, 80)`.
  **L86 CN**: 继续构造周围的声明或表达式：`gpr_lr_arm64,  // ARM64_LR, 80)`。
- **L87 EN**: Continues the surrounding declaration or expression: `gpr_sp_arm64,  // ARM64_SP, 81)`.
  **L87 CN**: 继续构造周围的声明或表达式：`gpr_sp_arm64,  // ARM64_SP, 81)`。
- **L88 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L88 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L89 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L89 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L90 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L90 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L91 EN**: Continues the surrounding declaration or expression: `gpr_cpsr_arm64, // ARM64_NZCV, 90)`.
  **L91 CN**: 继续构造周围的声明或表达式：`gpr_cpsr_arm64, // ARM64_NZCV, 90)`。
- **L92 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L92 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L93 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L93 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L94 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L94 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L95 EN**: Continues the surrounding declaration or expression: `fpu_s0_arm64,  // (ARM64_S0, 100)`.
  **L95 CN**: 继续构造周围的声明或表达式：`fpu_s0_arm64,  // (ARM64_S0, 100)`。
- **L96 EN**: Continues the surrounding declaration or expression: `fpu_s1_arm64,  // (ARM64_S1, 101)`.
  **L96 CN**: 继续构造周围的声明或表达式：`fpu_s1_arm64,  // (ARM64_S1, 101)`。

### Lines 97-120 / 第 97-120 行

````cpp
    fpu_s2_arm64,  // (ARM64_S2, 102)
    fpu_s3_arm64,  // (ARM64_S3, 103)
    fpu_s4_arm64,  // (ARM64_S4, 104)
    fpu_s5_arm64,  // (ARM64_S5, 105)
    fpu_s6_arm64,  // (ARM64_S6, 106)
    fpu_s7_arm64,  // (ARM64_S7, 107)
    fpu_s8_arm64,  // (ARM64_S8, 108)
    fpu_s9_arm64,  // (ARM64_S9, 109)
    fpu_s10_arm64, // (ARM64_S10, 110)
    fpu_s11_arm64, // (ARM64_S11, 111)
    fpu_s12_arm64, // (ARM64_S12, 112)
    fpu_s13_arm64, // (ARM64_S13, 113)
    fpu_s14_arm64, // (ARM64_S14, 114)
    fpu_s15_arm64, // (ARM64_S15, 115)
    fpu_s16_arm64, // (ARM64_S16, 116)
    fpu_s17_arm64, // (ARM64_S17, 117)
    fpu_s18_arm64, // (ARM64_S18, 118)
    fpu_s19_arm64, // (ARM64_S19, 119)
    fpu_s20_arm64, // (ARM64_S20, 120)
    fpu_s21_arm64, // (ARM64_S21, 121)
    fpu_s22_arm64, // (ARM64_S22, 122)
    fpu_s23_arm64, // (ARM64_S23, 123)
    fpu_s24_arm64, // (ARM64_S24, 124)
    fpu_s25_arm64, // (ARM64_S25, 125)
````
- **L97 EN**: Continues the surrounding declaration or expression: `fpu_s2_arm64,  // (ARM64_S2, 102)`.
  **L97 CN**: 继续构造周围的声明或表达式：`fpu_s2_arm64,  // (ARM64_S2, 102)`。
- **L98 EN**: Continues the surrounding declaration or expression: `fpu_s3_arm64,  // (ARM64_S3, 103)`.
  **L98 CN**: 继续构造周围的声明或表达式：`fpu_s3_arm64,  // (ARM64_S3, 103)`。
- **L99 EN**: Continues the surrounding declaration or expression: `fpu_s4_arm64,  // (ARM64_S4, 104)`.
  **L99 CN**: 继续构造周围的声明或表达式：`fpu_s4_arm64,  // (ARM64_S4, 104)`。
- **L100 EN**: Continues the surrounding declaration or expression: `fpu_s5_arm64,  // (ARM64_S5, 105)`.
  **L100 CN**: 继续构造周围的声明或表达式：`fpu_s5_arm64,  // (ARM64_S5, 105)`。
- **L101 EN**: Continues the surrounding declaration or expression: `fpu_s6_arm64,  // (ARM64_S6, 106)`.
  **L101 CN**: 继续构造周围的声明或表达式：`fpu_s6_arm64,  // (ARM64_S6, 106)`。
- **L102 EN**: Continues the surrounding declaration or expression: `fpu_s7_arm64,  // (ARM64_S7, 107)`.
  **L102 CN**: 继续构造周围的声明或表达式：`fpu_s7_arm64,  // (ARM64_S7, 107)`。
- **L103 EN**: Continues the surrounding declaration or expression: `fpu_s8_arm64,  // (ARM64_S8, 108)`.
  **L103 CN**: 继续构造周围的声明或表达式：`fpu_s8_arm64,  // (ARM64_S8, 108)`。
- **L104 EN**: Continues the surrounding declaration or expression: `fpu_s9_arm64,  // (ARM64_S9, 109)`.
  **L104 CN**: 继续构造周围的声明或表达式：`fpu_s9_arm64,  // (ARM64_S9, 109)`。
- **L105 EN**: Continues the surrounding declaration or expression: `fpu_s10_arm64, // (ARM64_S10, 110)`.
  **L105 CN**: 继续构造周围的声明或表达式：`fpu_s10_arm64, // (ARM64_S10, 110)`。
- **L106 EN**: Continues the surrounding declaration or expression: `fpu_s11_arm64, // (ARM64_S11, 111)`.
  **L106 CN**: 继续构造周围的声明或表达式：`fpu_s11_arm64, // (ARM64_S11, 111)`。
- **L107 EN**: Continues the surrounding declaration or expression: `fpu_s12_arm64, // (ARM64_S12, 112)`.
  **L107 CN**: 继续构造周围的声明或表达式：`fpu_s12_arm64, // (ARM64_S12, 112)`。
- **L108 EN**: Continues the surrounding declaration or expression: `fpu_s13_arm64, // (ARM64_S13, 113)`.
  **L108 CN**: 继续构造周围的声明或表达式：`fpu_s13_arm64, // (ARM64_S13, 113)`。
- **L109 EN**: Continues the surrounding declaration or expression: `fpu_s14_arm64, // (ARM64_S14, 114)`.
  **L109 CN**: 继续构造周围的声明或表达式：`fpu_s14_arm64, // (ARM64_S14, 114)`。
- **L110 EN**: Continues the surrounding declaration or expression: `fpu_s15_arm64, // (ARM64_S15, 115)`.
  **L110 CN**: 继续构造周围的声明或表达式：`fpu_s15_arm64, // (ARM64_S15, 115)`。
- **L111 EN**: Continues the surrounding declaration or expression: `fpu_s16_arm64, // (ARM64_S16, 116)`.
  **L111 CN**: 继续构造周围的声明或表达式：`fpu_s16_arm64, // (ARM64_S16, 116)`。
- **L112 EN**: Continues the surrounding declaration or expression: `fpu_s17_arm64, // (ARM64_S17, 117)`.
  **L112 CN**: 继续构造周围的声明或表达式：`fpu_s17_arm64, // (ARM64_S17, 117)`。
- **L113 EN**: Continues the surrounding declaration or expression: `fpu_s18_arm64, // (ARM64_S18, 118)`.
  **L113 CN**: 继续构造周围的声明或表达式：`fpu_s18_arm64, // (ARM64_S18, 118)`。
- **L114 EN**: Continues the surrounding declaration or expression: `fpu_s19_arm64, // (ARM64_S19, 119)`.
  **L114 CN**: 继续构造周围的声明或表达式：`fpu_s19_arm64, // (ARM64_S19, 119)`。
- **L115 EN**: Continues the surrounding declaration or expression: `fpu_s20_arm64, // (ARM64_S20, 120)`.
  **L115 CN**: 继续构造周围的声明或表达式：`fpu_s20_arm64, // (ARM64_S20, 120)`。
- **L116 EN**: Continues the surrounding declaration or expression: `fpu_s21_arm64, // (ARM64_S21, 121)`.
  **L116 CN**: 继续构造周围的声明或表达式：`fpu_s21_arm64, // (ARM64_S21, 121)`。
- **L117 EN**: Continues the surrounding declaration or expression: `fpu_s22_arm64, // (ARM64_S22, 122)`.
  **L117 CN**: 继续构造周围的声明或表达式：`fpu_s22_arm64, // (ARM64_S22, 122)`。
- **L118 EN**: Continues the surrounding declaration or expression: `fpu_s23_arm64, // (ARM64_S23, 123)`.
  **L118 CN**: 继续构造周围的声明或表达式：`fpu_s23_arm64, // (ARM64_S23, 123)`。
- **L119 EN**: Continues the surrounding declaration or expression: `fpu_s24_arm64, // (ARM64_S24, 124)`.
  **L119 CN**: 继续构造周围的声明或表达式：`fpu_s24_arm64, // (ARM64_S24, 124)`。
- **L120 EN**: Continues the surrounding declaration or expression: `fpu_s25_arm64, // (ARM64_S25, 125)`.
  **L120 CN**: 继续构造周围的声明或表达式：`fpu_s25_arm64, // (ARM64_S25, 125)`。

### Lines 121-144 / 第 121-144 行

````cpp
    fpu_s26_arm64, // (ARM64_S26, 126)
    fpu_s27_arm64, // (ARM64_S27, 127)
    fpu_s28_arm64, // (ARM64_S28, 128)
    fpu_s29_arm64, // (ARM64_S29, 129)
    fpu_s30_arm64, // (ARM64_S30, 130)
    fpu_s31_arm64, // (ARM64_S31, 131)
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    fpu_d0_arm64,  // (ARM64_D0, 140)
    fpu_d1_arm64,  // (ARM64_D1, 141)
    fpu_d2_arm64,  // (ARM64_D2, 142)
    fpu_d3_arm64,  // (ARM64_D3, 143)
    fpu_d4_arm64,  // (ARM64_D4, 144)
    fpu_d5_arm64,  // (ARM64_D5, 145)
    fpu_d6_arm64,  // (ARM64_D6, 146)
    fpu_d7_arm64,  // (ARM64_D7, 147)
    fpu_d8_arm64,  // (ARM64_D8, 148)
    fpu_d9_arm64,  // (ARM64_D9, 149)
    fpu_d10_arm64, // (ARM64_D10, 150)
    fpu_d11_arm64, // (ARM64_D11, 151)
    fpu_d12_arm64, // (ARM64_D12, 152)
    fpu_d13_arm64, // (ARM64_D13, 153)
    fpu_d14_arm64, // (ARM64_D14, 154)
````
- **L121 EN**: Continues the surrounding declaration or expression: `fpu_s26_arm64, // (ARM64_S26, 126)`.
  **L121 CN**: 继续构造周围的声明或表达式：`fpu_s26_arm64, // (ARM64_S26, 126)`。
- **L122 EN**: Continues the surrounding declaration or expression: `fpu_s27_arm64, // (ARM64_S27, 127)`.
  **L122 CN**: 继续构造周围的声明或表达式：`fpu_s27_arm64, // (ARM64_S27, 127)`。
- **L123 EN**: Continues the surrounding declaration or expression: `fpu_s28_arm64, // (ARM64_S28, 128)`.
  **L123 CN**: 继续构造周围的声明或表达式：`fpu_s28_arm64, // (ARM64_S28, 128)`。
- **L124 EN**: Continues the surrounding declaration or expression: `fpu_s29_arm64, // (ARM64_S29, 129)`.
  **L124 CN**: 继续构造周围的声明或表达式：`fpu_s29_arm64, // (ARM64_S29, 129)`。
- **L125 EN**: Continues the surrounding declaration or expression: `fpu_s30_arm64, // (ARM64_S30, 130)`.
  **L125 CN**: 继续构造周围的声明或表达式：`fpu_s30_arm64, // (ARM64_S30, 130)`。
- **L126 EN**: Continues the surrounding declaration or expression: `fpu_s31_arm64, // (ARM64_S31, 131)`.
  **L126 CN**: 继续构造周围的声明或表达式：`fpu_s31_arm64, // (ARM64_S31, 131)`。
- **L127 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L127 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L128 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L128 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L129 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L129 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L130 EN**: Continues the surrounding declaration or expression: `fpu_d0_arm64,  // (ARM64_D0, 140)`.
  **L130 CN**: 继续构造周围的声明或表达式：`fpu_d0_arm64,  // (ARM64_D0, 140)`。
- **L131 EN**: Continues the surrounding declaration or expression: `fpu_d1_arm64,  // (ARM64_D1, 141)`.
  **L131 CN**: 继续构造周围的声明或表达式：`fpu_d1_arm64,  // (ARM64_D1, 141)`。
- **L132 EN**: Continues the surrounding declaration or expression: `fpu_d2_arm64,  // (ARM64_D2, 142)`.
  **L132 CN**: 继续构造周围的声明或表达式：`fpu_d2_arm64,  // (ARM64_D2, 142)`。
- **L133 EN**: Continues the surrounding declaration or expression: `fpu_d3_arm64,  // (ARM64_D3, 143)`.
  **L133 CN**: 继续构造周围的声明或表达式：`fpu_d3_arm64,  // (ARM64_D3, 143)`。
- **L134 EN**: Continues the surrounding declaration or expression: `fpu_d4_arm64,  // (ARM64_D4, 144)`.
  **L134 CN**: 继续构造周围的声明或表达式：`fpu_d4_arm64,  // (ARM64_D4, 144)`。
- **L135 EN**: Continues the surrounding declaration or expression: `fpu_d5_arm64,  // (ARM64_D5, 145)`.
  **L135 CN**: 继续构造周围的声明或表达式：`fpu_d5_arm64,  // (ARM64_D5, 145)`。
- **L136 EN**: Continues the surrounding declaration or expression: `fpu_d6_arm64,  // (ARM64_D6, 146)`.
  **L136 CN**: 继续构造周围的声明或表达式：`fpu_d6_arm64,  // (ARM64_D6, 146)`。
- **L137 EN**: Continues the surrounding declaration or expression: `fpu_d7_arm64,  // (ARM64_D7, 147)`.
  **L137 CN**: 继续构造周围的声明或表达式：`fpu_d7_arm64,  // (ARM64_D7, 147)`。
- **L138 EN**: Continues the surrounding declaration or expression: `fpu_d8_arm64,  // (ARM64_D8, 148)`.
  **L138 CN**: 继续构造周围的声明或表达式：`fpu_d8_arm64,  // (ARM64_D8, 148)`。
- **L139 EN**: Continues the surrounding declaration or expression: `fpu_d9_arm64,  // (ARM64_D9, 149)`.
  **L139 CN**: 继续构造周围的声明或表达式：`fpu_d9_arm64,  // (ARM64_D9, 149)`。
- **L140 EN**: Continues the surrounding declaration or expression: `fpu_d10_arm64, // (ARM64_D10, 150)`.
  **L140 CN**: 继续构造周围的声明或表达式：`fpu_d10_arm64, // (ARM64_D10, 150)`。
- **L141 EN**: Continues the surrounding declaration or expression: `fpu_d11_arm64, // (ARM64_D11, 151)`.
  **L141 CN**: 继续构造周围的声明或表达式：`fpu_d11_arm64, // (ARM64_D11, 151)`。
- **L142 EN**: Continues the surrounding declaration or expression: `fpu_d12_arm64, // (ARM64_D12, 152)`.
  **L142 CN**: 继续构造周围的声明或表达式：`fpu_d12_arm64, // (ARM64_D12, 152)`。
- **L143 EN**: Continues the surrounding declaration or expression: `fpu_d13_arm64, // (ARM64_D13, 153)`.
  **L143 CN**: 继续构造周围的声明或表达式：`fpu_d13_arm64, // (ARM64_D13, 153)`。
- **L144 EN**: Continues the surrounding declaration or expression: `fpu_d14_arm64, // (ARM64_D14, 154)`.
  **L144 CN**: 继续构造周围的声明或表达式：`fpu_d14_arm64, // (ARM64_D14, 154)`。

### Lines 145-168 / 第 145-168 行

````cpp
    fpu_d15_arm64, // (ARM64_D15, 155)
    fpu_d16_arm64, // (ARM64_D16, 156)
    fpu_d17_arm64, // (ARM64_D17, 157)
    fpu_d18_arm64, // (ARM64_D18, 158)
    fpu_d19_arm64, // (ARM64_D19, 159)
    fpu_d20_arm64, // (ARM64_D20, 160)
    fpu_d21_arm64, // (ARM64_D21, 161)
    fpu_d22_arm64, // (ARM64_D22, 162)
    fpu_d23_arm64, // (ARM64_D23, 163)
    fpu_d24_arm64, // (ARM64_D24, 164)
    fpu_d25_arm64, // (ARM64_D25, 165)
    fpu_d26_arm64, // (ARM64_D26, 166)
    fpu_d27_arm64, // (ARM64_D27, 167)
    fpu_d28_arm64, // (ARM64_D28, 168)
    fpu_d29_arm64, // (ARM64_D29, 169)
    fpu_d30_arm64, // (ARM64_D30, 170)
    fpu_d31_arm64, // (ARM64_D31, 171)
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    fpu_v0_arm64,  // (ARM64_Q0, 180)
    fpu_v1_arm64,  // (ARM64_Q1, 181)
    fpu_v2_arm64,  // (ARM64_Q2, 182)
    fpu_v3_arm64,  // (ARM64_Q3, 183)
````
- **L145 EN**: Continues the surrounding declaration or expression: `fpu_d15_arm64, // (ARM64_D15, 155)`.
  **L145 CN**: 继续构造周围的声明或表达式：`fpu_d15_arm64, // (ARM64_D15, 155)`。
- **L146 EN**: Continues the surrounding declaration or expression: `fpu_d16_arm64, // (ARM64_D16, 156)`.
  **L146 CN**: 继续构造周围的声明或表达式：`fpu_d16_arm64, // (ARM64_D16, 156)`。
- **L147 EN**: Continues the surrounding declaration or expression: `fpu_d17_arm64, // (ARM64_D17, 157)`.
  **L147 CN**: 继续构造周围的声明或表达式：`fpu_d17_arm64, // (ARM64_D17, 157)`。
- **L148 EN**: Continues the surrounding declaration or expression: `fpu_d18_arm64, // (ARM64_D18, 158)`.
  **L148 CN**: 继续构造周围的声明或表达式：`fpu_d18_arm64, // (ARM64_D18, 158)`。
- **L149 EN**: Continues the surrounding declaration or expression: `fpu_d19_arm64, // (ARM64_D19, 159)`.
  **L149 CN**: 继续构造周围的声明或表达式：`fpu_d19_arm64, // (ARM64_D19, 159)`。
- **L150 EN**: Continues the surrounding declaration or expression: `fpu_d20_arm64, // (ARM64_D20, 160)`.
  **L150 CN**: 继续构造周围的声明或表达式：`fpu_d20_arm64, // (ARM64_D20, 160)`。
- **L151 EN**: Continues the surrounding declaration or expression: `fpu_d21_arm64, // (ARM64_D21, 161)`.
  **L151 CN**: 继续构造周围的声明或表达式：`fpu_d21_arm64, // (ARM64_D21, 161)`。
- **L152 EN**: Continues the surrounding declaration or expression: `fpu_d22_arm64, // (ARM64_D22, 162)`.
  **L152 CN**: 继续构造周围的声明或表达式：`fpu_d22_arm64, // (ARM64_D22, 162)`。
- **L153 EN**: Continues the surrounding declaration or expression: `fpu_d23_arm64, // (ARM64_D23, 163)`.
  **L153 CN**: 继续构造周围的声明或表达式：`fpu_d23_arm64, // (ARM64_D23, 163)`。
- **L154 EN**: Continues the surrounding declaration or expression: `fpu_d24_arm64, // (ARM64_D24, 164)`.
  **L154 CN**: 继续构造周围的声明或表达式：`fpu_d24_arm64, // (ARM64_D24, 164)`。
- **L155 EN**: Continues the surrounding declaration or expression: `fpu_d25_arm64, // (ARM64_D25, 165)`.
  **L155 CN**: 继续构造周围的声明或表达式：`fpu_d25_arm64, // (ARM64_D25, 165)`。
- **L156 EN**: Continues the surrounding declaration or expression: `fpu_d26_arm64, // (ARM64_D26, 166)`.
  **L156 CN**: 继续构造周围的声明或表达式：`fpu_d26_arm64, // (ARM64_D26, 166)`。
- **L157 EN**: Continues the surrounding declaration or expression: `fpu_d27_arm64, // (ARM64_D27, 167)`.
  **L157 CN**: 继续构造周围的声明或表达式：`fpu_d27_arm64, // (ARM64_D27, 167)`。
- **L158 EN**: Continues the surrounding declaration or expression: `fpu_d28_arm64, // (ARM64_D28, 168)`.
  **L158 CN**: 继续构造周围的声明或表达式：`fpu_d28_arm64, // (ARM64_D28, 168)`。
- **L159 EN**: Continues the surrounding declaration or expression: `fpu_d29_arm64, // (ARM64_D29, 169)`.
  **L159 CN**: 继续构造周围的声明或表达式：`fpu_d29_arm64, // (ARM64_D29, 169)`。
- **L160 EN**: Continues the surrounding declaration or expression: `fpu_d30_arm64, // (ARM64_D30, 170)`.
  **L160 CN**: 继续构造周围的声明或表达式：`fpu_d30_arm64, // (ARM64_D30, 170)`。
- **L161 EN**: Continues the surrounding declaration or expression: `fpu_d31_arm64, // (ARM64_D31, 171)`.
  **L161 CN**: 继续构造周围的声明或表达式：`fpu_d31_arm64, // (ARM64_D31, 171)`。
- **L162 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L162 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L163 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L163 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L164 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L164 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L165 EN**: Continues the surrounding declaration or expression: `fpu_v0_arm64,  // (ARM64_Q0, 180)`.
  **L165 CN**: 继续构造周围的声明或表达式：`fpu_v0_arm64,  // (ARM64_Q0, 180)`。
- **L166 EN**: Continues the surrounding declaration or expression: `fpu_v1_arm64,  // (ARM64_Q1, 181)`.
  **L166 CN**: 继续构造周围的声明或表达式：`fpu_v1_arm64,  // (ARM64_Q1, 181)`。
- **L167 EN**: Continues the surrounding declaration or expression: `fpu_v2_arm64,  // (ARM64_Q2, 182)`.
  **L167 CN**: 继续构造周围的声明或表达式：`fpu_v2_arm64,  // (ARM64_Q2, 182)`。
- **L168 EN**: Continues the surrounding declaration or expression: `fpu_v3_arm64,  // (ARM64_Q3, 183)`.
  **L168 CN**: 继续构造周围的声明或表达式：`fpu_v3_arm64,  // (ARM64_Q3, 183)`。

### Lines 169-192 / 第 169-192 行

````cpp
    fpu_v4_arm64,  // (ARM64_Q4, 184)
    fpu_v5_arm64,  // (ARM64_Q5, 185)
    fpu_v6_arm64,  // (ARM64_Q6, 186)
    fpu_v7_arm64,  // (ARM64_Q7, 187)
    fpu_v8_arm64,  // (ARM64_Q8, 188)
    fpu_v9_arm64,  // (ARM64_Q9, 189)
    fpu_v10_arm64, // (ARM64_Q10, 190)
    fpu_v11_arm64, // (ARM64_Q11, 191)
    fpu_v12_arm64, // (ARM64_Q12, 192)
    fpu_v13_arm64, // (ARM64_Q13, 193)
    fpu_v14_arm64, // (ARM64_Q14, 194)
    fpu_v15_arm64, // (ARM64_Q15, 195)
    fpu_v16_arm64, // (ARM64_Q16, 196)
    fpu_v17_arm64, // (ARM64_Q17, 197)
    fpu_v18_arm64, // (ARM64_Q18, 198)
    fpu_v19_arm64, // (ARM64_Q19, 199)
    fpu_v20_arm64, // (ARM64_Q20, 200)
    fpu_v21_arm64, // (ARM64_Q21, 201)
    fpu_v22_arm64, // (ARM64_Q22, 202)
    fpu_v23_arm64, // (ARM64_Q23, 203)
    fpu_v24_arm64, // (ARM64_Q24, 204)
    fpu_v25_arm64, // (ARM64_Q25, 205)
    fpu_v26_arm64, // (ARM64_Q26, 206)
    fpu_v27_arm64, // (ARM64_Q27, 207)
````
- **L169 EN**: Continues the surrounding declaration or expression: `fpu_v4_arm64,  // (ARM64_Q4, 184)`.
  **L169 CN**: 继续构造周围的声明或表达式：`fpu_v4_arm64,  // (ARM64_Q4, 184)`。
- **L170 EN**: Continues the surrounding declaration or expression: `fpu_v5_arm64,  // (ARM64_Q5, 185)`.
  **L170 CN**: 继续构造周围的声明或表达式：`fpu_v5_arm64,  // (ARM64_Q5, 185)`。
- **L171 EN**: Continues the surrounding declaration or expression: `fpu_v6_arm64,  // (ARM64_Q6, 186)`.
  **L171 CN**: 继续构造周围的声明或表达式：`fpu_v6_arm64,  // (ARM64_Q6, 186)`。
- **L172 EN**: Continues the surrounding declaration or expression: `fpu_v7_arm64,  // (ARM64_Q7, 187)`.
  **L172 CN**: 继续构造周围的声明或表达式：`fpu_v7_arm64,  // (ARM64_Q7, 187)`。
- **L173 EN**: Continues the surrounding declaration or expression: `fpu_v8_arm64,  // (ARM64_Q8, 188)`.
  **L173 CN**: 继续构造周围的声明或表达式：`fpu_v8_arm64,  // (ARM64_Q8, 188)`。
- **L174 EN**: Continues the surrounding declaration or expression: `fpu_v9_arm64,  // (ARM64_Q9, 189)`.
  **L174 CN**: 继续构造周围的声明或表达式：`fpu_v9_arm64,  // (ARM64_Q9, 189)`。
- **L175 EN**: Continues the surrounding declaration or expression: `fpu_v10_arm64, // (ARM64_Q10, 190)`.
  **L175 CN**: 继续构造周围的声明或表达式：`fpu_v10_arm64, // (ARM64_Q10, 190)`。
- **L176 EN**: Continues the surrounding declaration or expression: `fpu_v11_arm64, // (ARM64_Q11, 191)`.
  **L176 CN**: 继续构造周围的声明或表达式：`fpu_v11_arm64, // (ARM64_Q11, 191)`。
- **L177 EN**: Continues the surrounding declaration or expression: `fpu_v12_arm64, // (ARM64_Q12, 192)`.
  **L177 CN**: 继续构造周围的声明或表达式：`fpu_v12_arm64, // (ARM64_Q12, 192)`。
- **L178 EN**: Continues the surrounding declaration or expression: `fpu_v13_arm64, // (ARM64_Q13, 193)`.
  **L178 CN**: 继续构造周围的声明或表达式：`fpu_v13_arm64, // (ARM64_Q13, 193)`。
- **L179 EN**: Continues the surrounding declaration or expression: `fpu_v14_arm64, // (ARM64_Q14, 194)`.
  **L179 CN**: 继续构造周围的声明或表达式：`fpu_v14_arm64, // (ARM64_Q14, 194)`。
- **L180 EN**: Continues the surrounding declaration or expression: `fpu_v15_arm64, // (ARM64_Q15, 195)`.
  **L180 CN**: 继续构造周围的声明或表达式：`fpu_v15_arm64, // (ARM64_Q15, 195)`。
- **L181 EN**: Continues the surrounding declaration or expression: `fpu_v16_arm64, // (ARM64_Q16, 196)`.
  **L181 CN**: 继续构造周围的声明或表达式：`fpu_v16_arm64, // (ARM64_Q16, 196)`。
- **L182 EN**: Continues the surrounding declaration or expression: `fpu_v17_arm64, // (ARM64_Q17, 197)`.
  **L182 CN**: 继续构造周围的声明或表达式：`fpu_v17_arm64, // (ARM64_Q17, 197)`。
- **L183 EN**: Continues the surrounding declaration or expression: `fpu_v18_arm64, // (ARM64_Q18, 198)`.
  **L183 CN**: 继续构造周围的声明或表达式：`fpu_v18_arm64, // (ARM64_Q18, 198)`。
- **L184 EN**: Continues the surrounding declaration or expression: `fpu_v19_arm64, // (ARM64_Q19, 199)`.
  **L184 CN**: 继续构造周围的声明或表达式：`fpu_v19_arm64, // (ARM64_Q19, 199)`。
- **L185 EN**: Continues the surrounding declaration or expression: `fpu_v20_arm64, // (ARM64_Q20, 200)`.
  **L185 CN**: 继续构造周围的声明或表达式：`fpu_v20_arm64, // (ARM64_Q20, 200)`。
- **L186 EN**: Continues the surrounding declaration or expression: `fpu_v21_arm64, // (ARM64_Q21, 201)`.
  **L186 CN**: 继续构造周围的声明或表达式：`fpu_v21_arm64, // (ARM64_Q21, 201)`。
- **L187 EN**: Continues the surrounding declaration or expression: `fpu_v22_arm64, // (ARM64_Q22, 202)`.
  **L187 CN**: 继续构造周围的声明或表达式：`fpu_v22_arm64, // (ARM64_Q22, 202)`。
- **L188 EN**: Continues the surrounding declaration or expression: `fpu_v23_arm64, // (ARM64_Q23, 203)`.
  **L188 CN**: 继续构造周围的声明或表达式：`fpu_v23_arm64, // (ARM64_Q23, 203)`。
- **L189 EN**: Continues the surrounding declaration or expression: `fpu_v24_arm64, // (ARM64_Q24, 204)`.
  **L189 CN**: 继续构造周围的声明或表达式：`fpu_v24_arm64, // (ARM64_Q24, 204)`。
- **L190 EN**: Continues the surrounding declaration or expression: `fpu_v25_arm64, // (ARM64_Q25, 205)`.
  **L190 CN**: 继续构造周围的声明或表达式：`fpu_v25_arm64, // (ARM64_Q25, 205)`。
- **L191 EN**: Continues the surrounding declaration or expression: `fpu_v26_arm64, // (ARM64_Q26, 206)`.
  **L191 CN**: 继续构造周围的声明或表达式：`fpu_v26_arm64, // (ARM64_Q26, 206)`。
- **L192 EN**: Continues the surrounding declaration or expression: `fpu_v27_arm64, // (ARM64_Q27, 207)`.
  **L192 CN**: 继续构造周围的声明或表达式：`fpu_v27_arm64, // (ARM64_Q27, 207)`。

### Lines 193-216 / 第 193-216 行

````cpp
    fpu_v28_arm64, // (ARM64_Q28, 208)
    fpu_v29_arm64, // (ARM64_Q29, 209)
    fpu_v30_arm64, // (ARM64_Q30, 210)
    fpu_v31_arm64, // (ARM64_Q31, 211)
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    fpu_fpsr_arm64 // ARM64_FPSR, 220)
};

static const uint32_t g_code_view_to_lldb_registers_x86[] = {
    LLDB_INVALID_REGNUM, // NONE
    lldb_al_i386,        // AL
    lldb_cl_i386,        // CL
    lldb_dl_i386,        // DL
    lldb_bl_i386,        // BL
    lldb_ah_i386,        // AH
    lldb_ch_i386,        // CH
    lldb_dh_i386,        // DH
    lldb_bh_i386,        // BH
    lldb_ax_i386,        // AX
    lldb_cx_i386,        // CX
    lldb_dx_i386,        // DX
    lldb_bx_i386,        // BX
````
- **L193 EN**: Continues the surrounding declaration or expression: `fpu_v28_arm64, // (ARM64_Q28, 208)`.
  **L193 CN**: 继续构造周围的声明或表达式：`fpu_v28_arm64, // (ARM64_Q28, 208)`。
- **L194 EN**: Continues the surrounding declaration or expression: `fpu_v29_arm64, // (ARM64_Q29, 209)`.
  **L194 CN**: 继续构造周围的声明或表达式：`fpu_v29_arm64, // (ARM64_Q29, 209)`。
- **L195 EN**: Continues the surrounding declaration or expression: `fpu_v30_arm64, // (ARM64_Q30, 210)`.
  **L195 CN**: 继续构造周围的声明或表达式：`fpu_v30_arm64, // (ARM64_Q30, 210)`。
- **L196 EN**: Continues the surrounding declaration or expression: `fpu_v31_arm64, // (ARM64_Q31, 211)`.
  **L196 CN**: 继续构造周围的声明或表达式：`fpu_v31_arm64, // (ARM64_Q31, 211)`。
- **L197 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L197 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L198 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L198 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L199 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L199 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L200 EN**: Continues the surrounding declaration or expression: `fpu_fpsr_arm64 // ARM64_FPSR, 220)`.
  **L200 CN**: 继续构造周围的声明或表达式：`fpu_fpsr_arm64 // ARM64_FPSR, 220)`。
- **L201 EN**: Closes the current declaration scope such as a class or struct.
  **L201 CN**: 结束当前声明作用域，例如类或结构体。
- **L202 EN**: Blank line separates nearby declarations or logic blocks.
  **L202 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L203 EN**: Continues the surrounding declaration or expression: `static const uint32_t g_code_view_to_lldb_registers_x86[] = {`.
  **L203 CN**: 继续构造周围的声明或表达式：`static const uint32_t g_code_view_to_lldb_registers_x86[] = {`。
- **L204 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // NONE`.
  **L204 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // NONE`。
- **L205 EN**: Continues the surrounding declaration or expression: `lldb_al_i386,        // AL`.
  **L205 CN**: 继续构造周围的声明或表达式：`lldb_al_i386,        // AL`。
- **L206 EN**: Continues the surrounding declaration or expression: `lldb_cl_i386,        // CL`.
  **L206 CN**: 继续构造周围的声明或表达式：`lldb_cl_i386,        // CL`。
- **L207 EN**: Continues the surrounding declaration or expression: `lldb_dl_i386,        // DL`.
  **L207 CN**: 继续构造周围的声明或表达式：`lldb_dl_i386,        // DL`。
- **L208 EN**: Continues the surrounding declaration or expression: `lldb_bl_i386,        // BL`.
  **L208 CN**: 继续构造周围的声明或表达式：`lldb_bl_i386,        // BL`。
- **L209 EN**: Continues the surrounding declaration or expression: `lldb_ah_i386,        // AH`.
  **L209 CN**: 继续构造周围的声明或表达式：`lldb_ah_i386,        // AH`。
- **L210 EN**: Continues the surrounding declaration or expression: `lldb_ch_i386,        // CH`.
  **L210 CN**: 继续构造周围的声明或表达式：`lldb_ch_i386,        // CH`。
- **L211 EN**: Continues the surrounding declaration or expression: `lldb_dh_i386,        // DH`.
  **L211 CN**: 继续构造周围的声明或表达式：`lldb_dh_i386,        // DH`。
- **L212 EN**: Continues the surrounding declaration or expression: `lldb_bh_i386,        // BH`.
  **L212 CN**: 继续构造周围的声明或表达式：`lldb_bh_i386,        // BH`。
- **L213 EN**: Continues the surrounding declaration or expression: `lldb_ax_i386,        // AX`.
  **L213 CN**: 继续构造周围的声明或表达式：`lldb_ax_i386,        // AX`。
- **L214 EN**: Continues the surrounding declaration or expression: `lldb_cx_i386,        // CX`.
  **L214 CN**: 继续构造周围的声明或表达式：`lldb_cx_i386,        // CX`。
- **L215 EN**: Continues the surrounding declaration or expression: `lldb_dx_i386,        // DX`.
  **L215 CN**: 继续构造周围的声明或表达式：`lldb_dx_i386,        // DX`。
- **L216 EN**: Continues the surrounding declaration or expression: `lldb_bx_i386,        // BX`.
  **L216 CN**: 继续构造周围的声明或表达式：`lldb_bx_i386,        // BX`。

### Lines 217-240 / 第 217-240 行

````cpp
    lldb_sp_i386,        // SP
    lldb_bp_i386,        // BP
    lldb_si_i386,        // SI
    lldb_di_i386,        // DI
    lldb_eax_i386,       // EAX
    lldb_ecx_i386,       // ECX
    lldb_edx_i386,       // EDX
    lldb_ebx_i386,       // EBX
    lldb_esp_i386,       // ESP
    lldb_ebp_i386,       // EBP
    lldb_esi_i386,       // ESI
    lldb_edi_i386,       // EDI
    lldb_es_i386,        // ES
    lldb_cs_i386,        // CS
    lldb_ss_i386,        // SS
    lldb_ds_i386,        // DS
    lldb_fs_i386,        // FS
    lldb_gs_i386,        // GS
    LLDB_INVALID_REGNUM, // IP
    LLDB_INVALID_REGNUM, // FLAGS
    lldb_eip_i386,       // EIP
    lldb_eflags_i386,    // EFLAGS
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
````
- **L217 EN**: Continues the surrounding declaration or expression: `lldb_sp_i386,        // SP`.
  **L217 CN**: 继续构造周围的声明或表达式：`lldb_sp_i386,        // SP`。
- **L218 EN**: Continues the surrounding declaration or expression: `lldb_bp_i386,        // BP`.
  **L218 CN**: 继续构造周围的声明或表达式：`lldb_bp_i386,        // BP`。
- **L219 EN**: Continues the surrounding declaration or expression: `lldb_si_i386,        // SI`.
  **L219 CN**: 继续构造周围的声明或表达式：`lldb_si_i386,        // SI`。
- **L220 EN**: Continues the surrounding declaration or expression: `lldb_di_i386,        // DI`.
  **L220 CN**: 继续构造周围的声明或表达式：`lldb_di_i386,        // DI`。
- **L221 EN**: Continues the surrounding declaration or expression: `lldb_eax_i386,       // EAX`.
  **L221 CN**: 继续构造周围的声明或表达式：`lldb_eax_i386,       // EAX`。
- **L222 EN**: Continues the surrounding declaration or expression: `lldb_ecx_i386,       // ECX`.
  **L222 CN**: 继续构造周围的声明或表达式：`lldb_ecx_i386,       // ECX`。
- **L223 EN**: Continues the surrounding declaration or expression: `lldb_edx_i386,       // EDX`.
  **L223 CN**: 继续构造周围的声明或表达式：`lldb_edx_i386,       // EDX`。
- **L224 EN**: Continues the surrounding declaration or expression: `lldb_ebx_i386,       // EBX`.
  **L224 CN**: 继续构造周围的声明或表达式：`lldb_ebx_i386,       // EBX`。
- **L225 EN**: Continues the surrounding declaration or expression: `lldb_esp_i386,       // ESP`.
  **L225 CN**: 继续构造周围的声明或表达式：`lldb_esp_i386,       // ESP`。
- **L226 EN**: Continues the surrounding declaration or expression: `lldb_ebp_i386,       // EBP`.
  **L226 CN**: 继续构造周围的声明或表达式：`lldb_ebp_i386,       // EBP`。
- **L227 EN**: Continues the surrounding declaration or expression: `lldb_esi_i386,       // ESI`.
  **L227 CN**: 继续构造周围的声明或表达式：`lldb_esi_i386,       // ESI`。
- **L228 EN**: Continues the surrounding declaration or expression: `lldb_edi_i386,       // EDI`.
  **L228 CN**: 继续构造周围的声明或表达式：`lldb_edi_i386,       // EDI`。
- **L229 EN**: Continues the surrounding declaration or expression: `lldb_es_i386,        // ES`.
  **L229 CN**: 继续构造周围的声明或表达式：`lldb_es_i386,        // ES`。
- **L230 EN**: Continues the surrounding declaration or expression: `lldb_cs_i386,        // CS`.
  **L230 CN**: 继续构造周围的声明或表达式：`lldb_cs_i386,        // CS`。
- **L231 EN**: Continues the surrounding declaration or expression: `lldb_ss_i386,        // SS`.
  **L231 CN**: 继续构造周围的声明或表达式：`lldb_ss_i386,        // SS`。
- **L232 EN**: Continues the surrounding declaration or expression: `lldb_ds_i386,        // DS`.
  **L232 CN**: 继续构造周围的声明或表达式：`lldb_ds_i386,        // DS`。
- **L233 EN**: Continues the surrounding declaration or expression: `lldb_fs_i386,        // FS`.
  **L233 CN**: 继续构造周围的声明或表达式：`lldb_fs_i386,        // FS`。
- **L234 EN**: Continues the surrounding declaration or expression: `lldb_gs_i386,        // GS`.
  **L234 CN**: 继续构造周围的声明或表达式：`lldb_gs_i386,        // GS`。
- **L235 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // IP`.
  **L235 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // IP`。
- **L236 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FLAGS`.
  **L236 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FLAGS`。
- **L237 EN**: Continues the surrounding declaration or expression: `lldb_eip_i386,       // EIP`.
  **L237 CN**: 继续构造周围的声明或表达式：`lldb_eip_i386,       // EIP`。
- **L238 EN**: Continues the surrounding declaration or expression: `lldb_eflags_i386,    // EFLAGS`.
  **L238 CN**: 继续构造周围的声明或表达式：`lldb_eflags_i386,    // EFLAGS`。
- **L239 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L239 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L240 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L240 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。

### Lines 241-264 / 第 241-264 行

````cpp
    LLDB_INVALID_REGNUM, // TEMP
    LLDB_INVALID_REGNUM, // TEMPH
    LLDB_INVALID_REGNUM, // QUOTE
    LLDB_INVALID_REGNUM, // PCDR3
    LLDB_INVALID_REGNUM, // PCDR4
    LLDB_INVALID_REGNUM, // PCDR5
    LLDB_INVALID_REGNUM, // PCDR6
    LLDB_INVALID_REGNUM, // PCDR7
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, // CR0
    LLDB_INVALID_REGNUM, // CR1
    LLDB_INVALID_REGNUM, // CR2
    LLDB_INVALID_REGNUM, // CR3
    LLDB_INVALID_REGNUM, // CR4
````
- **L241 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // TEMP`.
  **L241 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // TEMP`。
- **L242 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // TEMPH`.
  **L242 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // TEMPH`。
- **L243 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // QUOTE`.
  **L243 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // QUOTE`。
- **L244 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR3`.
  **L244 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR3`。
- **L245 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR4`.
  **L245 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR4`。
- **L246 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR5`.
  **L246 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR5`。
- **L247 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR6`.
  **L247 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR6`。
- **L248 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR7`.
  **L248 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR7`。
- **L249 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L249 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L250 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L250 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L251 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L251 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L252 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L252 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L253 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L253 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L254 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L254 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L255 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L255 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L256 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L256 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L257 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L257 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L258 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L258 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L259 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L259 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L260 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR0`.
  **L260 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR0`。
- **L261 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR1`.
  **L261 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR1`。
- **L262 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR2`.
  **L262 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR2`。
- **L263 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR3`.
  **L263 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR3`。
- **L264 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR4`.
  **L264 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR4`。

### Lines 265-288 / 第 265-288 行

````cpp
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    lldb_dr0_i386, // DR0
    lldb_dr1_i386, // DR1
    lldb_dr2_i386, // DR2
    lldb_dr3_i386, // DR3
    lldb_dr4_i386, // DR4
    lldb_dr5_i386, // DR5
    lldb_dr6_i386, // DR6
    lldb_dr7_i386, // DR7
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, // GDTR
    LLDB_INVALID_REGNUM, // GDTL
    LLDB_INVALID_REGNUM, // IDTR
    LLDB_INVALID_REGNUM, // IDTL
    LLDB_INVALID_REGNUM, // LDTR
    LLDB_INVALID_REGNUM, // TR
    LLDB_INVALID_REGNUM, // PSEUDO1
    LLDB_INVALID_REGNUM, // PSEUDO2
    LLDB_INVALID_REGNUM, // PSEUDO3
    LLDB_INVALID_REGNUM, // PSEUDO4
````
- **L265 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L265 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L266 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L266 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L267 EN**: Continues the surrounding declaration or expression: `lldb_dr0_i386, // DR0`.
  **L267 CN**: 继续构造周围的声明或表达式：`lldb_dr0_i386, // DR0`。
- **L268 EN**: Continues the surrounding declaration or expression: `lldb_dr1_i386, // DR1`.
  **L268 CN**: 继续构造周围的声明或表达式：`lldb_dr1_i386, // DR1`。
- **L269 EN**: Continues the surrounding declaration or expression: `lldb_dr2_i386, // DR2`.
  **L269 CN**: 继续构造周围的声明或表达式：`lldb_dr2_i386, // DR2`。
- **L270 EN**: Continues the surrounding declaration or expression: `lldb_dr3_i386, // DR3`.
  **L270 CN**: 继续构造周围的声明或表达式：`lldb_dr3_i386, // DR3`。
- **L271 EN**: Continues the surrounding declaration or expression: `lldb_dr4_i386, // DR4`.
  **L271 CN**: 继续构造周围的声明或表达式：`lldb_dr4_i386, // DR4`。
- **L272 EN**: Continues the surrounding declaration or expression: `lldb_dr5_i386, // DR5`.
  **L272 CN**: 继续构造周围的声明或表达式：`lldb_dr5_i386, // DR5`。
- **L273 EN**: Continues the surrounding declaration or expression: `lldb_dr6_i386, // DR6`.
  **L273 CN**: 继续构造周围的声明或表达式：`lldb_dr6_i386, // DR6`。
- **L274 EN**: Continues the surrounding declaration or expression: `lldb_dr7_i386, // DR7`.
  **L274 CN**: 继续构造周围的声明或表达式：`lldb_dr7_i386, // DR7`。
- **L275 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L275 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L276 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L276 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L277 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L277 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L278 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L278 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L279 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // GDTR`.
  **L279 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // GDTR`。
- **L280 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // GDTL`.
  **L280 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // GDTL`。
- **L281 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // IDTR`.
  **L281 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // IDTR`。
- **L282 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // IDTL`.
  **L282 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // IDTL`。
- **L283 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // LDTR`.
  **L283 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // LDTR`。
- **L284 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // TR`.
  **L284 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // TR`。
- **L285 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO1`.
  **L285 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO1`。
- **L286 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO2`.
  **L286 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO2`。
- **L287 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO3`.
  **L287 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO3`。
- **L288 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO4`.
  **L288 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO4`。

### Lines 289-312 / 第 289-312 行

````cpp
    LLDB_INVALID_REGNUM, // PSEUDO5
    LLDB_INVALID_REGNUM, // PSEUDO6
    LLDB_INVALID_REGNUM, // PSEUDO7
    LLDB_INVALID_REGNUM, // PSEUDO8
    LLDB_INVALID_REGNUM, // PSEUDO9
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    lldb_st0_i386,       // ST0
    lldb_st1_i386,       // ST1
    lldb_st2_i386,       // ST2
    lldb_st3_i386,       // ST3
    lldb_st4_i386,       // ST4
    lldb_st5_i386,       // ST5
    lldb_st6_i386,       // ST6
    lldb_st7_i386,       // ST7
    LLDB_INVALID_REGNUM, // CTRL
    LLDB_INVALID_REGNUM, // STAT
    LLDB_INVALID_REGNUM, // TAG
    LLDB_INVALID_REGNUM, // FPIP
    LLDB_INVALID_REGNUM, // FPCS
    LLDB_INVALID_REGNUM, // FPDO
    LLDB_INVALID_REGNUM, // FPDS
    LLDB_INVALID_REGNUM, // ISEM
    LLDB_INVALID_REGNUM, // FPEIP
    LLDB_INVALID_REGNUM, // FPEDO
````
- **L289 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO5`.
  **L289 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO5`。
- **L290 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO6`.
  **L290 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO6`。
- **L291 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO7`.
  **L291 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO7`。
- **L292 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO8`.
  **L292 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO8`。
- **L293 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO9`.
  **L293 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO9`。
- **L294 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L294 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L295 EN**: Continues the surrounding declaration or expression: `lldb_st0_i386,       // ST0`.
  **L295 CN**: 继续构造周围的声明或表达式：`lldb_st0_i386,       // ST0`。
- **L296 EN**: Continues the surrounding declaration or expression: `lldb_st1_i386,       // ST1`.
  **L296 CN**: 继续构造周围的声明或表达式：`lldb_st1_i386,       // ST1`。
- **L297 EN**: Continues the surrounding declaration or expression: `lldb_st2_i386,       // ST2`.
  **L297 CN**: 继续构造周围的声明或表达式：`lldb_st2_i386,       // ST2`。
- **L298 EN**: Continues the surrounding declaration or expression: `lldb_st3_i386,       // ST3`.
  **L298 CN**: 继续构造周围的声明或表达式：`lldb_st3_i386,       // ST3`。
- **L299 EN**: Continues the surrounding declaration or expression: `lldb_st4_i386,       // ST4`.
  **L299 CN**: 继续构造周围的声明或表达式：`lldb_st4_i386,       // ST4`。
- **L300 EN**: Continues the surrounding declaration or expression: `lldb_st5_i386,       // ST5`.
  **L300 CN**: 继续构造周围的声明或表达式：`lldb_st5_i386,       // ST5`。
- **L301 EN**: Continues the surrounding declaration or expression: `lldb_st6_i386,       // ST6`.
  **L301 CN**: 继续构造周围的声明或表达式：`lldb_st6_i386,       // ST6`。
- **L302 EN**: Continues the surrounding declaration or expression: `lldb_st7_i386,       // ST7`.
  **L302 CN**: 继续构造周围的声明或表达式：`lldb_st7_i386,       // ST7`。
- **L303 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CTRL`.
  **L303 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CTRL`。
- **L304 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // STAT`.
  **L304 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // STAT`。
- **L305 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // TAG`.
  **L305 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // TAG`。
- **L306 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPIP`.
  **L306 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPIP`。
- **L307 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPCS`.
  **L307 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPCS`。
- **L308 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPDO`.
  **L308 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPDO`。
- **L309 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPDS`.
  **L309 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPDS`。
- **L310 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // ISEM`.
  **L310 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // ISEM`。
- **L311 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPEIP`.
  **L311 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPEIP`。
- **L312 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPEDO`.
  **L312 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPEDO`。

### Lines 313-336 / 第 313-336 行

````cpp
    lldb_mm0_i386,       // MM0
    lldb_mm1_i386,       // MM1
    lldb_mm2_i386,       // MM2
    lldb_mm3_i386,       // MM3
    lldb_mm4_i386,       // MM4
    lldb_mm5_i386,       // MM5
    lldb_mm6_i386,       // MM6
    lldb_mm7_i386,       // MM7
    lldb_xmm0_i386,      // XMM0
    lldb_xmm1_i386,      // XMM1
    lldb_xmm2_i386,      // XMM2
    lldb_xmm3_i386,      // XMM3
    lldb_xmm4_i386,      // XMM4
    lldb_xmm5_i386,      // XMM5
    lldb_xmm6_i386,      // XMM6
    lldb_xmm7_i386       // XMM7
};

static const uint32_t g_code_view_to_lldb_registers_x86_64[] = {
    LLDB_INVALID_REGNUM, // NONE
    lldb_al_x86_64,      // AL
    lldb_cl_x86_64,      // CL
    lldb_dl_x86_64,      // DL
    lldb_bl_x86_64,      // BL
````
- **L313 EN**: Continues the surrounding declaration or expression: `lldb_mm0_i386,       // MM0`.
  **L313 CN**: 继续构造周围的声明或表达式：`lldb_mm0_i386,       // MM0`。
- **L314 EN**: Continues the surrounding declaration or expression: `lldb_mm1_i386,       // MM1`.
  **L314 CN**: 继续构造周围的声明或表达式：`lldb_mm1_i386,       // MM1`。
- **L315 EN**: Continues the surrounding declaration or expression: `lldb_mm2_i386,       // MM2`.
  **L315 CN**: 继续构造周围的声明或表达式：`lldb_mm2_i386,       // MM2`。
- **L316 EN**: Continues the surrounding declaration or expression: `lldb_mm3_i386,       // MM3`.
  **L316 CN**: 继续构造周围的声明或表达式：`lldb_mm3_i386,       // MM3`。
- **L317 EN**: Continues the surrounding declaration or expression: `lldb_mm4_i386,       // MM4`.
  **L317 CN**: 继续构造周围的声明或表达式：`lldb_mm4_i386,       // MM4`。
- **L318 EN**: Continues the surrounding declaration or expression: `lldb_mm5_i386,       // MM5`.
  **L318 CN**: 继续构造周围的声明或表达式：`lldb_mm5_i386,       // MM5`。
- **L319 EN**: Continues the surrounding declaration or expression: `lldb_mm6_i386,       // MM6`.
  **L319 CN**: 继续构造周围的声明或表达式：`lldb_mm6_i386,       // MM6`。
- **L320 EN**: Continues the surrounding declaration or expression: `lldb_mm7_i386,       // MM7`.
  **L320 CN**: 继续构造周围的声明或表达式：`lldb_mm7_i386,       // MM7`。
- **L321 EN**: Continues the surrounding declaration or expression: `lldb_xmm0_i386,      // XMM0`.
  **L321 CN**: 继续构造周围的声明或表达式：`lldb_xmm0_i386,      // XMM0`。
- **L322 EN**: Continues the surrounding declaration or expression: `lldb_xmm1_i386,      // XMM1`.
  **L322 CN**: 继续构造周围的声明或表达式：`lldb_xmm1_i386,      // XMM1`。
- **L323 EN**: Continues the surrounding declaration or expression: `lldb_xmm2_i386,      // XMM2`.
  **L323 CN**: 继续构造周围的声明或表达式：`lldb_xmm2_i386,      // XMM2`。
- **L324 EN**: Continues the surrounding declaration or expression: `lldb_xmm3_i386,      // XMM3`.
  **L324 CN**: 继续构造周围的声明或表达式：`lldb_xmm3_i386,      // XMM3`。
- **L325 EN**: Continues the surrounding declaration or expression: `lldb_xmm4_i386,      // XMM4`.
  **L325 CN**: 继续构造周围的声明或表达式：`lldb_xmm4_i386,      // XMM4`。
- **L326 EN**: Continues the surrounding declaration or expression: `lldb_xmm5_i386,      // XMM5`.
  **L326 CN**: 继续构造周围的声明或表达式：`lldb_xmm5_i386,      // XMM5`。
- **L327 EN**: Continues the surrounding declaration or expression: `lldb_xmm6_i386,      // XMM6`.
  **L327 CN**: 继续构造周围的声明或表达式：`lldb_xmm6_i386,      // XMM6`。
- **L328 EN**: Continues the surrounding declaration or expression: `lldb_xmm7_i386       // XMM7`.
  **L328 CN**: 继续构造周围的声明或表达式：`lldb_xmm7_i386       // XMM7`。
- **L329 EN**: Closes the current declaration scope such as a class or struct.
  **L329 CN**: 结束当前声明作用域，例如类或结构体。
- **L330 EN**: Blank line separates nearby declarations or logic blocks.
  **L330 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L331 EN**: Continues the surrounding declaration or expression: `static const uint32_t g_code_view_to_lldb_registers_x86_64[] = {`.
  **L331 CN**: 继续构造周围的声明或表达式：`static const uint32_t g_code_view_to_lldb_registers_x86_64[] = {`。
- **L332 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // NONE`.
  **L332 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // NONE`。
- **L333 EN**: Continues the surrounding declaration or expression: `lldb_al_x86_64,      // AL`.
  **L333 CN**: 继续构造周围的声明或表达式：`lldb_al_x86_64,      // AL`。
- **L334 EN**: Continues the surrounding declaration or expression: `lldb_cl_x86_64,      // CL`.
  **L334 CN**: 继续构造周围的声明或表达式：`lldb_cl_x86_64,      // CL`。
- **L335 EN**: Continues the surrounding declaration or expression: `lldb_dl_x86_64,      // DL`.
  **L335 CN**: 继续构造周围的声明或表达式：`lldb_dl_x86_64,      // DL`。
- **L336 EN**: Continues the surrounding declaration or expression: `lldb_bl_x86_64,      // BL`.
  **L336 CN**: 继续构造周围的声明或表达式：`lldb_bl_x86_64,      // BL`。

### Lines 337-360 / 第 337-360 行

````cpp
    lldb_ah_x86_64,      // AH
    lldb_ch_x86_64,      // CH
    lldb_dh_x86_64,      // DH
    lldb_bh_x86_64,      // BH
    lldb_ax_x86_64,      // AX
    lldb_cx_x86_64,      // CX
    lldb_dx_x86_64,      // DX
    lldb_bx_x86_64,      // BX
    lldb_sp_x86_64,      // SP
    lldb_bp_x86_64,      // BP
    lldb_si_x86_64,      // SI
    lldb_di_x86_64,      // DI
    lldb_eax_x86_64,     // EAX
    lldb_ecx_x86_64,     // ECX
    lldb_edx_x86_64,     // EDX
    lldb_ebx_x86_64,     // EBX
    lldb_esp_x86_64,     // ESP
    lldb_ebp_x86_64,     // EBP
    lldb_esi_x86_64,     // ESI
    lldb_edi_x86_64,     // EDI
    lldb_es_x86_64,      // ES
    lldb_cs_x86_64,      // CS
    lldb_ss_x86_64,      // SS
    lldb_ds_x86_64,      // DS
````
- **L337 EN**: Continues the surrounding declaration or expression: `lldb_ah_x86_64,      // AH`.
  **L337 CN**: 继续构造周围的声明或表达式：`lldb_ah_x86_64,      // AH`。
- **L338 EN**: Continues the surrounding declaration or expression: `lldb_ch_x86_64,      // CH`.
  **L338 CN**: 继续构造周围的声明或表达式：`lldb_ch_x86_64,      // CH`。
- **L339 EN**: Continues the surrounding declaration or expression: `lldb_dh_x86_64,      // DH`.
  **L339 CN**: 继续构造周围的声明或表达式：`lldb_dh_x86_64,      // DH`。
- **L340 EN**: Continues the surrounding declaration or expression: `lldb_bh_x86_64,      // BH`.
  **L340 CN**: 继续构造周围的声明或表达式：`lldb_bh_x86_64,      // BH`。
- **L341 EN**: Continues the surrounding declaration or expression: `lldb_ax_x86_64,      // AX`.
  **L341 CN**: 继续构造周围的声明或表达式：`lldb_ax_x86_64,      // AX`。
- **L342 EN**: Continues the surrounding declaration or expression: `lldb_cx_x86_64,      // CX`.
  **L342 CN**: 继续构造周围的声明或表达式：`lldb_cx_x86_64,      // CX`。
- **L343 EN**: Continues the surrounding declaration or expression: `lldb_dx_x86_64,      // DX`.
  **L343 CN**: 继续构造周围的声明或表达式：`lldb_dx_x86_64,      // DX`。
- **L344 EN**: Continues the surrounding declaration or expression: `lldb_bx_x86_64,      // BX`.
  **L344 CN**: 继续构造周围的声明或表达式：`lldb_bx_x86_64,      // BX`。
- **L345 EN**: Continues the surrounding declaration or expression: `lldb_sp_x86_64,      // SP`.
  **L345 CN**: 继续构造周围的声明或表达式：`lldb_sp_x86_64,      // SP`。
- **L346 EN**: Continues the surrounding declaration or expression: `lldb_bp_x86_64,      // BP`.
  **L346 CN**: 继续构造周围的声明或表达式：`lldb_bp_x86_64,      // BP`。
- **L347 EN**: Continues the surrounding declaration or expression: `lldb_si_x86_64,      // SI`.
  **L347 CN**: 继续构造周围的声明或表达式：`lldb_si_x86_64,      // SI`。
- **L348 EN**: Continues the surrounding declaration or expression: `lldb_di_x86_64,      // DI`.
  **L348 CN**: 继续构造周围的声明或表达式：`lldb_di_x86_64,      // DI`。
- **L349 EN**: Continues the surrounding declaration or expression: `lldb_eax_x86_64,     // EAX`.
  **L349 CN**: 继续构造周围的声明或表达式：`lldb_eax_x86_64,     // EAX`。
- **L350 EN**: Continues the surrounding declaration or expression: `lldb_ecx_x86_64,     // ECX`.
  **L350 CN**: 继续构造周围的声明或表达式：`lldb_ecx_x86_64,     // ECX`。
- **L351 EN**: Continues the surrounding declaration or expression: `lldb_edx_x86_64,     // EDX`.
  **L351 CN**: 继续构造周围的声明或表达式：`lldb_edx_x86_64,     // EDX`。
- **L352 EN**: Continues the surrounding declaration or expression: `lldb_ebx_x86_64,     // EBX`.
  **L352 CN**: 继续构造周围的声明或表达式：`lldb_ebx_x86_64,     // EBX`。
- **L353 EN**: Continues the surrounding declaration or expression: `lldb_esp_x86_64,     // ESP`.
  **L353 CN**: 继续构造周围的声明或表达式：`lldb_esp_x86_64,     // ESP`。
- **L354 EN**: Continues the surrounding declaration or expression: `lldb_ebp_x86_64,     // EBP`.
  **L354 CN**: 继续构造周围的声明或表达式：`lldb_ebp_x86_64,     // EBP`。
- **L355 EN**: Continues the surrounding declaration or expression: `lldb_esi_x86_64,     // ESI`.
  **L355 CN**: 继续构造周围的声明或表达式：`lldb_esi_x86_64,     // ESI`。
- **L356 EN**: Continues the surrounding declaration or expression: `lldb_edi_x86_64,     // EDI`.
  **L356 CN**: 继续构造周围的声明或表达式：`lldb_edi_x86_64,     // EDI`。
- **L357 EN**: Continues the surrounding declaration or expression: `lldb_es_x86_64,      // ES`.
  **L357 CN**: 继续构造周围的声明或表达式：`lldb_es_x86_64,      // ES`。
- **L358 EN**: Continues the surrounding declaration or expression: `lldb_cs_x86_64,      // CS`.
  **L358 CN**: 继续构造周围的声明或表达式：`lldb_cs_x86_64,      // CS`。
- **L359 EN**: Continues the surrounding declaration or expression: `lldb_ss_x86_64,      // SS`.
  **L359 CN**: 继续构造周围的声明或表达式：`lldb_ss_x86_64,      // SS`。
- **L360 EN**: Continues the surrounding declaration or expression: `lldb_ds_x86_64,      // DS`.
  **L360 CN**: 继续构造周围的声明或表达式：`lldb_ds_x86_64,      // DS`。

### Lines 361-384 / 第 361-384 行

````cpp
    lldb_fs_x86_64,      // FS
    lldb_gs_x86_64,      // GS
    LLDB_INVALID_REGNUM, // IP
    LLDB_INVALID_REGNUM, // FLAGS
    LLDB_INVALID_REGNUM, // EIP
    LLDB_INVALID_REGNUM, // EFLAGS
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, // TEMP
    LLDB_INVALID_REGNUM, // TEMPH
    LLDB_INVALID_REGNUM, // QUOTE
    LLDB_INVALID_REGNUM, // PCDR3
    LLDB_INVALID_REGNUM, // PCDR4
    LLDB_INVALID_REGNUM, // PCDR5
    LLDB_INVALID_REGNUM, // PCDR6
    LLDB_INVALID_REGNUM, // PCDR7
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
````
- **L361 EN**: Continues the surrounding declaration or expression: `lldb_fs_x86_64,      // FS`.
  **L361 CN**: 继续构造周围的声明或表达式：`lldb_fs_x86_64,      // FS`。
- **L362 EN**: Continues the surrounding declaration or expression: `lldb_gs_x86_64,      // GS`.
  **L362 CN**: 继续构造周围的声明或表达式：`lldb_gs_x86_64,      // GS`。
- **L363 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // IP`.
  **L363 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // IP`。
- **L364 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FLAGS`.
  **L364 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FLAGS`。
- **L365 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EIP`.
  **L365 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EIP`。
- **L366 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EFLAGS`.
  **L366 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EFLAGS`。
- **L367 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L367 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L368 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L368 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L369 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // TEMP`.
  **L369 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // TEMP`。
- **L370 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // TEMPH`.
  **L370 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // TEMPH`。
- **L371 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // QUOTE`.
  **L371 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // QUOTE`。
- **L372 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR3`.
  **L372 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR3`。
- **L373 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR4`.
  **L373 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR4`。
- **L374 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR5`.
  **L374 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR5`。
- **L375 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR6`.
  **L375 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR6`。
- **L376 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PCDR7`.
  **L376 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PCDR7`。
- **L377 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L377 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L378 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L378 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L379 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L379 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L380 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L380 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L381 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L381 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L382 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L382 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L383 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L383 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L384 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L384 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。

### Lines 385-408 / 第 385-408 行

````cpp
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, // CR0
    LLDB_INVALID_REGNUM, // CR1
    LLDB_INVALID_REGNUM, // CR2
    LLDB_INVALID_REGNUM, // CR3
    LLDB_INVALID_REGNUM, // CR4
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    lldb_dr0_x86_64, // DR0
    lldb_dr1_x86_64, // DR1
    lldb_dr2_x86_64, // DR2
    lldb_dr3_x86_64, // DR3
    lldb_dr4_x86_64, // DR4
    lldb_dr5_x86_64, // DR5
    lldb_dr6_x86_64, // DR6
    lldb_dr7_x86_64, // DR7
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, // GDTR
    LLDB_INVALID_REGNUM, // GDTL
````
- **L385 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L385 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L386 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L386 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L387 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L387 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L388 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR0`.
  **L388 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR0`。
- **L389 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR1`.
  **L389 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR1`。
- **L390 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR2`.
  **L390 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR2`。
- **L391 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR3`.
  **L391 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR3`。
- **L392 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CR4`.
  **L392 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CR4`。
- **L393 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L393 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L394 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L394 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L395 EN**: Continues the surrounding declaration or expression: `lldb_dr0_x86_64, // DR0`.
  **L395 CN**: 继续构造周围的声明或表达式：`lldb_dr0_x86_64, // DR0`。
- **L396 EN**: Continues the surrounding declaration or expression: `lldb_dr1_x86_64, // DR1`.
  **L396 CN**: 继续构造周围的声明或表达式：`lldb_dr1_x86_64, // DR1`。
- **L397 EN**: Continues the surrounding declaration or expression: `lldb_dr2_x86_64, // DR2`.
  **L397 CN**: 继续构造周围的声明或表达式：`lldb_dr2_x86_64, // DR2`。
- **L398 EN**: Continues the surrounding declaration or expression: `lldb_dr3_x86_64, // DR3`.
  **L398 CN**: 继续构造周围的声明或表达式：`lldb_dr3_x86_64, // DR3`。
- **L399 EN**: Continues the surrounding declaration or expression: `lldb_dr4_x86_64, // DR4`.
  **L399 CN**: 继续构造周围的声明或表达式：`lldb_dr4_x86_64, // DR4`。
- **L400 EN**: Continues the surrounding declaration or expression: `lldb_dr5_x86_64, // DR5`.
  **L400 CN**: 继续构造周围的声明或表达式：`lldb_dr5_x86_64, // DR5`。
- **L401 EN**: Continues the surrounding declaration or expression: `lldb_dr6_x86_64, // DR6`.
  **L401 CN**: 继续构造周围的声明或表达式：`lldb_dr6_x86_64, // DR6`。
- **L402 EN**: Continues the surrounding declaration or expression: `lldb_dr7_x86_64, // DR7`.
  **L402 CN**: 继续构造周围的声明或表达式：`lldb_dr7_x86_64, // DR7`。
- **L403 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L403 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L404 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L404 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L405 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L405 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L406 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L406 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L407 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // GDTR`.
  **L407 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // GDTR`。
- **L408 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // GDTL`.
  **L408 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // GDTL`。

### Lines 409-432 / 第 409-432 行

````cpp
    LLDB_INVALID_REGNUM, // IDTR
    LLDB_INVALID_REGNUM, // IDTL
    LLDB_INVALID_REGNUM, // LDTR
    LLDB_INVALID_REGNUM, // TR
    LLDB_INVALID_REGNUM, // PSEUDO1
    LLDB_INVALID_REGNUM, // PSEUDO2
    LLDB_INVALID_REGNUM, // PSEUDO3
    LLDB_INVALID_REGNUM, // PSEUDO4
    LLDB_INVALID_REGNUM, // PSEUDO5
    LLDB_INVALID_REGNUM, // PSEUDO6
    LLDB_INVALID_REGNUM, // PSEUDO7
    LLDB_INVALID_REGNUM, // PSEUDO8
    LLDB_INVALID_REGNUM, // PSEUDO9
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    lldb_st0_x86_64,     // ST0
    lldb_st1_x86_64,     // ST1
    lldb_st2_x86_64,     // ST2
    lldb_st3_x86_64,     // ST3
    lldb_st4_x86_64,     // ST4
    lldb_st5_x86_64,     // ST5
    lldb_st6_x86_64,     // ST6
    lldb_st7_x86_64,     // ST7
    LLDB_INVALID_REGNUM, // CTRL
    LLDB_INVALID_REGNUM, // STAT
````
- **L409 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // IDTR`.
  **L409 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // IDTR`。
- **L410 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // IDTL`.
  **L410 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // IDTL`。
- **L411 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // LDTR`.
  **L411 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // LDTR`。
- **L412 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // TR`.
  **L412 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // TR`。
- **L413 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO1`.
  **L413 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO1`。
- **L414 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO2`.
  **L414 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO2`。
- **L415 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO3`.
  **L415 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO3`。
- **L416 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO4`.
  **L416 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO4`。
- **L417 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO5`.
  **L417 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO5`。
- **L418 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO6`.
  **L418 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO6`。
- **L419 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO7`.
  **L419 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO7`。
- **L420 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO8`.
  **L420 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO8`。
- **L421 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // PSEUDO9`.
  **L421 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // PSEUDO9`。
- **L422 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L422 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L423 EN**: Continues the surrounding declaration or expression: `lldb_st0_x86_64,     // ST0`.
  **L423 CN**: 继续构造周围的声明或表达式：`lldb_st0_x86_64,     // ST0`。
- **L424 EN**: Continues the surrounding declaration or expression: `lldb_st1_x86_64,     // ST1`.
  **L424 CN**: 继续构造周围的声明或表达式：`lldb_st1_x86_64,     // ST1`。
- **L425 EN**: Continues the surrounding declaration or expression: `lldb_st2_x86_64,     // ST2`.
  **L425 CN**: 继续构造周围的声明或表达式：`lldb_st2_x86_64,     // ST2`。
- **L426 EN**: Continues the surrounding declaration or expression: `lldb_st3_x86_64,     // ST3`.
  **L426 CN**: 继续构造周围的声明或表达式：`lldb_st3_x86_64,     // ST3`。
- **L427 EN**: Continues the surrounding declaration or expression: `lldb_st4_x86_64,     // ST4`.
  **L427 CN**: 继续构造周围的声明或表达式：`lldb_st4_x86_64,     // ST4`。
- **L428 EN**: Continues the surrounding declaration or expression: `lldb_st5_x86_64,     // ST5`.
  **L428 CN**: 继续构造周围的声明或表达式：`lldb_st5_x86_64,     // ST5`。
- **L429 EN**: Continues the surrounding declaration or expression: `lldb_st6_x86_64,     // ST6`.
  **L429 CN**: 继续构造周围的声明或表达式：`lldb_st6_x86_64,     // ST6`。
- **L430 EN**: Continues the surrounding declaration or expression: `lldb_st7_x86_64,     // ST7`.
  **L430 CN**: 继续构造周围的声明或表达式：`lldb_st7_x86_64,     // ST7`。
- **L431 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // CTRL`.
  **L431 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // CTRL`。
- **L432 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // STAT`.
  **L432 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // STAT`。

### Lines 433-456 / 第 433-456 行

````cpp
    LLDB_INVALID_REGNUM, // TAG
    LLDB_INVALID_REGNUM, // FPIP
    LLDB_INVALID_REGNUM, // FPCS
    LLDB_INVALID_REGNUM, // FPDO
    LLDB_INVALID_REGNUM, // FPDS
    LLDB_INVALID_REGNUM, // ISEM
    LLDB_INVALID_REGNUM, // FPEIP
    LLDB_INVALID_REGNUM, // FPEDO
    lldb_mm0_x86_64,     // MM0
    lldb_mm1_x86_64,     // MM1
    lldb_mm2_x86_64,     // MM2
    lldb_mm3_x86_64,     // MM3
    lldb_mm4_x86_64,     // MM4
    lldb_mm5_x86_64,     // MM5
    lldb_mm6_x86_64,     // MM6
    lldb_mm7_x86_64,     // MM7
    lldb_xmm0_x86_64,    // XMM0
    lldb_xmm1_x86_64,    // XMM1
    lldb_xmm2_x86_64,    // XMM2
    lldb_xmm3_x86_64,    // XMM3
    lldb_xmm4_x86_64,    // XMM4
    lldb_xmm5_x86_64,    // XMM5
    lldb_xmm6_x86_64,    // XMM6
    lldb_xmm7_x86_64,    // XMM7
````
- **L433 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // TAG`.
  **L433 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // TAG`。
- **L434 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPIP`.
  **L434 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPIP`。
- **L435 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPCS`.
  **L435 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPCS`。
- **L436 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPDO`.
  **L436 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPDO`。
- **L437 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPDS`.
  **L437 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPDS`。
- **L438 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // ISEM`.
  **L438 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // ISEM`。
- **L439 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPEIP`.
  **L439 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPEIP`。
- **L440 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // FPEDO`.
  **L440 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // FPEDO`。
- **L441 EN**: Continues the surrounding declaration or expression: `lldb_mm0_x86_64,     // MM0`.
  **L441 CN**: 继续构造周围的声明或表达式：`lldb_mm0_x86_64,     // MM0`。
- **L442 EN**: Continues the surrounding declaration or expression: `lldb_mm1_x86_64,     // MM1`.
  **L442 CN**: 继续构造周围的声明或表达式：`lldb_mm1_x86_64,     // MM1`。
- **L443 EN**: Continues the surrounding declaration or expression: `lldb_mm2_x86_64,     // MM2`.
  **L443 CN**: 继续构造周围的声明或表达式：`lldb_mm2_x86_64,     // MM2`。
- **L444 EN**: Continues the surrounding declaration or expression: `lldb_mm3_x86_64,     // MM3`.
  **L444 CN**: 继续构造周围的声明或表达式：`lldb_mm3_x86_64,     // MM3`。
- **L445 EN**: Continues the surrounding declaration or expression: `lldb_mm4_x86_64,     // MM4`.
  **L445 CN**: 继续构造周围的声明或表达式：`lldb_mm4_x86_64,     // MM4`。
- **L446 EN**: Continues the surrounding declaration or expression: `lldb_mm5_x86_64,     // MM5`.
  **L446 CN**: 继续构造周围的声明或表达式：`lldb_mm5_x86_64,     // MM5`。
- **L447 EN**: Continues the surrounding declaration or expression: `lldb_mm6_x86_64,     // MM6`.
  **L447 CN**: 继续构造周围的声明或表达式：`lldb_mm6_x86_64,     // MM6`。
- **L448 EN**: Continues the surrounding declaration or expression: `lldb_mm7_x86_64,     // MM7`.
  **L448 CN**: 继续构造周围的声明或表达式：`lldb_mm7_x86_64,     // MM7`。
- **L449 EN**: Continues the surrounding declaration or expression: `lldb_xmm0_x86_64,    // XMM0`.
  **L449 CN**: 继续构造周围的声明或表达式：`lldb_xmm0_x86_64,    // XMM0`。
- **L450 EN**: Continues the surrounding declaration or expression: `lldb_xmm1_x86_64,    // XMM1`.
  **L450 CN**: 继续构造周围的声明或表达式：`lldb_xmm1_x86_64,    // XMM1`。
- **L451 EN**: Continues the surrounding declaration or expression: `lldb_xmm2_x86_64,    // XMM2`.
  **L451 CN**: 继续构造周围的声明或表达式：`lldb_xmm2_x86_64,    // XMM2`。
- **L452 EN**: Continues the surrounding declaration or expression: `lldb_xmm3_x86_64,    // XMM3`.
  **L452 CN**: 继续构造周围的声明或表达式：`lldb_xmm3_x86_64,    // XMM3`。
- **L453 EN**: Continues the surrounding declaration or expression: `lldb_xmm4_x86_64,    // XMM4`.
  **L453 CN**: 继续构造周围的声明或表达式：`lldb_xmm4_x86_64,    // XMM4`。
- **L454 EN**: Continues the surrounding declaration or expression: `lldb_xmm5_x86_64,    // XMM5`.
  **L454 CN**: 继续构造周围的声明或表达式：`lldb_xmm5_x86_64,    // XMM5`。
- **L455 EN**: Continues the surrounding declaration or expression: `lldb_xmm6_x86_64,    // XMM6`.
  **L455 CN**: 继续构造周围的声明或表达式：`lldb_xmm6_x86_64,    // XMM6`。
- **L456 EN**: Continues the surrounding declaration or expression: `lldb_xmm7_x86_64,    // XMM7`.
  **L456 CN**: 继续构造周围的声明或表达式：`lldb_xmm7_x86_64,    // XMM7`。

### Lines 457-480 / 第 457-480 行

````cpp
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM,
    lldb_mxcsr_x86_64,   // MXCSR
    LLDB_INVALID_REGNUM, // EDXEAX
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, // EMM0L
    LLDB_INVALID_REGNUM, // EMM1L
````
- **L457 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L457 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L458 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L458 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L459 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L459 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L460 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L460 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L461 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L461 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L462 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L462 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L463 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L463 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L464 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L464 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L465 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L465 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L466 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L466 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L467 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L467 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L468 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L468 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L469 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L469 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L470 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L470 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L471 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L471 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L472 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L472 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L473 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM,`.
  **L473 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM,`。
- **L474 EN**: Continues the surrounding declaration or expression: `lldb_mxcsr_x86_64,   // MXCSR`.
  **L474 CN**: 继续构造周围的声明或表达式：`lldb_mxcsr_x86_64,   // MXCSR`。
- **L475 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EDXEAX`.
  **L475 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EDXEAX`。
- **L476 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L476 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L477 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L477 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L478 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM,`.
  **L478 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM,`。
- **L479 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM0L`.
  **L479 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM0L`。
- **L480 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM1L`.
  **L480 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM1L`。

### Lines 481-504 / 第 481-504 行

````cpp
    LLDB_INVALID_REGNUM, // EMM2L
    LLDB_INVALID_REGNUM, // EMM3L
    LLDB_INVALID_REGNUM, // EMM4L
    LLDB_INVALID_REGNUM, // EMM5L
    LLDB_INVALID_REGNUM, // EMM6L
    LLDB_INVALID_REGNUM, // EMM7L
    LLDB_INVALID_REGNUM, // EMM0H
    LLDB_INVALID_REGNUM, // EMM1H
    LLDB_INVALID_REGNUM, // EMM2H
    LLDB_INVALID_REGNUM, // EMM3H
    LLDB_INVALID_REGNUM, // EMM4H
    LLDB_INVALID_REGNUM, // EMM5H
    LLDB_INVALID_REGNUM, // EMM6H
    LLDB_INVALID_REGNUM, // EMM7H
    LLDB_INVALID_REGNUM, // MM00
    LLDB_INVALID_REGNUM, // MM01
    LLDB_INVALID_REGNUM, // MM10
    LLDB_INVALID_REGNUM, // MM11
    LLDB_INVALID_REGNUM, // MM20
    LLDB_INVALID_REGNUM, // MM21
    LLDB_INVALID_REGNUM, // MM30
    LLDB_INVALID_REGNUM, // MM31
    LLDB_INVALID_REGNUM, // MM40
    LLDB_INVALID_REGNUM, // MM41
````
- **L481 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM2L`.
  **L481 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM2L`。
- **L482 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM3L`.
  **L482 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM3L`。
- **L483 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM4L`.
  **L483 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM4L`。
- **L484 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM5L`.
  **L484 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM5L`。
- **L485 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM6L`.
  **L485 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM6L`。
- **L486 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM7L`.
  **L486 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM7L`。
- **L487 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM0H`.
  **L487 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM0H`。
- **L488 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM1H`.
  **L488 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM1H`。
- **L489 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM2H`.
  **L489 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM2H`。
- **L490 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM3H`.
  **L490 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM3H`。
- **L491 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM4H`.
  **L491 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM4H`。
- **L492 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM5H`.
  **L492 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM5H`。
- **L493 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM6H`.
  **L493 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM6H`。
- **L494 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // EMM7H`.
  **L494 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // EMM7H`。
- **L495 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM00`.
  **L495 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM00`。
- **L496 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM01`.
  **L496 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM01`。
- **L497 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM10`.
  **L497 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM10`。
- **L498 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM11`.
  **L498 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM11`。
- **L499 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM20`.
  **L499 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM20`。
- **L500 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM21`.
  **L500 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM21`。
- **L501 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM30`.
  **L501 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM30`。
- **L502 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM31`.
  **L502 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM31`。
- **L503 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM40`.
  **L503 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM40`。
- **L504 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM41`.
  **L504 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM41`。

### Lines 505-528 / 第 505-528 行

````cpp
    LLDB_INVALID_REGNUM, // MM50
    LLDB_INVALID_REGNUM, // MM51
    LLDB_INVALID_REGNUM, // MM60
    LLDB_INVALID_REGNUM, // MM61
    LLDB_INVALID_REGNUM, // MM70
    LLDB_INVALID_REGNUM, // MM71
    lldb_xmm8_x86_64,    // XMM8
    lldb_xmm9_x86_64,    // XMM9
    lldb_xmm10_x86_64,   // XMM10
    lldb_xmm11_x86_64,   // XMM11
    lldb_xmm12_x86_64,   // XMM12
    lldb_xmm13_x86_64,   // XMM13
    lldb_xmm14_x86_64,   // XMM14
    lldb_xmm15_x86_64,   // XMM15
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
````
- **L505 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM50`.
  **L505 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM50`。
- **L506 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM51`.
  **L506 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM51`。
- **L507 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM60`.
  **L507 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM60`。
- **L508 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM61`.
  **L508 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM61`。
- **L509 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM70`.
  **L509 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM70`。
- **L510 EN**: Continues the surrounding declaration or expression: `LLDB_INVALID_REGNUM, // MM71`.
  **L510 CN**: 继续构造周围的声明或表达式：`LLDB_INVALID_REGNUM, // MM71`。
- **L511 EN**: Continues the surrounding declaration or expression: `lldb_xmm8_x86_64,    // XMM8`.
  **L511 CN**: 继续构造周围的声明或表达式：`lldb_xmm8_x86_64,    // XMM8`。
- **L512 EN**: Continues the surrounding declaration or expression: `lldb_xmm9_x86_64,    // XMM9`.
  **L512 CN**: 继续构造周围的声明或表达式：`lldb_xmm9_x86_64,    // XMM9`。
- **L513 EN**: Continues the surrounding declaration or expression: `lldb_xmm10_x86_64,   // XMM10`.
  **L513 CN**: 继续构造周围的声明或表达式：`lldb_xmm10_x86_64,   // XMM10`。
- **L514 EN**: Continues the surrounding declaration or expression: `lldb_xmm11_x86_64,   // XMM11`.
  **L514 CN**: 继续构造周围的声明或表达式：`lldb_xmm11_x86_64,   // XMM11`。
- **L515 EN**: Continues the surrounding declaration or expression: `lldb_xmm12_x86_64,   // XMM12`.
  **L515 CN**: 继续构造周围的声明或表达式：`lldb_xmm12_x86_64,   // XMM12`。
- **L516 EN**: Continues the surrounding declaration or expression: `lldb_xmm13_x86_64,   // XMM13`.
  **L516 CN**: 继续构造周围的声明或表达式：`lldb_xmm13_x86_64,   // XMM13`。
- **L517 EN**: Continues the surrounding declaration or expression: `lldb_xmm14_x86_64,   // XMM14`.
  **L517 CN**: 继续构造周围的声明或表达式：`lldb_xmm14_x86_64,   // XMM14`。
- **L518 EN**: Continues the surrounding declaration or expression: `lldb_xmm15_x86_64,   // XMM15`.
  **L518 CN**: 继续构造周围的声明或表达式：`lldb_xmm15_x86_64,   // XMM15`。
- **L519 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L519 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L520 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L520 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L521 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L521 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L522 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L522 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L523 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L523 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L524 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L524 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L525 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L525 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L526 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L526 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L527 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L527 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L528 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L528 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。

### Lines 529-552 / 第 529-552 行

````cpp
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM,
    lldb_sil_x86_64,   // SIL
    lldb_dil_x86_64,   // DIL
    lldb_bpl_x86_64,   // BPL
    lldb_spl_x86_64,   // SPL
    lldb_rax_x86_64,   // RAX
    lldb_rbx_x86_64,   // RBX
    lldb_rcx_x86_64,   // RCX
    lldb_rdx_x86_64,   // RDX
    lldb_rsi_x86_64,   // RSI
    lldb_rdi_x86_64,   // RDI
    lldb_rbp_x86_64,   // RBP
    lldb_rsp_x86_64,   // RSP
````
- **L529 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L529 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L530 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L530 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L531 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L531 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L532 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L532 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L533 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L533 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L534 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L534 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L535 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L535 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L536 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L536 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L537 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L537 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L538 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L538 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L539 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L539 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L540 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM,`.
  **L540 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM,`。
- **L541 EN**: Continues the surrounding declaration or expression: `lldb_sil_x86_64,   // SIL`.
  **L541 CN**: 继续构造周围的声明或表达式：`lldb_sil_x86_64,   // SIL`。
- **L542 EN**: Continues the surrounding declaration or expression: `lldb_dil_x86_64,   // DIL`.
  **L542 CN**: 继续构造周围的声明或表达式：`lldb_dil_x86_64,   // DIL`。
- **L543 EN**: Continues the surrounding declaration or expression: `lldb_bpl_x86_64,   // BPL`.
  **L543 CN**: 继续构造周围的声明或表达式：`lldb_bpl_x86_64,   // BPL`。
- **L544 EN**: Continues the surrounding declaration or expression: `lldb_spl_x86_64,   // SPL`.
  **L544 CN**: 继续构造周围的声明或表达式：`lldb_spl_x86_64,   // SPL`。
- **L545 EN**: Continues the surrounding declaration or expression: `lldb_rax_x86_64,   // RAX`.
  **L545 CN**: 继续构造周围的声明或表达式：`lldb_rax_x86_64,   // RAX`。
- **L546 EN**: Continues the surrounding declaration or expression: `lldb_rbx_x86_64,   // RBX`.
  **L546 CN**: 继续构造周围的声明或表达式：`lldb_rbx_x86_64,   // RBX`。
- **L547 EN**: Continues the surrounding declaration or expression: `lldb_rcx_x86_64,   // RCX`.
  **L547 CN**: 继续构造周围的声明或表达式：`lldb_rcx_x86_64,   // RCX`。
- **L548 EN**: Continues the surrounding declaration or expression: `lldb_rdx_x86_64,   // RDX`.
  **L548 CN**: 继续构造周围的声明或表达式：`lldb_rdx_x86_64,   // RDX`。
- **L549 EN**: Continues the surrounding declaration or expression: `lldb_rsi_x86_64,   // RSI`.
  **L549 CN**: 继续构造周围的声明或表达式：`lldb_rsi_x86_64,   // RSI`。
- **L550 EN**: Continues the surrounding declaration or expression: `lldb_rdi_x86_64,   // RDI`.
  **L550 CN**: 继续构造周围的声明或表达式：`lldb_rdi_x86_64,   // RDI`。
- **L551 EN**: Continues the surrounding declaration or expression: `lldb_rbp_x86_64,   // RBP`.
  **L551 CN**: 继续构造周围的声明或表达式：`lldb_rbp_x86_64,   // RBP`。
- **L552 EN**: Continues the surrounding declaration or expression: `lldb_rsp_x86_64,   // RSP`.
  **L552 CN**: 继续构造周围的声明或表达式：`lldb_rsp_x86_64,   // RSP`。

### Lines 553-576 / 第 553-576 行

````cpp
    lldb_r8_x86_64,    // R8
    lldb_r9_x86_64,    // R9
    lldb_r10_x86_64,   // R10
    lldb_r11_x86_64,   // R11
    lldb_r12_x86_64,   // R12
    lldb_r13_x86_64,   // R13
    lldb_r14_x86_64,   // R14
    lldb_r15_x86_64,   // R15
    lldb_r8l_x86_64,   // R8B
    lldb_r9l_x86_64,   // R9B
    lldb_r10l_x86_64,  // R10B
    lldb_r11l_x86_64,  // R11B
    lldb_r12l_x86_64,  // R12B
    lldb_r13l_x86_64,  // R13B
    lldb_r14l_x86_64,  // R14B
    lldb_r15l_x86_64,  // R15B
    lldb_r8w_x86_64,   // R8W
    lldb_r9w_x86_64,   // R9W
    lldb_r10w_x86_64,  // R10W
    lldb_r11w_x86_64,  // R11W
    lldb_r12w_x86_64,  // R12W
    lldb_r13w_x86_64,  // R13W
    lldb_r14w_x86_64,  // R14W
    lldb_r15w_x86_64,  // R15W
````
- **L553 EN**: Continues the surrounding declaration or expression: `lldb_r8_x86_64,    // R8`.
  **L553 CN**: 继续构造周围的声明或表达式：`lldb_r8_x86_64,    // R8`。
- **L554 EN**: Continues the surrounding declaration or expression: `lldb_r9_x86_64,    // R9`.
  **L554 CN**: 继续构造周围的声明或表达式：`lldb_r9_x86_64,    // R9`。
- **L555 EN**: Continues the surrounding declaration or expression: `lldb_r10_x86_64,   // R10`.
  **L555 CN**: 继续构造周围的声明或表达式：`lldb_r10_x86_64,   // R10`。
- **L556 EN**: Continues the surrounding declaration or expression: `lldb_r11_x86_64,   // R11`.
  **L556 CN**: 继续构造周围的声明或表达式：`lldb_r11_x86_64,   // R11`。
- **L557 EN**: Continues the surrounding declaration or expression: `lldb_r12_x86_64,   // R12`.
  **L557 CN**: 继续构造周围的声明或表达式：`lldb_r12_x86_64,   // R12`。
- **L558 EN**: Continues the surrounding declaration or expression: `lldb_r13_x86_64,   // R13`.
  **L558 CN**: 继续构造周围的声明或表达式：`lldb_r13_x86_64,   // R13`。
- **L559 EN**: Continues the surrounding declaration or expression: `lldb_r14_x86_64,   // R14`.
  **L559 CN**: 继续构造周围的声明或表达式：`lldb_r14_x86_64,   // R14`。
- **L560 EN**: Continues the surrounding declaration or expression: `lldb_r15_x86_64,   // R15`.
  **L560 CN**: 继续构造周围的声明或表达式：`lldb_r15_x86_64,   // R15`。
- **L561 EN**: Continues the surrounding declaration or expression: `lldb_r8l_x86_64,   // R8B`.
  **L561 CN**: 继续构造周围的声明或表达式：`lldb_r8l_x86_64,   // R8B`。
- **L562 EN**: Continues the surrounding declaration or expression: `lldb_r9l_x86_64,   // R9B`.
  **L562 CN**: 继续构造周围的声明或表达式：`lldb_r9l_x86_64,   // R9B`。
- **L563 EN**: Continues the surrounding declaration or expression: `lldb_r10l_x86_64,  // R10B`.
  **L563 CN**: 继续构造周围的声明或表达式：`lldb_r10l_x86_64,  // R10B`。
- **L564 EN**: Continues the surrounding declaration or expression: `lldb_r11l_x86_64,  // R11B`.
  **L564 CN**: 继续构造周围的声明或表达式：`lldb_r11l_x86_64,  // R11B`。
- **L565 EN**: Continues the surrounding declaration or expression: `lldb_r12l_x86_64,  // R12B`.
  **L565 CN**: 继续构造周围的声明或表达式：`lldb_r12l_x86_64,  // R12B`。
- **L566 EN**: Continues the surrounding declaration or expression: `lldb_r13l_x86_64,  // R13B`.
  **L566 CN**: 继续构造周围的声明或表达式：`lldb_r13l_x86_64,  // R13B`。
- **L567 EN**: Continues the surrounding declaration or expression: `lldb_r14l_x86_64,  // R14B`.
  **L567 CN**: 继续构造周围的声明或表达式：`lldb_r14l_x86_64,  // R14B`。
- **L568 EN**: Continues the surrounding declaration or expression: `lldb_r15l_x86_64,  // R15B`.
  **L568 CN**: 继续构造周围的声明或表达式：`lldb_r15l_x86_64,  // R15B`。
- **L569 EN**: Continues the surrounding declaration or expression: `lldb_r8w_x86_64,   // R8W`.
  **L569 CN**: 继续构造周围的声明或表达式：`lldb_r8w_x86_64,   // R8W`。
- **L570 EN**: Continues the surrounding declaration or expression: `lldb_r9w_x86_64,   // R9W`.
  **L570 CN**: 继续构造周围的声明或表达式：`lldb_r9w_x86_64,   // R9W`。
- **L571 EN**: Continues the surrounding declaration or expression: `lldb_r10w_x86_64,  // R10W`.
  **L571 CN**: 继续构造周围的声明或表达式：`lldb_r10w_x86_64,  // R10W`。
- **L572 EN**: Continues the surrounding declaration or expression: `lldb_r11w_x86_64,  // R11W`.
  **L572 CN**: 继续构造周围的声明或表达式：`lldb_r11w_x86_64,  // R11W`。
- **L573 EN**: Continues the surrounding declaration or expression: `lldb_r12w_x86_64,  // R12W`.
  **L573 CN**: 继续构造周围的声明或表达式：`lldb_r12w_x86_64,  // R12W`。
- **L574 EN**: Continues the surrounding declaration or expression: `lldb_r13w_x86_64,  // R13W`.
  **L574 CN**: 继续构造周围的声明或表达式：`lldb_r13w_x86_64,  // R13W`。
- **L575 EN**: Continues the surrounding declaration or expression: `lldb_r14w_x86_64,  // R14W`.
  **L575 CN**: 继续构造周围的声明或表达式：`lldb_r14w_x86_64,  // R14W`。
- **L576 EN**: Continues the surrounding declaration or expression: `lldb_r15w_x86_64,  // R15W`.
  **L576 CN**: 继续构造周围的声明或表达式：`lldb_r15w_x86_64,  // R15W`。

### Lines 577-600 / 第 577-600 行

````cpp
    lldb_r8d_x86_64,   // R8D
    lldb_r9d_x86_64,   // R9D
    lldb_r10d_x86_64,  // R10D
    lldb_r11d_x86_64,  // R11D
    lldb_r12d_x86_64,  // R12D
    lldb_r13d_x86_64,  // R13D
    lldb_r14d_x86_64,  // R14D
    lldb_r15d_x86_64,  // R15D
    lldb_ymm0_x86_64,  // AMD64_YMM0
    lldb_ymm1_x86_64,  // AMD64_YMM1
    lldb_ymm2_x86_64,  // AMD64_YMM2
    lldb_ymm3_x86_64,  // AMD64_YMM3
    lldb_ymm4_x86_64,  // AMD64_YMM4
    lldb_ymm5_x86_64,  // AMD64_YMM5
    lldb_ymm6_x86_64,  // AMD64_YMM6
    lldb_ymm7_x86_64,  // AMD64_YMM7
    lldb_ymm8_x86_64,  // AMD64_YMM8
    lldb_ymm9_x86_64,  // AMD64_YMM9
    lldb_ymm10_x86_64, // AMD64_YMM10
    lldb_ymm11_x86_64, // AMD64_YMM11
    lldb_ymm12_x86_64, // AMD64_YMM12
    lldb_ymm13_x86_64, // AMD64_YMM13
    lldb_ymm14_x86_64, // AMD64_YMM14
    lldb_ymm15_x86_64, // AMD64_YMM15
````
- **L577 EN**: Continues the surrounding declaration or expression: `lldb_r8d_x86_64,   // R8D`.
  **L577 CN**: 继续构造周围的声明或表达式：`lldb_r8d_x86_64,   // R8D`。
- **L578 EN**: Continues the surrounding declaration or expression: `lldb_r9d_x86_64,   // R9D`.
  **L578 CN**: 继续构造周围的声明或表达式：`lldb_r9d_x86_64,   // R9D`。
- **L579 EN**: Continues the surrounding declaration or expression: `lldb_r10d_x86_64,  // R10D`.
  **L579 CN**: 继续构造周围的声明或表达式：`lldb_r10d_x86_64,  // R10D`。
- **L580 EN**: Continues the surrounding declaration or expression: `lldb_r11d_x86_64,  // R11D`.
  **L580 CN**: 继续构造周围的声明或表达式：`lldb_r11d_x86_64,  // R11D`。
- **L581 EN**: Continues the surrounding declaration or expression: `lldb_r12d_x86_64,  // R12D`.
  **L581 CN**: 继续构造周围的声明或表达式：`lldb_r12d_x86_64,  // R12D`。
- **L582 EN**: Continues the surrounding declaration or expression: `lldb_r13d_x86_64,  // R13D`.
  **L582 CN**: 继续构造周围的声明或表达式：`lldb_r13d_x86_64,  // R13D`。
- **L583 EN**: Continues the surrounding declaration or expression: `lldb_r14d_x86_64,  // R14D`.
  **L583 CN**: 继续构造周围的声明或表达式：`lldb_r14d_x86_64,  // R14D`。
- **L584 EN**: Continues the surrounding declaration or expression: `lldb_r15d_x86_64,  // R15D`.
  **L584 CN**: 继续构造周围的声明或表达式：`lldb_r15d_x86_64,  // R15D`。
- **L585 EN**: Continues the surrounding declaration or expression: `lldb_ymm0_x86_64,  // AMD64_YMM0`.
  **L585 CN**: 继续构造周围的声明或表达式：`lldb_ymm0_x86_64,  // AMD64_YMM0`。
- **L586 EN**: Continues the surrounding declaration or expression: `lldb_ymm1_x86_64,  // AMD64_YMM1`.
  **L586 CN**: 继续构造周围的声明或表达式：`lldb_ymm1_x86_64,  // AMD64_YMM1`。
- **L587 EN**: Continues the surrounding declaration or expression: `lldb_ymm2_x86_64,  // AMD64_YMM2`.
  **L587 CN**: 继续构造周围的声明或表达式：`lldb_ymm2_x86_64,  // AMD64_YMM2`。
- **L588 EN**: Continues the surrounding declaration or expression: `lldb_ymm3_x86_64,  // AMD64_YMM3`.
  **L588 CN**: 继续构造周围的声明或表达式：`lldb_ymm3_x86_64,  // AMD64_YMM3`。
- **L589 EN**: Continues the surrounding declaration or expression: `lldb_ymm4_x86_64,  // AMD64_YMM4`.
  **L589 CN**: 继续构造周围的声明或表达式：`lldb_ymm4_x86_64,  // AMD64_YMM4`。
- **L590 EN**: Continues the surrounding declaration or expression: `lldb_ymm5_x86_64,  // AMD64_YMM5`.
  **L590 CN**: 继续构造周围的声明或表达式：`lldb_ymm5_x86_64,  // AMD64_YMM5`。
- **L591 EN**: Continues the surrounding declaration or expression: `lldb_ymm6_x86_64,  // AMD64_YMM6`.
  **L591 CN**: 继续构造周围的声明或表达式：`lldb_ymm6_x86_64,  // AMD64_YMM6`。
- **L592 EN**: Continues the surrounding declaration or expression: `lldb_ymm7_x86_64,  // AMD64_YMM7`.
  **L592 CN**: 继续构造周围的声明或表达式：`lldb_ymm7_x86_64,  // AMD64_YMM7`。
- **L593 EN**: Continues the surrounding declaration or expression: `lldb_ymm8_x86_64,  // AMD64_YMM8`.
  **L593 CN**: 继续构造周围的声明或表达式：`lldb_ymm8_x86_64,  // AMD64_YMM8`。
- **L594 EN**: Continues the surrounding declaration or expression: `lldb_ymm9_x86_64,  // AMD64_YMM9`.
  **L594 CN**: 继续构造周围的声明或表达式：`lldb_ymm9_x86_64,  // AMD64_YMM9`。
- **L595 EN**: Continues the surrounding declaration or expression: `lldb_ymm10_x86_64, // AMD64_YMM10`.
  **L595 CN**: 继续构造周围的声明或表达式：`lldb_ymm10_x86_64, // AMD64_YMM10`。
- **L596 EN**: Continues the surrounding declaration or expression: `lldb_ymm11_x86_64, // AMD64_YMM11`.
  **L596 CN**: 继续构造周围的声明或表达式：`lldb_ymm11_x86_64, // AMD64_YMM11`。
- **L597 EN**: Continues the surrounding declaration or expression: `lldb_ymm12_x86_64, // AMD64_YMM12`.
  **L597 CN**: 继续构造周围的声明或表达式：`lldb_ymm12_x86_64, // AMD64_YMM12`。
- **L598 EN**: Continues the surrounding declaration or expression: `lldb_ymm13_x86_64, // AMD64_YMM13`.
  **L598 CN**: 继续构造周围的声明或表达式：`lldb_ymm13_x86_64, // AMD64_YMM13`。
- **L599 EN**: Continues the surrounding declaration or expression: `lldb_ymm14_x86_64, // AMD64_YMM14`.
  **L599 CN**: 继续构造周围的声明或表达式：`lldb_ymm14_x86_64, // AMD64_YMM14`。
- **L600 EN**: Continues the surrounding declaration or expression: `lldb_ymm15_x86_64, // AMD64_YMM15`.
  **L600 CN**: 继续构造周围的声明或表达式：`lldb_ymm15_x86_64, // AMD64_YMM15`。

### Lines 601-624 / 第 601-624 行

````cpp
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,
    lldb_bnd0_x86_64, // BND0
    lldb_bnd1_x86_64, // BND1
    lldb_bnd2_x86_64  // BND2
};

uint32_t lldb_private::npdb::GetLLDBRegisterNumber(
    llvm::Triple::ArchType arch_type, llvm::codeview::RegisterId register_id) {
  switch (arch_type) {
  case llvm::Triple::aarch64:
    if (static_cast<uint16_t>(register_id) <
        sizeof(g_code_view_to_lldb_registers_arm64) /
            sizeof(g_code_view_to_lldb_registers_arm64[0]))
      return g_code_view_to_lldb_registers_arm64[static_cast<uint16_t>(
          register_id)];

    return LLDB_INVALID_REGNUM;
  case llvm::Triple::x86:
    if (static_cast<uint16_t>(register_id) <
        sizeof(g_code_view_to_lldb_registers_x86) /
            sizeof(g_code_view_to_lldb_registers_x86[0]))
````
- **L601 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L601 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L602 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L602 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L603 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L603 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L604 EN**: Continues a multi-line list, initializer, or aggregate entry: `LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`.
  **L604 CN**: 继续一个多行列表、初始化器或聚合项：`LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,`。
- **L605 EN**: Continues the surrounding declaration or expression: `lldb_bnd0_x86_64, // BND0`.
  **L605 CN**: 继续构造周围的声明或表达式：`lldb_bnd0_x86_64, // BND0`。
- **L606 EN**: Continues the surrounding declaration or expression: `lldb_bnd1_x86_64, // BND1`.
  **L606 CN**: 继续构造周围的声明或表达式：`lldb_bnd1_x86_64, // BND1`。
- **L607 EN**: Continues the surrounding declaration or expression: `lldb_bnd2_x86_64  // BND2`.
  **L607 CN**: 继续构造周围的声明或表达式：`lldb_bnd2_x86_64  // BND2`。
- **L608 EN**: Closes the current declaration scope such as a class or struct.
  **L608 CN**: 结束当前声明作用域，例如类或结构体。
- **L609 EN**: Blank line separates nearby declarations or logic blocks.
  **L609 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L610 EN**: Continues logic associated with callable symbol `GetLLDBRegisterNumber`.
  **L610 CN**: 继续与可调用符号 `GetLLDBRegisterNumber` 相关的逻辑。
- **L611 EN**: Continues the surrounding declaration or expression: `llvm::Triple::ArchType arch_type, llvm::codeview::RegisterId register_id) {`.
  **L611 CN**: 继续构造周围的声明或表达式：`llvm::Triple::ArchType arch_type, llvm::codeview::RegisterId register_id) {`。
- **L612 EN**: Begins a `switch` control-flow statement.
  **L612 CN**: 开始一个 `switch` 控制流语句。
- **L613 EN**: Introduces a `switch` dispatch label: `case llvm::Triple::aarch64:`.
  **L613 CN**: 引入一个 `switch` 分发标签：`case llvm::Triple::aarch64:`。
- **L614 EN**: Begins a `if` control-flow statement.
  **L614 CN**: 开始一个 `if` 控制流语句。
- **L615 EN**: Continues the surrounding declaration or expression: `sizeof(g_code_view_to_lldb_registers_arm64) /`.
  **L615 CN**: 继续构造周围的声明或表达式：`sizeof(g_code_view_to_lldb_registers_arm64) /`。
- **L616 EN**: Continues the surrounding declaration or expression: `sizeof(g_code_view_to_lldb_registers_arm64[0]))`.
  **L616 CN**: 继续构造周围的声明或表达式：`sizeof(g_code_view_to_lldb_registers_arm64[0]))`。
- **L617 EN**: Returns from the current function with `g_code_view_to_lldb_registers_arm64[static_cast<uint16_t>(`.
  **L617 CN**: 以 `g_code_view_to_lldb_registers_arm64[static_cast<uint16_t>(` 从当前函数返回。
- **L618 EN**: Completes a standalone declaration or statement: `register_id)];`.
  **L618 CN**: 完成一条独立声明或语句：`register_id)];`。
- **L619 EN**: Blank line separates nearby declarations or logic blocks.
  **L619 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L620 EN**: Returns from the current function with `LLDB_INVALID_REGNUM`.
  **L620 CN**: 以 `LLDB_INVALID_REGNUM` 从当前函数返回。
- **L621 EN**: Introduces a `switch` dispatch label: `case llvm::Triple::x86:`.
  **L621 CN**: 引入一个 `switch` 分发标签：`case llvm::Triple::x86:`。
- **L622 EN**: Begins a `if` control-flow statement.
  **L622 CN**: 开始一个 `if` 控制流语句。
- **L623 EN**: Continues the surrounding declaration or expression: `sizeof(g_code_view_to_lldb_registers_x86) /`.
  **L623 CN**: 继续构造周围的声明或表达式：`sizeof(g_code_view_to_lldb_registers_x86) /`。
- **L624 EN**: Continues the surrounding declaration or expression: `sizeof(g_code_view_to_lldb_registers_x86[0]))`.
  **L624 CN**: 继续构造周围的声明或表达式：`sizeof(g_code_view_to_lldb_registers_x86[0]))`。

### Lines 625-648 / 第 625-648 行

````cpp
      return g_code_view_to_lldb_registers_x86[static_cast<uint16_t>(
          register_id)];

    switch (register_id) {
    case llvm::codeview::RegisterId::MXCSR:
      return lldb_mxcsr_i386;
    case llvm::codeview::RegisterId::BND0:
      return lldb_bnd0_i386;
    case llvm::codeview::RegisterId::BND1:
      return lldb_bnd1_i386;
    case llvm::codeview::RegisterId::BND2:
      return lldb_bnd2_i386;
    default:
      return LLDB_INVALID_REGNUM;
    }
  case llvm::Triple::x86_64:
    if (static_cast<uint16_t>(register_id) <
        sizeof(g_code_view_to_lldb_registers_x86_64) /
            sizeof(g_code_view_to_lldb_registers_x86_64[0]))
      return g_code_view_to_lldb_registers_x86_64[static_cast<uint16_t>(
          register_id)];

    return LLDB_INVALID_REGNUM;
  default:
````
- **L625 EN**: Returns from the current function with `g_code_view_to_lldb_registers_x86[static_cast<uint16_t>(`.
  **L625 CN**: 以 `g_code_view_to_lldb_registers_x86[static_cast<uint16_t>(` 从当前函数返回。
- **L626 EN**: Completes a standalone declaration or statement: `register_id)];`.
  **L626 CN**: 完成一条独立声明或语句：`register_id)];`。
- **L627 EN**: Blank line separates nearby declarations or logic blocks.
  **L627 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L628 EN**: Begins a `switch` control-flow statement.
  **L628 CN**: 开始一个 `switch` 控制流语句。
- **L629 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::MXCSR:`.
  **L629 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::MXCSR:`。
- **L630 EN**: Returns from the current function with `lldb_mxcsr_i386`.
  **L630 CN**: 以 `lldb_mxcsr_i386` 从当前函数返回。
- **L631 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::BND0:`.
  **L631 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::BND0:`。
- **L632 EN**: Returns from the current function with `lldb_bnd0_i386`.
  **L632 CN**: 以 `lldb_bnd0_i386` 从当前函数返回。
- **L633 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::BND1:`.
  **L633 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::BND1:`。
- **L634 EN**: Returns from the current function with `lldb_bnd1_i386`.
  **L634 CN**: 以 `lldb_bnd1_i386` 从当前函数返回。
- **L635 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::BND2:`.
  **L635 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::BND2:`。
- **L636 EN**: Returns from the current function with `lldb_bnd2_i386`.
  **L636 CN**: 以 `lldb_bnd2_i386` 从当前函数返回。
- **L637 EN**: Introduces a `switch` dispatch label: `default:`.
  **L637 CN**: 引入一个 `switch` 分发标签：`default:`。
- **L638 EN**: Returns from the current function with `LLDB_INVALID_REGNUM`.
  **L638 CN**: 以 `LLDB_INVALID_REGNUM` 从当前函数返回。
- **L639 EN**: Closes the current lexical scope or body.
  **L639 CN**: 关闭当前词法作用域或代码体。
- **L640 EN**: Introduces a `switch` dispatch label: `case llvm::Triple::x86_64:`.
  **L640 CN**: 引入一个 `switch` 分发标签：`case llvm::Triple::x86_64:`。
- **L641 EN**: Begins a `if` control-flow statement.
  **L641 CN**: 开始一个 `if` 控制流语句。
- **L642 EN**: Continues the surrounding declaration or expression: `sizeof(g_code_view_to_lldb_registers_x86_64) /`.
  **L642 CN**: 继续构造周围的声明或表达式：`sizeof(g_code_view_to_lldb_registers_x86_64) /`。
- **L643 EN**: Continues the surrounding declaration or expression: `sizeof(g_code_view_to_lldb_registers_x86_64[0]))`.
  **L643 CN**: 继续构造周围的声明或表达式：`sizeof(g_code_view_to_lldb_registers_x86_64[0]))`。
- **L644 EN**: Returns from the current function with `g_code_view_to_lldb_registers_x86_64[static_cast<uint16_t>(`.
  **L644 CN**: 以 `g_code_view_to_lldb_registers_x86_64[static_cast<uint16_t>(` 从当前函数返回。
- **L645 EN**: Completes a standalone declaration or statement: `register_id)];`.
  **L645 CN**: 完成一条独立声明或语句：`register_id)];`。
- **L646 EN**: Blank line separates nearby declarations or logic blocks.
  **L646 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L647 EN**: Returns from the current function with `LLDB_INVALID_REGNUM`.
  **L647 CN**: 以 `LLDB_INVALID_REGNUM` 从当前函数返回。
- **L648 EN**: Introduces a `switch` dispatch label: `default:`.
  **L648 CN**: 引入一个 `switch` 分发标签：`default:`。

### Lines 649-672 / 第 649-672 行

````cpp
    return LLDB_INVALID_REGNUM;
  }
}

uint32_t
lldb_private::npdb::GetRegisterSize(llvm::codeview::RegisterId register_id) {
  switch(register_id) {
    case llvm::codeview::RegisterId::AL:
    case llvm::codeview::RegisterId::BL:
    case llvm::codeview::RegisterId::CL:
    case llvm::codeview::RegisterId::DL:
    case llvm::codeview::RegisterId::AH:
    case llvm::codeview::RegisterId::BH:
    case llvm::codeview::RegisterId::CH:
    case llvm::codeview::RegisterId::DH:
    case llvm::codeview::RegisterId::SIL:
    case llvm::codeview::RegisterId::DIL:
    case llvm::codeview::RegisterId::BPL:
    case llvm::codeview::RegisterId::SPL:
    case llvm::codeview::RegisterId::R8B:
    case llvm::codeview::RegisterId::R9B:
    case llvm::codeview::RegisterId::R10B:
    case llvm::codeview::RegisterId::R11B:
    case llvm::codeview::RegisterId::R12B:
````
- **L649 EN**: Returns from the current function with `LLDB_INVALID_REGNUM`.
  **L649 CN**: 以 `LLDB_INVALID_REGNUM` 从当前函数返回。
- **L650 EN**: Closes the current lexical scope or body.
  **L650 CN**: 关闭当前词法作用域或代码体。
- **L651 EN**: Closes the current lexical scope or body.
  **L651 CN**: 关闭当前词法作用域或代码体。
- **L652 EN**: Blank line separates nearby declarations or logic blocks.
  **L652 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L653 EN**: Continues the surrounding declaration or expression: `uint32_t`.
  **L653 CN**: 继续构造周围的声明或表达式：`uint32_t`。
- **L654 EN**: Starts a function, method, lambda, or structured scope: `lldb_private::npdb::GetRegisterSize(llvm::codeview::RegisterId register_id) {`.
  **L654 CN**: 开始一个函数、方法、lambda 或结构化作用域：`lldb_private::npdb::GetRegisterSize(llvm::codeview::RegisterId register_id) {`。
- **L655 EN**: Begins a `switch` control-flow statement.
  **L655 CN**: 开始一个 `switch` 控制流语句。
- **L656 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::AL:`.
  **L656 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::AL:`。
- **L657 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::BL:`.
  **L657 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::BL:`。
- **L658 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::CL:`.
  **L658 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::CL:`。
- **L659 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::DL:`.
  **L659 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::DL:`。
- **L660 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::AH:`.
  **L660 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::AH:`。
- **L661 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::BH:`.
  **L661 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::BH:`。
- **L662 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::CH:`.
  **L662 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::CH:`。
- **L663 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::DH:`.
  **L663 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::DH:`。
- **L664 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::SIL:`.
  **L664 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::SIL:`。
- **L665 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::DIL:`.
  **L665 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::DIL:`。
- **L666 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::BPL:`.
  **L666 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::BPL:`。
- **L667 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::SPL:`.
  **L667 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::SPL:`。
- **L668 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R8B:`.
  **L668 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R8B:`。
- **L669 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R9B:`.
  **L669 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R9B:`。
- **L670 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R10B:`.
  **L670 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R10B:`。
- **L671 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R11B:`.
  **L671 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R11B:`。
- **L672 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R12B:`.
  **L672 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R12B:`。

### Lines 673-696 / 第 673-696 行

````cpp
    case llvm::codeview::RegisterId::R13B:
    case llvm::codeview::RegisterId::R14B:
    case llvm::codeview::RegisterId::R15B:
      return 1;
    case llvm::codeview::RegisterId::AX:
    case llvm::codeview::RegisterId::BX:
    case llvm::codeview::RegisterId::CX:
    case llvm::codeview::RegisterId::DX:
    case llvm::codeview::RegisterId::SP:
    case llvm::codeview::RegisterId::BP:
    case llvm::codeview::RegisterId::SI:
    case llvm::codeview::RegisterId::DI:
    case llvm::codeview::RegisterId::R8W:
    case llvm::codeview::RegisterId::R9W:
    case llvm::codeview::RegisterId::R10W:
    case llvm::codeview::RegisterId::R11W:
    case llvm::codeview::RegisterId::R12W:
    case llvm::codeview::RegisterId::R13W:
    case llvm::codeview::RegisterId::R14W:
    case llvm::codeview::RegisterId::R15W:
      return 2;
    case llvm::codeview::RegisterId::EAX:
    case llvm::codeview::RegisterId::EBX:
    case llvm::codeview::RegisterId::ECX:
````
- **L673 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R13B:`.
  **L673 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R13B:`。
- **L674 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R14B:`.
  **L674 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R14B:`。
- **L675 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R15B:`.
  **L675 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R15B:`。
- **L676 EN**: Returns from the current function with `1`.
  **L676 CN**: 以 `1` 从当前函数返回。
- **L677 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::AX:`.
  **L677 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::AX:`。
- **L678 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::BX:`.
  **L678 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::BX:`。
- **L679 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::CX:`.
  **L679 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::CX:`。
- **L680 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::DX:`.
  **L680 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::DX:`。
- **L681 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::SP:`.
  **L681 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::SP:`。
- **L682 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::BP:`.
  **L682 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::BP:`。
- **L683 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::SI:`.
  **L683 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::SI:`。
- **L684 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::DI:`.
  **L684 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::DI:`。
- **L685 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R8W:`.
  **L685 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R8W:`。
- **L686 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R9W:`.
  **L686 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R9W:`。
- **L687 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R10W:`.
  **L687 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R10W:`。
- **L688 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R11W:`.
  **L688 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R11W:`。
- **L689 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R12W:`.
  **L689 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R12W:`。
- **L690 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R13W:`.
  **L690 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R13W:`。
- **L691 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R14W:`.
  **L691 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R14W:`。
- **L692 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R15W:`.
  **L692 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R15W:`。
- **L693 EN**: Returns from the current function with `2`.
  **L693 CN**: 以 `2` 从当前函数返回。
- **L694 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::EAX:`.
  **L694 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::EAX:`。
- **L695 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::EBX:`.
  **L695 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::EBX:`。
- **L696 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::ECX:`.
  **L696 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::ECX:`。

### Lines 697-720 / 第 697-720 行

````cpp
    case llvm::codeview::RegisterId::EDX:
    case llvm::codeview::RegisterId::ESP:
    case llvm::codeview::RegisterId::EBP:
    case llvm::codeview::RegisterId::ESI:
    case llvm::codeview::RegisterId::EDI:
    case llvm::codeview::RegisterId::R8D:
    case llvm::codeview::RegisterId::R9D:
    case llvm::codeview::RegisterId::R10D:
    case llvm::codeview::RegisterId::R11D:
    case llvm::codeview::RegisterId::R12D:
    case llvm::codeview::RegisterId::R13D:
    case llvm::codeview::RegisterId::R14D:
    case llvm::codeview::RegisterId::R15D:
      return 4;
    case llvm::codeview::RegisterId::RAX:
    case llvm::codeview::RegisterId::RBX:
    case llvm::codeview::RegisterId::RCX:
    case llvm::codeview::RegisterId::RDX:
    case llvm::codeview::RegisterId::RSI:
    case llvm::codeview::RegisterId::RDI:
    case llvm::codeview::RegisterId::RBP:
    case llvm::codeview::RegisterId::RSP:
    case llvm::codeview::RegisterId::R8:
    case llvm::codeview::RegisterId::R9:
````
- **L697 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::EDX:`.
  **L697 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::EDX:`。
- **L698 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::ESP:`.
  **L698 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::ESP:`。
- **L699 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::EBP:`.
  **L699 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::EBP:`。
- **L700 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::ESI:`.
  **L700 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::ESI:`。
- **L701 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::EDI:`.
  **L701 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::EDI:`。
- **L702 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R8D:`.
  **L702 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R8D:`。
- **L703 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R9D:`.
  **L703 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R9D:`。
- **L704 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R10D:`.
  **L704 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R10D:`。
- **L705 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R11D:`.
  **L705 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R11D:`。
- **L706 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R12D:`.
  **L706 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R12D:`。
- **L707 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R13D:`.
  **L707 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R13D:`。
- **L708 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R14D:`.
  **L708 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R14D:`。
- **L709 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R15D:`.
  **L709 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R15D:`。
- **L710 EN**: Returns from the current function with `4`.
  **L710 CN**: 以 `4` 从当前函数返回。
- **L711 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::RAX:`.
  **L711 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::RAX:`。
- **L712 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::RBX:`.
  **L712 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::RBX:`。
- **L713 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::RCX:`.
  **L713 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::RCX:`。
- **L714 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::RDX:`.
  **L714 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::RDX:`。
- **L715 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::RSI:`.
  **L715 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::RSI:`。
- **L716 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::RDI:`.
  **L716 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::RDI:`。
- **L717 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::RBP:`.
  **L717 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::RBP:`。
- **L718 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::RSP:`.
  **L718 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::RSP:`。
- **L719 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R8:`.
  **L719 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R8:`。
- **L720 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R9:`.
  **L720 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R9:`。

### Lines 721-744 / 第 721-744 行

````cpp
    case llvm::codeview::RegisterId::R10:
    case llvm::codeview::RegisterId::R11:
    case llvm::codeview::RegisterId::R12:
    case llvm::codeview::RegisterId::R13:
    case llvm::codeview::RegisterId::R14:
    case llvm::codeview::RegisterId::R15:
      return 8;
    case llvm::codeview::RegisterId::XMM0:
    case llvm::codeview::RegisterId::XMM1:
    case llvm::codeview::RegisterId::XMM2:
    case llvm::codeview::RegisterId::XMM3:
    case llvm::codeview::RegisterId::XMM4:
    case llvm::codeview::RegisterId::XMM5:
    case llvm::codeview::RegisterId::XMM6:
    case llvm::codeview::RegisterId::XMM7:
    case llvm::codeview::RegisterId::XMM8:
    case llvm::codeview::RegisterId::XMM9:
    case llvm::codeview::RegisterId::XMM10:
    case llvm::codeview::RegisterId::XMM11:
    case llvm::codeview::RegisterId::XMM12:
    case llvm::codeview::RegisterId::XMM13:
    case llvm::codeview::RegisterId::XMM14:
    case llvm::codeview::RegisterId::XMM15:
      return 16;
````
- **L721 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R10:`.
  **L721 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R10:`。
- **L722 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R11:`.
  **L722 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R11:`。
- **L723 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R12:`.
  **L723 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R12:`。
- **L724 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R13:`.
  **L724 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R13:`。
- **L725 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R14:`.
  **L725 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R14:`。
- **L726 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::R15:`.
  **L726 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::R15:`。
- **L727 EN**: Returns from the current function with `8`.
  **L727 CN**: 以 `8` 从当前函数返回。
- **L728 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM0:`.
  **L728 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM0:`。
- **L729 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM1:`.
  **L729 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM1:`。
- **L730 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM2:`.
  **L730 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM2:`。
- **L731 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM3:`.
  **L731 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM3:`。
- **L732 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM4:`.
  **L732 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM4:`。
- **L733 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM5:`.
  **L733 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM5:`。
- **L734 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM6:`.
  **L734 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM6:`。
- **L735 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM7:`.
  **L735 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM7:`。
- **L736 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM8:`.
  **L736 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM8:`。
- **L737 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM9:`.
  **L737 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM9:`。
- **L738 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM10:`.
  **L738 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM10:`。
- **L739 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM11:`.
  **L739 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM11:`。
- **L740 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM12:`.
  **L740 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM12:`。
- **L741 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM13:`.
  **L741 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM13:`。
- **L742 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM14:`.
  **L742 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM14:`。
- **L743 EN**: Introduces a `switch` dispatch label: `case llvm::codeview::RegisterId::XMM15:`.
  **L743 CN**: 引入一个 `switch` 分发标签：`case llvm::codeview::RegisterId::XMM15:`。
- **L744 EN**: Returns from the current function with `16`.
  **L744 CN**: 以 `16` 从当前函数返回。

### Lines 745-748 / 第 745-748 行

````cpp
    default:
      return 0;
  }
}
````
- **L745 EN**: Introduces a `switch` dispatch label: `default:`.
  **L745 CN**: 引入一个 `switch` 分发标签：`default:`。
- **L746 EN**: Returns from the current function with `0`.
  **L746 CN**: 以 `0` 从当前函数返回。
- **L747 EN**: Closes the current lexical scope or body.
  **L747 CN**: 关闭当前词法作用域或代码体。
- **L748 EN**: Closes the current lexical scope or body.
  **L748 CN**: 关闭当前词法作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This file is a implementation unit inside LLDB's **SymbolFile** area. / 该文件是 LLDB **SymbolFile** 范围内的实现文件。
- **Scale / 规模**: 748 lines with 4 direct includes. / 共 748 行，直接包含 4 个头文件。
- **Subsystem focus / 子系统关注点**: DWARF/PDB parsing, compile-unit indexing, symbol and type reconstruction. / DWARF/PDB 解析、编译单元索引、符号与类型重建。
- **Visible entry points / 关键入口**: `lldb_private::npdb::GetRegisterSize`. / 可见的关键入口包括 `lldb_private::npdb::GetRegisterSize`。
- **Concept / 概念**: Process control and state tracking. / 进程控制与状态跟踪。

## Dependencies / 依赖关系

- **LLDB headers / LLDB 头文件**: `lldb/lldb-defines.h`.
- **System/other headers / 系统或其他头文件**: `CodeViewRegisterMapping.h`, `Plugins/Process/Utility/lldb-arm64-register-enums.h`, `Plugins/Process/Utility/lldb-x86-register-enums.h`.
- **Callable interfaces / 可调用接口**: `lldb_private::npdb::GetRegisterSize`.
