# LangStandards.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/LangStandards.def`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: Language Standard Data *- C++.
- **Purpose (CN)**: 声明与 `LangStandards` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 269

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- LangStandards.def - Language Standard Data --------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LANGSTANDARD
#error "LANGSTANDARD must be defined before including this file"
#endif

/// LANGSTANDARD(IDENT, NAME, LANG, DESC, FEATURES, VERSION)
///
/// \param IDENT - The name of the standard as a C++ identifier.
/// \param NAME - The name of the standard.
/// \param LANG - The Language for which this is a standard.
/// \param DESC - A short description of the standard.
/// \param FEATURES - The standard features as flags, these are enums from the
/// clang::frontend namespace, which is assumed to be available.
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
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L9 EN**: Starts a preprocessor conditional block: `#ifndef LANGSTANDARD`.
  **L9 CN**: 开始一个预处理条件块：`#ifndef LANGSTANDARD`。
- **L10 EN**: Emits a compilation error for an unsupported configuration: `#error "LANGSTANDARD must be defined before including this file"`.
  **L10 CN**: 为不受支持的配置触发编译错误：`#error "LANGSTANDARD must be defined before including this file"`。
- **L11 EN**: Closes the current preprocessor conditional block.
  **L11 CN**: 结束当前预处理条件块。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Comment explains nearby logic, constraints, or intent: `LANGSTANDARD(IDENT, NAME, LANG, DESC, FEATURES, VERSION)`.
  **L13 CN**: 注释解释附近代码的逻辑、约束或设计意图：`LANGSTANDARD(IDENT, NAME, LANG, DESC, FEATURES, VERSION)`。
- **L14 EN**: Separator comment used for visual grouping.
  **L14 CN**: 用于视觉分组的分隔注释。
- **L15 EN**: Comment explains nearby logic, constraints, or intent: `param IDENT - The name of the standard as a C++ identifier.`.
  **L15 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param IDENT - The name of the standard as a C++ identifier.`。
- **L16 EN**: Comment explains nearby logic, constraints, or intent: `param NAME - The name of the standard.`.
  **L16 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param NAME - The name of the standard.`。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `param LANG - The Language for which this is a standard.`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param LANG - The Language for which this is a standard.`。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `param DESC - A short description of the standard.`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param DESC - A short description of the standard.`。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `param FEATURES - The standard features as flags, these are enums from the`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param FEATURES - The standard features as flags, these are enums from the`。
- **L20 EN**: Comment explains nearby logic, constraints, or intent: `clang::frontend namespace, which is assumed to be available.`.
  **L20 CN**: 注释解释附近代码的逻辑、约束或设计意图：`clang::frontend namespace, which is assumed to be available.`。

### Lines 21-40

````cpp
/// \param VERSION - The official version code for this standard.
/// Has value 'std::nullopt' if no official version exists.

/// LANGSTANDARD_ALIAS(IDENT, ALIAS)
/// \param IDENT - The name of the standard as a C++ identifier.
/// \param ALIAS - The alias of the standard.

/// LANGSTANDARD_ALIAS_DEPR(IDENT, ALIAS)
/// Same as LANGSTANDARD_ALIAS, but for a deprecated alias.

#ifndef LANGSTANDARD_ALIAS
#define LANGSTANDARD_ALIAS(IDENT, ALIAS)
#endif

#ifndef LANGSTANDARD_ALIAS_DEPR
#define LANGSTANDARD_ALIAS_DEPR(IDENT, ALIAS) LANGSTANDARD_ALIAS(IDENT, ALIAS)
#endif

// C89-ish modes.
LANGSTANDARD(c89, "c89",
````
- **L21 EN**: Comment explains nearby logic, constraints, or intent: `param VERSION - The official version code for this standard.`.
  **L21 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param VERSION - The official version code for this standard.`。
- **L22 EN**: Comment explains nearby logic, constraints, or intent: `Has value 'std::nullopt' if no official version exists.`.
  **L22 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Has value 'std::nullopt' if no official version exists.`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Comment explains nearby logic, constraints, or intent: `LANGSTANDARD_ALIAS(IDENT, ALIAS)`.
  **L24 CN**: 注释解释附近代码的逻辑、约束或设计意图：`LANGSTANDARD_ALIAS(IDENT, ALIAS)`。
- **L25 EN**: Comment explains nearby logic, constraints, or intent: `param IDENT - The name of the standard as a C++ identifier.`.
  **L25 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param IDENT - The name of the standard as a C++ identifier.`。
