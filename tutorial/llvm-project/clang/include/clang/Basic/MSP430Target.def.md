# MSP430Target.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/MSP430Target.def`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: MSP430 Feature/Processor Database *- C++.
- **Purpose (CN)**: 声明与 `MSP430Target` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 662

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===--- MSP430Target.def - MSP430 Feature/Processor Database----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the MSP430 devices and their features.
//
// Generated from TI's devices.csv in version 1.212 using the script in
// Target/MSP430/gen-msp430-def.py - use this tool rather than adding
// new MCUs by hand.
//
//===----------------------------------------------------------------------===//

#ifndef MSP430_MCU_FEAT
#define MSP430_MCU_FEAT(NAME, HWMULT) MSP430_MCU(NAME)
#endif

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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the MSP430 devices and their features.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the MSP430 devices and their features.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Comment explains nearby logic, constraints, or intent: `Generated from TI's devices.csv in version 1.212 using the script in`.
  **L11 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generated from TI's devices.csv in version 1.212 using the script in`。
- **L12 EN**: Comment explains nearby logic, constraints, or intent: `Target/MSP430/gen-msp430-def.py - use this tool rather than adding`.
  **L12 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Target/MSP430/gen-msp430-def.py - use this tool rather than adding`。
- **L13 EN**: Comment explains nearby logic, constraints, or intent: `new MCUs by hand.`.
  **L13 CN**: 注释解释附近代码的逻辑、约束或设计意图：`new MCUs by hand.`。
- **L14 EN**: Separator comment used for visual grouping.
  **L14 CN**: 用于视觉分组的分隔注释。
- **L15 EN**: Banner comment marking a file or section boundary.
  **L15 CN**: 横幅注释，用于标记文件或章节边界。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Starts a preprocessor conditional block: `#ifndef MSP430_MCU_FEAT`.
  **L17 CN**: 开始一个预处理条件块：`#ifndef MSP430_MCU_FEAT`。
- **L18 EN**: Defines macro `MSP430_MCU_FEAT(NAME,` for conditional compilation, shorthand, or table-driven expansion.
  **L18 CN**: 定义宏 `MSP430_MCU_FEAT(NAME,`，用于条件编译、简写或表驱动展开。
- **L19 EN**: Closes the current preprocessor conditional block.
  **L19 CN**: 结束当前预处理条件块。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 21-40

````cpp
#ifndef MSP430_MCU
#define MSP430_MCU(NAME)
#endif

MSP430_MCU("msp430c111")
MSP430_MCU("msp430c1111")
MSP430_MCU("msp430c112")
MSP430_MCU("msp430c1121")
MSP430_MCU("msp430c1331")
MSP430_MCU("msp430c1351")
MSP430_MCU("msp430c311s")
MSP430_MCU("msp430c312")
MSP430_MCU("msp430c313")
MSP430_MCU("msp430c314")
MSP430_MCU("msp430c315")
MSP430_MCU("msp430c323")
MSP430_MCU("msp430c325")
MSP430_MCU("msp430c412")
MSP430_MCU("msp430c413")
MSP430_MCU("msp430e112")
````
- **L21 EN**: Starts a preprocessor conditional block: `#ifndef MSP430_MCU`.
  **L21 CN**: 开始一个预处理条件块：`#ifndef MSP430_MCU`。
- **L22 EN**: Defines macro `MSP430_MCU(NAME)` for conditional compilation, shorthand, or table-driven expansion.
  **L22 CN**: 定义宏 `MSP430_MCU(NAME)`，用于条件编译、简写或表驱动展开。
- **L23 EN**: Closes the current preprocessor conditional block.
  **L23 CN**: 结束当前预处理条件块。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L25 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L25 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L26 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L26 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L27 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L27 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L28 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L28 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L29 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L29 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L30 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L30 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L31 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L31 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L32 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L32 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L33 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L33 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L34 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L34 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L35 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L35 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L36 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L36 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L37 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L37 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L38 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L38 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L39 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L39 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L40 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L40 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 41-60

````cpp
MSP430_MCU("msp430e313")
MSP430_MCU("msp430e315")
MSP430_MCU("msp430e325")
MSP430_MCU("msp430f110")
MSP430_MCU("msp430f1101")
MSP430_MCU("msp430f1101a")
MSP430_MCU("msp430f1111")
MSP430_MCU("msp430f1111a")
MSP430_MCU("msp430f112")
MSP430_MCU("msp430f1121")
MSP430_MCU("msp430f1121a")
MSP430_MCU("msp430f1122")
MSP430_MCU("msp430f1132")
MSP430_MCU("msp430f122")
MSP430_MCU("msp430f1222")
MSP430_MCU("msp430f123")
MSP430_MCU("msp430f1232")
MSP430_MCU("msp430f133")
MSP430_MCU("msp430f135")
MSP430_MCU("msp430f155")
````
- **L41 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L41 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L42 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L42 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L43 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L43 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L44 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L44 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L45 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L45 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L46 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L46 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L47 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L47 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L48 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L48 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L49 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L49 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L50 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L50 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L51 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L51 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L52 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L52 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L53 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L53 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L54 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L54 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L55 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L55 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L56 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L56 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L57 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L57 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L58 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L58 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L59 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L59 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L60 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L60 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 61-80

````cpp
MSP430_MCU("msp430f156")
MSP430_MCU("msp430f157")
MSP430_MCU("msp430f2001")
MSP430_MCU("msp430f2011")
MSP430_MCU("msp430f2002")
MSP430_MCU("msp430f2012")
MSP430_MCU("msp430f2003")
MSP430_MCU("msp430f2013")
MSP430_MCU("msp430f2101")
MSP430_MCU("msp430f2111")
MSP430_MCU("msp430f2121")
MSP430_MCU("msp430f2131")
MSP430_MCU("msp430f2112")
MSP430_MCU("msp430f2122")
MSP430_MCU("msp430f2132")
MSP430_MCU("msp430f2232")
MSP430_MCU("msp430f2252")
MSP430_MCU("msp430f2272")
MSP430_MCU("msp430f2234")
MSP430_MCU("msp430f2254")
````
- **L61 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L61 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L62 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L62 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L63 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L63 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L64 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L64 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L65 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L65 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L66 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L66 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L67 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L67 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L68 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L68 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L69 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L69 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L70 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L70 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L71 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L71 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L72 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L72 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L73 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L73 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L74 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L74 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L75 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L75 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L76 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L76 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L77 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L77 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L78 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L78 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L79 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L79 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L80 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L80 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 81-100

````cpp
MSP430_MCU("msp430f2274")
MSP430_MCU("msp430f412")
MSP430_MCU("msp430f413")
MSP430_MCU("msp430f415")
MSP430_MCU("msp430f417")
MSP430_MCU("msp430f4132")
MSP430_MCU("msp430f4152")
MSP430_MCU("msp430f435")
MSP430_MCU("msp430f436")
MSP430_MCU("msp430f437")
MSP430_MCU("msp430f4351")
MSP430_MCU("msp430f4361")
MSP430_MCU("msp430f4371")
MSP430_MCU("msp430fe423")
MSP430_MCU("msp430fe425")
MSP430_MCU("msp430fe427")
MSP430_MCU("msp430fe423a")
MSP430_MCU("msp430fe425a")
MSP430_MCU("msp430fe427a")
MSP430_MCU("msp430fe4232")
````
- **L81 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L81 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L82 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L82 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L83 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L83 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L84 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L84 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L85 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L85 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L86 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L86 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L87 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L87 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L88 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L88 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L89 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L89 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L90 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L90 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L91 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L91 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L92 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L92 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L93 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L93 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L94 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L94 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L95 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L95 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L96 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L96 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L97 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L97 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L98 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L98 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L99 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L99 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L100 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L100 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 101-120