- **L26 EN**: Comment explains nearby logic, constraints, or intent: `param ALIAS - The alias of the standard.`.
  **L26 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param ALIAS - The alias of the standard.`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `LANGSTANDARD_ALIAS_DEPR(IDENT, ALIAS)`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`LANGSTANDARD_ALIAS_DEPR(IDENT, ALIAS)`。
- **L29 EN**: Comment explains nearby logic, constraints, or intent: `Same as LANGSTANDARD_ALIAS, but for a deprecated alias.`.
  **L29 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Same as LANGSTANDARD_ALIAS, but for a deprecated alias.`。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Starts a preprocessor conditional block: `#ifndef LANGSTANDARD_ALIAS`.
  **L31 CN**: 开始一个预处理条件块：`#ifndef LANGSTANDARD_ALIAS`。
- **L32 EN**: Defines macro `LANGSTANDARD_ALIAS(IDENT,` for conditional compilation, shorthand, or table-driven expansion.
  **L32 CN**: 定义宏 `LANGSTANDARD_ALIAS(IDENT,`，用于条件编译、简写或表驱动展开。
- **L33 EN**: Closes the current preprocessor conditional block.
  **L33 CN**: 结束当前预处理条件块。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Starts a preprocessor conditional block: `#ifndef LANGSTANDARD_ALIAS_DEPR`.
  **L35 CN**: 开始一个预处理条件块：`#ifndef LANGSTANDARD_ALIAS_DEPR`。
- **L36 EN**: Defines macro `LANGSTANDARD_ALIAS_DEPR(IDENT,` for conditional compilation, shorthand, or table-driven expansion.
  **L36 CN**: 定义宏 `LANGSTANDARD_ALIAS_DEPR(IDENT,`，用于条件编译、简写或表驱动展开。
- **L37 EN**: Closes the current preprocessor conditional block.
  **L37 CN**: 结束当前预处理条件块。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L39 EN**: Comment explains nearby logic, constraints, or intent: `C89-ish modes.`.
  **L39 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C89-ish modes.`。
- **L40 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L40 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。

### Lines 41-60

````cpp
             C, "ISO C 1990", 0, std::nullopt)
LANGSTANDARD_ALIAS(c89, "c90")
LANGSTANDARD_ALIAS(c89, "iso9899:1990")

LANGSTANDARD(c94, "iso9899:199409",
             C, "ISO C 1990 with amendment 1",
             Digraphs, 199409)

LANGSTANDARD(gnu89, "gnu89",
             C, "ISO C 1990 with GNU extensions",
             LineComment | Digraphs | GNUMode, std::nullopt)
LANGSTANDARD_ALIAS(gnu89, "gnu90")

// C99-ish modes
LANGSTANDARD(c99, "c99",
             C, "ISO C 1999",
             LineComment | C99 | Digraphs | HexFloat, 199901)
LANGSTANDARD_ALIAS(c99, "iso9899:1999")
LANGSTANDARD_ALIAS_DEPR(c99, "c9x")
LANGSTANDARD_ALIAS_DEPR(c99, "iso9899:199x")
````
- **L41 EN**: Continues the surrounding expression or declaration: `C, "ISO C 1990", 0, std::nullopt)`.
  **L41 CN**: 继续构造周围的表达式或声明：`C, "ISO C 1990", 0, std::nullopt)`。
- **L42 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L42 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L43 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L43 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L45 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L45 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L46 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 1990 with amendment 1",`.
  **L46 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 1990 with amendment 1",`。
- **L47 EN**: Continues the surrounding expression or declaration: `Digraphs, 199409)`.
  **L47 CN**: 继续构造周围的表达式或声明：`Digraphs, 199409)`。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L49 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L49 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L50 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 1990 with GNU extensions",`.
  **L50 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 1990 with GNU extensions",`。
- **L51 EN**: Continues the surrounding expression or declaration: `LineComment | Digraphs | GNUMode, std::nullopt)`.
  **L51 CN**: 继续构造周围的表达式或声明：`LineComment | Digraphs | GNUMode, std::nullopt)`。
- **L52 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L52 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L54 EN**: Comment explains nearby logic, constraints, or intent: `C99-ish modes`.
  **L54 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C99-ish modes`。
- **L55 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L55 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L56 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 1999",`.
  **L56 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 1999",`。
- **L57 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | Digraphs | HexFloat, 199901)`.
  **L57 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | Digraphs | HexFloat, 199901)`。
- **L58 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L58 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L59 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L59 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L60 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L60 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。

### Lines 61-80

````cpp

LANGSTANDARD(gnu99, "gnu99",
             C, "ISO C 1999 with GNU extensions",
             LineComment | C99 | Digraphs | GNUMode | HexFloat, 199901)
LANGSTANDARD_ALIAS_DEPR(gnu99, "gnu9x")

// C11 modes
LANGSTANDARD(c11, "c11",
             C, "ISO C 2011",
             LineComment | C99 | C11 | Digraphs | HexFloat, 201112)
LANGSTANDARD_ALIAS(c11, "iso9899:2011")
LANGSTANDARD_ALIAS_DEPR(c11, "c1x")
LANGSTANDARD_ALIAS_DEPR(c11, "iso9899:201x")

LANGSTANDARD(gnu11, "gnu11",
             C, "ISO C 2011 with GNU extensions",
             LineComment | C99 | C11 | Digraphs | GNUMode | HexFloat, 201112)
LANGSTANDARD_ALIAS_DEPR(gnu11, "gnu1x")

// C17 modes
````
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L62 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L62 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 1999 with GNU extensions",`.
  **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 1999 with GNU extensions",`。
- **L64 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | Digraphs | GNUMode | HexFloat, 199901)`.
  **L64 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | Digraphs | GNUMode | HexFloat, 199901)`。
- **L65 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L65 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L67 EN**: Comment explains nearby logic, constraints, or intent: `C11 modes`.
  **L67 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C11 modes`。
- **L68 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L68 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 2011",`.
  **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 2011",`。
- **L70 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | C11 | Digraphs | HexFloat, 201112)`.
  **L70 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | C11 | Digraphs | HexFloat, 201112)`。
- **L71 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L71 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L72 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L72 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L73 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L73 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L75 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L75 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 2011 with GNU extensions",`.
  **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 2011 with GNU extensions",`。
- **L77 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | C11 | Digraphs | GNUMode | HexFloat, 201112)`.
  **L77 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | C11 | Digraphs | GNUMode | HexFloat, 201112)`。
- **L78 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L78 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `C17 modes`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C17 modes`。

### Lines 81-100

````cpp
LANGSTANDARD(c17, "c17",
             C, "ISO C 2017",
             LineComment | C99 | C11 | C17 | Digraphs | HexFloat, 201710)
LANGSTANDARD_ALIAS(c17, "iso9899:2017")
LANGSTANDARD_ALIAS(c17, "c18")
LANGSTANDARD_ALIAS(c17, "iso9899:2018")
LANGSTANDARD(gnu17, "gnu17",
             C, "ISO C 2017 with GNU extensions",
             LineComment | C99 | C11 | C17 | Digraphs | GNUMode | HexFloat, 201710)
LANGSTANDARD_ALIAS(gnu17, "gnu18")

// C23 modes
LANGSTANDARD(c23, "c23",
             C, "ISO C 2023",
             LineComment | C99 | C11 | C17 | C23 | Digraphs | HexFloat, 202311)
LANGSTANDARD_ALIAS(c23, "iso9899:2024")
LANGSTANDARD_ALIAS_DEPR(c23, "c2x")
LANGSTANDARD(gnu23, "gnu23",
             C, "ISO C 2023 with GNU extensions",
             LineComment | C99 | C11 | C17 | C23 | Digraphs | GNUMode | HexFloat, 202311)
````
- **L81 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L81 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 2017",`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 2017",`。
- **L83 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | C11 | C17 | Digraphs | HexFloat, 201710)`.
  **L83 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | C11 | C17 | Digraphs | HexFloat, 201710)`。
- **L84 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L84 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L85 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L85 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L86 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L86 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L87 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L87 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 2017 with GNU extensions",`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 2017 with GNU extensions",`。
- **L89 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | C11 | C17 | Digraphs | GNUMode | HexFloat, 201710)`.
  **L89 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | C11 | C17 | Digraphs | GNUMode | HexFloat, 201710)`。
- **L90 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L90 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L92 EN**: Comment explains nearby logic, constraints, or intent: `C23 modes`.
  **L92 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C23 modes`。