````cpp
MSP430_MCU("msp430fe4242")
MSP430_MCU("msp430fe4252")
MSP430_MCU("msp430fe4272")
MSP430_MCU("msp430f4250")
MSP430_MCU("msp430f4260")
MSP430_MCU("msp430f4270")
MSP430_MCU("msp430fg4250")
MSP430_MCU("msp430fg4260")
MSP430_MCU("msp430fg4270")
MSP430_MCU("msp430fw423")
MSP430_MCU("msp430fw425")
MSP430_MCU("msp430fw427")
MSP430_MCU("msp430fw428")
MSP430_MCU("msp430fw429")
MSP430_MCU("msp430fg437")
MSP430_MCU("msp430fg438")
MSP430_MCU("msp430fg439")
MSP430_MCU("msp430f438")
MSP430_MCU("msp430f439")
MSP430_MCU("msp430f477")
````
- **L101 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L101 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L102 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L102 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L103 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L103 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L104 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L104 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L105 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L105 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L106 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L106 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L107 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L107 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L108 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L108 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L109 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L109 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L110 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L110 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L111 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L111 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L112 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L112 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L113 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L113 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L114 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L114 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L115 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L115 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L116 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L116 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L117 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L117 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L118 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L118 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L119 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L119 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L120 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L120 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 121-140

````cpp
MSP430_MCU("msp430f478")
MSP430_MCU("msp430f479")
MSP430_MCU("msp430fg477")
MSP430_MCU("msp430fg478")
MSP430_MCU("msp430fg479")
MSP430_MCU("msp430p112")
MSP430_MCU("msp430p313")
MSP430_MCU("msp430p315")
MSP430_MCU("msp430p315s")
MSP430_MCU("msp430p325")
MSP430_MCU("msp430l092")
MSP430_MCU("msp430c091")
MSP430_MCU("msp430c092")
MSP430_MCU("msp430g2211")
MSP430_MCU("msp430g2201")
MSP430_MCU("msp430g2111")
MSP430_MCU("msp430g2101")
MSP430_MCU("msp430g2001")
MSP430_MCU("msp430g2231")
MSP430_MCU("msp430g2221")
````
- **L121 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L121 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L122 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L122 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L123 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L123 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L124 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L124 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L125 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L125 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L126 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L126 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L127 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L127 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L128 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L128 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L129 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L129 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L130 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L130 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L131 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L131 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L132 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L132 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L133 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L133 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L134 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L134 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L135 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L135 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L136 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L136 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L137 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L137 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L138 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L138 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L139 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L139 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L140 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L140 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 141-160

````cpp
MSP430_MCU("msp430g2131")
MSP430_MCU("msp430g2121")
MSP430_MCU("msp430g2102")
MSP430_MCU("msp430g2202")
MSP430_MCU("msp430g2302")
MSP430_MCU("msp430g2402")
MSP430_MCU("msp430g2132")
MSP430_MCU("msp430g2232")
MSP430_MCU("msp430g2332")
MSP430_MCU("msp430g2432")
MSP430_MCU("msp430g2112")
MSP430_MCU("msp430g2212")
MSP430_MCU("msp430g2312")
MSP430_MCU("msp430g2412")
MSP430_MCU("msp430g2152")
MSP430_MCU("msp430g2252")
MSP430_MCU("msp430g2352")
MSP430_MCU("msp430g2452")
MSP430_MCU("msp430g2113")
MSP430_MCU("msp430g2213")
````
- **L141 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L141 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L142 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L142 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L143 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L143 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L144 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L144 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L145 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L145 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L146 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L146 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L147 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L147 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L148 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L148 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L149 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L149 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L150 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L150 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L151 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L151 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L152 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L152 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L153 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L153 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L154 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L154 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L155 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L155 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L156 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L156 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L157 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L157 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L158 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L158 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L159 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L159 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L160 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L160 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 161-180

````cpp
MSP430_MCU("msp430g2313")
MSP430_MCU("msp430g2413")
MSP430_MCU("msp430g2513")
MSP430_MCU("msp430g2153")
MSP430_MCU("msp430g2253")
MSP430_MCU("msp430g2353")
MSP430_MCU("msp430g2453")
MSP430_MCU("msp430g2553")
MSP430_MCU("msp430g2203")
MSP430_MCU("msp430g2303")
MSP430_MCU("msp430g2403")
MSP430_MCU("msp430g2233")
MSP430_MCU("msp430g2333")
MSP430_MCU("msp430g2433")
MSP430_MCU("msp430g2533")
MSP430_MCU("msp430tch5e")
MSP430_MCU("msp430g2444")
MSP430_MCU("msp430g2544")
MSP430_MCU("msp430g2744")
MSP430_MCU("msp430g2755")
````
- **L161 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L161 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L162 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L162 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L163 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L163 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L164 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L164 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L165 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L165 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L166 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L166 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L167 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L167 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L168 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L168 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L169 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L169 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L170 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L170 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L171 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L171 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L172 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L172 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L173 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L173 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L174 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L174 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L175 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L175 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L176 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L176 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L177 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L177 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L178 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L178 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L179 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L179 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L180 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L180 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 181-200

````cpp
MSP430_MCU("msp430g2855")
MSP430_MCU("msp430g2955")
MSP430_MCU("msp430g2230")
MSP430_MCU("msp430g2210")
MSP430_MCU("rf430frl152h")
MSP430_MCU("rf430frl153h")
MSP430_MCU("rf430frl154h")
MSP430_MCU("rf430frl152h_rom")
MSP430_MCU("rf430frl153h_rom")
MSP430_MCU("rf430frl154h_rom")
MSP430_MCU("msp430fr4131")
MSP430_MCU("msp430fr4132")
MSP430_MCU("msp430fr4133")
MSP430_MCU("msp430fr2032")
MSP430_MCU("msp430fr2033")
MSP430_MCU("msp430fr2110")
MSP430_MCU("msp430fr2111")
MSP430_MCU("msp430fr2310")
MSP430_MCU("msp430fr2311")
MSP430_MCU("msp430fr2100")
````
- **L181 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L181 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L182 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L182 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L183 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L183 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L184 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L184 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L185 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L185 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L186 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L186 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L187 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L187 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L188 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L188 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L189 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L189 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L190 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L190 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L191 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L191 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L192 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L192 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L193 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L193 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L194 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L194 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L195 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L195 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L196 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L196 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L197 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L197 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L198 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L198 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L199 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L199 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L200 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L200 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。

### Lines 201-220

````cpp
MSP430_MCU("msp430fr2000")