- **L93 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L93 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 2023",`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 2023",`。
- **L95 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | C11 | C17 | C23 | Digraphs | HexFloat, 202311)`.
  **L95 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | C11 | C17 | C23 | Digraphs | HexFloat, 202311)`。
- **L96 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L96 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L97 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L97 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L98 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L98 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "ISO C 2023 with GNU extensions",`.
  **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "ISO C 2023 with GNU extensions",`。
- **L100 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | C11 | C17 | C23 | Digraphs | GNUMode | HexFloat, 202311)`.
  **L100 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | C11 | C17 | C23 | Digraphs | GNUMode | HexFloat, 202311)`。

### Lines 101-120

````cpp
LANGSTANDARD_ALIAS_DEPR(gnu23, "gnu2x")

// C2y modes
// FIXME: Use correct version code for C2y once published.
LANGSTANDARD(c2y, "c2y",
             C, "Working Draft for ISO C2y",
             LineComment | C99 | C11 | C17 | C23 | C2y | Digraphs | HexFloat, 202400)
LANGSTANDARD(gnu2y, "gnu2y",
             C, "Working Draft for ISO C2y with GNU extensions",
             LineComment | C99 | C11 | C17 | C23 | C2y | Digraphs | GNUMode | HexFloat, 202400)
// TODO: Add the iso9899:202y alias once ISO publishes the standard.

// C++ modes
LANGSTANDARD(cxx98, "c++98",
             CXX, "ISO C++ 1998 with amendments",
             LineComment | CPlusPlus | Digraphs, 199711)
LANGSTANDARD_ALIAS(cxx98, "c++03")

LANGSTANDARD(gnucxx98, "gnu++98",
             CXX, "ISO C++ 1998 with amendments and GNU extensions",
````
- **L101 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L101 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L103 EN**: Comment explains nearby logic, constraints, or intent: `C2y modes`.
  **L103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C2y modes`。
- **L104 EN**: Comment records a pending task or caution: `FIXME: Use correct version code for C2y once published.`.
  **L104 CN**: 注释记录待办事项或注意点：`FIXME: Use correct version code for C2y once published.`。
- **L105 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L105 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "Working Draft for ISO C2y",`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "Working Draft for ISO C2y",`。
- **L107 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | C11 | C17 | C23 | C2y | Digraphs | HexFloat, 202400)`.
  **L107 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | C11 | C17 | C23 | C2y | Digraphs | HexFloat, 202400)`。
- **L108 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L108 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `C, "Working Draft for ISO C2y with GNU extensions",`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`C, "Working Draft for ISO C2y with GNU extensions",`。
- **L110 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | C11 | C17 | C23 | C2y | Digraphs | GNUMode | HexFloat, 202400)`.
  **L110 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | C11 | C17 | C23 | C2y | Digraphs | GNUMode | HexFloat, 202400)`。
- **L111 EN**: Comment records a pending task or caution: `TODO: Add the iso9899:202y alias once ISO publishes the standard.`.
  **L111 CN**: 注释记录待办事项或注意点：`TODO: Add the iso9899:202y alias once ISO publishes the standard.`。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L113 EN**: Comment explains nearby logic, constraints, or intent: `C++ modes`.
  **L113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++ modes`。
- **L114 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L114 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 1998 with amendments",`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 1998 with amendments",`。
- **L116 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | Digraphs, 199711)`.
  **L116 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | Digraphs, 199711)`。
- **L117 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L117 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L119 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L119 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 1998 with amendments and GNU extensions",`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 1998 with amendments and GNU extensions",`。

### Lines 121-140

````cpp
             LineComment | CPlusPlus | Digraphs | GNUMode, 199711)
LANGSTANDARD_ALIAS(gnucxx98, "gnu++03")

LANGSTANDARD(cxx11, "c++11",
             CXX, "ISO C++ 2011 with amendments",
             LineComment | CPlusPlus | CPlusPlus11 | Digraphs, 201103)
LANGSTANDARD_ALIAS_DEPR(cxx11, "c++0x")

LANGSTANDARD(gnucxx11, "gnu++11", CXX,
             "ISO C++ 2011 with amendments and GNU extensions",
             LineComment | CPlusPlus | CPlusPlus11 | Digraphs | GNUMode, 201103)
LANGSTANDARD_ALIAS_DEPR(gnucxx11, "gnu++0x")

LANGSTANDARD(cxx14, "c++14",
             CXX, "ISO C++ 2014 with amendments",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | Digraphs, 201402)
LANGSTANDARD_ALIAS_DEPR(cxx14, "c++1y")

LANGSTANDARD(gnucxx14, "gnu++14",
             CXX, "ISO C++ 2014 with amendments and GNU extensions",
````
- **L121 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | Digraphs | GNUMode, 199711)`.
  **L121 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | Digraphs | GNUMode, 199711)`。
- **L122 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L122 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L123 EN**: Blank line separating nearby declarations or logic blocks.
  **L123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L124 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L124 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2011 with amendments",`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2011 with amendments",`。
- **L126 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | Digraphs, 201103)`.
  **L126 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | Digraphs, 201103)`。
- **L127 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L127 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L129 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L129 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ISO C++ 2011 with amendments and GNU extensions",`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ISO C++ 2011 with amendments and GNU extensions",`。
- **L131 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | Digraphs | GNUMode, 201103)`.
  **L131 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | Digraphs | GNUMode, 201103)`。
- **L132 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L132 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L134 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L134 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2014 with amendments",`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2014 with amendments",`。
- **L136 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | Digraphs, 201402)`.
  **L136 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | Digraphs, 201402)`。
- **L137 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L137 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L139 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L139 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2014 with amendments and GNU extensions",`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2014 with amendments and GNU extensions",`。

### Lines 141-160

````cpp
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | Digraphs |
             GNUMode, 201402)
LANGSTANDARD_ALIAS_DEPR(gnucxx14, "gnu++1y")

LANGSTANDARD(cxx17, "c++17",
             CXX, "ISO C++ 2017 with amendments",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             Digraphs | HexFloat, 201703)
LANGSTANDARD_ALIAS_DEPR(cxx17, "c++1z")

LANGSTANDARD(gnucxx17, "gnu++17",
             CXX, "ISO C++ 2017 with amendments and GNU extensions",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             Digraphs | HexFloat | GNUMode, 201703)
LANGSTANDARD_ALIAS_DEPR(gnucxx17, "gnu++1z")

LANGSTANDARD(cxx20, "c++20",
             CXX, "ISO C++ 2020 DIS",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             CPlusPlus20 | Digraphs | HexFloat, 202002)
````
- **L141 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | Digraphs |`.
  **L141 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | Digraphs |`。
- **L142 EN**: Continues the surrounding expression or declaration: `GNUMode, 201402)`.
  **L142 CN**: 继续构造周围的表达式或声明：`GNUMode, 201402)`。
- **L143 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L143 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L144 EN**: Blank line separating nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L145 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L145 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2017 with amendments",`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2017 with amendments",`。
- **L147 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L147 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L148 EN**: Continues the surrounding expression or declaration: `Digraphs | HexFloat, 201703)`.
  **L148 CN**: 继续构造周围的表达式或声明：`Digraphs | HexFloat, 201703)`。
- **L149 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L149 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L151 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L151 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2017 with amendments and GNU extensions",`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2017 with amendments and GNU extensions",`。
- **L153 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L153 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L154 EN**: Continues the surrounding expression or declaration: `Digraphs | HexFloat | GNUMode, 201703)`.
  **L154 CN**: 继续构造周围的表达式或声明：`Digraphs | HexFloat | GNUMode, 201703)`。
- **L155 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L155 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L157 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L157 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2020 DIS",`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2020 DIS",`。
- **L159 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L159 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L160 EN**: Continues the surrounding expression or declaration: `CPlusPlus20 | Digraphs | HexFloat, 202002)`.
  **L160 CN**: 继续构造周围的表达式或声明：`CPlusPlus20 | Digraphs | HexFloat, 202002)`。

### Lines 161-180

````cpp
LANGSTANDARD_ALIAS_DEPR(cxx20, "c++2a")

LANGSTANDARD(gnucxx20, "gnu++20",
             CXX, "ISO C++ 2020 DIS with GNU extensions",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             CPlusPlus20 | Digraphs | HexFloat | GNUMode, 202002)
LANGSTANDARD_ALIAS_DEPR(gnucxx20, "gnu++2a")

LANGSTANDARD(cxx23, "c++23",
             CXX, "ISO C++ 2023 DIS",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             CPlusPlus20 | CPlusPlus23 | Digraphs | HexFloat, 202302)
LANGSTANDARD_ALIAS_DEPR(cxx23, "c++2b")

LANGSTANDARD(gnucxx23, "gnu++23",
             CXX, "ISO C++ 2023 DIS with GNU extensions",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             CPlusPlus20 | CPlusPlus23 | Digraphs | HexFloat | GNUMode, 202302)
LANGSTANDARD_ALIAS_DEPR(gnucxx23, "gnu++2b")