// With 16-bit hardware multiplier
MSP430_MCU_FEAT("msp430c336", "16bit")
MSP430_MCU_FEAT("msp430c337", "16bit")
MSP430_MCU_FEAT("msp430cg4616", "16bit")
MSP430_MCU_FEAT("msp430cg4617", "16bit")
MSP430_MCU_FEAT("msp430cg4618", "16bit")
MSP430_MCU_FEAT("msp430cg4619", "16bit")
MSP430_MCU_FEAT("msp430e337", "16bit")
MSP430_MCU_FEAT("msp430f147", "16bit")
MSP430_MCU_FEAT("msp430f148", "16bit")
MSP430_MCU_FEAT("msp430f149", "16bit")
MSP430_MCU_FEAT("msp430f1471", "16bit")
MSP430_MCU_FEAT("msp430f1481", "16bit")
MSP430_MCU_FEAT("msp430f1491", "16bit")
MSP430_MCU_FEAT("msp430f167", "16bit")
MSP430_MCU_FEAT("msp430f168", "16bit")
MSP430_MCU_FEAT("msp430f169", "16bit")
MSP430_MCU_FEAT("msp430f1610", "16bit")
````
- **L201 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L201 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L203 EN**: Comment explains nearby logic, constraints, or intent: `With 16-bit hardware multiplier`.
  **L203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`With 16-bit hardware multiplier`。
- **L204 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L204 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L205 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L205 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L206 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L206 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L207 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L207 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L208 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L208 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L209 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L209 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L210 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L210 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L211 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L211 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L212 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L212 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L213 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L213 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L214 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L214 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L215 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L215 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L216 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L216 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L217 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L217 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L218 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L218 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L219 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L219 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L220 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L220 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 221-240

````cpp
MSP430_MCU_FEAT("msp430f1611", "16bit")
MSP430_MCU_FEAT("msp430f1612", "16bit")
MSP430_MCU_FEAT("msp430f2330", "16bit")
MSP430_MCU_FEAT("msp430f2350", "16bit")
MSP430_MCU_FEAT("msp430f2370", "16bit")
MSP430_MCU_FEAT("msp430f233", "16bit")
MSP430_MCU_FEAT("msp430f235", "16bit")
MSP430_MCU_FEAT("msp430f247", "16bit")
MSP430_MCU_FEAT("msp430f248", "16bit")
MSP430_MCU_FEAT("msp430f249", "16bit")
MSP430_MCU_FEAT("msp430f2410", "16bit")
MSP430_MCU_FEAT("msp430f2471", "16bit")
MSP430_MCU_FEAT("msp430f2481", "16bit")
MSP430_MCU_FEAT("msp430f2491", "16bit")
MSP430_MCU_FEAT("msp430f2416", "16bit")
MSP430_MCU_FEAT("msp430f2417", "16bit")
MSP430_MCU_FEAT("msp430f2418", "16bit")
MSP430_MCU_FEAT("msp430f2419", "16bit")
MSP430_MCU_FEAT("msp430f2616", "16bit")
MSP430_MCU_FEAT("msp430f2617", "16bit")
````
- **L221 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L221 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L222 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L222 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L223 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L223 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L224 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L224 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L225 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L225 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L226 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L226 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L227 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L227 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L228 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L228 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L229 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L229 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L230 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L230 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L231 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L231 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L232 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L232 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L233 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L233 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L234 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L234 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L235 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L235 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L236 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L236 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L237 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L237 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L238 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L238 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L239 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L239 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L240 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L240 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 241-260

````cpp
MSP430_MCU_FEAT("msp430f2618", "16bit")
MSP430_MCU_FEAT("msp430f2619", "16bit")
MSP430_MCU_FEAT("msp430f423", "16bit")
MSP430_MCU_FEAT("msp430f425", "16bit")
MSP430_MCU_FEAT("msp430f427", "16bit")
MSP430_MCU_FEAT("msp430f423a", "16bit")
MSP430_MCU_FEAT("msp430f425a", "16bit")
MSP430_MCU_FEAT("msp430f427a", "16bit")
MSP430_MCU_FEAT("msp430f4481", "16bit")
MSP430_MCU_FEAT("msp430f4491", "16bit")
MSP430_MCU_FEAT("msp430f447", "16bit")
MSP430_MCU_FEAT("msp430f448", "16bit")
MSP430_MCU_FEAT("msp430f449", "16bit")
MSP430_MCU_FEAT("msp430f46161", "16bit")
MSP430_MCU_FEAT("msp430f46171", "16bit")
MSP430_MCU_FEAT("msp430f46181", "16bit")
MSP430_MCU_FEAT("msp430f46191", "16bit")
MSP430_MCU_FEAT("msp430f4616", "16bit")
MSP430_MCU_FEAT("msp430f4617", "16bit")
MSP430_MCU_FEAT("msp430f4618", "16bit")
````
- **L241 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L241 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L242 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L242 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L243 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L243 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L244 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L244 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L245 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L245 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L246 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L246 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L247 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L247 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L248 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L248 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L249 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L249 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L250 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L250 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L251 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L251 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L252 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L252 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L253 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L253 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L254 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L254 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L255 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L255 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L256 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L256 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L257 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L257 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L258 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L258 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L259 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L259 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L260 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L260 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 261-280

````cpp
MSP430_MCU_FEAT("msp430f4619", "16bit")
MSP430_MCU_FEAT("msp430fg4616", "16bit")
MSP430_MCU_FEAT("msp430fg4617", "16bit")
MSP430_MCU_FEAT("msp430fg4618", "16bit")
MSP430_MCU_FEAT("msp430fg4619", "16bit")
MSP430_MCU_FEAT("msp430p337", "16bit")
MSP430_MCU_FEAT("msp430afe221", "16bit")
MSP430_MCU_FEAT("msp430afe231", "16bit")
MSP430_MCU_FEAT("msp430afe251", "16bit")
MSP430_MCU_FEAT("msp430afe222", "16bit")
MSP430_MCU_FEAT("msp430afe232", "16bit")
MSP430_MCU_FEAT("msp430afe252", "16bit")
MSP430_MCU_FEAT("msp430afe223", "16bit")
MSP430_MCU_FEAT("msp430afe233", "16bit")
MSP430_MCU_FEAT("msp430afe253", "16bit")
MSP430_MCU_FEAT("msp430i2020", "16bit")
MSP430_MCU_FEAT("msp430i2021", "16bit")
MSP430_MCU_FEAT("msp430i2030", "16bit")
MSP430_MCU_FEAT("msp430i2031", "16bit")
MSP430_MCU_FEAT("msp430i2040", "16bit")
````
- **L261 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L261 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L262 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L262 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L263 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L263 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L264 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L264 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L265 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L265 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L266 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L266 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L267 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L267 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L268 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L268 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L269 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L269 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L270 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L270 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L271 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L271 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L272 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L272 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L273 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L273 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L274 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L274 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L275 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L275 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L276 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L276 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L277 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L277 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L278 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L278 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L279 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L279 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L280 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L280 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 281-300

````cpp
MSP430_MCU_FEAT("msp430i2041", "16bit")