````
- **L161 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L161 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L163 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L163 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2020 DIS with GNU extensions",`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2020 DIS with GNU extensions",`。
- **L165 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L165 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L166 EN**: Continues the surrounding expression or declaration: `CPlusPlus20 | Digraphs | HexFloat | GNUMode, 202002)`.
  **L166 CN**: 继续构造周围的表达式或声明：`CPlusPlus20 | Digraphs | HexFloat | GNUMode, 202002)`。
- **L167 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L167 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L169 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L169 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2023 DIS",`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2023 DIS",`。
- **L171 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L171 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L172 EN**: Continues the surrounding expression or declaration: `CPlusPlus20 | CPlusPlus23 | Digraphs | HexFloat, 202302)`.
  **L172 CN**: 继续构造周围的表达式或声明：`CPlusPlus20 | CPlusPlus23 | Digraphs | HexFloat, 202302)`。
- **L173 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L173 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L175 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L175 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "ISO C++ 2023 DIS with GNU extensions",`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "ISO C++ 2023 DIS with GNU extensions",`。
- **L177 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L177 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L178 EN**: Continues the surrounding expression or declaration: `CPlusPlus20 | CPlusPlus23 | Digraphs | HexFloat | GNUMode, 202302)`.
  **L178 CN**: 继续构造周围的表达式或声明：`CPlusPlus20 | CPlusPlus23 | Digraphs | HexFloat | GNUMode, 202302)`。
- **L179 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L179 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 181-200

````cpp
// FIXME: Use correct version code for C++26 once published.
LANGSTANDARD(cxx26, "c++2c",
             CXX, "Working draft for C++2c",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             CPlusPlus20 | CPlusPlus23 | CPlusPlus26 | Digraphs | HexFloat, 202400)
LANGSTANDARD_ALIAS(cxx26, "c++26")

LANGSTANDARD(gnucxx26, "gnu++2c",
             CXX, "Working draft for C++2c with GNU extensions",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             CPlusPlus20 | CPlusPlus23 | CPlusPlus26 | Digraphs | HexFloat | GNUMode, 202400)
LANGSTANDARD_ALIAS(gnucxx26, "gnu++26")

// OpenCL
LANGSTANDARD(opencl10, "cl1.0",
             OpenCL, "OpenCL 1.0",
             LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)
LANGSTANDARD_ALIAS_DEPR(opencl10, "cl")