// With 32-bit hardware multiplier
MSP430_MCU_FEAT("msp430f4783", "32bit")
MSP430_MCU_FEAT("msp430f4793", "32bit")
MSP430_MCU_FEAT("msp430f4784", "32bit")
MSP430_MCU_FEAT("msp430f4794", "32bit")
MSP430_MCU_FEAT("msp430f47126", "32bit")
MSP430_MCU_FEAT("msp430f47127", "32bit")
MSP430_MCU_FEAT("msp430f47163", "32bit")
MSP430_MCU_FEAT("msp430f47173", "32bit")
MSP430_MCU_FEAT("msp430f47183", "32bit")
MSP430_MCU_FEAT("msp430f47193", "32bit")
MSP430_MCU_FEAT("msp430f47166", "32bit")
MSP430_MCU_FEAT("msp430f47176", "32bit")
MSP430_MCU_FEAT("msp430f47186", "32bit")
MSP430_MCU_FEAT("msp430f47196", "32bit")
MSP430_MCU_FEAT("msp430f47167", "32bit")
MSP430_MCU_FEAT("msp430f47177", "32bit")
MSP430_MCU_FEAT("msp430f47187", "32bit")
````
- **L281 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L281 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L283 EN**: Comment explains nearby logic, constraints, or intent: `With 32-bit hardware multiplier`.
  **L283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`With 32-bit hardware multiplier`。
- **L284 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L284 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L285 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L285 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L286 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L286 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L287 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L287 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L288 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L288 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L289 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L289 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L290 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L290 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L291 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L291 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L292 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L292 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L293 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L293 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L294 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L294 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L295 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L295 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L296 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L296 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L297 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L297 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L298 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L298 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L299 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L299 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L300 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L300 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 301-320

````cpp
MSP430_MCU_FEAT("msp430f47197", "32bit")
MSP430_MCU_FEAT("msp430f5418", "32bit")
MSP430_MCU_FEAT("msp430f5419", "32bit")
MSP430_MCU_FEAT("msp430f5435", "32bit")
MSP430_MCU_FEAT("msp430f5436", "32bit")
MSP430_MCU_FEAT("msp430f5437", "32bit")
MSP430_MCU_FEAT("msp430f5438", "32bit")
MSP430_MCU_FEAT("msp430f5418a", "32bit")
MSP430_MCU_FEAT("msp430f5419a", "32bit")
MSP430_MCU_FEAT("msp430f5435a", "32bit")
MSP430_MCU_FEAT("msp430f5436a", "32bit")
MSP430_MCU_FEAT("msp430f5437a", "32bit")
MSP430_MCU_FEAT("msp430f5438a", "32bit")
MSP430_MCU_FEAT("msp430f5212", "32bit")
MSP430_MCU_FEAT("msp430f5213", "32bit")
MSP430_MCU_FEAT("msp430f5214", "32bit")
MSP430_MCU_FEAT("msp430f5217", "32bit")
MSP430_MCU_FEAT("msp430f5218", "32bit")
MSP430_MCU_FEAT("msp430f5219", "32bit")
MSP430_MCU_FEAT("msp430f5222", "32bit")
````
- **L301 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L301 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L302 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L302 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L303 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L303 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L304 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L304 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L305 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L305 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L306 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L306 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L307 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L307 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L308 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L308 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L309 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L309 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L310 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L310 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L311 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L311 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L312 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L312 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L313 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L313 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L314 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L314 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L315 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L315 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L316 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L316 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L317 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L317 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L318 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L318 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L319 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L319 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L320 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L320 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 321-340

````cpp
MSP430_MCU_FEAT("msp430f5223", "32bit")
MSP430_MCU_FEAT("msp430f5224", "32bit")
MSP430_MCU_FEAT("msp430f5227", "32bit")
MSP430_MCU_FEAT("msp430f5228", "32bit")
MSP430_MCU_FEAT("msp430f5229", "32bit")
MSP430_MCU_FEAT("msp430f5232", "32bit")
MSP430_MCU_FEAT("msp430f5234", "32bit")
MSP430_MCU_FEAT("msp430f5237", "32bit")
MSP430_MCU_FEAT("msp430f5239", "32bit")
MSP430_MCU_FEAT("msp430f5242", "32bit")
MSP430_MCU_FEAT("msp430f5244", "32bit")
MSP430_MCU_FEAT("msp430f5247", "32bit")
MSP430_MCU_FEAT("msp430f5249", "32bit")
MSP430_MCU_FEAT("msp430f5304", "32bit")
MSP430_MCU_FEAT("msp430f5308", "32bit")
MSP430_MCU_FEAT("msp430f5309", "32bit")
MSP430_MCU_FEAT("msp430f5310", "32bit")
MSP430_MCU_FEAT("msp430f5340", "32bit")
MSP430_MCU_FEAT("msp430f5341", "32bit")
MSP430_MCU_FEAT("msp430f5342", "32bit")
````
- **L321 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L321 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L322 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L322 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L323 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L323 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L324 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L324 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L325 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L325 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L326 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L326 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L327 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L327 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L328 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L328 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L329 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L329 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L330 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L330 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L331 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L331 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L332 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L332 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L333 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L333 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L334 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L334 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L335 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L335 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L336 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L336 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L337 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L337 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L338 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L338 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L339 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L339 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L340 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L340 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 341-360

````cpp
MSP430_MCU_FEAT("msp430f5324", "32bit")
MSP430_MCU_FEAT("msp430f5325", "32bit")
MSP430_MCU_FEAT("msp430f5326", "32bit")
MSP430_MCU_FEAT("msp430f5327", "32bit")
MSP430_MCU_FEAT("msp430f5328", "32bit")
MSP430_MCU_FEAT("msp430f5329", "32bit")
MSP430_MCU_FEAT("msp430f5500", "32bit")
MSP430_MCU_FEAT("msp430f5501", "32bit")
MSP430_MCU_FEAT("msp430f5502", "32bit")
MSP430_MCU_FEAT("msp430f5503", "32bit")
MSP430_MCU_FEAT("msp430f5504", "32bit")
MSP430_MCU_FEAT("msp430f5505", "32bit")
MSP430_MCU_FEAT("msp430f5506", "32bit")
MSP430_MCU_FEAT("msp430f5507", "32bit")
MSP430_MCU_FEAT("msp430f5508", "32bit")
MSP430_MCU_FEAT("msp430f5509", "32bit")
MSP430_MCU_FEAT("msp430f5510", "32bit")
MSP430_MCU_FEAT("msp430f5513", "32bit")
MSP430_MCU_FEAT("msp430f5514", "32bit")
MSP430_MCU_FEAT("msp430f5515", "32bit")
````
- **L341 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L341 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L342 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L342 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L343 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L343 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L344 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L344 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L345 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L345 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L346 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L346 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L347 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L347 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L348 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L348 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L349 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L349 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L350 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L350 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L351 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L351 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L352 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L352 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L353 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L353 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L354 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L354 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L355 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L355 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L356 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L356 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L357 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L357 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L358 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L358 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L359 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L359 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L360 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L360 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 361-380