LANGSTANDARD(opencl11, "cl1.1",
````
- **L181 EN**: Comment records a pending task or caution: `FIXME: Use correct version code for C++26 once published.`.
  **L181 CN**: 注释记录待办事项或注意点：`FIXME: Use correct version code for C++26 once published.`。
- **L182 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L182 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "Working draft for C++2c",`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "Working draft for C++2c",`。
- **L184 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L184 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L185 EN**: Continues the surrounding expression or declaration: `CPlusPlus20 | CPlusPlus23 | CPlusPlus26 | Digraphs | HexFloat, 202400)`.
  **L185 CN**: 继续构造周围的表达式或声明：`CPlusPlus20 | CPlusPlus23 | CPlusPlus26 | Digraphs | HexFloat, 202400)`。
- **L186 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L186 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L188 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L188 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CXX, "Working draft for C++2c with GNU extensions",`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`CXX, "Working draft for C++2c with GNU extensions",`。
- **L190 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L190 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L191 EN**: Continues the surrounding expression or declaration: `CPlusPlus20 | CPlusPlus23 | CPlusPlus26 | Digraphs | HexFloat | GNUMode, 202400)`.
  **L191 CN**: 继续构造周围的表达式或声明：`CPlusPlus20 | CPlusPlus23 | CPlusPlus26 | Digraphs | HexFloat | GNUMode, 202400)`。
- **L192 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L192 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `OpenCL`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`OpenCL`。
- **L195 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L195 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpenCL, "OpenCL 1.0",`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpenCL, "OpenCL 1.0",`。
- **L197 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`.
  **L197 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`。
- **L198 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L198 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L200 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L200 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。

### Lines 201-220

````cpp
             OpenCL, "OpenCL 1.1",
             LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)
LANGSTANDARD(opencl12, "cl1.2",
             OpenCL, "OpenCL 1.2",
             LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)
LANGSTANDARD(opencl20, "cl2.0",
             OpenCL, "OpenCL 2.0",
             LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)
LANGSTANDARD(opencl30, "cl3.0",
             OpenCL, "OpenCL 3.0",
             LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)

LANGSTANDARD(openclcpp10, "clc++1.0",
             OpenCL, "C++ for OpenCL 1.0",
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             Digraphs | HexFloat | OpenCL, std::nullopt)
LANGSTANDARD_ALIAS(openclcpp10, "clc++")

LANGSTANDARD(openclcpp2021, "clc++2021",
             OpenCL, "C++ for OpenCL 2021",
````
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpenCL, "OpenCL 1.1",`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpenCL, "OpenCL 1.1",`。
- **L202 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`.
  **L202 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`。
- **L203 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L203 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpenCL, "OpenCL 1.2",`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpenCL, "OpenCL 1.2",`。
- **L205 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`.
  **L205 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`。
- **L206 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L206 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpenCL, "OpenCL 2.0",`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpenCL, "OpenCL 2.0",`。
- **L208 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`.
  **L208 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`。
- **L209 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L209 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpenCL, "OpenCL 3.0",`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpenCL, "OpenCL 3.0",`。
- **L211 EN**: Continues the surrounding expression or declaration: `LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`.
  **L211 CN**: 继续构造周围的表达式或声明：`LineComment | C99 | Digraphs | HexFloat | OpenCL, std::nullopt)`。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L213 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L213 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpenCL, "C++ for OpenCL 1.0",`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpenCL, "C++ for OpenCL 1.0",`。
- **L215 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L215 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L216 EN**: Continues the surrounding expression or declaration: `Digraphs | HexFloat | OpenCL, std::nullopt)`.
  **L216 CN**: 继续构造周围的表达式或声明：`Digraphs | HexFloat | OpenCL, std::nullopt)`。
- **L217 EN**: Invokes macro `LANGSTANDARD_ALIAS` to contribute one entry to a table-driven definition list.
  **L217 CN**: 调用宏 `LANGSTANDARD_ALIAS`，向表驱动定义列表贡献一个条目。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L219 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L219 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpenCL, "C++ for OpenCL 2021",`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpenCL, "C++ for OpenCL 2021",`。

### Lines 221-240

````cpp
             LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |
             Digraphs | HexFloat | OpenCL, std::nullopt)

LANGSTANDARD_ALIAS_DEPR(opencl10, "CL")
LANGSTANDARD_ALIAS_DEPR(opencl11, "CL1.1")
LANGSTANDARD_ALIAS_DEPR(opencl12, "CL1.2")
LANGSTANDARD_ALIAS_DEPR(opencl20, "CL2.0")
LANGSTANDARD_ALIAS_DEPR(opencl30, "CL3.0")
LANGSTANDARD_ALIAS_DEPR(openclcpp10, "CLC++")
LANGSTANDARD_ALIAS_DEPR(openclcpp10, "CLC++1.0")
LANGSTANDARD_ALIAS_DEPR(openclcpp2021, "CLC++2021")

// HLSL
LANGSTANDARD(hlsl, "hlsl",
             HLSL, "High Level Shader Language",
             LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)

LANGSTANDARD(hlsl2015, "hlsl2015",
             HLSL, "High Level Shader Language 2015",
             LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)
````
- **L221 EN**: Continues the surrounding expression or declaration: `LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`.
  **L221 CN**: 继续构造周围的表达式或声明：`LineComment | CPlusPlus | CPlusPlus11 | CPlusPlus14 | CPlusPlus17 |`。
- **L222 EN**: Continues the surrounding expression or declaration: `Digraphs | HexFloat | OpenCL, std::nullopt)`.
  **L222 CN**: 继续构造周围的表达式或声明：`Digraphs | HexFloat | OpenCL, std::nullopt)`。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L224 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L224 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L225 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L225 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L226 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L226 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L227 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L227 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L228 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L228 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L229 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L229 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L230 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L230 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L231 EN**: Invokes macro `LANGSTANDARD_ALIAS_DEPR` to contribute one entry to a table-driven definition list.
  **L231 CN**: 调用宏 `LANGSTANDARD_ALIAS_DEPR`，向表驱动定义列表贡献一个条目。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `HLSL`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HLSL`。