````cpp
MSP430_MCU_FEAT("msp430f5517", "32bit")
MSP430_MCU_FEAT("msp430f5519", "32bit")
MSP430_MCU_FEAT("msp430f5521", "32bit")
MSP430_MCU_FEAT("msp430f5522", "32bit")
MSP430_MCU_FEAT("msp430f5524", "32bit")
MSP430_MCU_FEAT("msp430f5525", "32bit")
MSP430_MCU_FEAT("msp430f5526", "32bit")
MSP430_MCU_FEAT("msp430f5527", "32bit")
MSP430_MCU_FEAT("msp430f5528", "32bit")
MSP430_MCU_FEAT("msp430f5529", "32bit")
MSP430_MCU_FEAT("cc430f5133", "32bit")
MSP430_MCU_FEAT("cc430f5135", "32bit")
MSP430_MCU_FEAT("cc430f5137", "32bit")
MSP430_MCU_FEAT("cc430f6125", "32bit")
MSP430_MCU_FEAT("cc430f6126", "32bit")
MSP430_MCU_FEAT("cc430f6127", "32bit")
MSP430_MCU_FEAT("cc430f6135", "32bit")
MSP430_MCU_FEAT("cc430f6137", "32bit")
MSP430_MCU_FEAT("cc430f5123", "32bit")
MSP430_MCU_FEAT("cc430f5125", "32bit")
````
- **L361 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L361 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L362 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L362 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L363 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L363 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L364 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L364 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L365 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L365 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L366 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L366 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L367 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L367 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L368 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L368 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L369 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L369 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L370 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L370 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L371 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L371 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L372 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L372 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L373 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L373 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L374 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L374 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L375 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L375 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L376 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L376 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L377 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L377 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L378 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L378 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L379 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L379 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L380 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L380 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 381-400

````cpp
MSP430_MCU_FEAT("cc430f5143", "32bit")
MSP430_MCU_FEAT("cc430f5145", "32bit")
MSP430_MCU_FEAT("cc430f5147", "32bit")
MSP430_MCU_FEAT("cc430f6143", "32bit")
MSP430_MCU_FEAT("cc430f6145", "32bit")
MSP430_MCU_FEAT("cc430f6147", "32bit")
MSP430_MCU_FEAT("msp430f5333", "32bit")
MSP430_MCU_FEAT("msp430f5335", "32bit")
MSP430_MCU_FEAT("msp430f5336", "32bit")
MSP430_MCU_FEAT("msp430f5338", "32bit")
MSP430_MCU_FEAT("msp430f5630", "32bit")
MSP430_MCU_FEAT("msp430f5631", "32bit")
MSP430_MCU_FEAT("msp430f5632", "32bit")
MSP430_MCU_FEAT("msp430f5633", "32bit")
MSP430_MCU_FEAT("msp430f5634", "32bit")
MSP430_MCU_FEAT("msp430f5635", "32bit")
MSP430_MCU_FEAT("msp430f5636", "32bit")
MSP430_MCU_FEAT("msp430f5637", "32bit")
MSP430_MCU_FEAT("msp430f5638", "32bit")
MSP430_MCU_FEAT("msp430f6433", "32bit")
````
- **L381 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L381 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L382 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L382 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L383 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L383 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L384 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L384 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L385 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L385 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L386 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L386 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L387 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L387 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L388 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L388 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L389 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L389 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L390 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L390 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L391 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L391 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L392 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L392 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L393 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L393 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L394 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L394 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L395 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L395 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L396 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L396 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L397 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L397 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L398 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L398 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L399 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L399 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L400 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L400 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 401-420

````cpp
MSP430_MCU_FEAT("msp430f6435", "32bit")
MSP430_MCU_FEAT("msp430f6436", "32bit")
MSP430_MCU_FEAT("msp430f6438", "32bit")
MSP430_MCU_FEAT("msp430f6630", "32bit")
MSP430_MCU_FEAT("msp430f6631", "32bit")
MSP430_MCU_FEAT("msp430f6632", "32bit")
MSP430_MCU_FEAT("msp430f6633", "32bit")
MSP430_MCU_FEAT("msp430f6634", "32bit")
MSP430_MCU_FEAT("msp430f6635", "32bit")
MSP430_MCU_FEAT("msp430f6636", "32bit")
MSP430_MCU_FEAT("msp430f6637", "32bit")
MSP430_MCU_FEAT("msp430f6638", "32bit")
MSP430_MCU_FEAT("msp430f5358", "32bit")
MSP430_MCU_FEAT("msp430f5359", "32bit")
MSP430_MCU_FEAT("msp430f5658", "32bit")
MSP430_MCU_FEAT("msp430f5659", "32bit")
MSP430_MCU_FEAT("msp430f6458", "32bit")
MSP430_MCU_FEAT("msp430f6459", "32bit")
MSP430_MCU_FEAT("msp430f6658", "32bit")
MSP430_MCU_FEAT("msp430f6659", "32bit")
````
- **L401 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L401 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L402 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L402 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L403 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L403 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L404 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L404 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L405 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L405 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L406 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L406 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L407 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L407 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L408 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L408 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L409 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L409 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L410 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L410 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L411 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L411 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L412 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L412 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L413 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L413 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L414 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L414 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L415 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L415 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L416 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L416 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L417 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L417 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L418 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L418 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L419 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L419 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L420 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L420 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 421-440

````cpp
MSP430_MCU_FEAT("msp430fg6425", "32bit")
MSP430_MCU_FEAT("msp430fg6426", "32bit")
MSP430_MCU_FEAT("msp430fg6625", "32bit")
MSP430_MCU_FEAT("msp430fg6626", "32bit")
MSP430_MCU_FEAT("msp430f5131", "32bit")
MSP430_MCU_FEAT("msp430f5151", "32bit")
MSP430_MCU_FEAT("msp430f5171", "32bit")
MSP430_MCU_FEAT("msp430f5132", "32bit")
MSP430_MCU_FEAT("msp430f5152", "32bit")
MSP430_MCU_FEAT("msp430f5172", "32bit")
MSP430_MCU_FEAT("msp430f6720", "32bit")
MSP430_MCU_FEAT("msp430f6721", "32bit")
MSP430_MCU_FEAT("msp430f6723", "32bit")
MSP430_MCU_FEAT("msp430f6724", "32bit")
MSP430_MCU_FEAT("msp430f6725", "32bit")
MSP430_MCU_FEAT("msp430f6726", "32bit")
MSP430_MCU_FEAT("msp430f6730", "32bit")
MSP430_MCU_FEAT("msp430f6731", "32bit")
MSP430_MCU_FEAT("msp430f6733", "32bit")
MSP430_MCU_FEAT("msp430f6734", "32bit")
````
- **L421 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L421 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L422 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L422 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L423 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L423 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L424 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L424 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L425 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L425 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L426 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L426 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L427 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L427 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L428 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L428 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L429 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L429 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L430 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L430 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L431 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L431 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L432 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L432 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L433 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L433 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L434 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L434 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L435 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L435 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L436 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L436 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L437 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L437 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L438 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L438 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L439 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L439 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L440 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L440 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 441-460

````cpp
MSP430_MCU_FEAT("msp430f6735", "32bit")
MSP430_MCU_FEAT("msp430f6736", "32bit")
MSP430_MCU_FEAT("msp430f67621", "32bit")
MSP430_MCU_FEAT("msp430f67641", "32bit")
MSP430_MCU_FEAT("msp430f6720a", "32bit")
MSP430_MCU_FEAT("msp430f6721a", "32bit")
MSP430_MCU_FEAT("msp430f6723a", "32bit")
MSP430_MCU_FEAT("msp430f6724a", "32bit")
MSP430_MCU_FEAT("msp430f6725a", "32bit")
MSP430_MCU_FEAT("msp430f6726a", "32bit")
MSP430_MCU_FEAT("msp430f6730a", "32bit")
MSP430_MCU_FEAT("msp430f6731a", "32bit")
MSP430_MCU_FEAT("msp430f6733a", "32bit")
MSP430_MCU_FEAT("msp430f6734a", "32bit")
MSP430_MCU_FEAT("msp430f6735a", "32bit")
MSP430_MCU_FEAT("msp430f6736a", "32bit")
MSP430_MCU_FEAT("msp430f67621a", "32bit")
MSP430_MCU_FEAT("msp430f67641a", "32bit")
MSP430_MCU_FEAT("msp430f67451", "32bit")
MSP430_MCU_FEAT("msp430f67651", "32bit")
````
- **L441 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L441 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L442 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L442 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L443 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L443 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L444 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L444 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L445 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L445 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L446 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L446 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L447 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L447 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L448 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L448 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L449 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L449 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L450 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L450 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L451 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L451 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L452 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L452 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L453 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L453 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L454 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L454 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L455 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L455 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L456 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L456 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L457 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L457 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L458 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L458 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L459 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L459 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L460 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L460 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 461-480

````cpp
MSP430_MCU_FEAT("msp430f67751", "32bit")
MSP430_MCU_FEAT("msp430f67461", "32bit")
MSP430_MCU_FEAT("msp430f67661", "32bit")
MSP430_MCU_FEAT("msp430f67761", "32bit")
MSP430_MCU_FEAT("msp430f67471", "32bit")
MSP430_MCU_FEAT("msp430f67671", "32bit")
MSP430_MCU_FEAT("msp430f67771", "32bit")
MSP430_MCU_FEAT("msp430f67481", "32bit")
MSP430_MCU_FEAT("msp430f67681", "32bit")
MSP430_MCU_FEAT("msp430f67781", "32bit")
MSP430_MCU_FEAT("msp430f67491", "32bit")
MSP430_MCU_FEAT("msp430f67691", "32bit")
MSP430_MCU_FEAT("msp430f67791", "32bit")
MSP430_MCU_FEAT("msp430f6745", "32bit")
MSP430_MCU_FEAT("msp430f6765", "32bit")
MSP430_MCU_FEAT("msp430f6775", "32bit")
MSP430_MCU_FEAT("msp430f6746", "32bit")
MSP430_MCU_FEAT("msp430f6766", "32bit")
MSP430_MCU_FEAT("msp430f6776", "32bit")
MSP430_MCU_FEAT("msp430f6747", "32bit")
````
- **L461 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L461 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L462 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L462 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L463 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L463 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L464 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L464 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L465 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L465 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L466 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L466 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L467 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L467 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L468 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L468 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L469 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L469 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L470 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L470 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L471 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L471 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L472 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L472 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L473 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L473 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L474 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L474 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L475 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L475 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L476 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L476 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L477 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L477 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L478 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L478 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L479 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L479 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L480 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L480 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 481-500

````cpp
MSP430_MCU_FEAT("msp430f6767", "32bit")
MSP430_MCU_FEAT("msp430f6777", "32bit")
MSP430_MCU_FEAT("msp430f6748", "32bit")
MSP430_MCU_FEAT("msp430f6768", "32bit")
MSP430_MCU_FEAT("msp430f6778", "32bit")
MSP430_MCU_FEAT("msp430f6749", "32bit")
MSP430_MCU_FEAT("msp430f6769", "32bit")
MSP430_MCU_FEAT("msp430f6779", "32bit")
MSP430_MCU_FEAT("msp430f67451a", "32bit")
MSP430_MCU_FEAT("msp430f67651a", "32bit")
MSP430_MCU_FEAT("msp430f67751a", "32bit")
MSP430_MCU_FEAT("msp430f67461a", "32bit")
MSP430_MCU_FEAT("msp430f67661a", "32bit")
MSP430_MCU_FEAT("msp430f67761a", "32bit")
MSP430_MCU_FEAT("msp430f67471a", "32bit")
MSP430_MCU_FEAT("msp430f67671a", "32bit")
MSP430_MCU_FEAT("msp430f67771a", "32bit")
MSP430_MCU_FEAT("msp430f67481a", "32bit")
MSP430_MCU_FEAT("msp430f67681a", "32bit")
MSP430_MCU_FEAT("msp430f67781a", "32bit")
````
- **L481 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L481 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L482 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L482 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L483 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L483 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L484 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L484 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L485 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L485 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L486 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L486 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L487 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L487 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L488 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L488 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L489 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L489 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L490 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L490 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L491 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L491 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L492 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L492 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L493 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L493 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L494 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L494 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L495 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L495 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L496 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L496 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L497 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L497 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L498 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L498 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L499 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L499 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L500 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L500 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 501-520

````cpp
MSP430_MCU_FEAT("msp430f67491a", "32bit")
MSP430_MCU_FEAT("msp430f67691a", "32bit")
MSP430_MCU_FEAT("msp430f67791a", "32bit")
MSP430_MCU_FEAT("msp430f6745a", "32bit")
MSP430_MCU_FEAT("msp430f6765a", "32bit")
MSP430_MCU_FEAT("msp430f6775a", "32bit")
MSP430_MCU_FEAT("msp430f6746a", "32bit")
MSP430_MCU_FEAT("msp430f6766a", "32bit")
MSP430_MCU_FEAT("msp430f6776a", "32bit")
MSP430_MCU_FEAT("msp430f6747a", "32bit")
MSP430_MCU_FEAT("msp430f6767a", "32bit")
MSP430_MCU_FEAT("msp430f6777a", "32bit")
MSP430_MCU_FEAT("msp430f6748a", "32bit")
MSP430_MCU_FEAT("msp430f6768a", "32bit")
MSP430_MCU_FEAT("msp430f6778a", "32bit")
MSP430_MCU_FEAT("msp430f6749a", "32bit")
MSP430_MCU_FEAT("msp430f6769a", "32bit")
MSP430_MCU_FEAT("msp430f6779a", "32bit")
MSP430_MCU_FEAT("msp430fr5720", "32bit")
MSP430_MCU_FEAT("msp430fr5721", "32bit")
````
- **L501 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L501 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L502 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L502 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L503 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L503 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L504 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L504 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L505 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L505 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L506 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L506 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L507 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L507 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L508 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L508 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L509 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L509 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L510 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L510 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L511 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L511 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L512 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L512 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L513 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L513 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L514 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L514 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L515 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L515 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L516 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L516 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L517 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L517 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L518 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L518 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L519 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L519 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L520 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L520 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 521-540