- **L234 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L234 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSL, "High Level Shader Language",`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSL, "High Level Shader Language",`。
- **L236 EN**: Continues the surrounding expression or declaration: `LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`.
  **L236 CN**: 继续构造周围的表达式或声明：`LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L238 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L238 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSL, "High Level Shader Language 2015",`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSL, "High Level Shader Language 2015",`。
- **L240 EN**: Continues the surrounding expression or declaration: `LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`.
  **L240 CN**: 继续构造周围的表达式或声明：`LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`。

### Lines 241-260

````cpp

LANGSTANDARD(hlsl2016, "hlsl2016",
             HLSL, "High Level Shader Language 2016",
             LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)

LANGSTANDARD(hlsl2017, "hlsl2017",
             HLSL, "High Level Shader Language 2017",
             LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)

LANGSTANDARD(hlsl2018, "hlsl2018",
             HLSL, "High Level Shader Language 2018",
             LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)

LANGSTANDARD(hlsl2021, "hlsl2021",
             HLSL, "High Level Shader Language 2021",
             LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)

LANGSTANDARD(hlsl202x, "hlsl202x",
             HLSL, "High Level Shader Language 202x",
             LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)
````
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L242 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L242 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSL, "High Level Shader Language 2016",`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSL, "High Level Shader Language 2016",`。
- **L244 EN**: Continues the surrounding expression or declaration: `LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`.
  **L244 CN**: 继续构造周围的表达式或声明：`LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L246 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L246 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSL, "High Level Shader Language 2017",`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSL, "High Level Shader Language 2017",`。
- **L248 EN**: Continues the surrounding expression or declaration: `LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`.
  **L248 CN**: 继续构造周围的表达式或声明：`LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L250 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L250 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSL, "High Level Shader Language 2018",`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSL, "High Level Shader Language 2018",`。
- **L252 EN**: Continues the surrounding expression or declaration: `LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`.
  **L252 CN**: 继续构造周围的表达式或声明：`LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`。
- **L253 EN**: Blank line separating nearby declarations or logic blocks.
  **L253 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L254 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L254 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSL, "High Level Shader Language 2021",`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSL, "High Level Shader Language 2021",`。
- **L256 EN**: Continues the surrounding expression or declaration: `LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`.
  **L256 CN**: 继续构造周围的表达式或声明：`LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`。
- **L257 EN**: Blank line separating nearby declarations or logic blocks.
  **L257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L258 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L258 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSL, "High Level Shader Language 202x",`.
  **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSL, "High Level Shader Language 202x",`。
- **L260 EN**: Continues the surrounding expression or declaration: `LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`.
  **L260 CN**: 继续构造周围的表达式或声明：`LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`。

### Lines 261-269

````cpp

LANGSTANDARD(hlsl202y, "hlsl202y",
             HLSL, "High Level Shader Language 202y",
             LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)


#undef LANGSTANDARD
#undef LANGSTANDARD_ALIAS
#undef LANGSTANDARD_ALIAS_DEPR
````
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L262 EN**: Invokes macro `LANGSTANDARD` to contribute one entry to a table-driven definition list.
  **L262 CN**: 调用宏 `LANGSTANDARD`，向表驱动定义列表贡献一个条目。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HLSL, "High Level Shader Language 202y",`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`HLSL, "High Level Shader Language 202y",`。
- **L264 EN**: Continues the surrounding expression or declaration: `LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`.
  **L264 CN**: 继续构造周围的表达式或声明：`LineComment | HLSL | CPlusPlus | CPlusPlus11, std::nullopt)`。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L267 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef LANGSTANDARD`.
  **L267 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef LANGSTANDARD`。
- **L268 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef LANGSTANDARD_ALIAS`.
  **L268 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef LANGSTANDARD_ALIAS`。
- **L269 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef LANGSTANDARD_ALIAS_DEPR`.
  **L269 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef LANGSTANDARD_ALIAS_DEPR`。

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
- **OpenCL integration / OpenCL 集成**
  - **EN**: Represents OpenCL-specific qualifiers, builtins, or declarative metadata.
  - **CN**: 表示 OpenCL 专用限定符、builtin 或声明式元数据。
- **HLSL integration / HLSL 集成**
  - **EN**: Carries HLSL-specific address spaces, builtins, or declarative metadata.
  - **CN**: 承载 HLSL 专用地址空间、builtin 或声明式元数据。
- **Macro-driven generation / 宏驱动生成**
  - **EN**: Relies on macros or generated include fragments to keep large definition sets synchronized.
  - **CN**: 依赖宏或生成式 include 片段来同步大规模定义集合。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: `LANGSTANDARD_ALIAS(IDENT,`, `LANGSTANDARD_ALIAS_DEPR(IDENT,`
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `LANGSTANDARD`, `LANGSTANDARD_ALIAS`, `LANGSTANDARD_ALIAS_DEPR`
- **TableGen records / TableGen 记录**: No prominent TableGen record names detected. / 未检测到明显的 TableGen 记录名称。
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