````cpp
MSP430_MCU_FEAT("msp430fr5722", "32bit")
MSP430_MCU_FEAT("msp430fr5723", "32bit")
MSP430_MCU_FEAT("msp430fr5724", "32bit")
MSP430_MCU_FEAT("msp430fr5725", "32bit")
MSP430_MCU_FEAT("msp430fr5726", "32bit")
MSP430_MCU_FEAT("msp430fr5727", "32bit")
MSP430_MCU_FEAT("msp430fr5728", "32bit")
MSP430_MCU_FEAT("msp430fr5729", "32bit")
MSP430_MCU_FEAT("msp430fr5730", "32bit")
MSP430_MCU_FEAT("msp430fr5731", "32bit")
MSP430_MCU_FEAT("msp430fr5732", "32bit")
MSP430_MCU_FEAT("msp430fr5733", "32bit")
MSP430_MCU_FEAT("msp430fr5734", "32bit")
MSP430_MCU_FEAT("msp430fr5735", "32bit")
MSP430_MCU_FEAT("msp430fr5736", "32bit")
MSP430_MCU_FEAT("msp430fr5737", "32bit")
MSP430_MCU_FEAT("msp430fr5738", "32bit")
MSP430_MCU_FEAT("msp430fr5739", "32bit")
MSP430_MCU_FEAT("msp430bt5190", "32bit")
MSP430_MCU_FEAT("msp430fr5857", "32bit")
````
- **L521 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L521 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L522 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L522 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L523 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L523 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L524 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L524 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L525 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L525 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L526 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L526 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L527 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L527 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L528 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L528 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L529 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L529 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L530 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L530 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L531 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L531 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L532 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L532 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L533 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L533 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L534 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L534 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L535 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L535 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L536 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L536 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L537 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L537 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L538 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L538 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L539 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L539 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L540 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L540 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 541-560

````cpp
MSP430_MCU_FEAT("msp430fr5858", "32bit")
MSP430_MCU_FEAT("msp430fr5859", "32bit")
MSP430_MCU_FEAT("msp430fr5847", "32bit")
MSP430_MCU_FEAT("msp430fr58471", "32bit")
MSP430_MCU_FEAT("msp430fr5848", "32bit")
MSP430_MCU_FEAT("msp430fr5849", "32bit")
MSP430_MCU_FEAT("msp430fr5867", "32bit")
MSP430_MCU_FEAT("msp430fr58671", "32bit")
MSP430_MCU_FEAT("msp430fr5868", "32bit")
MSP430_MCU_FEAT("msp430fr5869", "32bit")
MSP430_MCU_FEAT("msp430fr5957", "32bit")
MSP430_MCU_FEAT("msp430fr5958", "32bit")
MSP430_MCU_FEAT("msp430fr5959", "32bit")
MSP430_MCU_FEAT("msp430fr5947", "32bit")
MSP430_MCU_FEAT("msp430fr59471", "32bit")
MSP430_MCU_FEAT("msp430fr5948", "32bit")
MSP430_MCU_FEAT("msp430fr5949", "32bit")
MSP430_MCU_FEAT("msp430fr5967", "32bit")
MSP430_MCU_FEAT("msp430fr5968", "32bit")
MSP430_MCU_FEAT("msp430fr5969", "32bit")
````
- **L541 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L541 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L542 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L542 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L543 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L543 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L544 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L544 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L545 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L545 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L546 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L546 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L547 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L547 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L548 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L548 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L549 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L549 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L550 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L550 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L551 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L551 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L552 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L552 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L553 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L553 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L554 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L554 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L555 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L555 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L556 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L556 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L557 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L557 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L558 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L558 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L559 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L559 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L560 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L560 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 561-580

````cpp
MSP430_MCU_FEAT("msp430fr59691", "32bit")
MSP430_MCU_FEAT("rf430f5175", "32bit")
MSP430_MCU_FEAT("rf430f5155", "32bit")
MSP430_MCU_FEAT("rf430f5144", "32bit")
MSP430_MCU_FEAT("msp430fr69271", "32bit")
MSP430_MCU_FEAT("msp430fr68791", "32bit")
MSP430_MCU_FEAT("msp430fr69791", "32bit")
MSP430_MCU_FEAT("msp430fr6927", "32bit")
MSP430_MCU_FEAT("msp430fr6928", "32bit")
MSP430_MCU_FEAT("msp430fr6877", "32bit")
MSP430_MCU_FEAT("msp430fr6977", "32bit")
MSP430_MCU_FEAT("msp430fr6879", "32bit")
MSP430_MCU_FEAT("msp430fr6979", "32bit")
MSP430_MCU_FEAT("msp430fr58891", "32bit")
MSP430_MCU_FEAT("msp430fr68891", "32bit")
MSP430_MCU_FEAT("msp430fr59891", "32bit")
MSP430_MCU_FEAT("msp430fr69891", "32bit")
MSP430_MCU_FEAT("msp430fr5887", "32bit")
MSP430_MCU_FEAT("msp430fr5888", "32bit")
MSP430_MCU_FEAT("msp430fr5889", "32bit")
````
- **L561 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L561 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L562 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L562 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L563 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L563 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L564 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L564 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L565 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L565 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L566 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L566 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L567 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L567 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L568 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L568 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L569 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L569 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L570 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L570 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L571 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L571 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L572 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L572 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L573 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L573 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L574 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L574 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L575 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L575 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L576 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L576 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L577 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L577 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L578 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L578 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L579 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L579 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L580 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L580 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 581-600

````cpp
MSP430_MCU_FEAT("msp430fr6887", "32bit")
MSP430_MCU_FEAT("msp430fr6888", "32bit")
MSP430_MCU_FEAT("msp430fr6889", "32bit")
MSP430_MCU_FEAT("msp430fr5986", "32bit")
MSP430_MCU_FEAT("msp430fr5987", "32bit")
MSP430_MCU_FEAT("msp430fr5988", "32bit")
MSP430_MCU_FEAT("msp430fr5989", "32bit")
MSP430_MCU_FEAT("msp430fr6987", "32bit")
MSP430_MCU_FEAT("msp430fr6988", "32bit")
MSP430_MCU_FEAT("msp430fr6989", "32bit")
MSP430_MCU_FEAT("msp430fr5922", "32bit")
MSP430_MCU_FEAT("msp430fr5870", "32bit")
MSP430_MCU_FEAT("msp430fr5970", "32bit")
MSP430_MCU_FEAT("msp430fr5872", "32bit")
MSP430_MCU_FEAT("msp430fr5972", "32bit")
MSP430_MCU_FEAT("msp430fr6820", "32bit")
MSP430_MCU_FEAT("msp430fr6920", "32bit")
MSP430_MCU_FEAT("msp430fr6822", "32bit")
MSP430_MCU_FEAT("msp430fr6922", "32bit")
MSP430_MCU_FEAT("msp430fr6870", "32bit")
````
- **L581 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L581 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L582 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L582 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L583 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L583 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L584 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L584 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L585 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L585 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L586 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L586 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L587 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L587 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L588 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L588 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L589 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L589 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L590 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L590 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L591 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L591 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L592 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L592 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L593 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L593 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L594 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L594 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L595 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L595 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L596 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L596 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L597 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L597 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L598 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L598 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L599 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L599 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L600 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L600 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 601-620

````cpp
MSP430_MCU_FEAT("msp430fr6970", "32bit")
MSP430_MCU_FEAT("msp430fr6872", "32bit")
MSP430_MCU_FEAT("msp430fr6972", "32bit")
MSP430_MCU_FEAT("msp430fr59221", "32bit")
MSP430_MCU_FEAT("msp430fr58721", "32bit")
MSP430_MCU_FEAT("msp430fr59721", "32bit")
MSP430_MCU_FEAT("msp430fr68221", "32bit")
MSP430_MCU_FEAT("msp430fr69221", "32bit")
MSP430_MCU_FEAT("msp430fr68721", "32bit")
MSP430_MCU_FEAT("msp430fr69721", "32bit")
MSP430_MCU_FEAT("msp430sl5438a", "32bit")
MSP430_MCU_FEAT("msp430fr2433", "32bit")
MSP430_MCU_FEAT("msp430fr2532", "32bit")
MSP430_MCU_FEAT("msp430fr2533", "32bit")
MSP430_MCU_FEAT("msp430fr2632", "32bit")
MSP430_MCU_FEAT("msp430fr2633", "32bit")
MSP430_MCU_FEAT("msp430f5252", "32bit")
MSP430_MCU_FEAT("msp430f5253", "32bit")
MSP430_MCU_FEAT("msp430f5254", "32bit")
MSP430_MCU_FEAT("msp430f5255", "32bit")
````
- **L601 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L601 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L602 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L602 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L603 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L603 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L604 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L604 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L605 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L605 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L606 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L606 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L607 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L607 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L608 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L608 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L609 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L609 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L610 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L610 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L611 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L611 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L612 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L612 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L613 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L613 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L614 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L614 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L615 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L615 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L616 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L616 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L617 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L617 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L618 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L618 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L619 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L619 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L620 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L620 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 621-640

````cpp
MSP430_MCU_FEAT("msp430f5256", "32bit")
MSP430_MCU_FEAT("msp430f5257", "32bit")
MSP430_MCU_FEAT("msp430f5258", "32bit")
MSP430_MCU_FEAT("msp430f5259", "32bit")
MSP430_MCU_FEAT("msp430fr5962", "32bit")
MSP430_MCU_FEAT("msp430fr5964", "32bit")
MSP430_MCU_FEAT("msp430fr5992", "32bit")
MSP430_MCU_FEAT("msp430fr5994", "32bit")
MSP430_MCU_FEAT("msp430fr59941", "32bit")
MSP430_MCU_FEAT("msp430fr2355", "32bit")
MSP430_MCU_FEAT("msp430fr2155", "32bit")
MSP430_MCU_FEAT("msp430fr2353", "32bit")
MSP430_MCU_FEAT("msp430fr2153", "32bit")
MSP430_MCU_FEAT("msp430fr2522", "32bit")
MSP430_MCU_FEAT("msp430fr2512", "32bit")
MSP430_MCU_FEAT("msp430fr2422", "32bit")
MSP430_MCU_FEAT("msp430fr2676", "32bit")
MSP430_MCU_FEAT("msp430fr2476", "32bit")
MSP430_MCU_FEAT("msp430fr2675", "32bit")
MSP430_MCU_FEAT("msp430fr2673", "32bit")
````
- **L621 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L621 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L622 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L622 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L623 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L623 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L624 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L624 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L625 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L625 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L626 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L626 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L627 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L627 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L628 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L628 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L629 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L629 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L630 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L630 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L631 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L631 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L632 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L632 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L633 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L633 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L634 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L634 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L635 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L635 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L636 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L636 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L637 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L637 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L638 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L638 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L639 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L639 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L640 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L640 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。

### Lines 641-660

````cpp
MSP430_MCU_FEAT("msp430fr2475", "32bit")
MSP430_MCU_FEAT("msp430fr2672", "32bit")
MSP430_MCU_FEAT("msp430fr6043", "32bit")
MSP430_MCU_FEAT("msp430fr5043", "32bit")
MSP430_MCU_FEAT("msp430fr6041", "32bit")
MSP430_MCU_FEAT("msp430fr60431", "32bit")
MSP430_MCU_FEAT("msp430fr5041", "32bit")
MSP430_MCU_FEAT("msp430fr50431", "32bit")
MSP430_MCU_FEAT("msp430fr6005", "32bit")
MSP430_MCU_FEAT("msp430fr6047", "32bit")
MSP430_MCU_FEAT("msp430fr6037", "32bit")
MSP430_MCU_FEAT("msp430fr6045", "32bit")
MSP430_MCU_FEAT("msp430fr60471", "32bit")
MSP430_MCU_FEAT("msp430fr6035", "32bit")
MSP430_MCU_FEAT("msp430fr6007", "32bit")
MSP430_MCU_FEAT("msp430fr60371", "32bit")

// Generic MCUs
MSP430_MCU("msp430i2xxgeneric")

````
- **L641 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L641 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L642 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L642 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L643 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L643 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L644 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L644 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L645 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L645 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L646 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L646 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L647 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L647 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L648 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L648 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L649 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L649 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L650 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L650 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L651 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L651 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L652 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L652 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L653 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L653 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L654 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L654 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L655 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L655 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L656 EN**: Invokes macro `MSP430_MCU_FEAT` to contribute one entry to a table-driven definition list.
  **L656 CN**: 调用宏 `MSP430_MCU_FEAT`，向表驱动定义列表贡献一个条目。
- **L657 EN**: Blank line separating nearby declarations or logic blocks.
  **L657 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L658 EN**: Comment explains nearby logic, constraints, or intent: `Generic MCUs`.
  **L658 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generic MCUs`。
- **L659 EN**: Invokes macro `MSP430_MCU` to contribute one entry to a table-driven definition list.
  **L659 CN**: 调用宏 `MSP430_MCU`，向表驱动定义列表贡献一个条目。
- **L660 EN**: Blank line separating nearby declarations or logic blocks.
  **L660 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 661-662

````cpp
#undef MSP430_MCU
#undef MSP430_MCU_FEAT
````
- **L661 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef MSP430_MCU`.
  **L661 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef MSP430_MCU`。
- **L662 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef MSP430_MCU_FEAT`.
  **L662 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef MSP430_MCU_FEAT`。

## Key Concepts / 关键概念

- **Compiler configuration / 编译器配置**
  - **EN**: Holds low-level enums, constants, feature switches, and descriptive metadata used across Clang.
  - **CN**: 保存 Clang 各处复用的底层枚举、常量、特性开关与描述性元数据。
- **Target and language metadata / 目标与语言元数据**
  - **EN**: Models ABI, address-space, builtin, and target-feature information shared by many subsystems.
  - **CN**: 建模多个子系统共享的 ABI、地址空间、builtin 与目标特性信息。
- **Generated definition tables / 生成式定义表**
  - **EN**: Many Basic headers use `.td` or `.def` files to describe families of diagnostics, attributes, or builtins declaratively.
  - **CN**: 许多 Basic 头文件使用 `.td` 或 `.def` 文件以声明式方式描述诊断、属性或 builtin 家族。
- **Macro-driven definition lists / 宏驱动定义列表**
  - **EN**: Provides reusable entry lists that different includers expand under different macro definitions.
  - **CN**: 提供可复用的条目列表，不同包含者可在不同宏定义下对其展开。
- **Macro-driven generation / 宏驱动生成**
  - **EN**: Relies on macros or generated include fragments to keep large definition sets synchronized.
  - **CN**: 依赖宏或生成式 include 片段来同步大规模定义集合。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: `MSP430_MCU_FEAT(NAME,`, `MSP430_MCU(NAME)`
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `MSP430_MCU`, `MSP430_MCU_FEAT`
- **TableGen records / TableGen 记录**: No prominent TableGen record names detected. / 未检测到明显的 TableGen 记录名称。
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
