# Fortran-parsers.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Parser/Fortran-parsers.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Top-level grammar specification for Fortran. These parsers drive the tokenization parsers in cooked-tokens.h to consume characters, recognize the productions of Fortran, and to construct a parse tree. See ParserCombinators.md for documentation on the parser co
- **Purpose (CN)**: 实现 Fortran parsers 相关的解析、语法树支持或源码级处理流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Parser/Fortran-parsers.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// Top-level grammar specification for Fortran.  These parsers drive
// the tokenization parsers in cooked-tokens.h to consume characters,
// recognize the productions of Fortran, and to construct a parse tree.
// See ParserCombinators.md for documentation on the parser combinator
// library used here to implement an LL recursive descent recognizer.

// The productions that follow are derived from the draft Fortran 2018
// standard, with some necessary modifications to remove left recursion
// and some generalization in order to defer cases where parses depend
// on the definitions of symbols.  The "Rxxx" numbers that appear in
// comments refer to these numbered requirements in the Fortran standard.

// The whole Fortran grammar originally constituted one header file,
// but that turned out to require more memory to compile with current
// C++ compilers than some people were willing to accept, so now the
// various per-type parsers are partitioned into several C++ source
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, intent, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的逻辑、意图或元数据：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, intent, or metadata: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的逻辑、意图或元数据：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, intent, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的逻辑、意图或元数据：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `Top-level grammar specification for Fortran.  These parsers drive`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`Top-level grammar specification for Fortran.  These parsers drive`。
- **L10 EN**: Comment explains nearby logic, intent, or metadata: `the tokenization parsers in cooked-tokens.h to consume characters,`.
  **L10 CN**: 注释说明附近代码的逻辑、意图或元数据：`the tokenization parsers in cooked-tokens.h to consume characters,`。
- **L11 EN**: Comment explains nearby logic, intent, or metadata: `recognize the productions of Fortran, and to construct a parse tree.`.
  **L11 CN**: 注释说明附近代码的逻辑、意图或元数据：`recognize the productions of Fortran, and to construct a parse tree.`。
- **L12 EN**: Comment explains nearby logic, intent, or metadata: `See ParserCombinators.md for documentation on the parser combinator`.
  **L12 CN**: 注释说明附近代码的逻辑、意图或元数据：`See ParserCombinators.md for documentation on the parser combinator`。
- **L13 EN**: Comment explains nearby logic, intent, or metadata: `library used here to implement an LL recursive descent recognizer.`.
  **L13 CN**: 注释说明附近代码的逻辑、意图或元数据：`library used here to implement an LL recursive descent recognizer.`。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Comment explains nearby logic, intent, or metadata: `The productions that follow are derived from the draft Fortran 2018`.
  **L15 CN**: 注释说明附近代码的逻辑、意图或元数据：`The productions that follow are derived from the draft Fortran 2018`。
- **L16 EN**: Comment explains nearby logic, intent, or metadata: `standard, with some necessary modifications to remove left recursion`.
  **L16 CN**: 注释说明附近代码的逻辑、意图或元数据：`standard, with some necessary modifications to remove left recursion`。
- **L17 EN**: Comment explains nearby logic, intent, or metadata: `and some generalization in order to defer cases where parses depend`.
  **L17 CN**: 注释说明附近代码的逻辑、意图或元数据：`and some generalization in order to defer cases where parses depend`。
- **L18 EN**: Comment explains nearby logic, intent, or metadata: `on the definitions of symbols.  The "Rxxx" numbers that appear in`.
  **L18 CN**: 注释说明附近代码的逻辑、意图或元数据：`on the definitions of symbols.  The "Rxxx" numbers that appear in`。
- **L19 EN**: Comment explains nearby logic, intent, or metadata: `comments refer to these numbered requirements in the Fortran standard.`.
  **L19 CN**: 注释说明附近代码的逻辑、意图或元数据：`comments refer to these numbered requirements in the Fortran standard.`。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Comment explains nearby logic, intent, or metadata: `The whole Fortran grammar originally constituted one header file,`.
  **L21 CN**: 注释说明附近代码的逻辑、意图或元数据：`The whole Fortran grammar originally constituted one header file,`。
- **L22 EN**: Comment explains nearby logic, intent, or metadata: `but that turned out to require more memory to compile with current`.
  **L22 CN**: 注释说明附近代码的逻辑、意图或元数据：`but that turned out to require more memory to compile with current`。
- **L23 EN**: Comment explains nearby logic, intent, or metadata: `C++ compilers than some people were willing to accept, so now the`.
  **L23 CN**: 注释说明附近代码的逻辑、意图或元数据：`C++ compilers than some people were willing to accept, so now the`。
- **L24 EN**: Comment explains nearby logic, intent, or metadata: `various per-type parsers are partitioned into several C++ source`.
  **L24 CN**: 注释说明附近代码的逻辑、意图或元数据：`various per-type parsers are partitioned into several C++ source`。

### Lines 25-48

````cpp
// files.  This file contains parsers for constants, types, declarations,
// and misfits (mostly clauses 7, 8, & 9 of Fortran 2018).  The others:
//  executable-parsers.cpp  Executable statements
//  expr-parsers.cpp        Expressions
//  io-parsers.cpp          I/O statements and FORMAT
//  openmp-parsers.cpp      OpenMP directives
//  program-parsers.cpp     Program units

#include "basic-parsers.h"
#include "expr-parsers.h"
#include "misc-parsers.h"
#include "stmt-parser.h"
#include "token-parsers.h"
#include "type-parser-implementation.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Parser/user-state.h"

namespace Fortran::parser {

// R601 alphanumeric-character -> letter | digit | underscore
// R603 name -> letter [alphanumeric-character]...
constexpr auto nonDigitIdChar{letter || otherIdChar};
constexpr auto rawName{nonDigitIdChar >> many(nonDigitIdChar || digit)};
TYPE_PARSER(space >> sourced(rawName >> construct<Name>()))
````
- **L25 EN**: Comment explains nearby logic, intent, or metadata: `files.  This file contains parsers for constants, types, declarations,`.
  **L25 CN**: 注释说明附近代码的逻辑、意图或元数据：`files.  This file contains parsers for constants, types, declarations,`。
- **L26 EN**: Comment explains nearby logic, intent, or metadata: `and misfits (mostly clauses 7, 8, & 9 of Fortran 2018).  The others:`.
  **L26 CN**: 注释说明附近代码的逻辑、意图或元数据：`and misfits (mostly clauses 7, 8, & 9 of Fortran 2018).  The others:`。
- **L27 EN**: Comment explains nearby logic, intent, or metadata: `executable-parsers.cpp  Executable statements`.
  **L27 CN**: 注释说明附近代码的逻辑、意图或元数据：`executable-parsers.cpp  Executable statements`。
- **L28 EN**: Comment explains nearby logic, intent, or metadata: `expr-parsers.cpp        Expressions`.
  **L28 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr-parsers.cpp        Expressions`。
- **L29 EN**: Comment explains nearby logic, intent, or metadata: `io-parsers.cpp          I/O statements and FORMAT`.
  **L29 CN**: 注释说明附近代码的逻辑、意图或元数据：`io-parsers.cpp          I/O statements and FORMAT`。
- **L30 EN**: Comment explains nearby logic, intent, or metadata: `openmp-parsers.cpp      OpenMP directives`.
  **L30 CN**: 注释说明附近代码的逻辑、意图或元数据：`openmp-parsers.cpp      OpenMP directives`。
- **L31 EN**: Comment explains nearby logic, intent, or metadata: `program-parsers.cpp     Program units`.
  **L31 CN**: 注释说明附近代码的逻辑、意图或元数据：`program-parsers.cpp     Program units`。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Includes "basic-parsers.h" to access local declarations paired with this implementation.
  **L33 CN**: 引入 "basic-parsers.h" 以使用与该实现配套的本地声明。
- **L34 EN**: Includes "expr-parsers.h" to access local declarations paired with this implementation.
  **L34 CN**: 引入 "expr-parsers.h" 以使用与该实现配套的本地声明。
- **L35 EN**: Includes "misc-parsers.h" to access local declarations paired with this implementation.
  **L35 CN**: 引入 "misc-parsers.h" 以使用与该实现配套的本地声明。
- **L36 EN**: Includes "stmt-parser.h" to access local declarations paired with this implementation.
  **L36 CN**: 引入 "stmt-parser.h" 以使用与该实现配套的本地声明。
- **L37 EN**: Includes "token-parsers.h" to access local declarations paired with this implementation.
  **L37 CN**: 引入 "token-parsers.h" 以使用与该实现配套的本地声明。
- **L38 EN**: Includes "type-parser-implementation.h" to access local declarations paired with this implementation.
  **L38 CN**: 引入 "type-parser-implementation.h" 以使用与该实现配套的本地声明。
- **L39 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L39 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L40 EN**: Includes "flang/Parser/user-state.h" to access parse-tree, token, or source representation support.
  **L40 CN**: 引入 "flang/Parser/user-state.h" 以使用语法树、词法单元或源码表示支持。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Opens namespace scope `Fortran::parser`.
  **L42 CN**: 打开命名空间作用域 `Fortran::parser`。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Comment explains nearby logic, intent, or metadata: `R601 alphanumeric-character -> letter | digit | underscore`.
  **L44 CN**: 注释说明附近代码的逻辑、意图或元数据：`R601 alphanumeric-character -> letter | digit | underscore`。
- **L45 EN**: Comment explains nearby logic, intent, or metadata: `R603 name -> letter [alphanumeric-character]...`.
  **L45 CN**: 注释说明附近代码的逻辑、意图或元数据：`R603 name -> letter [alphanumeric-character]...`。
- **L46 EN**: Executes a standalone statement or declaration: `constexpr auto nonDigitIdChar{letter || otherIdChar};`.
  **L46 CN**: 执行一条独立语句或声明：`constexpr auto nonDigitIdChar{letter || otherIdChar};`。
- **L47 EN**: Executes a call or declaration centered on `many`.
  **L47 CN**: 执行以 `many` 为核心的调用或声明。
- **L48 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L48 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。

### Lines 49-72

````cpp

// R608 intrinsic-operator ->
//        power-op | mult-op | add-op | concat-op | rel-op |
//        not-op | and-op | or-op | equiv-op
// R610 extended-intrinsic-op -> intrinsic-operator
// These parsers must be ordered carefully to avoid misrecognition.
constexpr auto namedIntrinsicOperator{
    ".LT." >> pure(DefinedOperator::IntrinsicOperator::LT) ||
    ".LE." >> pure(DefinedOperator::IntrinsicOperator::LE) ||
    ".EQ." >> pure(DefinedOperator::IntrinsicOperator::EQ) ||
    ".NE." >> pure(DefinedOperator::IntrinsicOperator::NE) ||
    ".GE." >> pure(DefinedOperator::IntrinsicOperator::GE) ||
    ".GT." >> pure(DefinedOperator::IntrinsicOperator::GT) ||
    ".NOT." >> pure(DefinedOperator::IntrinsicOperator::NOT) ||
    ".AND." >> pure(DefinedOperator::IntrinsicOperator::AND) ||
    ".OR." >> pure(DefinedOperator::IntrinsicOperator::OR) ||
    ".EQV." >> pure(DefinedOperator::IntrinsicOperator::EQV) ||
    ".NEQV." >> pure(DefinedOperator::IntrinsicOperator::NEQV) ||
    extension<LanguageFeature::XOROperator>(
        "nonstandard usage: .XOR. spelling of .NEQV."_port_en_US,
        ".XOR." >> pure(DefinedOperator::IntrinsicOperator::NEQV)) ||
    extension<LanguageFeature::LogicalAbbreviations>(
        "nonstandard usage: abbreviated logical operator"_port_en_US,
        ".N." >> pure(DefinedOperator::IntrinsicOperator::NOT) ||
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Comment explains nearby logic, intent, or metadata: `R608 intrinsic-operator ->`.
  **L50 CN**: 注释说明附近代码的逻辑、意图或元数据：`R608 intrinsic-operator ->`。
- **L51 EN**: Comment explains nearby logic, intent, or metadata: `power-op | mult-op | add-op | concat-op | rel-op |`.
  **L51 CN**: 注释说明附近代码的逻辑、意图或元数据：`power-op | mult-op | add-op | concat-op | rel-op |`。
- **L52 EN**: Comment explains nearby logic, intent, or metadata: `not-op | and-op | or-op | equiv-op`.
  **L52 CN**: 注释说明附近代码的逻辑、意图或元数据：`not-op | and-op | or-op | equiv-op`。
- **L53 EN**: Comment explains nearby logic, intent, or metadata: `R610 extended-intrinsic-op -> intrinsic-operator`.
  **L53 CN**: 注释说明附近代码的逻辑、意图或元数据：`R610 extended-intrinsic-op -> intrinsic-operator`。
- **L54 EN**: Comment explains nearby logic, intent, or metadata: `These parsers must be ordered carefully to avoid misrecognition.`.
  **L54 CN**: 注释说明附近代码的逻辑、意图或元数据：`These parsers must be ordered carefully to avoid misrecognition.`。
- **L55 EN**: Continues the surrounding expression or declaration: `constexpr auto namedIntrinsicOperator{`.
  **L55 CN**: 继续构造周围的表达式或声明：`constexpr auto namedIntrinsicOperator{`。
- **L56 EN**: Continues logic associated with callable symbol `pure`.
  **L56 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L57 EN**: Continues logic associated with callable symbol `pure`.
  **L57 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L58 EN**: Continues logic associated with callable symbol `pure`.
  **L58 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L59 EN**: Continues logic associated with callable symbol `pure`.
  **L59 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L60 EN**: Continues logic associated with callable symbol `pure`.
  **L60 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L61 EN**: Continues logic associated with callable symbol `pure`.
  **L61 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L62 EN**: Continues logic associated with callable symbol `pure`.
  **L62 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L63 EN**: Continues logic associated with callable symbol `pure`.
  **L63 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L64 EN**: Continues logic associated with callable symbol `pure`.
  **L64 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L65 EN**: Continues logic associated with callable symbol `pure`.
  **L65 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L66 EN**: Continues logic associated with callable symbol `pure`.
  **L66 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L67 EN**: Continues logic associated with callable symbol `XOROperator>`.
  **L67 CN**: 继续与可调用符号 `XOROperator>` 相关的逻辑。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: .XOR. spelling of .NEQV."_port_en_US,`.
  **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: .XOR. spelling of .NEQV."_port_en_US,`。
- **L69 EN**: Continues logic associated with callable symbol `pure`.
  **L69 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L70 EN**: Continues logic associated with callable symbol `LogicalAbbreviations>`.
  **L70 CN**: 继续与可调用符号 `LogicalAbbreviations>` 相关的逻辑。
- **L71 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: abbreviated logical operator"_port_en_US,`.
  **L71 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: abbreviated logical operator"_port_en_US,`。
- **L72 EN**: Continues logic associated with callable symbol `pure`.
  **L72 CN**: 继续与可调用符号 `pure` 相关的逻辑。

### Lines 73-96

````cpp
            ".A." >> pure(DefinedOperator::IntrinsicOperator::AND) ||
            ".O." >> pure(DefinedOperator::IntrinsicOperator::OR) ||
            extension<LanguageFeature::XOROperator>(
                "nonstandard usage: .X. spelling of .NEQV."_port_en_US,
                ".X." >> pure(DefinedOperator::IntrinsicOperator::NEQV)))};

constexpr auto intrinsicOperator{
    "**" >> pure(DefinedOperator::IntrinsicOperator::Power) ||
    "*" >> pure(DefinedOperator::IntrinsicOperator::Multiply) ||
    "//" >> pure(DefinedOperator::IntrinsicOperator::Concat) ||
    "/=" >> pure(DefinedOperator::IntrinsicOperator::NE) ||
    "/" >> pure(DefinedOperator::IntrinsicOperator::Divide) ||
    "+" >> pure(DefinedOperator::IntrinsicOperator::Add) ||
    "-" >> pure(DefinedOperator::IntrinsicOperator::Subtract) ||
    "<=" >> pure(DefinedOperator::IntrinsicOperator::LE) ||
    extension<LanguageFeature::AlternativeNE>(
        "nonstandard usage: <> spelling of /= or .NE."_port_en_US,
        "<>" >> pure(DefinedOperator::IntrinsicOperator::NE)) ||
    "<" >> pure(DefinedOperator::IntrinsicOperator::LT) ||
    "==" >> pure(DefinedOperator::IntrinsicOperator::EQ) ||
    ">=" >> pure(DefinedOperator::IntrinsicOperator::GE) ||
    ">" >> pure(DefinedOperator::IntrinsicOperator::GT) ||
    namedIntrinsicOperator};

````
- **L73 EN**: Continues logic associated with callable symbol `pure`.
  **L73 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L74 EN**: Continues logic associated with callable symbol `pure`.
  **L74 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L75 EN**: Continues logic associated with callable symbol `XOROperator>`.
  **L75 CN**: 继续与可调用符号 `XOROperator>` 相关的逻辑。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: .X. spelling of .NEQV."_port_en_US,`.
  **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: .X. spelling of .NEQV."_port_en_US,`。
- **L77 EN**: Executes a call or declaration centered on `pure`.
  **L77 CN**: 执行以 `pure` 为核心的调用或声明。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L79 EN**: Continues the surrounding expression or declaration: `constexpr auto intrinsicOperator{`.
  **L79 CN**: 继续构造周围的表达式或声明：`constexpr auto intrinsicOperator{`。
- **L80 EN**: Continues logic associated with callable symbol `pure`.
  **L80 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L81 EN**: Continues logic associated with callable symbol `pure`.
  **L81 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L82 EN**: Continues logic associated with callable symbol `pure`.
  **L82 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L83 EN**: Continues logic associated with callable symbol `pure`.
  **L83 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L84 EN**: Continues logic associated with callable symbol `pure`.
  **L84 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L85 EN**: Continues logic associated with callable symbol `pure`.
  **L85 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L86 EN**: Continues logic associated with callable symbol `pure`.
  **L86 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L87 EN**: Continues logic associated with callable symbol `pure`.
  **L87 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L88 EN**: Continues logic associated with callable symbol `AlternativeNE>`.
  **L88 CN**: 继续与可调用符号 `AlternativeNE>` 相关的逻辑。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: <> spelling of /= or .NE."_port_en_US,`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: <> spelling of /= or .NE."_port_en_US,`。
- **L90 EN**: Continues logic associated with callable symbol `pure`.
  **L90 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L91 EN**: Continues logic associated with callable symbol `pure`.
  **L91 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L92 EN**: Continues logic associated with callable symbol `pure`.
  **L92 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L93 EN**: Continues logic associated with callable symbol `pure`.
  **L93 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L94 EN**: Continues logic associated with callable symbol `pure`.
  **L94 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L95 EN**: Executes a standalone statement or declaration: `namedIntrinsicOperator};`.
  **L95 CN**: 执行一条独立语句或声明：`namedIntrinsicOperator};`。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 97-120

````cpp
// R609 defined-operator ->
//        defined-unary-op | defined-binary-op | extended-intrinsic-op
TYPE_PARSER(construct<DefinedOperator>(intrinsicOperator) ||
    construct<DefinedOperator>(definedOpName))

// R505 implicit-part -> [implicit-part-stmt]... implicit-stmt
// N.B. PARAMETER, FORMAT, & ENTRY statements that appear before any
// other kind of declaration-construct will be parsed into the
// implicit-part.
TYPE_CONTEXT_PARSER("implicit part"_en_US,
    construct<ImplicitPart>(many(Parser<ImplicitPartStmt>{})))

// R506 implicit-part-stmt ->
//         implicit-stmt | parameter-stmt | format-stmt | entry-stmt
TYPE_PARSER(first(
    construct<ImplicitPartStmt>(statement(indirect(Parser<ImplicitStmt>{}))),
    construct<ImplicitPartStmt>(statement(indirect(parameterStmt))),
    construct<ImplicitPartStmt>(statement(indirect(oldParameterStmt))),
    construct<ImplicitPartStmt>(statement(indirect(formatStmt))),
    construct<ImplicitPartStmt>(statement(indirect(entryStmt))),
    construct<ImplicitPartStmt>(indirect(compilerDirective)),
    construct<ImplicitPartStmt>(indirect(openaccDeclarativeConstruct))))

// R512 internal-subprogram -> function-subprogram | subroutine-subprogram
````
- **L97 EN**: Comment explains nearby logic, intent, or metadata: `R609 defined-operator ->`.
  **L97 CN**: 注释说明附近代码的逻辑、意图或元数据：`R609 defined-operator ->`。
- **L98 EN**: Comment explains nearby logic, intent, or metadata: `defined-unary-op | defined-binary-op | extended-intrinsic-op`.
  **L98 CN**: 注释说明附近代码的逻辑、意图或元数据：`defined-unary-op | defined-binary-op | extended-intrinsic-op`。
- **L99 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L99 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L100 EN**: Continues logic associated with callable symbol `construct<DefinedOperator>`.
  **L100 CN**: 继续与可调用符号 `construct<DefinedOperator>` 相关的逻辑。
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L102 EN**: Comment explains nearby logic, intent, or metadata: `R505 implicit-part -> [implicit-part-stmt]... implicit-stmt`.
  **L102 CN**: 注释说明附近代码的逻辑、意图或元数据：`R505 implicit-part -> [implicit-part-stmt]... implicit-stmt`。
- **L103 EN**: Comment explains nearby logic, intent, or metadata: `N.B. PARAMETER, FORMAT, & ENTRY statements that appear before any`.
  **L103 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. PARAMETER, FORMAT, & ENTRY statements that appear before any`。
- **L104 EN**: Comment explains nearby logic, intent, or metadata: `other kind of declaration-construct will be parsed into the`.
  **L104 CN**: 注释说明附近代码的逻辑、意图或元数据：`other kind of declaration-construct will be parsed into the`。
- **L105 EN**: Comment explains nearby logic, intent, or metadata: `implicit-part.`.
  **L105 CN**: 注释说明附近代码的逻辑、意图或元数据：`implicit-part.`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("implicit part"_en_US,`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("implicit part"_en_US,`。
- **L107 EN**: Continues logic associated with callable symbol `construct<ImplicitPart>`.
  **L107 CN**: 继续与可调用符号 `construct<ImplicitPart>` 相关的逻辑。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L109 EN**: Comment explains nearby logic, intent, or metadata: `R506 implicit-part-stmt ->`.
  **L109 CN**: 注释说明附近代码的逻辑、意图或元数据：`R506 implicit-part-stmt ->`。
- **L110 EN**: Comment explains nearby logic, intent, or metadata: `implicit-stmt | parameter-stmt | format-stmt | entry-stmt`.
  **L110 CN**: 注释说明附近代码的逻辑、意图或元数据：`implicit-stmt | parameter-stmt | format-stmt | entry-stmt`。
- **L111 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L111 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ImplicitPartStmt>(statement(indirect(Parser<ImplicitStmt>{}))),`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ImplicitPartStmt>(statement(indirect(Parser<ImplicitStmt>{}))),`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ImplicitPartStmt>(statement(indirect(parameterStmt))),`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ImplicitPartStmt>(statement(indirect(parameterStmt))),`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ImplicitPartStmt>(statement(indirect(oldParameterStmt))),`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ImplicitPartStmt>(statement(indirect(oldParameterStmt))),`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ImplicitPartStmt>(statement(indirect(formatStmt))),`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ImplicitPartStmt>(statement(indirect(formatStmt))),`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ImplicitPartStmt>(statement(indirect(entryStmt))),`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ImplicitPartStmt>(statement(indirect(entryStmt))),`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ImplicitPartStmt>(indirect(compilerDirective)),`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ImplicitPartStmt>(indirect(compilerDirective)),`。
- **L118 EN**: Continues logic associated with callable symbol `construct<ImplicitPartStmt>`.
  **L118 CN**: 继续与可调用符号 `construct<ImplicitPartStmt>` 相关的逻辑。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L120 EN**: Comment explains nearby logic, intent, or metadata: `R512 internal-subprogram -> function-subprogram | subroutine-subprogram`.
  **L120 CN**: 注释说明附近代码的逻辑、意图或元数据：`R512 internal-subprogram -> function-subprogram | subroutine-subprogram`。

### Lines 121-144

````cpp
// Internal subprograms are not program units, so their END statements
// can be followed by ';' and another statement on the same line.
TYPE_CONTEXT_PARSER("internal subprogram"_en_US,
    (construct<InternalSubprogram>(indirect(functionSubprogram)) ||
        construct<InternalSubprogram>(indirect(subroutineSubprogram))) /
            forceEndOfStmt ||
        construct<InternalSubprogram>(indirect(compilerDirective)))

// R511 internal-subprogram-part -> contains-stmt [internal-subprogram]...
TYPE_CONTEXT_PARSER("internal subprogram part"_en_US,
    construct<InternalSubprogramPart>(statement(containsStmt),
        many(StartNewSubprogram{} >> Parser<InternalSubprogram>{})))

// R605 literal-constant ->
//        int-literal-constant | real-literal-constant |
//        complex-literal-constant | logical-literal-constant |
//        char-literal-constant | boz-literal-constant |
//        unsigned-literal-constant
TYPE_PARSER(
    first(construct<LiteralConstant>(Parser<HollerithLiteralConstant>{}),
        construct<LiteralConstant>(realLiteralConstant),
        construct<LiteralConstant>(intLiteralConstant),
        construct<LiteralConstant>(Parser<ComplexLiteralConstant>{}),
        construct<LiteralConstant>(Parser<BOZLiteralConstant>{}),
````
- **L121 EN**: Comment explains nearby logic, intent, or metadata: `Internal subprograms are not program units, so their END statements`.
  **L121 CN**: 注释说明附近代码的逻辑、意图或元数据：`Internal subprograms are not program units, so their END statements`。
- **L122 EN**: Comment explains nearby logic, intent, or metadata: `can be followed by ';' and another statement on the same line.`.
  **L122 CN**: 注释说明附近代码的逻辑、意图或元数据：`can be followed by ';' and another statement on the same line.`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("internal subprogram"_en_US,`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("internal subprogram"_en_US,`。
- **L124 EN**: Continues logic associated with callable symbol `construct<InternalSubprogram>`.
  **L124 CN**: 继续与可调用符号 `construct<InternalSubprogram>` 相关的逻辑。
- **L125 EN**: Continues logic associated with callable symbol `construct<InternalSubprogram>`.
  **L125 CN**: 继续与可调用符号 `construct<InternalSubprogram>` 相关的逻辑。
- **L126 EN**: Continues the surrounding expression or declaration: `forceEndOfStmt ||`.
  **L126 CN**: 继续构造周围的表达式或声明：`forceEndOfStmt ||`。
- **L127 EN**: Continues logic associated with callable symbol `construct<InternalSubprogram>`.
  **L127 CN**: 继续与可调用符号 `construct<InternalSubprogram>` 相关的逻辑。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L129 EN**: Comment explains nearby logic, intent, or metadata: `R511 internal-subprogram-part -> contains-stmt [internal-subprogram]...`.
  **L129 CN**: 注释说明附近代码的逻辑、意图或元数据：`R511 internal-subprogram-part -> contains-stmt [internal-subprogram]...`。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("internal subprogram part"_en_US,`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("internal subprogram part"_en_US,`。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<InternalSubprogramPart>(statement(containsStmt),`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<InternalSubprogramPart>(statement(containsStmt),`。
- **L132 EN**: Continues logic associated with callable symbol `many`.
  **L132 CN**: 继续与可调用符号 `many` 相关的逻辑。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L134 EN**: Comment explains nearby logic, intent, or metadata: `R605 literal-constant ->`.
  **L134 CN**: 注释说明附近代码的逻辑、意图或元数据：`R605 literal-constant ->`。
- **L135 EN**: Comment explains nearby logic, intent, or metadata: `int-literal-constant | real-literal-constant |`.
  **L135 CN**: 注释说明附近代码的逻辑、意图或元数据：`int-literal-constant | real-literal-constant |`。
- **L136 EN**: Comment explains nearby logic, intent, or metadata: `complex-literal-constant | logical-literal-constant |`.
  **L136 CN**: 注释说明附近代码的逻辑、意图或元数据：`complex-literal-constant | logical-literal-constant |`。
- **L137 EN**: Comment explains nearby logic, intent, or metadata: `char-literal-constant | boz-literal-constant |`.
  **L137 CN**: 注释说明附近代码的逻辑、意图或元数据：`char-literal-constant | boz-literal-constant |`。
- **L138 EN**: Comment explains nearby logic, intent, or metadata: `unsigned-literal-constant`.
  **L138 CN**: 注释说明附近代码的逻辑、意图或元数据：`unsigned-literal-constant`。
- **L139 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L139 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first(construct<LiteralConstant>(Parser<HollerithLiteralConstant>{}),`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`first(construct<LiteralConstant>(Parser<HollerithLiteralConstant>{}),`。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<LiteralConstant>(realLiteralConstant),`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<LiteralConstant>(realLiteralConstant),`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<LiteralConstant>(intLiteralConstant),`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<LiteralConstant>(intLiteralConstant),`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<LiteralConstant>(Parser<ComplexLiteralConstant>{}),`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<LiteralConstant>(Parser<ComplexLiteralConstant>{}),`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<LiteralConstant>(Parser<BOZLiteralConstant>{}),`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<LiteralConstant>(Parser<BOZLiteralConstant>{}),`。

### Lines 145-168

````cpp
        construct<LiteralConstant>(charLiteralConstant),
        construct<LiteralConstant>(Parser<LogicalLiteralConstant>{}),
        construct<LiteralConstant>(unsignedLiteralConstant)))

// R606 named-constant -> name
TYPE_PARSER(construct<NamedConstant>(name))

// R701 type-param-value -> scalar-int-expr | * | :
TYPE_PARSER(construct<TypeParamValue>(scalarIntExpr) ||
    construct<TypeParamValue>(star) ||
    construct<TypeParamValue>(construct<TypeParamValue::Deferred>(":"_tok)))

// R702 type-spec -> intrinsic-type-spec | derived-type-spec
// N.B. This type-spec production is one of two instances in the Fortran
// grammar where intrinsic types and bare derived type names can clash;
// the other is below in R703 declaration-type-spec.  Look-ahead is required
// to disambiguate the cases where a derived type name begins with the name
// of an intrinsic type, e.g., REALITY.
TYPE_CONTEXT_PARSER("type spec"_en_US,
    construct<TypeSpec>(intrinsicTypeSpec / lookAhead("::"_tok || ")"_tok)) ||
        construct<TypeSpec>(derivedTypeSpec))

// R703 declaration-type-spec ->
//        intrinsic-type-spec | TYPE ( intrinsic-type-spec ) |
````
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<LiteralConstant>(charLiteralConstant),`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<LiteralConstant>(charLiteralConstant),`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<LiteralConstant>(Parser<LogicalLiteralConstant>{}),`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<LiteralConstant>(Parser<LogicalLiteralConstant>{}),`。
- **L147 EN**: Continues logic associated with callable symbol `construct<LiteralConstant>`.
  **L147 CN**: 继续与可调用符号 `construct<LiteralConstant>` 相关的逻辑。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L149 EN**: Comment explains nearby logic, intent, or metadata: `R606 named-constant -> name`.
  **L149 CN**: 注释说明附近代码的逻辑、意图或元数据：`R606 named-constant -> name`。
- **L150 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L150 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L152 EN**: Comment explains nearby logic, intent, or metadata: `R701 type-param-value -> scalar-int-expr | * | :`.
  **L152 CN**: 注释说明附近代码的逻辑、意图或元数据：`R701 type-param-value -> scalar-int-expr | * | :`。
- **L153 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L153 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L154 EN**: Continues logic associated with callable symbol `construct<TypeParamValue>`.
  **L154 CN**: 继续与可调用符号 `construct<TypeParamValue>` 相关的逻辑。
- **L155 EN**: Continues logic associated with callable symbol `construct<TypeParamValue>`.
  **L155 CN**: 继续与可调用符号 `construct<TypeParamValue>` 相关的逻辑。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L157 EN**: Comment explains nearby logic, intent, or metadata: `R702 type-spec -> intrinsic-type-spec | derived-type-spec`.
  **L157 CN**: 注释说明附近代码的逻辑、意图或元数据：`R702 type-spec -> intrinsic-type-spec | derived-type-spec`。
- **L158 EN**: Comment explains nearby logic, intent, or metadata: `N.B. This type-spec production is one of two instances in the Fortran`.
  **L158 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. This type-spec production is one of two instances in the Fortran`。
- **L159 EN**: Comment explains nearby logic, intent, or metadata: `grammar where intrinsic types and bare derived type names can clash;`.
  **L159 CN**: 注释说明附近代码的逻辑、意图或元数据：`grammar where intrinsic types and bare derived type names can clash;`。
- **L160 EN**: Comment explains nearby logic, intent, or metadata: `the other is below in R703 declaration-type-spec.  Look-ahead is required`.
  **L160 CN**: 注释说明附近代码的逻辑、意图或元数据：`the other is below in R703 declaration-type-spec.  Look-ahead is required`。
- **L161 EN**: Comment explains nearby logic, intent, or metadata: `to disambiguate the cases where a derived type name begins with the name`.
  **L161 CN**: 注释说明附近代码的逻辑、意图或元数据：`to disambiguate the cases where a derived type name begins with the name`。
- **L162 EN**: Comment explains nearby logic, intent, or metadata: `of an intrinsic type, e.g., REALITY.`.
  **L162 CN**: 注释说明附近代码的逻辑、意图或元数据：`of an intrinsic type, e.g., REALITY.`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("type spec"_en_US,`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("type spec"_en_US,`。
- **L164 EN**: Continues logic associated with callable symbol `construct<TypeSpec>`.
  **L164 CN**: 继续与可调用符号 `construct<TypeSpec>` 相关的逻辑。
- **L165 EN**: Continues logic associated with callable symbol `construct<TypeSpec>`.
  **L165 CN**: 继续与可调用符号 `construct<TypeSpec>` 相关的逻辑。
- **L166 EN**: Blank line separating nearby declarations or logic blocks.
  **L166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L167 EN**: Comment explains nearby logic, intent, or metadata: `R703 declaration-type-spec ->`.
  **L167 CN**: 注释说明附近代码的逻辑、意图或元数据：`R703 declaration-type-spec ->`。
- **L168 EN**: Comment explains nearby logic, intent, or metadata: `intrinsic-type-spec | TYPE ( intrinsic-type-spec ) |`.
  **L168 CN**: 注释说明附近代码的逻辑、意图或元数据：`intrinsic-type-spec | TYPE ( intrinsic-type-spec ) |`。

### Lines 169-192

````cpp
//        TYPE ( derived-type-spec ) | CLASS ( derived-type-spec ) |
//        CLASS ( * ) | TYPE ( * )
// N.B. It is critical to distribute "parenthesized()" over the alternatives
// for TYPE (...), rather than putting the alternatives within it, which
// would fail on "TYPE(real_derived)" with a misrecognition of "real" as an
// intrinsic-type-spec.
// N.B. TYPE(x) is a derived type if x is a one-word extension intrinsic
// type (BYTE or DOUBLECOMPLEX), not the extension intrinsic type.
TYPE_CONTEXT_PARSER("declaration type spec"_en_US,
    construct<DeclarationTypeSpec>(intrinsicTypeSpec) ||
        "TYPE" >>
            (parenthesized(construct<DeclarationTypeSpec>(
                 !"DOUBLECOMPLEX"_tok >> !"BYTE"_tok >> intrinsicTypeSpec)) ||
                parenthesized(construct<DeclarationTypeSpec>(
                    construct<DeclarationTypeSpec::Type>(derivedTypeSpec))) ||
                construct<DeclarationTypeSpec>(
                    "( * )" >> construct<DeclarationTypeSpec::TypeStar>())) ||
        "CLASS" >> parenthesized(construct<DeclarationTypeSpec>(
                                     construct<DeclarationTypeSpec::Class>(
                                         derivedTypeSpec)) ||
                       construct<DeclarationTypeSpec>("*" >>
                           construct<DeclarationTypeSpec::ClassStar>())) ||
        extension<LanguageFeature::DECStructures>(
            "nonstandard usage: STRUCTURE"_port_en_US,
````
- **L169 EN**: Comment explains nearby logic, intent, or metadata: `TYPE ( derived-type-spec ) | CLASS ( derived-type-spec ) |`.
  **L169 CN**: 注释说明附近代码的逻辑、意图或元数据：`TYPE ( derived-type-spec ) | CLASS ( derived-type-spec ) |`。
- **L170 EN**: Comment explains nearby logic, intent, or metadata: `CLASS ( * ) | TYPE ( * )`.
  **L170 CN**: 注释说明附近代码的逻辑、意图或元数据：`CLASS ( * ) | TYPE ( * )`。
- **L171 EN**: Comment explains nearby logic, intent, or metadata: `N.B. It is critical to distribute "parenthesized()" over the alternatives`.
  **L171 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. It is critical to distribute "parenthesized()" over the alternatives`。
- **L172 EN**: Comment explains nearby logic, intent, or metadata: `for TYPE (...), rather than putting the alternatives within it, which`.
  **L172 CN**: 注释说明附近代码的逻辑、意图或元数据：`for TYPE (...), rather than putting the alternatives within it, which`。
- **L173 EN**: Comment explains nearby logic, intent, or metadata: `would fail on "TYPE(real_derived)" with a misrecognition of "real" as an`.
  **L173 CN**: 注释说明附近代码的逻辑、意图或元数据：`would fail on "TYPE(real_derived)" with a misrecognition of "real" as an`。
- **L174 EN**: Comment explains nearby logic, intent, or metadata: `intrinsic-type-spec.`.
  **L174 CN**: 注释说明附近代码的逻辑、意图或元数据：`intrinsic-type-spec.`。
- **L175 EN**: Comment explains nearby logic, intent, or metadata: `N.B. TYPE(x) is a derived type if x is a one-word extension intrinsic`.
  **L175 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. TYPE(x) is a derived type if x is a one-word extension intrinsic`。
- **L176 EN**: Comment explains nearby logic, intent, or metadata: `type (BYTE or DOUBLECOMPLEX), not the extension intrinsic type.`.
  **L176 CN**: 注释说明附近代码的逻辑、意图或元数据：`type (BYTE or DOUBLECOMPLEX), not the extension intrinsic type.`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("declaration type spec"_en_US,`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("declaration type spec"_en_US,`。
- **L178 EN**: Continues logic associated with callable symbol `construct<DeclarationTypeSpec>`.
  **L178 CN**: 继续与可调用符号 `construct<DeclarationTypeSpec>` 相关的逻辑。
- **L179 EN**: Continues the surrounding expression or declaration: `"TYPE" >>`.
  **L179 CN**: 继续构造周围的表达式或声明：`"TYPE" >>`。
- **L180 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L180 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L181 EN**: Continues the surrounding expression or declaration: `!"DOUBLECOMPLEX"_tok >> !"BYTE"_tok >> intrinsicTypeSpec)) ||`.
  **L181 CN**: 继续构造周围的表达式或声明：`!"DOUBLECOMPLEX"_tok >> !"BYTE"_tok >> intrinsicTypeSpec)) ||`。
- **L182 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L182 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L183 EN**: Continues logic associated with callable symbol `Type>`.
  **L183 CN**: 继续与可调用符号 `Type>` 相关的逻辑。
- **L184 EN**: Continues logic associated with callable symbol `construct<DeclarationTypeSpec>`.
  **L184 CN**: 继续与可调用符号 `construct<DeclarationTypeSpec>` 相关的逻辑。
- **L185 EN**: Continues logic associated with callable symbol `TypeStar>`.
  **L185 CN**: 继续与可调用符号 `TypeStar>` 相关的逻辑。
- **L186 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L186 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L187 EN**: Continues logic associated with callable symbol `Class>`.
  **L187 CN**: 继续与可调用符号 `Class>` 相关的逻辑。
- **L188 EN**: Continues the surrounding expression or declaration: `derivedTypeSpec)) ||`.
  **L188 CN**: 继续构造周围的表达式或声明：`derivedTypeSpec)) ||`。
- **L189 EN**: Continues logic associated with callable symbol `construct<DeclarationTypeSpec>`.
  **L189 CN**: 继续与可调用符号 `construct<DeclarationTypeSpec>` 相关的逻辑。
- **L190 EN**: Continues logic associated with callable symbol `ClassStar>`.
  **L190 CN**: 继续与可调用符号 `ClassStar>` 相关的逻辑。
- **L191 EN**: Continues logic associated with callable symbol `DECStructures>`.
  **L191 CN**: 继续与可调用符号 `DECStructures>` 相关的逻辑。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: STRUCTURE"_port_en_US,`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: STRUCTURE"_port_en_US,`。

### Lines 193-216

````cpp
            construct<DeclarationTypeSpec>(
                // As is also done for the STRUCTURE statement, the name of
                // the structure includes the surrounding slashes to avoid
                // name clashes.
                construct<DeclarationTypeSpec::Record>(
                    "RECORD" >> sourced("/" >> name / "/")))) ||
        construct<DeclarationTypeSpec>(vectorTypeSpec))

// R704 intrinsic-type-spec ->
//        integer-type-spec | REAL [kind-selector] | DOUBLE PRECISION |
//        COMPLEX [kind-selector] | CHARACTER [char-selector] |
//        LOGICAL [kind-selector]
// Extensions: DOUBLE COMPLEX, BYTE
TYPE_CONTEXT_PARSER("intrinsic type spec"_en_US,
    first(construct<IntrinsicTypeSpec>(integerTypeSpec),
        construct<IntrinsicTypeSpec>(
            construct<IntrinsicTypeSpec::Real>("REAL" >> maybe(kindSelector))),
        construct<IntrinsicTypeSpec>("DOUBLE PRECISION" >>
            construct<IntrinsicTypeSpec::DoublePrecision>()),
        construct<IntrinsicTypeSpec>(construct<IntrinsicTypeSpec::Complex>(
            "COMPLEX" >> maybe(kindSelector))),
        construct<IntrinsicTypeSpec>(construct<IntrinsicTypeSpec::Character>(
            "CHARACTER" >> maybe(Parser<CharSelector>{}))),
        construct<IntrinsicTypeSpec>(construct<IntrinsicTypeSpec::Logical>(
````
- **L193 EN**: Continues logic associated with callable symbol `construct<DeclarationTypeSpec>`.
  **L193 CN**: 继续与可调用符号 `construct<DeclarationTypeSpec>` 相关的逻辑。
- **L194 EN**: Comment explains nearby logic, intent, or metadata: `As is also done for the STRUCTURE statement, the name of`.
  **L194 CN**: 注释说明附近代码的逻辑、意图或元数据：`As is also done for the STRUCTURE statement, the name of`。
- **L195 EN**: Comment explains nearby logic, intent, or metadata: `the structure includes the surrounding slashes to avoid`.
  **L195 CN**: 注释说明附近代码的逻辑、意图或元数据：`the structure includes the surrounding slashes to avoid`。
- **L196 EN**: Comment explains nearby logic, intent, or metadata: `name clashes.`.
  **L196 CN**: 注释说明附近代码的逻辑、意图或元数据：`name clashes.`。
- **L197 EN**: Continues logic associated with callable symbol `Record>`.
  **L197 CN**: 继续与可调用符号 `Record>` 相关的逻辑。
- **L198 EN**: Continues logic associated with callable symbol `sourced`.
  **L198 CN**: 继续与可调用符号 `sourced` 相关的逻辑。
- **L199 EN**: Continues logic associated with callable symbol `construct<DeclarationTypeSpec>`.
  **L199 CN**: 继续与可调用符号 `construct<DeclarationTypeSpec>` 相关的逻辑。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L201 EN**: Comment explains nearby logic, intent, or metadata: `R704 intrinsic-type-spec ->`.
  **L201 CN**: 注释说明附近代码的逻辑、意图或元数据：`R704 intrinsic-type-spec ->`。
- **L202 EN**: Comment explains nearby logic, intent, or metadata: `integer-type-spec | REAL [kind-selector] | DOUBLE PRECISION |`.
  **L202 CN**: 注释说明附近代码的逻辑、意图或元数据：`integer-type-spec | REAL [kind-selector] | DOUBLE PRECISION |`。
- **L203 EN**: Comment explains nearby logic, intent, or metadata: `COMPLEX [kind-selector] | CHARACTER [char-selector] |`.
  **L203 CN**: 注释说明附近代码的逻辑、意图或元数据：`COMPLEX [kind-selector] | CHARACTER [char-selector] |`。
- **L204 EN**: Comment explains nearby logic, intent, or metadata: `LOGICAL [kind-selector]`.
  **L204 CN**: 注释说明附近代码的逻辑、意图或元数据：`LOGICAL [kind-selector]`。
- **L205 EN**: Comment explains nearby logic, intent, or metadata: `Extensions: DOUBLE COMPLEX, BYTE`.
  **L205 CN**: 注释说明附近代码的逻辑、意图或元数据：`Extensions: DOUBLE COMPLEX, BYTE`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("intrinsic type spec"_en_US,`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("intrinsic type spec"_en_US,`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first(construct<IntrinsicTypeSpec>(integerTypeSpec),`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`first(construct<IntrinsicTypeSpec>(integerTypeSpec),`。
- **L208 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L208 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<IntrinsicTypeSpec::Real>("REAL" >> maybe(kindSelector))),`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<IntrinsicTypeSpec::Real>("REAL" >> maybe(kindSelector))),`。
- **L210 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L210 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<IntrinsicTypeSpec::DoublePrecision>()),`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<IntrinsicTypeSpec::DoublePrecision>()),`。
- **L212 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L212 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"COMPLEX" >> maybe(kindSelector))),`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`"COMPLEX" >> maybe(kindSelector))),`。
- **L214 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L214 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CHARACTER" >> maybe(Parser<CharSelector>{}))),`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CHARACTER" >> maybe(Parser<CharSelector>{}))),`。
- **L216 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L216 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。

### Lines 217-240

````cpp
            "LOGICAL" >> maybe(kindSelector))),
        construct<IntrinsicTypeSpec>(unsignedTypeSpec),
        extension<LanguageFeature::DoubleComplex>(
            "nonstandard usage: DOUBLE COMPLEX"_port_en_US,
            construct<IntrinsicTypeSpec>("DOUBLE COMPLEX"_sptok >>
                construct<IntrinsicTypeSpec::DoubleComplex>())),
        extension<LanguageFeature::Byte>("nonstandard usage: BYTE"_port_en_US,
            construct<IntrinsicTypeSpec>(construct<IntegerTypeSpec>(
                "BYTE" >> construct<std::optional<KindSelector>>(pure(1)))))))

// Extension: Vector type
// VECTOR(intrinsic-type-spec) | __VECTOR_PAIR | __VECTOR_QUAD
TYPE_CONTEXT_PARSER("vector type spec"_en_US,
    extension<LanguageFeature::PPCVector>(
        "nonstandard usage: Vector type"_port_en_US,
        first(construct<VectorTypeSpec>(intrinsicVectorTypeSpec),
            construct<VectorTypeSpec>("__VECTOR_PAIR" >>
                construct<VectorTypeSpec::PairVectorTypeSpec>()),
            construct<VectorTypeSpec>("__VECTOR_QUAD" >>
                construct<VectorTypeSpec::QuadVectorTypeSpec>()))))

// VECTOR(integer-type-spec) | VECTOR(real-type-spec) |
// VECTOR(unsigned-type-spec) |
TYPE_PARSER(construct<IntrinsicVectorTypeSpec>("VECTOR" >>
````
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"LOGICAL" >> maybe(kindSelector))),`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`"LOGICAL" >> maybe(kindSelector))),`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<IntrinsicTypeSpec>(unsignedTypeSpec),`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<IntrinsicTypeSpec>(unsignedTypeSpec),`。
- **L219 EN**: Continues logic associated with callable symbol `DoubleComplex>`.
  **L219 CN**: 继续与可调用符号 `DoubleComplex>` 相关的逻辑。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: DOUBLE COMPLEX"_port_en_US,`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: DOUBLE COMPLEX"_port_en_US,`。
- **L221 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L221 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<IntrinsicTypeSpec::DoubleComplex>())),`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<IntrinsicTypeSpec::DoubleComplex>())),`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `extension<LanguageFeature::Byte>("nonstandard usage: BYTE"_port_en_US,`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`extension<LanguageFeature::Byte>("nonstandard usage: BYTE"_port_en_US,`。
- **L224 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L224 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L225 EN**: Continues logic associated with callable symbol `optional<KindSelector>>`.
  **L225 CN**: 继续与可调用符号 `optional<KindSelector>>` 相关的逻辑。
- **L226 EN**: Blank line separating nearby declarations or logic blocks.
  **L226 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L227 EN**: Comment explains nearby logic, intent, or metadata: `Extension: Vector type`.
  **L227 CN**: 注释说明附近代码的逻辑、意图或元数据：`Extension: Vector type`。
- **L228 EN**: Comment explains nearby logic, intent, or metadata: `VECTOR(intrinsic-type-spec) | __VECTOR_PAIR | __VECTOR_QUAD`.
  **L228 CN**: 注释说明附近代码的逻辑、意图或元数据：`VECTOR(intrinsic-type-spec) | __VECTOR_PAIR | __VECTOR_QUAD`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("vector type spec"_en_US,`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("vector type spec"_en_US,`。
- **L230 EN**: Continues logic associated with callable symbol `PPCVector>`.
  **L230 CN**: 继续与可调用符号 `PPCVector>` 相关的逻辑。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: Vector type"_port_en_US,`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: Vector type"_port_en_US,`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first(construct<VectorTypeSpec>(intrinsicVectorTypeSpec),`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`first(construct<VectorTypeSpec>(intrinsicVectorTypeSpec),`。
- **L233 EN**: Continues logic associated with callable symbol `construct<VectorTypeSpec>`.
  **L233 CN**: 继续与可调用符号 `construct<VectorTypeSpec>` 相关的逻辑。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<VectorTypeSpec::PairVectorTypeSpec>()),`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<VectorTypeSpec::PairVectorTypeSpec>()),`。
- **L235 EN**: Continues logic associated with callable symbol `construct<VectorTypeSpec>`.
  **L235 CN**: 继续与可调用符号 `construct<VectorTypeSpec>` 相关的逻辑。
- **L236 EN**: Continues logic associated with callable symbol `QuadVectorTypeSpec>`.
  **L236 CN**: 继续与可调用符号 `QuadVectorTypeSpec>` 相关的逻辑。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L238 EN**: Comment explains nearby logic, intent, or metadata: `VECTOR(integer-type-spec) | VECTOR(real-type-spec) |`.
  **L238 CN**: 注释说明附近代码的逻辑、意图或元数据：`VECTOR(integer-type-spec) | VECTOR(real-type-spec) |`。
- **L239 EN**: Comment explains nearby logic, intent, or metadata: `VECTOR(unsigned-type-spec) |`.
  **L239 CN**: 注释说明附近代码的逻辑、意图或元数据：`VECTOR(unsigned-type-spec) |`。
- **L240 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L240 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。

### Lines 241-264

````cpp
    parenthesized(construct<VectorElementType>(integerTypeSpec) ||
        construct<VectorElementType>(unsignedTypeSpec) ||
        construct<VectorElementType>(construct<IntrinsicTypeSpec::Real>(
            "REAL" >> maybe(kindSelector))))))

// UNSIGNED type
TYPE_PARSER(construct<UnsignedTypeSpec>("UNSIGNED" >> maybe(kindSelector)))

// R705 integer-type-spec -> INTEGER [kind-selector]
TYPE_PARSER(construct<IntegerTypeSpec>("INTEGER" >> maybe(kindSelector)))

// R706 kind-selector -> ( [KIND =] scalar-int-constant-expr )
// Legacy extension: kind-selector -> * digit-string
TYPE_PARSER(construct<KindSelector>(
                parenthesized(maybe("KIND ="_tok) >> scalarIntConstantExpr)) ||
    extension<LanguageFeature::StarKind>(
        "nonstandard usage: TYPE*KIND syntax"_port_en_US,
        construct<KindSelector>(construct<KindSelector::StarSize>(
            "*" >> digitString64 / spaceCheck))))

constexpr auto noSpace{
    recovery(withMessage("invalid space"_err_en_US, !" "_ch), space)};

// R707 signed-int-literal-constant -> [sign] int-literal-constant
````
- **L241 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L241 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L242 EN**: Continues logic associated with callable symbol `construct<VectorElementType>`.
  **L242 CN**: 继续与可调用符号 `construct<VectorElementType>` 相关的逻辑。
- **L243 EN**: Continues logic associated with callable symbol `construct<VectorElementType>`.
  **L243 CN**: 继续与可调用符号 `construct<VectorElementType>` 相关的逻辑。
- **L244 EN**: Continues logic associated with callable symbol `maybe`.
  **L244 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L246 EN**: Comment explains nearby logic, intent, or metadata: `UNSIGNED type`.
  **L246 CN**: 注释说明附近代码的逻辑、意图或元数据：`UNSIGNED type`。
- **L247 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L247 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L249 EN**: Comment explains nearby logic, intent, or metadata: `R705 integer-type-spec -> INTEGER [kind-selector]`.
  **L249 CN**: 注释说明附近代码的逻辑、意图或元数据：`R705 integer-type-spec -> INTEGER [kind-selector]`。
- **L250 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L250 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L252 EN**: Comment explains nearby logic, intent, or metadata: `R706 kind-selector -> ( [KIND =] scalar-int-constant-expr )`.
  **L252 CN**: 注释说明附近代码的逻辑、意图或元数据：`R706 kind-selector -> ( [KIND =] scalar-int-constant-expr )`。
- **L253 EN**: Comment explains nearby logic, intent, or metadata: `Legacy extension: kind-selector -> * digit-string`.
  **L253 CN**: 注释说明附近代码的逻辑、意图或元数据：`Legacy extension: kind-selector -> * digit-string`。
- **L254 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L254 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L255 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L255 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L256 EN**: Continues logic associated with callable symbol `StarKind>`.
  **L256 CN**: 继续与可调用符号 `StarKind>` 相关的逻辑。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: TYPE*KIND syntax"_port_en_US,`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: TYPE*KIND syntax"_port_en_US,`。
- **L258 EN**: Continues logic associated with callable symbol `construct<KindSelector>`.
  **L258 CN**: 继续与可调用符号 `construct<KindSelector>` 相关的逻辑。
- **L259 EN**: Continues the surrounding expression or declaration: `"*" >> digitString64 / spaceCheck))))`.
  **L259 CN**: 继续构造周围的表达式或声明：`"*" >> digitString64 / spaceCheck))))`。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L261 EN**: Continues the surrounding expression or declaration: `constexpr auto noSpace{`.
  **L261 CN**: 继续构造周围的表达式或声明：`constexpr auto noSpace{`。
- **L262 EN**: Executes a call or declaration centered on `recovery`.
  **L262 CN**: 执行以 `recovery` 为核心的调用或声明。
- **L263 EN**: Blank line separating nearby declarations or logic blocks.
  **L263 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L264 EN**: Comment explains nearby logic, intent, or metadata: `R707 signed-int-literal-constant -> [sign] int-literal-constant`.
  **L264 CN**: 注释说明附近代码的逻辑、意图或元数据：`R707 signed-int-literal-constant -> [sign] int-literal-constant`。

### Lines 265-288

````cpp
TYPE_PARSER(sourced(
    construct<SignedIntLiteralConstant>(SignedIntLiteralConstantWithoutKind{},
        maybe(noSpace >> underscore >> noSpace >> kindParam))))

// R708 int-literal-constant -> digit-string [_ kind-param]
// The negated look-ahead for a trailing underscore prevents misrecognition
// when the digit string is a numeric kind parameter of a character literal.
TYPE_PARSER(construct<IntLiteralConstant>(space >> digitString / !"u"_ch,
    maybe(underscore >> noSpace >> kindParam) / !underscore))

// unsigned-literal-constant -> digit-string U [_ kind-param]
TYPE_PARSER(construct<UnsignedLiteralConstant>(space >> digitString / "u"_ch,
    maybe(underscore >> noSpace >> kindParam) / !underscore))

// R709 kind-param -> digit-string | scalar-int-constant-name
TYPE_PARSER(construct<KindParam>(digitString64) ||
    construct<KindParam>(
        scalar(integer(constant(sourced(rawName >> construct<Name>()))))))

// R712 sign -> + | -
// N.B. A sign constitutes a whole token, so a space is allowed in free form
// after the sign and before a real-literal-constant or
// complex-literal-constant.  A sign is not a unary operator in these contexts.
constexpr auto sign{
````
- **L265 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L265 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SignedIntLiteralConstant>(SignedIntLiteralConstantWithoutKind{},`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SignedIntLiteralConstant>(SignedIntLiteralConstantWithoutKind{},`。
- **L267 EN**: Continues logic associated with callable symbol `maybe`.
  **L267 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L269 EN**: Comment explains nearby logic, intent, or metadata: `R708 int-literal-constant -> digit-string [_ kind-param]`.
  **L269 CN**: 注释说明附近代码的逻辑、意图或元数据：`R708 int-literal-constant -> digit-string [_ kind-param]`。
- **L270 EN**: Comment explains nearby logic, intent, or metadata: `The negated look-ahead for a trailing underscore prevents misrecognition`.
  **L270 CN**: 注释说明附近代码的逻辑、意图或元数据：`The negated look-ahead for a trailing underscore prevents misrecognition`。
- **L271 EN**: Comment explains nearby logic, intent, or metadata: `when the digit string is a numeric kind parameter of a character literal.`.
  **L271 CN**: 注释说明附近代码的逻辑、意图或元数据：`when the digit string is a numeric kind parameter of a character literal.`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<IntLiteralConstant>(space >> digitString / !"u"_ch,`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<IntLiteralConstant>(space >> digitString / !"u"_ch,`。
- **L273 EN**: Continues logic associated with callable symbol `maybe`.
  **L273 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L275 EN**: Comment explains nearby logic, intent, or metadata: `unsigned-literal-constant -> digit-string U [_ kind-param]`.
  **L275 CN**: 注释说明附近代码的逻辑、意图或元数据：`unsigned-literal-constant -> digit-string U [_ kind-param]`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<UnsignedLiteralConstant>(space >> digitString / "u"_ch,`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<UnsignedLiteralConstant>(space >> digitString / "u"_ch,`。
- **L277 EN**: Continues logic associated with callable symbol `maybe`.
  **L277 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L279 EN**: Comment explains nearby logic, intent, or metadata: `R709 kind-param -> digit-string | scalar-int-constant-name`.
  **L279 CN**: 注释说明附近代码的逻辑、意图或元数据：`R709 kind-param -> digit-string | scalar-int-constant-name`。
- **L280 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L280 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L281 EN**: Continues logic associated with callable symbol `construct<KindParam>`.
  **L281 CN**: 继续与可调用符号 `construct<KindParam>` 相关的逻辑。
- **L282 EN**: Continues logic associated with callable symbol `scalar`.
  **L282 CN**: 继续与可调用符号 `scalar` 相关的逻辑。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L284 EN**: Comment explains nearby logic, intent, or metadata: `R712 sign -> + | -`.
  **L284 CN**: 注释说明附近代码的逻辑、意图或元数据：`R712 sign -> + | -`。
- **L285 EN**: Comment explains nearby logic, intent, or metadata: `N.B. A sign constitutes a whole token, so a space is allowed in free form`.
  **L285 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. A sign constitutes a whole token, so a space is allowed in free form`。
- **L286 EN**: Comment explains nearby logic, intent, or metadata: `after the sign and before a real-literal-constant or`.
  **L286 CN**: 注释说明附近代码的逻辑、意图或元数据：`after the sign and before a real-literal-constant or`。
- **L287 EN**: Comment explains nearby logic, intent, or metadata: `complex-literal-constant.  A sign is not a unary operator in these contexts.`.
  **L287 CN**: 注释说明附近代码的逻辑、意图或元数据：`complex-literal-constant.  A sign is not a unary operator in these contexts.`。
- **L288 EN**: Continues the surrounding expression or declaration: `constexpr auto sign{`.
  **L288 CN**: 继续构造周围的表达式或声明：`constexpr auto sign{`。

### Lines 289-312

````cpp
    "+"_tok >> pure(Sign::Positive) || "-"_tok >> pure(Sign::Negative)};

// R713 signed-real-literal-constant -> [sign] real-literal-constant
constexpr auto signedRealLiteralConstant{
    construct<SignedRealLiteralConstant>(maybe(sign), realLiteralConstant)};

// R714 real-literal-constant ->
//        significand [exponent-letter exponent] [_ kind-param] |
//        digit-string exponent-letter exponent [_ kind-param]
// R715 significand -> digit-string . [digit-string] | . digit-string
// R716 exponent-letter -> E | D
// Extension: Q
// R717 exponent -> signed-digit-string
constexpr auto exponentPart{
    ("ed"_ch ||
        extension<LanguageFeature::QuadPrecision>(
            "nonstandard usage: Q exponent"_port_en_US, "q"_ch)) >>
    SignedDigitString{}};

TYPE_CONTEXT_PARSER("REAL literal constant"_en_US,
    space >>
        construct<RealLiteralConstant>(
            sourced((digitString >> "."_ch >>
                            !(some(letter) >>
````
- **L289 EN**: Executes a call or declaration centered on `pure`.
  **L289 CN**: 执行以 `pure` 为核心的调用或声明。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L291 EN**: Comment explains nearby logic, intent, or metadata: `R713 signed-real-literal-constant -> [sign] real-literal-constant`.
  **L291 CN**: 注释说明附近代码的逻辑、意图或元数据：`R713 signed-real-literal-constant -> [sign] real-literal-constant`。
- **L292 EN**: Continues the surrounding expression or declaration: `constexpr auto signedRealLiteralConstant{`.
  **L292 CN**: 继续构造周围的表达式或声明：`constexpr auto signedRealLiteralConstant{`。
- **L293 EN**: Executes a call or declaration centered on `construct<SignedRealLiteralConstant>`.
  **L293 CN**: 执行以 `construct<SignedRealLiteralConstant>` 为核心的调用或声明。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L295 EN**: Comment explains nearby logic, intent, or metadata: `R714 real-literal-constant ->`.
  **L295 CN**: 注释说明附近代码的逻辑、意图或元数据：`R714 real-literal-constant ->`。
- **L296 EN**: Comment explains nearby logic, intent, or metadata: `significand [exponent-letter exponent] [_ kind-param] |`.
  **L296 CN**: 注释说明附近代码的逻辑、意图或元数据：`significand [exponent-letter exponent] [_ kind-param] |`。
- **L297 EN**: Comment explains nearby logic, intent, or metadata: `digit-string exponent-letter exponent [_ kind-param]`.
  **L297 CN**: 注释说明附近代码的逻辑、意图或元数据：`digit-string exponent-letter exponent [_ kind-param]`。
- **L298 EN**: Comment explains nearby logic, intent, or metadata: `R715 significand -> digit-string . [digit-string] | . digit-string`.
  **L298 CN**: 注释说明附近代码的逻辑、意图或元数据：`R715 significand -> digit-string . [digit-string] | . digit-string`。
- **L299 EN**: Comment explains nearby logic, intent, or metadata: `R716 exponent-letter -> E | D`.
  **L299 CN**: 注释说明附近代码的逻辑、意图或元数据：`R716 exponent-letter -> E | D`。
- **L300 EN**: Comment explains nearby logic, intent, or metadata: `Extension: Q`.
  **L300 CN**: 注释说明附近代码的逻辑、意图或元数据：`Extension: Q`。
- **L301 EN**: Comment explains nearby logic, intent, or metadata: `R717 exponent -> signed-digit-string`.
  **L301 CN**: 注释说明附近代码的逻辑、意图或元数据：`R717 exponent -> signed-digit-string`。
- **L302 EN**: Continues the surrounding expression or declaration: `constexpr auto exponentPart{`.
  **L302 CN**: 继续构造周围的表达式或声明：`constexpr auto exponentPart{`。
- **L303 EN**: Continues the surrounding expression or declaration: `("ed"_ch ||`.
  **L303 CN**: 继续构造周围的表达式或声明：`("ed"_ch ||`。
- **L304 EN**: Continues logic associated with callable symbol `QuadPrecision>`.
  **L304 CN**: 继续与可调用符号 `QuadPrecision>` 相关的逻辑。
- **L305 EN**: Continues the surrounding expression or declaration: `"nonstandard usage: Q exponent"_port_en_US, "q"_ch)) >>`.
  **L305 CN**: 继续构造周围的表达式或声明：`"nonstandard usage: Q exponent"_port_en_US, "q"_ch)) >>`。
- **L306 EN**: Executes a standalone statement or declaration: `SignedDigitString{}};`.
  **L306 CN**: 执行一条独立语句或声明：`SignedDigitString{}};`。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("REAL literal constant"_en_US,`.
  **L308 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("REAL literal constant"_en_US,`。
- **L309 EN**: Continues the surrounding expression or declaration: `space >>`.
  **L309 CN**: 继续构造周围的表达式或声明：`space >>`。
- **L310 EN**: Continues logic associated with callable symbol `construct<RealLiteralConstant>`.
  **L310 CN**: 继续与可调用符号 `construct<RealLiteralConstant>` 相关的逻辑。
- **L311 EN**: Continues logic associated with callable symbol `sourced`.
  **L311 CN**: 继续与可调用符号 `sourced` 相关的逻辑。
- **L312 EN**: Continues logic associated with callable symbol `some`.
  **L312 CN**: 继续与可调用符号 `some` 相关的逻辑。

### Lines 313-336

````cpp
                                "."_ch /* don't misinterpret 1.AND. */) >>
                            maybe(digitString) >> maybe(exponentPart) >> ok ||
                        "."_ch >> digitString >> maybe(exponentPart) >> ok ||
                        digitString >> exponentPart >> ok) >>
                construct<RealLiteralConstant::Real>()),
            maybe(noSpace >> underscore >> noSpace >> kindParam)))

// R718 complex-literal-constant -> ( real-part , imag-part )
TYPE_CONTEXT_PARSER("COMPLEX literal constant"_en_US,
    parenthesized(construct<ComplexLiteralConstant>(
        Parser<ComplexPart>{} / ",", Parser<ComplexPart>{})))

// PGI/Intel extension: signed complex literal constant
TYPE_PARSER(construct<SignedComplexLiteralConstant>(
    sign, Parser<ComplexLiteralConstant>{}))

// R719 real-part ->
//        signed-int-literal-constant | signed-real-literal-constant |
//        named-constant
// R720 imag-part ->
//        signed-int-literal-constant | signed-real-literal-constant |
//        named-constant
TYPE_PARSER(construct<ComplexPart>(signedRealLiteralConstant) ||
    construct<ComplexPart>(signedIntLiteralConstant) ||
````
- **L313 EN**: Continues the surrounding expression or declaration: `"."_ch /* don't misinterpret 1.AND. */) >>`.
  **L313 CN**: 继续构造周围的表达式或声明：`"."_ch /* don't misinterpret 1.AND. */) >>`。
- **L314 EN**: Continues logic associated with callable symbol `maybe`.
  **L314 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L315 EN**: Continues logic associated with callable symbol `maybe`.
  **L315 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L316 EN**: Continues the surrounding expression or declaration: `digitString >> exponentPart >> ok) >>`.
  **L316 CN**: 继续构造周围的表达式或声明：`digitString >> exponentPart >> ok) >>`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<RealLiteralConstant::Real>()),`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<RealLiteralConstant::Real>()),`。
- **L318 EN**: Continues logic associated with callable symbol `maybe`.
  **L318 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L319 EN**: Blank line separating nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L320 EN**: Comment explains nearby logic, intent, or metadata: `R718 complex-literal-constant -> ( real-part , imag-part )`.
  **L320 CN**: 注释说明附近代码的逻辑、意图或元数据：`R718 complex-literal-constant -> ( real-part , imag-part )`。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("COMPLEX literal constant"_en_US,`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("COMPLEX literal constant"_en_US,`。
- **L322 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L322 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L323 EN**: Continues the surrounding expression or declaration: `Parser<ComplexPart>{} / ",", Parser<ComplexPart>{})))`.
  **L323 CN**: 继续构造周围的表达式或声明：`Parser<ComplexPart>{} / ",", Parser<ComplexPart>{})))`。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L325 EN**: Comment explains nearby logic, intent, or metadata: `PGI/Intel extension: signed complex literal constant`.
  **L325 CN**: 注释说明附近代码的逻辑、意图或元数据：`PGI/Intel extension: signed complex literal constant`。
- **L326 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L326 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L327 EN**: Continues the surrounding expression or declaration: `sign, Parser<ComplexLiteralConstant>{}))`.
  **L327 CN**: 继续构造周围的表达式或声明：`sign, Parser<ComplexLiteralConstant>{}))`。
- **L328 EN**: Blank line separating nearby declarations or logic blocks.
  **L328 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L329 EN**: Comment explains nearby logic, intent, or metadata: `R719 real-part ->`.
  **L329 CN**: 注释说明附近代码的逻辑、意图或元数据：`R719 real-part ->`。
- **L330 EN**: Comment explains nearby logic, intent, or metadata: `signed-int-literal-constant | signed-real-literal-constant |`.
  **L330 CN**: 注释说明附近代码的逻辑、意图或元数据：`signed-int-literal-constant | signed-real-literal-constant |`。
- **L331 EN**: Comment explains nearby logic, intent, or metadata: `named-constant`.
  **L331 CN**: 注释说明附近代码的逻辑、意图或元数据：`named-constant`。
- **L332 EN**: Comment explains nearby logic, intent, or metadata: `R720 imag-part ->`.
  **L332 CN**: 注释说明附近代码的逻辑、意图或元数据：`R720 imag-part ->`。
- **L333 EN**: Comment explains nearby logic, intent, or metadata: `signed-int-literal-constant | signed-real-literal-constant |`.
  **L333 CN**: 注释说明附近代码的逻辑、意图或元数据：`signed-int-literal-constant | signed-real-literal-constant |`。
- **L334 EN**: Comment explains nearby logic, intent, or metadata: `named-constant`.
  **L334 CN**: 注释说明附近代码的逻辑、意图或元数据：`named-constant`。
- **L335 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L335 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L336 EN**: Continues logic associated with callable symbol `construct<ComplexPart>`.
  **L336 CN**: 继续与可调用符号 `construct<ComplexPart>` 相关的逻辑。

### Lines 337-360

````cpp
    construct<ComplexPart>(namedConstant))

// R721 char-selector ->
//        length-selector |
//        ( LEN = type-param-value , KIND = scalar-int-constant-expr ) |
//        ( type-param-value , [KIND =] scalar-int-constant-expr ) |
//        ( KIND = scalar-int-constant-expr [, LEN = type-param-value] )
TYPE_PARSER(construct<CharSelector>(Parser<LengthSelector>{}) ||
    parenthesized(construct<CharSelector>(
        "LEN =" >> typeParamValue, ", KIND =" >> scalarIntConstantExpr)) ||
    parenthesized(construct<CharSelector>(
        typeParamValue / ",", maybe("KIND ="_tok) >> scalarIntConstantExpr)) ||
    parenthesized(construct<CharSelector>(
        "KIND =" >> scalarIntConstantExpr, maybe(", LEN =" >> typeParamValue))))

// R722 length-selector -> ( [LEN =] type-param-value ) | * char-length [,]
// N.B. The trailing [,] in the production is permitted by the Standard
// only in the context of a type-declaration-stmt, but even with that
// limitation, it would seem to be unnecessary and buggy to consume the comma
// here.
TYPE_PARSER(construct<LengthSelector>(
                parenthesized(maybe("LEN ="_tok) >> typeParamValue)) ||
    construct<LengthSelector>("*" >> charLength /* / maybe(","_tok) */))

````
- **L337 EN**: Continues logic associated with callable symbol `construct<ComplexPart>`.
  **L337 CN**: 继续与可调用符号 `construct<ComplexPart>` 相关的逻辑。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L339 EN**: Comment explains nearby logic, intent, or metadata: `R721 char-selector ->`.
  **L339 CN**: 注释说明附近代码的逻辑、意图或元数据：`R721 char-selector ->`。
- **L340 EN**: Comment explains nearby logic, intent, or metadata: `length-selector |`.
  **L340 CN**: 注释说明附近代码的逻辑、意图或元数据：`length-selector |`。
- **L341 EN**: Comment explains nearby logic, intent, or metadata: `( LEN = type-param-value , KIND = scalar-int-constant-expr ) |`.
  **L341 CN**: 注释说明附近代码的逻辑、意图或元数据：`( LEN = type-param-value , KIND = scalar-int-constant-expr ) |`。
- **L342 EN**: Comment explains nearby logic, intent, or metadata: `( type-param-value , [KIND =] scalar-int-constant-expr ) |`.
  **L342 CN**: 注释说明附近代码的逻辑、意图或元数据：`( type-param-value , [KIND =] scalar-int-constant-expr ) |`。
- **L343 EN**: Comment explains nearby logic, intent, or metadata: `( KIND = scalar-int-constant-expr [, LEN = type-param-value] )`.
  **L343 CN**: 注释说明附近代码的逻辑、意图或元数据：`( KIND = scalar-int-constant-expr [, LEN = type-param-value] )`。
- **L344 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L344 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L345 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L345 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L346 EN**: Continues the surrounding expression or declaration: `"LEN =" >> typeParamValue, ", KIND =" >> scalarIntConstantExpr)) ||`.
  **L346 CN**: 继续构造周围的表达式或声明：`"LEN =" >> typeParamValue, ", KIND =" >> scalarIntConstantExpr)) ||`。
- **L347 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L347 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L348 EN**: Continues logic associated with callable symbol `maybe`.
  **L348 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L349 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L349 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L350 EN**: Continues logic associated with callable symbol `maybe`.
  **L350 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L352 EN**: Comment explains nearby logic, intent, or metadata: `R722 length-selector -> ( [LEN =] type-param-value ) | * char-length [,]`.
  **L352 CN**: 注释说明附近代码的逻辑、意图或元数据：`R722 length-selector -> ( [LEN =] type-param-value ) | * char-length [,]`。
- **L353 EN**: Comment explains nearby logic, intent, or metadata: `N.B. The trailing [,] in the production is permitted by the Standard`.
  **L353 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. The trailing [,] in the production is permitted by the Standard`。
- **L354 EN**: Comment explains nearby logic, intent, or metadata: `only in the context of a type-declaration-stmt, but even with that`.
  **L354 CN**: 注释说明附近代码的逻辑、意图或元数据：`only in the context of a type-declaration-stmt, but even with that`。
- **L355 EN**: Comment explains nearby logic, intent, or metadata: `limitation, it would seem to be unnecessary and buggy to consume the comma`.
  **L355 CN**: 注释说明附近代码的逻辑、意图或元数据：`limitation, it would seem to be unnecessary and buggy to consume the comma`。
- **L356 EN**: Comment explains nearby logic, intent, or metadata: `here.`.
  **L356 CN**: 注释说明附近代码的逻辑、意图或元数据：`here.`。
- **L357 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L357 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L358 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L358 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L359 EN**: Continues logic associated with callable symbol `construct<LengthSelector>`.
  **L359 CN**: 继续与可调用符号 `construct<LengthSelector>` 相关的逻辑。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 361-384

````cpp
// R723 char-length -> ( type-param-value ) | digit-string
TYPE_PARSER(construct<CharLength>(parenthesized(typeParamValue)) ||
    construct<CharLength>(space >> digitString64 / spaceCheck))

// R724 char-literal-constant ->
//        [kind-param _] ' [rep-char]... ' |
//        [kind-param _] " [rep-char]... "
// "rep-char" is any non-control character.  Doubled interior quotes are
// combined.  Backslash escapes can be enabled.
// N.B. the parsing of "kind-param" takes care to not consume the '_'.
TYPE_CONTEXT_PARSER("CHARACTER literal constant"_en_US,
    construct<CharLiteralConstant>(
        kindParam / underscore, charLiteralConstantWithoutKind) ||
        construct<CharLiteralConstant>(construct<std::optional<KindParam>>(),
            space >> charLiteralConstantWithoutKind))

TYPE_CONTEXT_PARSER(
    "Hollerith"_en_US, construct<HollerithLiteralConstant>(rawHollerithLiteral))

// R725 logical-literal-constant ->
//        .TRUE. [_ kind-param] | .FALSE. [_ kind-param]
// Also accept .T. and .F. as extensions.
TYPE_PARSER(construct<LogicalLiteralConstant>(logicalTRUE,
                maybe(noSpace >> underscore >> noSpace >> kindParam)) ||
````
- **L361 EN**: Comment explains nearby logic, intent, or metadata: `R723 char-length -> ( type-param-value ) | digit-string`.
  **L361 CN**: 注释说明附近代码的逻辑、意图或元数据：`R723 char-length -> ( type-param-value ) | digit-string`。
- **L362 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L362 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L363 EN**: Continues logic associated with callable symbol `construct<CharLength>`.
  **L363 CN**: 继续与可调用符号 `construct<CharLength>` 相关的逻辑。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L365 EN**: Comment explains nearby logic, intent, or metadata: `R724 char-literal-constant ->`.
  **L365 CN**: 注释说明附近代码的逻辑、意图或元数据：`R724 char-literal-constant ->`。
- **L366 EN**: Comment explains nearby logic, intent, or metadata: `[kind-param _] ' [rep-char]... ' |`.
  **L366 CN**: 注释说明附近代码的逻辑、意图或元数据：`[kind-param _] ' [rep-char]... ' |`。
- **L367 EN**: Comment explains nearby logic, intent, or metadata: `[kind-param _] " [rep-char]... "`.
  **L367 CN**: 注释说明附近代码的逻辑、意图或元数据：`[kind-param _] " [rep-char]... "`。
- **L368 EN**: Comment explains nearby logic, intent, or metadata: `"rep-char" is any non-control character.  Doubled interior quotes are`.
  **L368 CN**: 注释说明附近代码的逻辑、意图或元数据：`"rep-char" is any non-control character.  Doubled interior quotes are`。
- **L369 EN**: Comment explains nearby logic, intent, or metadata: `combined.  Backslash escapes can be enabled.`.
  **L369 CN**: 注释说明附近代码的逻辑、意图或元数据：`combined.  Backslash escapes can be enabled.`。
- **L370 EN**: Comment explains nearby logic, intent, or metadata: `N.B. the parsing of "kind-param" takes care to not consume the '_'.`.
  **L370 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. the parsing of "kind-param" takes care to not consume the '_'.`。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("CHARACTER literal constant"_en_US,`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("CHARACTER literal constant"_en_US,`。
- **L372 EN**: Continues logic associated with callable symbol `construct<CharLiteralConstant>`.
  **L372 CN**: 继续与可调用符号 `construct<CharLiteralConstant>` 相关的逻辑。
- **L373 EN**: Continues the surrounding expression or declaration: `kindParam / underscore, charLiteralConstantWithoutKind) ||`.
  **L373 CN**: 继续构造周围的表达式或声明：`kindParam / underscore, charLiteralConstantWithoutKind) ||`。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<CharLiteralConstant>(construct<std::optional<KindParam>>(),`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<CharLiteralConstant>(construct<std::optional<KindParam>>(),`。
- **L375 EN**: Continues the surrounding expression or declaration: `space >> charLiteralConstantWithoutKind))`.
  **L375 CN**: 继续构造周围的表达式或声明：`space >> charLiteralConstantWithoutKind))`。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L377 EN**: Continues logic associated with callable symbol `TYPE_CONTEXT_PARSER`.
  **L377 CN**: 继续与可调用符号 `TYPE_CONTEXT_PARSER` 相关的逻辑。
- **L378 EN**: Continues logic associated with callable symbol `construct<HollerithLiteralConstant>`.
  **L378 CN**: 继续与可调用符号 `construct<HollerithLiteralConstant>` 相关的逻辑。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L380 EN**: Comment explains nearby logic, intent, or metadata: `R725 logical-literal-constant ->`.
  **L380 CN**: 注释说明附近代码的逻辑、意图或元数据：`R725 logical-literal-constant ->`。
- **L381 EN**: Comment explains nearby logic, intent, or metadata: `.TRUE. [_ kind-param] | .FALSE. [_ kind-param]`.
  **L381 CN**: 注释说明附近代码的逻辑、意图或元数据：`.TRUE. [_ kind-param] | .FALSE. [_ kind-param]`。
- **L382 EN**: Comment explains nearby logic, intent, or metadata: `Also accept .T. and .F. as extensions.`.
  **L382 CN**: 注释说明附近代码的逻辑、意图或元数据：`Also accept .T. and .F. as extensions.`。
- **L383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<LogicalLiteralConstant>(logicalTRUE,`.
  **L383 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<LogicalLiteralConstant>(logicalTRUE,`。
- **L384 EN**: Continues logic associated with callable symbol `maybe`.
  **L384 CN**: 继续与可调用符号 `maybe` 相关的逻辑。

### Lines 385-408

````cpp
    construct<LogicalLiteralConstant>(
        logicalFALSE, maybe(noSpace >> underscore >> noSpace >> kindParam)))

// R726 derived-type-def ->
//        derived-type-stmt [type-param-def-stmt]...
//        [private-or-sequence]... [component-part]
//        [type-bound-procedure-part] end-type-stmt
// R735 component-part -> [component-def-stmt]...
TYPE_CONTEXT_PARSER("derived type definition"_en_US,
    construct<DerivedTypeDef>(statement(Parser<DerivedTypeStmt>{}),
        many(unambiguousStatement(Parser<TypeParamDefStmt>{})),
        many(statement(Parser<PrivateOrSequence>{})),
        many(inContext("component"_en_US,
            unambiguousStatement(Parser<ComponentDefStmt>{}))),
        maybe(Parser<TypeBoundProcedurePart>{}),
        statement(Parser<EndTypeStmt>{})))

// R727 derived-type-stmt ->
//        TYPE [[, type-attr-spec-list] ::] type-name [(
//        type-param-name-list )]
TYPE_CONTEXT_PARSER("TYPE statement"_en_US,
    construct<DerivedTypeStmt>(
        "TYPE" >> optionalListBeforeColons(Parser<TypeAttrSpec>{}), name,
        defaulted(parenthesized(nonemptyList(name)))))
````
- **L385 EN**: Continues logic associated with callable symbol `construct<LogicalLiteralConstant>`.
  **L385 CN**: 继续与可调用符号 `construct<LogicalLiteralConstant>` 相关的逻辑。
- **L386 EN**: Continues logic associated with callable symbol `maybe`.
  **L386 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L387 EN**: Blank line separating nearby declarations or logic blocks.
  **L387 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L388 EN**: Comment explains nearby logic, intent, or metadata: `R726 derived-type-def ->`.
  **L388 CN**: 注释说明附近代码的逻辑、意图或元数据：`R726 derived-type-def ->`。
- **L389 EN**: Comment explains nearby logic, intent, or metadata: `derived-type-stmt [type-param-def-stmt]...`.
  **L389 CN**: 注释说明附近代码的逻辑、意图或元数据：`derived-type-stmt [type-param-def-stmt]...`。
- **L390 EN**: Comment explains nearby logic, intent, or metadata: `[private-or-sequence]... [component-part]`.
  **L390 CN**: 注释说明附近代码的逻辑、意图或元数据：`[private-or-sequence]... [component-part]`。
- **L391 EN**: Comment explains nearby logic, intent, or metadata: `[type-bound-procedure-part] end-type-stmt`.
  **L391 CN**: 注释说明附近代码的逻辑、意图或元数据：`[type-bound-procedure-part] end-type-stmt`。
- **L392 EN**: Comment explains nearby logic, intent, or metadata: `R735 component-part -> [component-def-stmt]...`.
  **L392 CN**: 注释说明附近代码的逻辑、意图或元数据：`R735 component-part -> [component-def-stmt]...`。
- **L393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("derived type definition"_en_US,`.
  **L393 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("derived type definition"_en_US,`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DerivedTypeDef>(statement(Parser<DerivedTypeStmt>{}),`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DerivedTypeDef>(statement(Parser<DerivedTypeStmt>{}),`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(unambiguousStatement(Parser<TypeParamDefStmt>{})),`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(unambiguousStatement(Parser<TypeParamDefStmt>{})),`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(statement(Parser<PrivateOrSequence>{})),`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(statement(Parser<PrivateOrSequence>{})),`。
- **L397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(inContext("component"_en_US,`.
  **L397 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(inContext("component"_en_US,`。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unambiguousStatement(Parser<ComponentDefStmt>{}))),`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`unambiguousStatement(Parser<ComponentDefStmt>{}))),`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `maybe(Parser<TypeBoundProcedurePart>{}),`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`maybe(Parser<TypeBoundProcedurePart>{}),`。
- **L400 EN**: Continues logic associated with callable symbol `statement`.
  **L400 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L402 EN**: Comment explains nearby logic, intent, or metadata: `R727 derived-type-stmt ->`.
  **L402 CN**: 注释说明附近代码的逻辑、意图或元数据：`R727 derived-type-stmt ->`。
- **L403 EN**: Comment explains nearby logic, intent, or metadata: `TYPE [[, type-attr-spec-list] ::] type-name [(`.
  **L403 CN**: 注释说明附近代码的逻辑、意图或元数据：`TYPE [[, type-attr-spec-list] ::] type-name [(`。
- **L404 EN**: Comment explains nearby logic, intent, or metadata: `type-param-name-list )]`.
  **L404 CN**: 注释说明附近代码的逻辑、意图或元数据：`type-param-name-list )]`。
- **L405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("TYPE statement"_en_US,`.
  **L405 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("TYPE statement"_en_US,`。
- **L406 EN**: Continues logic associated with callable symbol `construct<DerivedTypeStmt>`.
  **L406 CN**: 继续与可调用符号 `construct<DerivedTypeStmt>` 相关的逻辑。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"TYPE" >> optionalListBeforeColons(Parser<TypeAttrSpec>{}), name,`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`"TYPE" >> optionalListBeforeColons(Parser<TypeAttrSpec>{}), name,`。
- **L408 EN**: Continues logic associated with callable symbol `defaulted`.
  **L408 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。

### Lines 409-432

````cpp

// R728 type-attr-spec ->
//        ABSTRACT | access-spec | BIND(C) | EXTENDS ( parent-type-name )
TYPE_PARSER(construct<TypeAttrSpec>(construct<Abstract>("ABSTRACT"_tok)) ||
    construct<TypeAttrSpec>(construct<TypeAttrSpec::BindC>("BIND ( C )"_tok)) ||
    construct<TypeAttrSpec>(
        construct<TypeAttrSpec::Extends>("EXTENDS" >> parenthesized(name))) ||
    construct<TypeAttrSpec>(accessSpec))

// R729 private-or-sequence -> private-components-stmt | sequence-stmt
TYPE_PARSER(construct<PrivateOrSequence>(Parser<PrivateStmt>{}) ||
    construct<PrivateOrSequence>(Parser<SequenceStmt>{}))

// R730 end-type-stmt -> END TYPE [type-name]
TYPE_PARSER(construct<EndTypeStmt>(
    recovery("END TYPE" >> maybe(name), namedConstructEndStmtErrorRecovery)))

// R731 sequence-stmt -> SEQUENCE
TYPE_PARSER(construct<SequenceStmt>("SEQUENCE"_tok))

// R732 type-param-def-stmt ->
//        integer-type-spec , type-param-attr-spec :: type-param-decl-list
// R734 type-param-attr-spec -> KIND | LEN
constexpr auto kindOrLen{"KIND" >> pure(common::TypeParamAttr::Kind) ||
````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L410 EN**: Comment explains nearby logic, intent, or metadata: `R728 type-attr-spec ->`.
  **L410 CN**: 注释说明附近代码的逻辑、意图或元数据：`R728 type-attr-spec ->`。
- **L411 EN**: Comment explains nearby logic, intent, or metadata: `ABSTRACT | access-spec | BIND(C) | EXTENDS ( parent-type-name )`.
  **L411 CN**: 注释说明附近代码的逻辑、意图或元数据：`ABSTRACT | access-spec | BIND(C) | EXTENDS ( parent-type-name )`。
- **L412 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L412 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L413 EN**: Continues logic associated with callable symbol `construct<TypeAttrSpec>`.
  **L413 CN**: 继续与可调用符号 `construct<TypeAttrSpec>` 相关的逻辑。
- **L414 EN**: Continues logic associated with callable symbol `construct<TypeAttrSpec>`.
  **L414 CN**: 继续与可调用符号 `construct<TypeAttrSpec>` 相关的逻辑。
- **L415 EN**: Continues logic associated with callable symbol `Extends>`.
  **L415 CN**: 继续与可调用符号 `Extends>` 相关的逻辑。
- **L416 EN**: Continues logic associated with callable symbol `construct<TypeAttrSpec>`.
  **L416 CN**: 继续与可调用符号 `construct<TypeAttrSpec>` 相关的逻辑。
- **L417 EN**: Blank line separating nearby declarations or logic blocks.
  **L417 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L418 EN**: Comment explains nearby logic, intent, or metadata: `R729 private-or-sequence -> private-components-stmt | sequence-stmt`.
  **L418 CN**: 注释说明附近代码的逻辑、意图或元数据：`R729 private-or-sequence -> private-components-stmt | sequence-stmt`。
- **L419 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L419 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L420 EN**: Continues logic associated with callable symbol `construct<PrivateOrSequence>`.
  **L420 CN**: 继续与可调用符号 `construct<PrivateOrSequence>` 相关的逻辑。
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L422 EN**: Comment explains nearby logic, intent, or metadata: `R730 end-type-stmt -> END TYPE [type-name]`.
  **L422 CN**: 注释说明附近代码的逻辑、意图或元数据：`R730 end-type-stmt -> END TYPE [type-name]`。
- **L423 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L423 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L424 EN**: Continues logic associated with callable symbol `recovery`.
  **L424 CN**: 继续与可调用符号 `recovery` 相关的逻辑。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L426 EN**: Comment explains nearby logic, intent, or metadata: `R731 sequence-stmt -> SEQUENCE`.
  **L426 CN**: 注释说明附近代码的逻辑、意图或元数据：`R731 sequence-stmt -> SEQUENCE`。
- **L427 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L427 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L428 EN**: Blank line separating nearby declarations or logic blocks.
  **L428 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L429 EN**: Comment explains nearby logic, intent, or metadata: `R732 type-param-def-stmt ->`.
  **L429 CN**: 注释说明附近代码的逻辑、意图或元数据：`R732 type-param-def-stmt ->`。
- **L430 EN**: Comment explains nearby logic, intent, or metadata: `integer-type-spec , type-param-attr-spec :: type-param-decl-list`.
  **L430 CN**: 注释说明附近代码的逻辑、意图或元数据：`integer-type-spec , type-param-attr-spec :: type-param-decl-list`。
- **L431 EN**: Comment explains nearby logic, intent, or metadata: `R734 type-param-attr-spec -> KIND | LEN`.
  **L431 CN**: 注释说明附近代码的逻辑、意图或元数据：`R734 type-param-attr-spec -> KIND | LEN`。
- **L432 EN**: Continues logic associated with callable symbol `pure`.
  **L432 CN**: 继续与可调用符号 `pure` 相关的逻辑。

### Lines 433-456

````cpp
    "LEN" >> pure(common::TypeParamAttr::Len)};
TYPE_PARSER(construct<TypeParamDefStmt>(integerTypeSpec / ",", kindOrLen,
    "::" >> nonemptyList("expected type parameter declarations"_err_en_US,
                Parser<TypeParamDecl>{})))

// R733 type-param-decl -> type-param-name [= scalar-int-constant-expr]
TYPE_PARSER(construct<TypeParamDecl>(name, maybe("=" >> scalarIntConstantExpr)))

// R736 component-def-stmt -> data-component-def-stmt |
//        proc-component-def-stmt
// Accidental extension not enabled here: PGI accepts type-param-def-stmt in
// component-part of derived-type-def.
TYPE_PARSER(recovery(
    withMessage("expected component definition"_err_en_US,
        first(construct<ComponentDefStmt>(Parser<DataComponentDefStmt>{}),
            construct<ComponentDefStmt>(Parser<ProcComponentDefStmt>{}),
            construct<ComponentDefStmt>(indirect(compilerDirective)))),
    construct<ComponentDefStmt>(inStmtErrorRecovery)))

// R737 data-component-def-stmt ->
//        declaration-type-spec [[, component-attr-spec-list] ::]
//        component-decl-list
// N.B. The standard requires double colons if there's an initializer.
TYPE_PARSER(construct<DataComponentDefStmt>(declarationTypeSpec,
````
- **L433 EN**: Executes a call or declaration centered on `pure`.
  **L433 CN**: 执行以 `pure` 为核心的调用或声明。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<TypeParamDefStmt>(integerTypeSpec / ",", kindOrLen,`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<TypeParamDefStmt>(integerTypeSpec / ",", kindOrLen,`。
- **L435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"::" >> nonemptyList("expected type parameter declarations"_err_en_US,`.
  **L435 CN**: 继续一个多行参数列表、初始化器或聚合项：`"::" >> nonemptyList("expected type parameter declarations"_err_en_US,`。
- **L436 EN**: Continues the surrounding expression or declaration: `Parser<TypeParamDecl>{})))`.
  **L436 CN**: 继续构造周围的表达式或声明：`Parser<TypeParamDecl>{})))`。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L438 EN**: Comment explains nearby logic, intent, or metadata: `R733 type-param-decl -> type-param-name [= scalar-int-constant-expr]`.
  **L438 CN**: 注释说明附近代码的逻辑、意图或元数据：`R733 type-param-decl -> type-param-name [= scalar-int-constant-expr]`。
- **L439 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L439 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L441 EN**: Comment explains nearby logic, intent, or metadata: `R736 component-def-stmt -> data-component-def-stmt |`.
  **L441 CN**: 注释说明附近代码的逻辑、意图或元数据：`R736 component-def-stmt -> data-component-def-stmt |`。
- **L442 EN**: Comment explains nearby logic, intent, or metadata: `proc-component-def-stmt`.
  **L442 CN**: 注释说明附近代码的逻辑、意图或元数据：`proc-component-def-stmt`。
- **L443 EN**: Comment explains nearby logic, intent, or metadata: `Accidental extension not enabled here: PGI accepts type-param-def-stmt in`.
  **L443 CN**: 注释说明附近代码的逻辑、意图或元数据：`Accidental extension not enabled here: PGI accepts type-param-def-stmt in`。
- **L444 EN**: Comment explains nearby logic, intent, or metadata: `component-part of derived-type-def.`.
  **L444 CN**: 注释说明附近代码的逻辑、意图或元数据：`component-part of derived-type-def.`。
- **L445 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L445 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `withMessage("expected component definition"_err_en_US,`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`withMessage("expected component definition"_err_en_US,`。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first(construct<ComponentDefStmt>(Parser<DataComponentDefStmt>{}),`.
  **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`first(construct<ComponentDefStmt>(Parser<DataComponentDefStmt>{}),`。
- **L448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ComponentDefStmt>(Parser<ProcComponentDefStmt>{}),`.
  **L448 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ComponentDefStmt>(Parser<ProcComponentDefStmt>{}),`。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ComponentDefStmt>(indirect(compilerDirective)))),`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ComponentDefStmt>(indirect(compilerDirective)))),`。
- **L450 EN**: Continues logic associated with callable symbol `construct<ComponentDefStmt>`.
  **L450 CN**: 继续与可调用符号 `construct<ComponentDefStmt>` 相关的逻辑。
- **L451 EN**: Blank line separating nearby declarations or logic blocks.
  **L451 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L452 EN**: Comment explains nearby logic, intent, or metadata: `R737 data-component-def-stmt ->`.
  **L452 CN**: 注释说明附近代码的逻辑、意图或元数据：`R737 data-component-def-stmt ->`。
- **L453 EN**: Comment explains nearby logic, intent, or metadata: `declaration-type-spec [[, component-attr-spec-list] ::]`.
  **L453 CN**: 注释说明附近代码的逻辑、意图或元数据：`declaration-type-spec [[, component-attr-spec-list] ::]`。
- **L454 EN**: Comment explains nearby logic, intent, or metadata: `component-decl-list`.
  **L454 CN**: 注释说明附近代码的逻辑、意图或元数据：`component-decl-list`。
- **L455 EN**: Comment explains nearby logic, intent, or metadata: `N.B. The standard requires double colons if there's an initializer.`.
  **L455 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. The standard requires double colons if there's an initializer.`。
- **L456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<DataComponentDefStmt>(declarationTypeSpec,`.
  **L456 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<DataComponentDefStmt>(declarationTypeSpec,`。

### Lines 457-480

````cpp
    optionalListBeforeColons(Parser<ComponentAttrSpec>{}),
    nonemptyList("expected component declarations"_err_en_US,
        Parser<ComponentOrFill>{})))

// R738 component-attr-spec ->
//        access-spec | ALLOCATABLE |
//        CODIMENSION lbracket coarray-spec rbracket |
//        CONTIGUOUS | DIMENSION ( component-array-spec ) | POINTER |
//        CUDA-data-attr
TYPE_PARSER(construct<ComponentAttrSpec>(accessSpec) ||
    construct<ComponentAttrSpec>(allocatable) ||
    construct<ComponentAttrSpec>("CODIMENSION" >> coarraySpec) ||
    construct<ComponentAttrSpec>(contiguous) ||
    construct<ComponentAttrSpec>("DIMENSION" >> componentArraySpec) ||
    construct<ComponentAttrSpec>(pointer) ||
    extension<LanguageFeature::CUDA>(
        construct<ComponentAttrSpec>(Parser<common::CUDADataAttr>{})) ||
    construct<ComponentAttrSpec>(recovery(
        fail<ErrorRecovery>(
            "type parameter definitions must appear before component declarations"_err_en_US),
        kindOrLen >> construct<ErrorRecovery>())))

// R739 component-decl ->
//        component-name [( component-array-spec )]
````
- **L457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `optionalListBeforeColons(Parser<ComponentAttrSpec>{}),`.
  **L457 CN**: 继续一个多行参数列表、初始化器或聚合项：`optionalListBeforeColons(Parser<ComponentAttrSpec>{}),`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected component declarations"_err_en_US,`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected component declarations"_err_en_US,`。
- **L459 EN**: Continues the surrounding expression or declaration: `Parser<ComponentOrFill>{})))`.
  **L459 CN**: 继续构造周围的表达式或声明：`Parser<ComponentOrFill>{})))`。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L461 EN**: Comment explains nearby logic, intent, or metadata: `R738 component-attr-spec ->`.
  **L461 CN**: 注释说明附近代码的逻辑、意图或元数据：`R738 component-attr-spec ->`。
- **L462 EN**: Comment explains nearby logic, intent, or metadata: `access-spec | ALLOCATABLE |`.
  **L462 CN**: 注释说明附近代码的逻辑、意图或元数据：`access-spec | ALLOCATABLE |`。
- **L463 EN**: Comment explains nearby logic, intent, or metadata: `CODIMENSION lbracket coarray-spec rbracket |`.
  **L463 CN**: 注释说明附近代码的逻辑、意图或元数据：`CODIMENSION lbracket coarray-spec rbracket |`。
- **L464 EN**: Comment explains nearby logic, intent, or metadata: `CONTIGUOUS | DIMENSION ( component-array-spec ) | POINTER |`.
  **L464 CN**: 注释说明附近代码的逻辑、意图或元数据：`CONTIGUOUS | DIMENSION ( component-array-spec ) | POINTER |`。
- **L465 EN**: Comment explains nearby logic, intent, or metadata: `CUDA-data-attr`.
  **L465 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUDA-data-attr`。
- **L466 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L466 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L467 EN**: Continues logic associated with callable symbol `construct<ComponentAttrSpec>`.
  **L467 CN**: 继续与可调用符号 `construct<ComponentAttrSpec>` 相关的逻辑。
- **L468 EN**: Continues logic associated with callable symbol `construct<ComponentAttrSpec>`.
  **L468 CN**: 继续与可调用符号 `construct<ComponentAttrSpec>` 相关的逻辑。
- **L469 EN**: Continues logic associated with callable symbol `construct<ComponentAttrSpec>`.
  **L469 CN**: 继续与可调用符号 `construct<ComponentAttrSpec>` 相关的逻辑。
- **L470 EN**: Continues logic associated with callable symbol `construct<ComponentAttrSpec>`.
  **L470 CN**: 继续与可调用符号 `construct<ComponentAttrSpec>` 相关的逻辑。
- **L471 EN**: Continues logic associated with callable symbol `construct<ComponentAttrSpec>`.
  **L471 CN**: 继续与可调用符号 `construct<ComponentAttrSpec>` 相关的逻辑。
- **L472 EN**: Continues logic associated with callable symbol `CUDA>`.
  **L472 CN**: 继续与可调用符号 `CUDA>` 相关的逻辑。
- **L473 EN**: Continues logic associated with callable symbol `construct<ComponentAttrSpec>`.
  **L473 CN**: 继续与可调用符号 `construct<ComponentAttrSpec>` 相关的逻辑。
- **L474 EN**: Continues logic associated with callable symbol `construct<ComponentAttrSpec>`.
  **L474 CN**: 继续与可调用符号 `construct<ComponentAttrSpec>` 相关的逻辑。
- **L475 EN**: Continues logic associated with callable symbol `fail<ErrorRecovery>`.
  **L475 CN**: 继续与可调用符号 `fail<ErrorRecovery>` 相关的逻辑。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"type parameter definitions must appear before component declarations"_err_en_US),`.
  **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`"type parameter definitions must appear before component declarations"_err_en_US),`。
- **L477 EN**: Continues logic associated with callable symbol `construct<ErrorRecovery>`.
  **L477 CN**: 继续与可调用符号 `construct<ErrorRecovery>` 相关的逻辑。
- **L478 EN**: Blank line separating nearby declarations or logic blocks.
  **L478 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L479 EN**: Comment explains nearby logic, intent, or metadata: `R739 component-decl ->`.
  **L479 CN**: 注释说明附近代码的逻辑、意图或元数据：`R739 component-decl ->`。
- **L480 EN**: Comment explains nearby logic, intent, or metadata: `component-name [( component-array-spec )]`.
  **L480 CN**: 注释说明附近代码的逻辑、意图或元数据：`component-name [( component-array-spec )]`。

### Lines 481-504

````cpp
//          [lbracket coarray-spec rbracket] [* char-length]
//          [component-initialization] |
// (ext.) component-name *char-length [(component-array-spec)]
//          [lbracket coarray-spec rbracket] [* char-length]
//          [component-initialization]
TYPE_CONTEXT_PARSER("component declaration"_en_US,
    construct<ComponentDecl>(name, "*" >> charLength, maybe(componentArraySpec),
        maybe(coarraySpec), maybe(initialization)) ||
        construct<ComponentDecl>(name, maybe(componentArraySpec),
            maybe(coarraySpec), maybe("*" >> charLength),
            maybe(initialization)))
// The source field of the Name will be replaced with a distinct generated name.
TYPE_CONTEXT_PARSER("%FILL item"_en_US,
    extension<LanguageFeature::DECStructures>(
        "nonstandard usage: %FILL"_port_en_US,
        construct<FillDecl>(space >> sourced("%FILL" >> construct<Name>()),
            maybe(componentArraySpec), maybe("*" >> charLength))))
TYPE_PARSER(construct<ComponentOrFill>(Parser<ComponentDecl>{}) ||
    construct<ComponentOrFill>(Parser<FillDecl>{}))

// R740 component-array-spec ->
//        explicit-shape-spec-list | deferred-shape-spec-list
// N.B. Parenthesized here rather than around references to this production.
TYPE_PARSER(construct<ComponentArraySpec>(parenthesized(
````
- **L481 EN**: Comment explains nearby logic, intent, or metadata: `[lbracket coarray-spec rbracket] [* char-length]`.
  **L481 CN**: 注释说明附近代码的逻辑、意图或元数据：`[lbracket coarray-spec rbracket] [* char-length]`。
- **L482 EN**: Comment explains nearby logic, intent, or metadata: `[component-initialization] |`.
  **L482 CN**: 注释说明附近代码的逻辑、意图或元数据：`[component-initialization] |`。
- **L483 EN**: Comment explains nearby logic, intent, or metadata: `(ext.) component-name *char-length [(component-array-spec)]`.
  **L483 CN**: 注释说明附近代码的逻辑、意图或元数据：`(ext.) component-name *char-length [(component-array-spec)]`。
- **L484 EN**: Comment explains nearby logic, intent, or metadata: `[lbracket coarray-spec rbracket] [* char-length]`.
  **L484 CN**: 注释说明附近代码的逻辑、意图或元数据：`[lbracket coarray-spec rbracket] [* char-length]`。
- **L485 EN**: Comment explains nearby logic, intent, or metadata: `[component-initialization]`.
  **L485 CN**: 注释说明附近代码的逻辑、意图或元数据：`[component-initialization]`。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("component declaration"_en_US,`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("component declaration"_en_US,`。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ComponentDecl>(name, "*" >> charLength, maybe(componentArraySpec),`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ComponentDecl>(name, "*" >> charLength, maybe(componentArraySpec),`。
- **L488 EN**: Continues logic associated with callable symbol `maybe`.
  **L488 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ComponentDecl>(name, maybe(componentArraySpec),`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ComponentDecl>(name, maybe(componentArraySpec),`。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `maybe(coarraySpec), maybe("*" >> charLength),`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`maybe(coarraySpec), maybe("*" >> charLength),`。
- **L491 EN**: Continues logic associated with callable symbol `maybe`.
  **L491 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L492 EN**: Comment explains nearby logic, intent, or metadata: `The source field of the Name will be replaced with a distinct generated name.`.
  **L492 CN**: 注释说明附近代码的逻辑、意图或元数据：`The source field of the Name will be replaced with a distinct generated name.`。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("%FILL item"_en_US,`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("%FILL item"_en_US,`。
- **L494 EN**: Continues logic associated with callable symbol `DECStructures>`.
  **L494 CN**: 继续与可调用符号 `DECStructures>` 相关的逻辑。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: %FILL"_port_en_US,`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: %FILL"_port_en_US,`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<FillDecl>(space >> sourced("%FILL" >> construct<Name>()),`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<FillDecl>(space >> sourced("%FILL" >> construct<Name>()),`。
- **L497 EN**: Continues logic associated with callable symbol `maybe`.
  **L497 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L498 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L498 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L499 EN**: Continues logic associated with callable symbol `construct<ComponentOrFill>`.
  **L499 CN**: 继续与可调用符号 `construct<ComponentOrFill>` 相关的逻辑。
- **L500 EN**: Blank line separating nearby declarations or logic blocks.
  **L500 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L501 EN**: Comment explains nearby logic, intent, or metadata: `R740 component-array-spec ->`.
  **L501 CN**: 注释说明附近代码的逻辑、意图或元数据：`R740 component-array-spec ->`。
- **L502 EN**: Comment explains nearby logic, intent, or metadata: `explicit-shape-spec-list | deferred-shape-spec-list`.
  **L502 CN**: 注释说明附近代码的逻辑、意图或元数据：`explicit-shape-spec-list | deferred-shape-spec-list`。
- **L503 EN**: Comment explains nearby logic, intent, or metadata: `N.B. Parenthesized here rather than around references to this production.`.
  **L503 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. Parenthesized here rather than around references to this production.`。
- **L504 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L504 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。

### Lines 505-528

````cpp
                nonemptyList("expected explicit shape specifications"_err_en_US,
                    explicitShapeSpec))) ||
    construct<ComponentArraySpec>(parenthesized(deferredShapeSpecList)))

// R741 proc-component-def-stmt ->
//        PROCEDURE ( [proc-interface] ) , proc-component-attr-spec-list
//          :: proc-decl-list
TYPE_CONTEXT_PARSER("PROCEDURE component definition statement"_en_US,
    construct<ProcComponentDefStmt>(
        "PROCEDURE" >> parenthesized(maybe(procInterface)),
        localRecovery("expected PROCEDURE component attributes"_err_en_US,
            "," >> nonemptyList(Parser<ProcComponentAttrSpec>{}), ok),
        localRecovery("expected PROCEDURE declarations"_err_en_US,
            "::" >> nonemptyList(procDecl), SkipTo<'\n'>{})))

// R742 proc-component-attr-spec ->
//        access-spec | NOPASS | PASS [(arg-name)] | POINTER
constexpr auto noPass{construct<NoPass>("NOPASS"_tok)};
constexpr auto pass{construct<Pass>("PASS" >> maybe(parenthesized(name)))};
TYPE_PARSER(construct<ProcComponentAttrSpec>(accessSpec) ||
    construct<ProcComponentAttrSpec>(noPass) ||
    construct<ProcComponentAttrSpec>(pass) ||
    construct<ProcComponentAttrSpec>(pointer))

````
- **L505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected explicit shape specifications"_err_en_US,`.
  **L505 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected explicit shape specifications"_err_en_US,`。
- **L506 EN**: Continues the surrounding expression or declaration: `explicitShapeSpec))) ||`.
  **L506 CN**: 继续构造周围的表达式或声明：`explicitShapeSpec))) ||`。
- **L507 EN**: Continues logic associated with callable symbol `construct<ComponentArraySpec>`.
  **L507 CN**: 继续与可调用符号 `construct<ComponentArraySpec>` 相关的逻辑。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L509 EN**: Comment explains nearby logic, intent, or metadata: `R741 proc-component-def-stmt ->`.
  **L509 CN**: 注释说明附近代码的逻辑、意图或元数据：`R741 proc-component-def-stmt ->`。
- **L510 EN**: Comment explains nearby logic, intent, or metadata: `PROCEDURE ( [proc-interface] ) , proc-component-attr-spec-list`.
  **L510 CN**: 注释说明附近代码的逻辑、意图或元数据：`PROCEDURE ( [proc-interface] ) , proc-component-attr-spec-list`。
- **L511 EN**: Comment explains nearby logic, intent, or metadata: `:: proc-decl-list`.
  **L511 CN**: 注释说明附近代码的逻辑、意图或元数据：`:: proc-decl-list`。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("PROCEDURE component definition statement"_en_US,`.
  **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("PROCEDURE component definition statement"_en_US,`。
- **L513 EN**: Continues logic associated with callable symbol `construct<ProcComponentDefStmt>`.
  **L513 CN**: 继续与可调用符号 `construct<ProcComponentDefStmt>` 相关的逻辑。
- **L514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"PROCEDURE" >> parenthesized(maybe(procInterface)),`.
  **L514 CN**: 继续一个多行参数列表、初始化器或聚合项：`"PROCEDURE" >> parenthesized(maybe(procInterface)),`。
- **L515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `localRecovery("expected PROCEDURE component attributes"_err_en_US,`.
  **L515 CN**: 继续一个多行参数列表、初始化器或聚合项：`localRecovery("expected PROCEDURE component attributes"_err_en_US,`。
- **L516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"," >> nonemptyList(Parser<ProcComponentAttrSpec>{}), ok),`.
  **L516 CN**: 继续一个多行参数列表、初始化器或聚合项：`"," >> nonemptyList(Parser<ProcComponentAttrSpec>{}), ok),`。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `localRecovery("expected PROCEDURE declarations"_err_en_US,`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`localRecovery("expected PROCEDURE declarations"_err_en_US,`。
- **L518 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L518 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L519 EN**: Blank line separating nearby declarations or logic blocks.
  **L519 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L520 EN**: Comment explains nearby logic, intent, or metadata: `R742 proc-component-attr-spec ->`.
  **L520 CN**: 注释说明附近代码的逻辑、意图或元数据：`R742 proc-component-attr-spec ->`。
- **L521 EN**: Comment explains nearby logic, intent, or metadata: `access-spec | NOPASS | PASS [(arg-name)] | POINTER`.
  **L521 CN**: 注释说明附近代码的逻辑、意图或元数据：`access-spec | NOPASS | PASS [(arg-name)] | POINTER`。
- **L522 EN**: Executes a call or declaration centered on `noPass{construct<NoPass>`.
  **L522 CN**: 执行以 `noPass{construct<NoPass>` 为核心的调用或声明。
- **L523 EN**: Executes a call or declaration centered on `pass{construct<Pass>`.
  **L523 CN**: 执行以 `pass{construct<Pass>` 为核心的调用或声明。
- **L524 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L524 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L525 EN**: Continues logic associated with callable symbol `construct<ProcComponentAttrSpec>`.
  **L525 CN**: 继续与可调用符号 `construct<ProcComponentAttrSpec>` 相关的逻辑。
- **L526 EN**: Continues logic associated with callable symbol `construct<ProcComponentAttrSpec>`.
  **L526 CN**: 继续与可调用符号 `construct<ProcComponentAttrSpec>` 相关的逻辑。
- **L527 EN**: Continues logic associated with callable symbol `construct<ProcComponentAttrSpec>`.
  **L527 CN**: 继续与可调用符号 `construct<ProcComponentAttrSpec>` 相关的逻辑。
- **L528 EN**: Blank line separating nearby declarations or logic blocks.
  **L528 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 529-552

````cpp
// R744 initial-data-target -> designator
constexpr auto initialDataTarget{indirect(designator)};

// R743 component-initialization ->
//        = constant-expr | => null-init | => initial-data-target
// R805 initialization ->
//        = constant-expr | => null-init | => initial-data-target
// Universal extension: initialization -> / data-stmt-value-list /
TYPE_PARSER(construct<Initialization>("=>" >> nullInit) ||
    construct<Initialization>("=>" >> initialDataTarget) ||
    construct<Initialization>("=" >> constantExpr) ||
    extension<LanguageFeature::SlashInitialization>(
        "nonstandard usage: /initialization/"_port_en_US,
        construct<Initialization>(
            "/" >> nonemptyList("expected values"_err_en_US,
                       indirect(Parser<DataStmtValue>{})) /
                "/")))

// R745 private-components-stmt -> PRIVATE
// R747 binding-private-stmt -> PRIVATE
TYPE_PARSER(construct<PrivateStmt>("PRIVATE"_tok))

// R746 type-bound-procedure-part ->
//        contains-stmt [binding-private-stmt] [type-bound-proc-binding]...
````
- **L529 EN**: Comment explains nearby logic, intent, or metadata: `R744 initial-data-target -> designator`.
  **L529 CN**: 注释说明附近代码的逻辑、意图或元数据：`R744 initial-data-target -> designator`。
- **L530 EN**: Executes a call or declaration centered on `initialDataTarget{indirect`.
  **L530 CN**: 执行以 `initialDataTarget{indirect` 为核心的调用或声明。
- **L531 EN**: Blank line separating nearby declarations or logic blocks.
  **L531 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L532 EN**: Comment explains nearby logic, intent, or metadata: `R743 component-initialization ->`.
  **L532 CN**: 注释说明附近代码的逻辑、意图或元数据：`R743 component-initialization ->`。
- **L533 EN**: Comment explains nearby logic, intent, or metadata: `= constant-expr | => null-init | => initial-data-target`.
  **L533 CN**: 注释说明附近代码的逻辑、意图或元数据：`= constant-expr | => null-init | => initial-data-target`。
- **L534 EN**: Comment explains nearby logic, intent, or metadata: `R805 initialization ->`.
  **L534 CN**: 注释说明附近代码的逻辑、意图或元数据：`R805 initialization ->`。
- **L535 EN**: Comment explains nearby logic, intent, or metadata: `= constant-expr | => null-init | => initial-data-target`.
  **L535 CN**: 注释说明附近代码的逻辑、意图或元数据：`= constant-expr | => null-init | => initial-data-target`。
- **L536 EN**: Comment explains nearby logic, intent, or metadata: `Universal extension: initialization -> / data-stmt-value-list`.
  **L536 CN**: 注释说明附近代码的逻辑、意图或元数据：`Universal extension: initialization -> / data-stmt-value-list`。
- **L537 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L537 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L538 EN**: Continues logic associated with callable symbol `construct<Initialization>`.
  **L538 CN**: 继续与可调用符号 `construct<Initialization>` 相关的逻辑。
- **L539 EN**: Continues logic associated with callable symbol `construct<Initialization>`.
  **L539 CN**: 继续与可调用符号 `construct<Initialization>` 相关的逻辑。
- **L540 EN**: Continues logic associated with callable symbol `SlashInitialization>`.
  **L540 CN**: 继续与可调用符号 `SlashInitialization>` 相关的逻辑。
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: /initialization/"_port_en_US,`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: /initialization/"_port_en_US,`。
- **L542 EN**: Continues logic associated with callable symbol `construct<Initialization>`.
  **L542 CN**: 继续与可调用符号 `construct<Initialization>` 相关的逻辑。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"/" >> nonemptyList("expected values"_err_en_US,`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`"/" >> nonemptyList("expected values"_err_en_US,`。
- **L544 EN**: Continues logic associated with callable symbol `indirect`.
  **L544 CN**: 继续与可调用符号 `indirect` 相关的逻辑。
- **L545 EN**: Continues the surrounding expression or declaration: `"/")))`.
  **L545 CN**: 继续构造周围的表达式或声明：`"/")))`。
- **L546 EN**: Blank line separating nearby declarations or logic blocks.
  **L546 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L547 EN**: Comment explains nearby logic, intent, or metadata: `R745 private-components-stmt -> PRIVATE`.
  **L547 CN**: 注释说明附近代码的逻辑、意图或元数据：`R745 private-components-stmt -> PRIVATE`。
- **L548 EN**: Comment explains nearby logic, intent, or metadata: `R747 binding-private-stmt -> PRIVATE`.
  **L548 CN**: 注释说明附近代码的逻辑、意图或元数据：`R747 binding-private-stmt -> PRIVATE`。
- **L549 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L549 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L550 EN**: Blank line separating nearby declarations or logic blocks.
  **L550 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L551 EN**: Comment explains nearby logic, intent, or metadata: `R746 type-bound-procedure-part ->`.
  **L551 CN**: 注释说明附近代码的逻辑、意图或元数据：`R746 type-bound-procedure-part ->`。
- **L552 EN**: Comment explains nearby logic, intent, or metadata: `contains-stmt [binding-private-stmt] [type-bound-proc-binding]...`.
  **L552 CN**: 注释说明附近代码的逻辑、意图或元数据：`contains-stmt [binding-private-stmt] [type-bound-proc-binding]...`。

### Lines 553-576

````cpp
TYPE_CONTEXT_PARSER("type bound procedure part"_en_US,
    construct<TypeBoundProcedurePart>(statement(containsStmt),
        maybe(statement(Parser<PrivateStmt>{})),
        many(statement(Parser<TypeBoundProcBinding>{}))))

// R748 type-bound-proc-binding ->
//        type-bound-procedure-stmt | type-bound-generic-stmt |
//        final-procedure-stmt
TYPE_CONTEXT_PARSER("type bound procedure binding"_en_US,
    recovery(
        first(construct<TypeBoundProcBinding>(Parser<TypeBoundProcedureStmt>{}),
            construct<TypeBoundProcBinding>(Parser<TypeBoundGenericStmt>{}),
            construct<TypeBoundProcBinding>(Parser<FinalProcedureStmt>{})),
        construct<TypeBoundProcBinding>(
            !"END"_tok >> SkipTo<'\n'>{} >> construct<ErrorRecovery>())))

// R749 type-bound-procedure-stmt ->
//        PROCEDURE [[, bind-attr-list] ::] type-bound-proc-decl-list |
//        PROCEDURE ( interface-name ) , bind-attr-list :: binding-name-list
// The "::" is required by the standard (C768) in the first production if
// any type-bound-proc-decl has a "=>', but it's not strictly necessary to
// avoid a bad parse.
TYPE_CONTEXT_PARSER("type bound PROCEDURE statement"_en_US,
    "PROCEDURE" >>
````
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("type bound procedure part"_en_US,`.
  **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("type bound procedure part"_en_US,`。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<TypeBoundProcedurePart>(statement(containsStmt),`.
  **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<TypeBoundProcedurePart>(statement(containsStmt),`。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `maybe(statement(Parser<PrivateStmt>{})),`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`maybe(statement(Parser<PrivateStmt>{})),`。
- **L556 EN**: Continues logic associated with callable symbol `many`.
  **L556 CN**: 继续与可调用符号 `many` 相关的逻辑。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L558 EN**: Comment explains nearby logic, intent, or metadata: `R748 type-bound-proc-binding ->`.
  **L558 CN**: 注释说明附近代码的逻辑、意图或元数据：`R748 type-bound-proc-binding ->`。
- **L559 EN**: Comment explains nearby logic, intent, or metadata: `type-bound-procedure-stmt | type-bound-generic-stmt |`.
  **L559 CN**: 注释说明附近代码的逻辑、意图或元数据：`type-bound-procedure-stmt | type-bound-generic-stmt |`。
- **L560 EN**: Comment explains nearby logic, intent, or metadata: `final-procedure-stmt`.
  **L560 CN**: 注释说明附近代码的逻辑、意图或元数据：`final-procedure-stmt`。
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("type bound procedure binding"_en_US,`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("type bound procedure binding"_en_US,`。
- **L562 EN**: Continues logic associated with callable symbol `recovery`.
  **L562 CN**: 继续与可调用符号 `recovery` 相关的逻辑。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first(construct<TypeBoundProcBinding>(Parser<TypeBoundProcedureStmt>{}),`.
  **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`first(construct<TypeBoundProcBinding>(Parser<TypeBoundProcedureStmt>{}),`。
- **L564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<TypeBoundProcBinding>(Parser<TypeBoundGenericStmt>{}),`.
  **L564 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<TypeBoundProcBinding>(Parser<TypeBoundGenericStmt>{}),`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<TypeBoundProcBinding>(Parser<FinalProcedureStmt>{})),`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<TypeBoundProcBinding>(Parser<FinalProcedureStmt>{})),`。
- **L566 EN**: Continues logic associated with callable symbol `construct<TypeBoundProcBinding>`.
  **L566 CN**: 继续与可调用符号 `construct<TypeBoundProcBinding>` 相关的逻辑。
- **L567 EN**: Continues logic associated with callable symbol `construct<ErrorRecovery>`.
  **L567 CN**: 继续与可调用符号 `construct<ErrorRecovery>` 相关的逻辑。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L569 EN**: Comment explains nearby logic, intent, or metadata: `R749 type-bound-procedure-stmt ->`.
  **L569 CN**: 注释说明附近代码的逻辑、意图或元数据：`R749 type-bound-procedure-stmt ->`。
- **L570 EN**: Comment explains nearby logic, intent, or metadata: `PROCEDURE [[, bind-attr-list] ::] type-bound-proc-decl-list |`.
  **L570 CN**: 注释说明附近代码的逻辑、意图或元数据：`PROCEDURE [[, bind-attr-list] ::] type-bound-proc-decl-list |`。
- **L571 EN**: Comment explains nearby logic, intent, or metadata: `PROCEDURE ( interface-name ) , bind-attr-list :: binding-name-list`.
  **L571 CN**: 注释说明附近代码的逻辑、意图或元数据：`PROCEDURE ( interface-name ) , bind-attr-list :: binding-name-list`。
- **L572 EN**: Comment explains nearby logic, intent, or metadata: `The "::" is required by the standard (C768) in the first production if`.
  **L572 CN**: 注释说明附近代码的逻辑、意图或元数据：`The "::" is required by the standard (C768) in the first production if`。
- **L573 EN**: Comment explains nearby logic, intent, or metadata: `any type-bound-proc-decl has a "=>', but it's not strictly necessary to`.
  **L573 CN**: 注释说明附近代码的逻辑、意图或元数据：`any type-bound-proc-decl has a "=>', but it's not strictly necessary to`。
- **L574 EN**: Comment explains nearby logic, intent, or metadata: `avoid a bad parse.`.
  **L574 CN**: 注释说明附近代码的逻辑、意图或元数据：`avoid a bad parse.`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("type bound PROCEDURE statement"_en_US,`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("type bound PROCEDURE statement"_en_US,`。
- **L576 EN**: Continues the surrounding expression or declaration: `"PROCEDURE" >>`.
  **L576 CN**: 继续构造周围的表达式或声明：`"PROCEDURE" >>`。

### Lines 577-600

````cpp
        (construct<TypeBoundProcedureStmt>(
             construct<TypeBoundProcedureStmt::WithInterface>(
                 parenthesized(name),
                 localRecovery("expected list of binding attributes"_err_en_US,
                     "," >> nonemptyList(Parser<BindAttr>{}), ok),
                 localRecovery("expected list of binding names"_err_en_US,
                     "::" >> listOfNames, SkipTo<'\n'>{}))) ||
            construct<TypeBoundProcedureStmt>(construct<
                TypeBoundProcedureStmt::WithoutInterface>(
                pure<std::list<BindAttr>>(),
                nonemptyList(
                    "expected type bound procedure declarations"_err_en_US,
                    construct<TypeBoundProcDecl>(name,
                        maybe(extension<LanguageFeature::MissingColons>(
                            "type-bound procedure statement should have '::' if it has '=>'"_port_en_US,
                            "=>" >> name)))))) ||
            construct<TypeBoundProcedureStmt>(
                construct<TypeBoundProcedureStmt::WithoutInterface>(
                    optionalListBeforeColons(Parser<BindAttr>{}),
                    nonemptyList(
                        "expected type bound procedure declarations"_err_en_US,
                        Parser<TypeBoundProcDecl>{})))))

// R750 type-bound-proc-decl -> binding-name [=> procedure-name]
````
- **L577 EN**: Continues logic associated with callable symbol `construct<TypeBoundProcedureStmt>`.
  **L577 CN**: 继续与可调用符号 `construct<TypeBoundProcedureStmt>` 相关的逻辑。
- **L578 EN**: Continues logic associated with callable symbol `WithInterface>`.
  **L578 CN**: 继续与可调用符号 `WithInterface>` 相关的逻辑。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parenthesized(name),`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`parenthesized(name),`。
- **L580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `localRecovery("expected list of binding attributes"_err_en_US,`.
  **L580 CN**: 继续一个多行参数列表、初始化器或聚合项：`localRecovery("expected list of binding attributes"_err_en_US,`。
- **L581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"," >> nonemptyList(Parser<BindAttr>{}), ok),`.
  **L581 CN**: 继续一个多行参数列表、初始化器或聚合项：`"," >> nonemptyList(Parser<BindAttr>{}), ok),`。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `localRecovery("expected list of binding names"_err_en_US,`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`localRecovery("expected list of binding names"_err_en_US,`。
- **L583 EN**: Continues the surrounding expression or declaration: `"::" >> listOfNames, SkipTo<'\n'>{}))) ||`.
  **L583 CN**: 继续构造周围的表达式或声明：`"::" >> listOfNames, SkipTo<'\n'>{}))) ||`。
- **L584 EN**: Continues logic associated with callable symbol `construct<TypeBoundProcedureStmt>`.
  **L584 CN**: 继续与可调用符号 `construct<TypeBoundProcedureStmt>` 相关的逻辑。
- **L585 EN**: Continues logic associated with callable symbol `WithoutInterface>`.
  **L585 CN**: 继续与可调用符号 `WithoutInterface>` 相关的逻辑。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `pure<std::list<BindAttr>>(),`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`pure<std::list<BindAttr>>(),`。
- **L587 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L587 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected type bound procedure declarations"_err_en_US,`.
  **L588 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected type bound procedure declarations"_err_en_US,`。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<TypeBoundProcDecl>(name,`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<TypeBoundProcDecl>(name,`。
- **L590 EN**: Continues logic associated with callable symbol `maybe`.
  **L590 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"type-bound procedure statement should have '::' if it has '=>'"_port_en_US,`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`"type-bound procedure statement should have '::' if it has '=>'"_port_en_US,`。
- **L592 EN**: Continues the surrounding expression or declaration: `"=>" >> name)))))) ||`.
  **L592 CN**: 继续构造周围的表达式或声明：`"=>" >> name)))))) ||`。
- **L593 EN**: Continues logic associated with callable symbol `construct<TypeBoundProcedureStmt>`.
  **L593 CN**: 继续与可调用符号 `construct<TypeBoundProcedureStmt>` 相关的逻辑。
- **L594 EN**: Continues logic associated with callable symbol `WithoutInterface>`.
  **L594 CN**: 继续与可调用符号 `WithoutInterface>` 相关的逻辑。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `optionalListBeforeColons(Parser<BindAttr>{}),`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`optionalListBeforeColons(Parser<BindAttr>{}),`。
- **L596 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L596 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected type bound procedure declarations"_err_en_US,`.
  **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected type bound procedure declarations"_err_en_US,`。
- **L598 EN**: Continues the surrounding expression or declaration: `Parser<TypeBoundProcDecl>{})))))`.
  **L598 CN**: 继续构造周围的表达式或声明：`Parser<TypeBoundProcDecl>{})))))`。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L600 EN**: Comment explains nearby logic, intent, or metadata: `R750 type-bound-proc-decl -> binding-name [=> procedure-name]`.
  **L600 CN**: 注释说明附近代码的逻辑、意图或元数据：`R750 type-bound-proc-decl -> binding-name [=> procedure-name]`。

### Lines 601-624

````cpp
TYPE_PARSER(construct<TypeBoundProcDecl>(name, maybe("=>" >> name)))

// R751 type-bound-generic-stmt ->
//        GENERIC [, access-spec] :: generic-spec => binding-name-list
TYPE_CONTEXT_PARSER("type bound GENERIC statement"_en_US,
    construct<TypeBoundGenericStmt>("GENERIC" >> maybe("," >> accessSpec),
        "::" >> indirect(genericSpec), "=>" >> listOfNames))

// R752 bind-attr ->
//        access-spec | DEFERRED | NON_OVERRIDABLE | NOPASS | PASS [(arg-name)]
TYPE_PARSER(construct<BindAttr>(accessSpec) ||
    construct<BindAttr>(construct<BindAttr::Deferred>("DEFERRED"_tok)) ||
    construct<BindAttr>(
        construct<BindAttr::Non_Overridable>("NON_OVERRIDABLE"_tok)) ||
    construct<BindAttr>(noPass) || construct<BindAttr>(pass))

// R753 final-procedure-stmt -> FINAL [::] final-subroutine-name-list
TYPE_CONTEXT_PARSER("FINAL statement"_en_US,
    construct<FinalProcedureStmt>("FINAL" >> maybe("::"_tok) >> listOfNames))

// R754 derived-type-spec -> type-name [(type-param-spec-list)]
TYPE_PARSER(construct<DerivedTypeSpec>(name,
    defaulted(parenthesized(nonemptyList(
        "expected type parameters"_err_en_US, Parser<TypeParamSpec>{})))))
````
- **L601 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L601 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L603 EN**: Comment explains nearby logic, intent, or metadata: `R751 type-bound-generic-stmt ->`.
  **L603 CN**: 注释说明附近代码的逻辑、意图或元数据：`R751 type-bound-generic-stmt ->`。
- **L604 EN**: Comment explains nearby logic, intent, or metadata: `GENERIC [, access-spec] :: generic-spec => binding-name-list`.
  **L604 CN**: 注释说明附近代码的逻辑、意图或元数据：`GENERIC [, access-spec] :: generic-spec => binding-name-list`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("type bound GENERIC statement"_en_US,`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("type bound GENERIC statement"_en_US,`。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<TypeBoundGenericStmt>("GENERIC" >> maybe("," >> accessSpec),`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<TypeBoundGenericStmt>("GENERIC" >> maybe("," >> accessSpec),`。
- **L607 EN**: Continues logic associated with callable symbol `indirect`.
  **L607 CN**: 继续与可调用符号 `indirect` 相关的逻辑。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L609 EN**: Comment explains nearby logic, intent, or metadata: `R752 bind-attr ->`.
  **L609 CN**: 注释说明附近代码的逻辑、意图或元数据：`R752 bind-attr ->`。
- **L610 EN**: Comment explains nearby logic, intent, or metadata: `access-spec | DEFERRED | NON_OVERRIDABLE | NOPASS | PASS [(arg-name)]`.
  **L610 CN**: 注释说明附近代码的逻辑、意图或元数据：`access-spec | DEFERRED | NON_OVERRIDABLE | NOPASS | PASS [(arg-name)]`。
- **L611 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L611 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L612 EN**: Continues logic associated with callable symbol `construct<BindAttr>`.
  **L612 CN**: 继续与可调用符号 `construct<BindAttr>` 相关的逻辑。
- **L613 EN**: Continues logic associated with callable symbol `construct<BindAttr>`.
  **L613 CN**: 继续与可调用符号 `construct<BindAttr>` 相关的逻辑。
- **L614 EN**: Continues logic associated with callable symbol `Non_Overridable>`.
  **L614 CN**: 继续与可调用符号 `Non_Overridable>` 相关的逻辑。
- **L615 EN**: Continues logic associated with callable symbol `construct<BindAttr>`.
  **L615 CN**: 继续与可调用符号 `construct<BindAttr>` 相关的逻辑。
- **L616 EN**: Blank line separating nearby declarations or logic blocks.
  **L616 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L617 EN**: Comment explains nearby logic, intent, or metadata: `R753 final-procedure-stmt -> FINAL [::] final-subroutine-name-list`.
  **L617 CN**: 注释说明附近代码的逻辑、意图或元数据：`R753 final-procedure-stmt -> FINAL [::] final-subroutine-name-list`。
- **L618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("FINAL statement"_en_US,`.
  **L618 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("FINAL statement"_en_US,`。
- **L619 EN**: Continues logic associated with callable symbol `construct<FinalProcedureStmt>`.
  **L619 CN**: 继续与可调用符号 `construct<FinalProcedureStmt>` 相关的逻辑。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L621 EN**: Comment explains nearby logic, intent, or metadata: `R754 derived-type-spec -> type-name [(type-param-spec-list)]`.
  **L621 CN**: 注释说明附近代码的逻辑、意图或元数据：`R754 derived-type-spec -> type-name [(type-param-spec-list)]`。
- **L622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<DerivedTypeSpec>(name,`.
  **L622 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<DerivedTypeSpec>(name,`。
- **L623 EN**: Continues logic associated with callable symbol `defaulted`.
  **L623 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L624 EN**: Continues the surrounding expression or declaration: `"expected type parameters"_err_en_US, Parser<TypeParamSpec>{})))))`.
  **L624 CN**: 继续构造周围的表达式或声明：`"expected type parameters"_err_en_US, Parser<TypeParamSpec>{})))))`。

### Lines 625-648

````cpp

// R755 type-param-spec -> [keyword =] type-param-value
TYPE_PARSER(construct<TypeParamSpec>(maybe(keyword / "="), typeParamValue))

// R756 structure-constructor -> derived-type-spec ( [component-spec-list] )
TYPE_PARSER((construct<StructureConstructor>(derivedTypeSpec,
                 parenthesized(optionalList(Parser<ComponentSpec>{}))) ||
                // This alternative corrects misrecognition of the
                // component-spec-list as the type-param-spec-list in
                // derived-type-spec.
                construct<StructureConstructor>(
                    construct<DerivedTypeSpec>(
                        name, construct<std::list<TypeParamSpec>>()),
                    parenthesized(optionalList(Parser<ComponentSpec>{})))) /
    !"("_tok)

// R757 component-spec -> [keyword =] component-data-source
TYPE_PARSER(construct<ComponentSpec>(
    maybe(keyword / "="), Parser<ComponentDataSource>{}))

// R758 component-data-source -> expr | data-target | proc-target
TYPE_PARSER(construct<ComponentDataSource>(indirect(expr)))

// R759 enum-def ->
````
- **L625 EN**: Blank line separating nearby declarations or logic blocks.
  **L625 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L626 EN**: Comment explains nearby logic, intent, or metadata: `R755 type-param-spec -> [keyword =] type-param-value`.
  **L626 CN**: 注释说明附近代码的逻辑、意图或元数据：`R755 type-param-spec -> [keyword =] type-param-value`。
- **L627 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L627 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L628 EN**: Blank line separating nearby declarations or logic blocks.
  **L628 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L629 EN**: Comment explains nearby logic, intent, or metadata: `R756 structure-constructor -> derived-type-spec ( [component-spec-list] )`.
  **L629 CN**: 注释说明附近代码的逻辑、意图或元数据：`R756 structure-constructor -> derived-type-spec ( [component-spec-list] )`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER((construct<StructureConstructor>(derivedTypeSpec,`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER((construct<StructureConstructor>(derivedTypeSpec,`。
- **L631 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L631 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L632 EN**: Comment explains nearby logic, intent, or metadata: `This alternative corrects misrecognition of the`.
  **L632 CN**: 注释说明附近代码的逻辑、意图或元数据：`This alternative corrects misrecognition of the`。
- **L633 EN**: Comment explains nearby logic, intent, or metadata: `component-spec-list as the type-param-spec-list in`.
  **L633 CN**: 注释说明附近代码的逻辑、意图或元数据：`component-spec-list as the type-param-spec-list in`。
- **L634 EN**: Comment explains nearby logic, intent, or metadata: `derived-type-spec.`.
  **L634 CN**: 注释说明附近代码的逻辑、意图或元数据：`derived-type-spec.`。
- **L635 EN**: Continues logic associated with callable symbol `construct<StructureConstructor>`.
  **L635 CN**: 继续与可调用符号 `construct<StructureConstructor>` 相关的逻辑。
- **L636 EN**: Continues logic associated with callable symbol `construct<DerivedTypeSpec>`.
  **L636 CN**: 继续与可调用符号 `construct<DerivedTypeSpec>` 相关的逻辑。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `name, construct<std::list<TypeParamSpec>>()),`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`name, construct<std::list<TypeParamSpec>>()),`。
- **L638 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L638 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L639 EN**: Continues the surrounding expression or declaration: `!"("_tok)`.
  **L639 CN**: 继续构造周围的表达式或声明：`!"("_tok)`。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L641 EN**: Comment explains nearby logic, intent, or metadata: `R757 component-spec -> [keyword =] component-data-source`.
  **L641 CN**: 注释说明附近代码的逻辑、意图或元数据：`R757 component-spec -> [keyword =] component-data-source`。
- **L642 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L642 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L643 EN**: Continues logic associated with callable symbol `maybe`.
  **L643 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L645 EN**: Comment explains nearby logic, intent, or metadata: `R758 component-data-source -> expr | data-target | proc-target`.
  **L645 CN**: 注释说明附近代码的逻辑、意图或元数据：`R758 component-data-source -> expr | data-target | proc-target`。
- **L646 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L646 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L648 EN**: Comment explains nearby logic, intent, or metadata: `R759 enum-def ->`.
  **L648 CN**: 注释说明附近代码的逻辑、意图或元数据：`R759 enum-def ->`。

### Lines 649-672

````cpp
//        enum-def-stmt enumerator-def-stmt [enumerator-def-stmt]...
//        end-enum-stmt
TYPE_CONTEXT_PARSER("enum definition"_en_US,
    construct<EnumDef>(statement(Parser<EnumDefStmt>{}),
        some(unambiguousStatement(Parser<EnumeratorDefStmt>{})),
        statement(Parser<EndEnumStmt>{})))

// R760 enum-def-stmt -> ENUM, BIND(C)
TYPE_PARSER(construct<EnumDefStmt>("ENUM , BIND ( C )"_tok))

// R761 enumerator-def-stmt -> ENUMERATOR [::] enumerator-list
TYPE_CONTEXT_PARSER("ENUMERATOR statement"_en_US,
    construct<EnumeratorDefStmt>("ENUMERATOR" >> maybe("::"_tok) >>
        nonemptyList("expected enumerators"_err_en_US, Parser<Enumerator>{})))

// R762 enumerator -> named-constant [= scalar-int-constant-expr]
TYPE_PARSER(
    construct<Enumerator>(namedConstant, maybe("=" >> scalarIntConstantExpr)))

// R763 end-enum-stmt -> END ENUM
TYPE_PARSER(recovery("END ENUM"_tok, constructEndStmtErrorRecovery) >>
    construct<EndEnumStmt>())

// R801 type-declaration-stmt ->
````
- **L649 EN**: Comment explains nearby logic, intent, or metadata: `enum-def-stmt enumerator-def-stmt [enumerator-def-stmt]...`.
  **L649 CN**: 注释说明附近代码的逻辑、意图或元数据：`enum-def-stmt enumerator-def-stmt [enumerator-def-stmt]...`。
- **L650 EN**: Comment explains nearby logic, intent, or metadata: `end-enum-stmt`.
  **L650 CN**: 注释说明附近代码的逻辑、意图或元数据：`end-enum-stmt`。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("enum definition"_en_US,`.
  **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("enum definition"_en_US,`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<EnumDef>(statement(Parser<EnumDefStmt>{}),`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<EnumDef>(statement(Parser<EnumDefStmt>{}),`。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `some(unambiguousStatement(Parser<EnumeratorDefStmt>{})),`.
  **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`some(unambiguousStatement(Parser<EnumeratorDefStmt>{})),`。
- **L654 EN**: Continues logic associated with callable symbol `statement`.
  **L654 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L656 EN**: Comment explains nearby logic, intent, or metadata: `R760 enum-def-stmt -> ENUM, BIND(C)`.
  **L656 CN**: 注释说明附近代码的逻辑、意图或元数据：`R760 enum-def-stmt -> ENUM, BIND(C)`。
- **L657 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L657 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L658 EN**: Blank line separating nearby declarations or logic blocks.
  **L658 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L659 EN**: Comment explains nearby logic, intent, or metadata: `R761 enumerator-def-stmt -> ENUMERATOR [::] enumerator-list`.
  **L659 CN**: 注释说明附近代码的逻辑、意图或元数据：`R761 enumerator-def-stmt -> ENUMERATOR [::] enumerator-list`。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("ENUMERATOR statement"_en_US,`.
  **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("ENUMERATOR statement"_en_US,`。
- **L661 EN**: Continues logic associated with callable symbol `construct<EnumeratorDefStmt>`.
  **L661 CN**: 继续与可调用符号 `construct<EnumeratorDefStmt>` 相关的逻辑。
- **L662 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L662 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L664 EN**: Comment explains nearby logic, intent, or metadata: `R762 enumerator -> named-constant [= scalar-int-constant-expr]`.
  **L664 CN**: 注释说明附近代码的逻辑、意图或元数据：`R762 enumerator -> named-constant [= scalar-int-constant-expr]`。
- **L665 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L665 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L666 EN**: Continues logic associated with callable symbol `construct<Enumerator>`.
  **L666 CN**: 继续与可调用符号 `construct<Enumerator>` 相关的逻辑。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L668 EN**: Comment explains nearby logic, intent, or metadata: `R763 end-enum-stmt -> END ENUM`.
  **L668 CN**: 注释说明附近代码的逻辑、意图或元数据：`R763 end-enum-stmt -> END ENUM`。
- **L669 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L669 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L670 EN**: Continues logic associated with callable symbol `construct<EndEnumStmt>`.
  **L670 CN**: 继续与可调用符号 `construct<EndEnumStmt>` 相关的逻辑。
- **L671 EN**: Blank line separating nearby declarations or logic blocks.
  **L671 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L672 EN**: Comment explains nearby logic, intent, or metadata: `R801 type-declaration-stmt ->`.
  **L672 CN**: 注释说明附近代码的逻辑、意图或元数据：`R801 type-declaration-stmt ->`。

### Lines 673-696

````cpp
//        declaration-type-spec [[, attr-spec]... ::] entity-decl-list
constexpr auto entityDeclWithoutEqInit{
    construct<EntityDecl>(name, "*" >> charLength, maybe(arraySpec),
        maybe(coarraySpec), !"="_tok >> maybe(initialization)) ||
    construct<EntityDecl>(name, maybe(arraySpec), maybe(coarraySpec),
        maybe("*" >> charLength),
        !"="_tok >>
            maybe(initialization) /* old-style REAL A/0/ still works */)};
TYPE_PARSER(
    construct<TypeDeclarationStmt>(declarationTypeSpec,
        defaulted("," >> nonemptyList(Parser<AttrSpec>{})) / "::",
        nonemptyList("expected entity declarations"_err_en_US, entityDecl)) ||
    // C806: no initializers allowed without colons ("REALA=1" is ambiguous)
    construct<TypeDeclarationStmt>(declarationTypeSpec,
        construct<std::list<AttrSpec>>(),
        nonemptyList("expected entity declarations"_err_en_US,
            entityDeclWithoutEqInit)) ||
    // PGI-only extension: comma in place of doubled colons
    extension<LanguageFeature::MissingColons>(
        "nonstandard usage: ',' in place of '::'"_port_en_US,
        construct<TypeDeclarationStmt>(declarationTypeSpec,
            defaulted("," >> nonemptyList(Parser<AttrSpec>{})),
            withMessage("expected entity declarations"_err_en_US,
                "," >> nonemptyList(entityDecl)))))
````
- **L673 EN**: Comment explains nearby logic, intent, or metadata: `declaration-type-spec [[, attr-spec]... ::] entity-decl-list`.
  **L673 CN**: 注释说明附近代码的逻辑、意图或元数据：`declaration-type-spec [[, attr-spec]... ::] entity-decl-list`。
- **L674 EN**: Continues the surrounding expression or declaration: `constexpr auto entityDeclWithoutEqInit{`.
  **L674 CN**: 继续构造周围的表达式或声明：`constexpr auto entityDeclWithoutEqInit{`。
- **L675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<EntityDecl>(name, "*" >> charLength, maybe(arraySpec),`.
  **L675 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<EntityDecl>(name, "*" >> charLength, maybe(arraySpec),`。
- **L676 EN**: Continues logic associated with callable symbol `maybe`.
  **L676 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<EntityDecl>(name, maybe(arraySpec), maybe(coarraySpec),`.
  **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<EntityDecl>(name, maybe(arraySpec), maybe(coarraySpec),`。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `maybe("*" >> charLength),`.
  **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`maybe("*" >> charLength),`。
- **L679 EN**: Continues the surrounding expression or declaration: `!"="_tok >>`.
  **L679 CN**: 继续构造周围的表达式或声明：`!"="_tok >>`。
- **L680 EN**: Executes a call or declaration centered on `maybe`.
  **L680 CN**: 执行以 `maybe` 为核心的调用或声明。
- **L681 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L681 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<TypeDeclarationStmt>(declarationTypeSpec,`.
  **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<TypeDeclarationStmt>(declarationTypeSpec,`。
- **L683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defaulted("," >> nonemptyList(Parser<AttrSpec>{})) / "::",`.
  **L683 CN**: 继续一个多行参数列表、初始化器或聚合项：`defaulted("," >> nonemptyList(Parser<AttrSpec>{})) / "::",`。
- **L684 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L684 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L685 EN**: Comment explains nearby logic, intent, or metadata: `C806: no initializers allowed without colons ("REALA=1" is ambiguous)`.
  **L685 CN**: 注释说明附近代码的逻辑、意图或元数据：`C806: no initializers allowed without colons ("REALA=1" is ambiguous)`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<TypeDeclarationStmt>(declarationTypeSpec,`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<TypeDeclarationStmt>(declarationTypeSpec,`。
- **L687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<std::list<AttrSpec>>(),`.
  **L687 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<std::list<AttrSpec>>(),`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected entity declarations"_err_en_US,`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected entity declarations"_err_en_US,`。
- **L689 EN**: Continues the surrounding expression or declaration: `entityDeclWithoutEqInit)) ||`.
  **L689 CN**: 继续构造周围的表达式或声明：`entityDeclWithoutEqInit)) ||`。
- **L690 EN**: Comment explains nearby logic, intent, or metadata: `PGI-only extension: comma in place of doubled colons`.
  **L690 CN**: 注释说明附近代码的逻辑、意图或元数据：`PGI-only extension: comma in place of doubled colons`。
- **L691 EN**: Continues logic associated with callable symbol `MissingColons>`.
  **L691 CN**: 继续与可调用符号 `MissingColons>` 相关的逻辑。
- **L692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: ',' in place of '::'"_port_en_US,`.
  **L692 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: ',' in place of '::'"_port_en_US,`。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<TypeDeclarationStmt>(declarationTypeSpec,`.
  **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<TypeDeclarationStmt>(declarationTypeSpec,`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defaulted("," >> nonemptyList(Parser<AttrSpec>{})),`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`defaulted("," >> nonemptyList(Parser<AttrSpec>{})),`。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `withMessage("expected entity declarations"_err_en_US,`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`withMessage("expected entity declarations"_err_en_US,`。
- **L696 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L696 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。

### Lines 697-720

````cpp

// R802 attr-spec ->
//        access-spec | ALLOCATABLE | ASYNCHRONOUS |
//        CODIMENSION lbracket coarray-spec rbracket | CONTIGUOUS |
//        DIMENSION ( array-spec ) | EXTERNAL | INTENT ( intent-spec ) |
//        INTRINSIC | language-binding-spec | OPTIONAL | PARAMETER | POINTER |
//        PROTECTED | RANK ( scalar-int-constant-expr ) | SAVE | TARGET |
//        VALUE | VOLATILE |
//        CUDA-data-attr
TYPE_PARSER(construct<AttrSpec>(accessSpec) ||
    construct<AttrSpec>(allocatable) ||
    construct<AttrSpec>(construct<Asynchronous>("ASYNCHRONOUS"_tok)) ||
    construct<AttrSpec>("CODIMENSION" >> coarraySpec) ||
    construct<AttrSpec>(contiguous) ||
    construct<AttrSpec>("DIMENSION" >> arraySpec) ||
    construct<AttrSpec>(construct<External>("EXTERNAL"_tok)) ||
    construct<AttrSpec>("INTENT" >> parenthesized(intentSpec)) ||
    construct<AttrSpec>(construct<Intrinsic>("INTRINSIC"_tok)) ||
    construct<AttrSpec>(languageBindingSpec) || construct<AttrSpec>(optional) ||
    construct<AttrSpec>(construct<Parameter>("PARAMETER"_tok)) ||
    construct<AttrSpec>(pointer) || construct<AttrSpec>(protectedAttr) ||
    construct<AttrSpec>("RANK" >>
        construct<RankClause>(parenthesized(scalarIntConstantExpr))) ||
    construct<AttrSpec>(save) ||
````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L698 EN**: Comment explains nearby logic, intent, or metadata: `R802 attr-spec ->`.
  **L698 CN**: 注释说明附近代码的逻辑、意图或元数据：`R802 attr-spec ->`。
- **L699 EN**: Comment explains nearby logic, intent, or metadata: `access-spec | ALLOCATABLE | ASYNCHRONOUS |`.
  **L699 CN**: 注释说明附近代码的逻辑、意图或元数据：`access-spec | ALLOCATABLE | ASYNCHRONOUS |`。
- **L700 EN**: Comment explains nearby logic, intent, or metadata: `CODIMENSION lbracket coarray-spec rbracket | CONTIGUOUS |`.
  **L700 CN**: 注释说明附近代码的逻辑、意图或元数据：`CODIMENSION lbracket coarray-spec rbracket | CONTIGUOUS |`。
- **L701 EN**: Comment explains nearby logic, intent, or metadata: `DIMENSION ( array-spec ) | EXTERNAL | INTENT ( intent-spec ) |`.
  **L701 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIMENSION ( array-spec ) | EXTERNAL | INTENT ( intent-spec ) |`。
- **L702 EN**: Comment explains nearby logic, intent, or metadata: `INTRINSIC | language-binding-spec | OPTIONAL | PARAMETER | POINTER |`.
  **L702 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTRINSIC | language-binding-spec | OPTIONAL | PARAMETER | POINTER |`。
- **L703 EN**: Comment explains nearby logic, intent, or metadata: `PROTECTED | RANK ( scalar-int-constant-expr ) | SAVE | TARGET |`.
  **L703 CN**: 注释说明附近代码的逻辑、意图或元数据：`PROTECTED | RANK ( scalar-int-constant-expr ) | SAVE | TARGET |`。
- **L704 EN**: Comment explains nearby logic, intent, or metadata: `VALUE | VOLATILE |`.
  **L704 CN**: 注释说明附近代码的逻辑、意图或元数据：`VALUE | VOLATILE |`。
- **L705 EN**: Comment explains nearby logic, intent, or metadata: `CUDA-data-attr`.
  **L705 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUDA-data-attr`。
- **L706 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L706 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L707 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L707 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L708 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L708 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L709 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L709 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L710 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L710 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L711 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L711 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L712 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L712 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L713 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L713 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L714 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L714 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L715 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L715 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L716 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L716 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L717 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L717 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L718 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L718 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L719 EN**: Continues logic associated with callable symbol `construct<RankClause>`.
  **L719 CN**: 继续与可调用符号 `construct<RankClause>` 相关的逻辑。
- **L720 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L720 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。

### Lines 721-744

````cpp
    construct<AttrSpec>(construct<Target>("TARGET"_tok)) ||
    construct<AttrSpec>(construct<Value>("VALUE"_tok)) ||
    construct<AttrSpec>(construct<Volatile>("VOLATILE"_tok)) ||
    extension<LanguageFeature::CUDA>(
        construct<AttrSpec>(Parser<common::CUDADataAttr>{})))

// CUDA-data-attr ->
//     CONSTANT | DEVICE | MANAGED | PINNED | SHARED | TEXTURE | UNIFIED
TYPE_PARSER("CONSTANT" >> pure(common::CUDADataAttr::Constant) ||
    "DEVICE" >> pure(common::CUDADataAttr::Device) ||
    "MANAGED" >> pure(common::CUDADataAttr::Managed) ||
    "PINNED" >> pure(common::CUDADataAttr::Pinned) ||
    "SHARED" >> pure(common::CUDADataAttr::Shared) ||
    "TEXTURE" >> pure(common::CUDADataAttr::Texture) ||
    "UNIFIED" >> pure(common::CUDADataAttr::Unified) ||
    "VALUE" >> pure(common::CUDADataAttr::Value))

// R804 object-name -> name
constexpr auto objectName{name};

// R803 entity-decl ->
//        object-name [( array-spec )] [lbracket coarray-spec rbracket]
//          [* char-length] [initialization] |
//        function-name [* char-length] |
````
- **L721 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L721 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L722 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L722 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L723 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L723 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L724 EN**: Continues logic associated with callable symbol `CUDA>`.
  **L724 CN**: 继续与可调用符号 `CUDA>` 相关的逻辑。
- **L725 EN**: Continues logic associated with callable symbol `construct<AttrSpec>`.
  **L725 CN**: 继续与可调用符号 `construct<AttrSpec>` 相关的逻辑。
- **L726 EN**: Blank line separating nearby declarations or logic blocks.
  **L726 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L727 EN**: Comment explains nearby logic, intent, or metadata: `CUDA-data-attr ->`.
  **L727 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUDA-data-attr ->`。
- **L728 EN**: Comment explains nearby logic, intent, or metadata: `CONSTANT | DEVICE | MANAGED | PINNED | SHARED | TEXTURE | UNIFIED`.
  **L728 CN**: 注释说明附近代码的逻辑、意图或元数据：`CONSTANT | DEVICE | MANAGED | PINNED | SHARED | TEXTURE | UNIFIED`。
- **L729 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L729 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L730 EN**: Continues logic associated with callable symbol `pure`.
  **L730 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L731 EN**: Continues logic associated with callable symbol `pure`.
  **L731 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L732 EN**: Continues logic associated with callable symbol `pure`.
  **L732 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L733 EN**: Continues logic associated with callable symbol `pure`.
  **L733 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L734 EN**: Continues logic associated with callable symbol `pure`.
  **L734 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L735 EN**: Continues logic associated with callable symbol `pure`.
  **L735 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L736 EN**: Continues logic associated with callable symbol `pure`.
  **L736 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L738 EN**: Comment explains nearby logic, intent, or metadata: `R804 object-name -> name`.
  **L738 CN**: 注释说明附近代码的逻辑、意图或元数据：`R804 object-name -> name`。
- **L739 EN**: Executes a standalone statement or declaration: `constexpr auto objectName{name};`.
  **L739 CN**: 执行一条独立语句或声明：`constexpr auto objectName{name};`。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L741 EN**: Comment explains nearby logic, intent, or metadata: `R803 entity-decl ->`.
  **L741 CN**: 注释说明附近代码的逻辑、意图或元数据：`R803 entity-decl ->`。
- **L742 EN**: Comment explains nearby logic, intent, or metadata: `object-name [( array-spec )] [lbracket coarray-spec rbracket]`.
  **L742 CN**: 注释说明附近代码的逻辑、意图或元数据：`object-name [( array-spec )] [lbracket coarray-spec rbracket]`。
- **L743 EN**: Comment explains nearby logic, intent, or metadata: `[* char-length] [initialization] |`.
  **L743 CN**: 注释说明附近代码的逻辑、意图或元数据：`[* char-length] [initialization] |`。
- **L744 EN**: Comment explains nearby logic, intent, or metadata: `function-name [* char-length] |`.
  **L744 CN**: 注释说明附近代码的逻辑、意图或元数据：`function-name [* char-length] |`。

### Lines 745-768

````cpp
// (ext.) object-name *char-length [(array-spec)]
//          [lbracket coarray-spec rbracket] [initialization]
TYPE_PARSER(construct<EntityDecl>(objectName, "*" >> charLength,
                maybe(arraySpec), maybe(coarraySpec), maybe(initialization)) ||
    construct<EntityDecl>(objectName, maybe(arraySpec), maybe(coarraySpec),
        maybe("*" >> charLength), maybe(initialization)))

// R806 null-init -> function-reference   ... which must resolve to NULL()
TYPE_PARSER(lookAhead(name / "( )") >> construct<NullInit>(expr))

// R807 access-spec -> PUBLIC | PRIVATE
TYPE_PARSER(construct<AccessSpec>("PUBLIC" >> pure(AccessSpec::Kind::Public)) ||
    construct<AccessSpec>("PRIVATE" >> pure(AccessSpec::Kind::Private)))

// R808 language-binding-spec ->
//        BIND ( C [, NAME = scalar-default-char-constant-expr] )
// R1528 proc-language-binding-spec -> language-binding-spec
TYPE_PARSER(construct<LanguageBindingSpec>(
    "BIND ( C" >> maybe(", NAME =" >> scalarDefaultCharConstantExpr),
    (", CDEFINED" >> pure(true) || pure(false)) / ")"))

// R809 coarray-spec -> deferred-coshape-spec-list | explicit-coshape-spec
// N.B. Bracketed here rather than around references, for consistency with
// array-spec.
````
- **L745 EN**: Comment explains nearby logic, intent, or metadata: `(ext.) object-name *char-length [(array-spec)]`.
  **L745 CN**: 注释说明附近代码的逻辑、意图或元数据：`(ext.) object-name *char-length [(array-spec)]`。
- **L746 EN**: Comment explains nearby logic, intent, or metadata: `[lbracket coarray-spec rbracket] [initialization]`.
  **L746 CN**: 注释说明附近代码的逻辑、意图或元数据：`[lbracket coarray-spec rbracket] [initialization]`。
- **L747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<EntityDecl>(objectName, "*" >> charLength,`.
  **L747 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<EntityDecl>(objectName, "*" >> charLength,`。
- **L748 EN**: Continues logic associated with callable symbol `maybe`.
  **L748 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<EntityDecl>(objectName, maybe(arraySpec), maybe(coarraySpec),`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<EntityDecl>(objectName, maybe(arraySpec), maybe(coarraySpec),`。
- **L750 EN**: Continues logic associated with callable symbol `maybe`.
  **L750 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L751 EN**: Blank line separating nearby declarations or logic blocks.
  **L751 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L752 EN**: Comment explains nearby logic, intent, or metadata: `R806 null-init -> function-reference   ... which must resolve to NULL()`.
  **L752 CN**: 注释说明附近代码的逻辑、意图或元数据：`R806 null-init -> function-reference   ... which must resolve to NULL()`。
- **L753 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L753 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L755 EN**: Comment explains nearby logic, intent, or metadata: `R807 access-spec -> PUBLIC | PRIVATE`.
  **L755 CN**: 注释说明附近代码的逻辑、意图或元数据：`R807 access-spec -> PUBLIC | PRIVATE`。
- **L756 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L756 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L757 EN**: Continues logic associated with callable symbol `construct<AccessSpec>`.
  **L757 CN**: 继续与可调用符号 `construct<AccessSpec>` 相关的逻辑。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L759 EN**: Comment explains nearby logic, intent, or metadata: `R808 language-binding-spec ->`.
  **L759 CN**: 注释说明附近代码的逻辑、意图或元数据：`R808 language-binding-spec ->`。
- **L760 EN**: Comment explains nearby logic, intent, or metadata: `BIND ( C [, NAME = scalar-default-char-constant-expr] )`.
  **L760 CN**: 注释说明附近代码的逻辑、意图或元数据：`BIND ( C [, NAME = scalar-default-char-constant-expr] )`。
- **L761 EN**: Comment explains nearby logic, intent, or metadata: `R1528 proc-language-binding-spec -> language-binding-spec`.
  **L761 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1528 proc-language-binding-spec -> language-binding-spec`。
- **L762 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L762 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"BIND ( C" >> maybe(", NAME =" >> scalarDefaultCharConstantExpr),`.
  **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`"BIND ( C" >> maybe(", NAME =" >> scalarDefaultCharConstantExpr),`。
- **L764 EN**: Continues logic associated with callable symbol `pure`.
  **L764 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L766 EN**: Comment explains nearby logic, intent, or metadata: `R809 coarray-spec -> deferred-coshape-spec-list | explicit-coshape-spec`.
  **L766 CN**: 注释说明附近代码的逻辑、意图或元数据：`R809 coarray-spec -> deferred-coshape-spec-list | explicit-coshape-spec`。
- **L767 EN**: Comment explains nearby logic, intent, or metadata: `N.B. Bracketed here rather than around references, for consistency with`.
  **L767 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. Bracketed here rather than around references, for consistency with`。
- **L768 EN**: Comment explains nearby logic, intent, or metadata: `array-spec.`.
  **L768 CN**: 注释说明附近代码的逻辑、意图或元数据：`array-spec.`。

### Lines 769-792

````cpp
TYPE_PARSER(
    construct<CoarraySpec>(bracketed(Parser<DeferredCoshapeSpecList>{})) ||
    construct<CoarraySpec>(bracketed(Parser<ExplicitCoshapeSpec>{})))

// R810 deferred-coshape-spec -> :
// deferred-coshape-spec-list - just a list of colons
inline int listLength(std::list<Success> &&xs) { return xs.size(); }

TYPE_PARSER(construct<DeferredCoshapeSpecList>(
    applyFunction(listLength, nonemptyList(":"_tok))))

// R811 explicit-coshape-spec ->
//        [[lower-cobound :] upper-cobound ,]... [lower-cobound :] *
// R812 lower-cobound -> specification-expr
// R813 upper-cobound -> specification-expr
TYPE_PARSER(construct<ExplicitCoshapeSpec>(
    many(explicitShapeSpec / ","), maybe(specificationExpr / ":") / "*"))

// R815 array-spec ->
//        explicit-shape-spec-list | assumed-shape-spec-list |
//        deferred-shape-spec-list | assumed-size-spec | implied-shape-spec |
//        implied-shape-or-assumed-size-spec | assumed-rank-spec
// N.B. Parenthesized here rather than around references to avoid
// a need for forced look-ahead.
````
- **L769 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L769 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L770 EN**: Continues logic associated with callable symbol `construct<CoarraySpec>`.
  **L770 CN**: 继续与可调用符号 `construct<CoarraySpec>` 相关的逻辑。
- **L771 EN**: Continues logic associated with callable symbol `construct<CoarraySpec>`.
  **L771 CN**: 继续与可调用符号 `construct<CoarraySpec>` 相关的逻辑。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L773 EN**: Comment explains nearby logic, intent, or metadata: `R810 deferred-coshape-spec -> :`.
  **L773 CN**: 注释说明附近代码的逻辑、意图或元数据：`R810 deferred-coshape-spec -> :`。
- **L774 EN**: Comment explains nearby logic, intent, or metadata: `deferred-coshape-spec-list - just a list of colons`.
  **L774 CN**: 注释说明附近代码的逻辑、意图或元数据：`deferred-coshape-spec-list - just a list of colons`。
- **L775 EN**: Continues logic associated with callable symbol `listLength`.
  **L775 CN**: 继续与可调用符号 `listLength` 相关的逻辑。
- **L776 EN**: Blank line separating nearby declarations or logic blocks.
  **L776 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L777 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L777 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L778 EN**: Continues logic associated with callable symbol `applyFunction`.
  **L778 CN**: 继续与可调用符号 `applyFunction` 相关的逻辑。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L780 EN**: Comment explains nearby logic, intent, or metadata: `R811 explicit-coshape-spec ->`.
  **L780 CN**: 注释说明附近代码的逻辑、意图或元数据：`R811 explicit-coshape-spec ->`。
- **L781 EN**: Comment explains nearby logic, intent, or metadata: `[[lower-cobound :] upper-cobound ,]... [lower-cobound :]`.
  **L781 CN**: 注释说明附近代码的逻辑、意图或元数据：`[[lower-cobound :] upper-cobound ,]... [lower-cobound :]`。
- **L782 EN**: Comment explains nearby logic, intent, or metadata: `R812 lower-cobound -> specification-expr`.
  **L782 CN**: 注释说明附近代码的逻辑、意图或元数据：`R812 lower-cobound -> specification-expr`。
- **L783 EN**: Comment explains nearby logic, intent, or metadata: `R813 upper-cobound -> specification-expr`.
  **L783 CN**: 注释说明附近代码的逻辑、意图或元数据：`R813 upper-cobound -> specification-expr`。
- **L784 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L784 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L785 EN**: Continues logic associated with callable symbol `many`.
  **L785 CN**: 继续与可调用符号 `many` 相关的逻辑。
- **L786 EN**: Blank line separating nearby declarations or logic blocks.
  **L786 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L787 EN**: Comment explains nearby logic, intent, or metadata: `R815 array-spec ->`.
  **L787 CN**: 注释说明附近代码的逻辑、意图或元数据：`R815 array-spec ->`。
- **L788 EN**: Comment explains nearby logic, intent, or metadata: `explicit-shape-spec-list | assumed-shape-spec-list |`.
  **L788 CN**: 注释说明附近代码的逻辑、意图或元数据：`explicit-shape-spec-list | assumed-shape-spec-list |`。
- **L789 EN**: Comment explains nearby logic, intent, or metadata: `deferred-shape-spec-list | assumed-size-spec | implied-shape-spec |`.
  **L789 CN**: 注释说明附近代码的逻辑、意图或元数据：`deferred-shape-spec-list | assumed-size-spec | implied-shape-spec |`。
- **L790 EN**: Comment explains nearby logic, intent, or metadata: `implied-shape-or-assumed-size-spec | assumed-rank-spec`.
  **L790 CN**: 注释说明附近代码的逻辑、意图或元数据：`implied-shape-or-assumed-size-spec | assumed-rank-spec`。
- **L791 EN**: Comment explains nearby logic, intent, or metadata: `N.B. Parenthesized here rather than around references to avoid`.
  **L791 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. Parenthesized here rather than around references to avoid`。
- **L792 EN**: Comment explains nearby logic, intent, or metadata: `a need for forced look-ahead.`.
  **L792 CN**: 注释说明附近代码的逻辑、意图或元数据：`a need for forced look-ahead.`。

### Lines 793-816

````cpp
// Shape specs that could be deferred-shape-spec or assumed-shape-spec
// (e.g. '(:,:)') are parsed as the former.
TYPE_PARSER(
    construct<ArraySpec>(parenthesized(nonemptyList(explicitShapeSpec))) ||
    construct<ArraySpec>(parenthesized(deferredShapeSpecList)) ||
    construct<ArraySpec>(
        parenthesized(nonemptyList(Parser<AssumedShapeSpec>{}))) ||
    construct<ArraySpec>(parenthesized(Parser<AssumedSizeSpec>{})) ||
    construct<ArraySpec>(parenthesized(Parser<ImpliedShapeSpec>{})) ||
    construct<ArraySpec>(parenthesized(Parser<AssumedRankSpec>{})))

// R816 explicit-shape-spec -> [lower-bound :] upper-bound
// R817 lower-bound -> specification-expr
// R818 upper-bound -> specification-expr
TYPE_PARSER(construct<ExplicitShapeSpec>(
    maybe(specificationExpr / ":"), specificationExpr))

// R819 assumed-shape-spec -> [lower-bound] :
TYPE_PARSER(construct<AssumedShapeSpec>(maybe(specificationExpr) / ":"))

// R820 deferred-shape-spec -> :
// deferred-shape-spec-list - just a list of colons
TYPE_PARSER(construct<DeferredShapeSpecList>(
    applyFunction(listLength, nonemptyList(":"_tok))))
````
- **L793 EN**: Comment explains nearby logic, intent, or metadata: `Shape specs that could be deferred-shape-spec or assumed-shape-spec`.
  **L793 CN**: 注释说明附近代码的逻辑、意图或元数据：`Shape specs that could be deferred-shape-spec or assumed-shape-spec`。
- **L794 EN**: Comment explains nearby logic, intent, or metadata: `(e.g. '(:,:)') are parsed as the former.`.
  **L794 CN**: 注释说明附近代码的逻辑、意图或元数据：`(e.g. '(:,:)') are parsed as the former.`。
- **L795 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L795 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L796 EN**: Continues logic associated with callable symbol `construct<ArraySpec>`.
  **L796 CN**: 继续与可调用符号 `construct<ArraySpec>` 相关的逻辑。
- **L797 EN**: Continues logic associated with callable symbol `construct<ArraySpec>`.
  **L797 CN**: 继续与可调用符号 `construct<ArraySpec>` 相关的逻辑。
- **L798 EN**: Continues logic associated with callable symbol `construct<ArraySpec>`.
  **L798 CN**: 继续与可调用符号 `construct<ArraySpec>` 相关的逻辑。
- **L799 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L799 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L800 EN**: Continues logic associated with callable symbol `construct<ArraySpec>`.
  **L800 CN**: 继续与可调用符号 `construct<ArraySpec>` 相关的逻辑。
- **L801 EN**: Continues logic associated with callable symbol `construct<ArraySpec>`.
  **L801 CN**: 继续与可调用符号 `construct<ArraySpec>` 相关的逻辑。
- **L802 EN**: Continues logic associated with callable symbol `construct<ArraySpec>`.
  **L802 CN**: 继续与可调用符号 `construct<ArraySpec>` 相关的逻辑。
- **L803 EN**: Blank line separating nearby declarations or logic blocks.
  **L803 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L804 EN**: Comment explains nearby logic, intent, or metadata: `R816 explicit-shape-spec -> [lower-bound :] upper-bound`.
  **L804 CN**: 注释说明附近代码的逻辑、意图或元数据：`R816 explicit-shape-spec -> [lower-bound :] upper-bound`。
- **L805 EN**: Comment explains nearby logic, intent, or metadata: `R817 lower-bound -> specification-expr`.
  **L805 CN**: 注释说明附近代码的逻辑、意图或元数据：`R817 lower-bound -> specification-expr`。
- **L806 EN**: Comment explains nearby logic, intent, or metadata: `R818 upper-bound -> specification-expr`.
  **L806 CN**: 注释说明附近代码的逻辑、意图或元数据：`R818 upper-bound -> specification-expr`。
- **L807 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L807 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L808 EN**: Continues logic associated with callable symbol `maybe`.
  **L808 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L809 EN**: Blank line separating nearby declarations or logic blocks.
  **L809 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L810 EN**: Comment explains nearby logic, intent, or metadata: `R819 assumed-shape-spec -> [lower-bound] :`.
  **L810 CN**: 注释说明附近代码的逻辑、意图或元数据：`R819 assumed-shape-spec -> [lower-bound] :`。
- **L811 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L811 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L813 EN**: Comment explains nearby logic, intent, or metadata: `R820 deferred-shape-spec -> :`.
  **L813 CN**: 注释说明附近代码的逻辑、意图或元数据：`R820 deferred-shape-spec -> :`。
- **L814 EN**: Comment explains nearby logic, intent, or metadata: `deferred-shape-spec-list - just a list of colons`.
  **L814 CN**: 注释说明附近代码的逻辑、意图或元数据：`deferred-shape-spec-list - just a list of colons`。
- **L815 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L815 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L816 EN**: Continues logic associated with callable symbol `applyFunction`.
  **L816 CN**: 继续与可调用符号 `applyFunction` 相关的逻辑。

### Lines 817-840

````cpp

// R821 assumed-implied-spec -> [lower-bound :] *
TYPE_PARSER(construct<AssumedImpliedSpec>(maybe(specificationExpr / ":") / "*"))

// R822 assumed-size-spec -> explicit-shape-spec-list , assumed-implied-spec
TYPE_PARSER(construct<AssumedSizeSpec>(
    nonemptyList(explicitShapeSpec) / ",", assumedImpliedSpec))

// R823 implied-shape-or-assumed-size-spec -> assumed-implied-spec
// R824 implied-shape-spec -> assumed-implied-spec , assumed-implied-spec-list
// I.e., when the assumed-implied-spec-list has a single item, it constitutes an
// implied-shape-or-assumed-size-spec; otherwise, an implied-shape-spec.
TYPE_PARSER(construct<ImpliedShapeSpec>(nonemptyList(assumedImpliedSpec)))

// R825 assumed-rank-spec -> ..
TYPE_PARSER(construct<AssumedRankSpec>(".."_tok))

// R826 intent-spec -> IN | OUT | INOUT
TYPE_PARSER(construct<IntentSpec>("IN OUT" >> pure(IntentSpec::Intent::InOut) ||
    "IN" >> pure(IntentSpec::Intent::In) ||
    "OUT" >> pure(IntentSpec::Intent::Out)))

// R827 access-stmt -> access-spec [[::] access-id-list]
TYPE_PARSER(construct<AccessStmt>(accessSpec,
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L818 EN**: Comment explains nearby logic, intent, or metadata: `R821 assumed-implied-spec -> [lower-bound :]`.
  **L818 CN**: 注释说明附近代码的逻辑、意图或元数据：`R821 assumed-implied-spec -> [lower-bound :]`。
- **L819 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L819 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L820 EN**: Blank line separating nearby declarations or logic blocks.
  **L820 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L821 EN**: Comment explains nearby logic, intent, or metadata: `R822 assumed-size-spec -> explicit-shape-spec-list , assumed-implied-spec`.
  **L821 CN**: 注释说明附近代码的逻辑、意图或元数据：`R822 assumed-size-spec -> explicit-shape-spec-list , assumed-implied-spec`。
- **L822 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L822 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L823 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L823 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L824 EN**: Blank line separating nearby declarations or logic blocks.
  **L824 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L825 EN**: Comment explains nearby logic, intent, or metadata: `R823 implied-shape-or-assumed-size-spec -> assumed-implied-spec`.
  **L825 CN**: 注释说明附近代码的逻辑、意图或元数据：`R823 implied-shape-or-assumed-size-spec -> assumed-implied-spec`。
- **L826 EN**: Comment explains nearby logic, intent, or metadata: `R824 implied-shape-spec -> assumed-implied-spec , assumed-implied-spec-list`.
  **L826 CN**: 注释说明附近代码的逻辑、意图或元数据：`R824 implied-shape-spec -> assumed-implied-spec , assumed-implied-spec-list`。
- **L827 EN**: Comment explains nearby logic, intent, or metadata: `I.e., when the assumed-implied-spec-list has a single item, it constitutes an`.
  **L827 CN**: 注释说明附近代码的逻辑、意图或元数据：`I.e., when the assumed-implied-spec-list has a single item, it constitutes an`。
- **L828 EN**: Comment explains nearby logic, intent, or metadata: `implied-shape-or-assumed-size-spec; otherwise, an implied-shape-spec.`.
  **L828 CN**: 注释说明附近代码的逻辑、意图或元数据：`implied-shape-or-assumed-size-spec; otherwise, an implied-shape-spec.`。
- **L829 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L829 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L831 EN**: Comment explains nearby logic, intent, or metadata: `R825 assumed-rank-spec -> ..`.
  **L831 CN**: 注释说明附近代码的逻辑、意图或元数据：`R825 assumed-rank-spec -> ..`。
- **L832 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L832 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L833 EN**: Blank line separating nearby declarations or logic blocks.
  **L833 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L834 EN**: Comment explains nearby logic, intent, or metadata: `R826 intent-spec -> IN | OUT | INOUT`.
  **L834 CN**: 注释说明附近代码的逻辑、意图或元数据：`R826 intent-spec -> IN | OUT | INOUT`。
- **L835 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L835 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L836 EN**: Continues logic associated with callable symbol `pure`.
  **L836 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L837 EN**: Continues logic associated with callable symbol `pure`.
  **L837 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L838 EN**: Blank line separating nearby declarations or logic blocks.
  **L838 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L839 EN**: Comment explains nearby logic, intent, or metadata: `R827 access-stmt -> access-spec [[::] access-id-list]`.
  **L839 CN**: 注释说明附近代码的逻辑、意图或元数据：`R827 access-stmt -> access-spec [[::] access-id-list]`。
- **L840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<AccessStmt>(accessSpec,`.
  **L840 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<AccessStmt>(accessSpec,`。

### Lines 841-864

````cpp
    defaulted(maybe("::"_tok) >>
        nonemptyList("expected names and generic specifications"_err_en_US,
            Parser<AccessId>{}))))

// R828 access-id -> access-name | generic-spec
// "access-name" is ambiguous with "generic-spec"
TYPE_PARSER(construct<AccessId>(indirect(genericSpec)))

// R829 allocatable-stmt -> ALLOCATABLE [::] allocatable-decl-list
TYPE_PARSER(construct<AllocatableStmt>("ALLOCATABLE" >> maybe("::"_tok) >>
    nonemptyList(
        "expected object declarations"_err_en_US, Parser<ObjectDecl>{})))

// R830 allocatable-decl ->
//        object-name [( array-spec )] [lbracket coarray-spec rbracket]
// R860 target-decl ->
//        object-name [( array-spec )] [lbracket coarray-spec rbracket]
TYPE_PARSER(
    construct<ObjectDecl>(objectName, maybe(arraySpec), maybe(coarraySpec)))

// R831 asynchronous-stmt -> ASYNCHRONOUS [::] object-name-list
TYPE_PARSER(construct<AsynchronousStmt>("ASYNCHRONOUS" >> maybe("::"_tok) >>
    nonemptyList("expected object names"_err_en_US, objectName)))

````
- **L841 EN**: Continues logic associated with callable symbol `defaulted`.
  **L841 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected names and generic specifications"_err_en_US,`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected names and generic specifications"_err_en_US,`。
- **L843 EN**: Continues the surrounding expression or declaration: `Parser<AccessId>{}))))`.
  **L843 CN**: 继续构造周围的表达式或声明：`Parser<AccessId>{}))))`。
- **L844 EN**: Blank line separating nearby declarations or logic blocks.
  **L844 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L845 EN**: Comment explains nearby logic, intent, or metadata: `R828 access-id -> access-name | generic-spec`.
  **L845 CN**: 注释说明附近代码的逻辑、意图或元数据：`R828 access-id -> access-name | generic-spec`。
- **L846 EN**: Comment explains nearby logic, intent, or metadata: `"access-name" is ambiguous with "generic-spec"`.
  **L846 CN**: 注释说明附近代码的逻辑、意图或元数据：`"access-name" is ambiguous with "generic-spec"`。
- **L847 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L847 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L849 EN**: Comment explains nearby logic, intent, or metadata: `R829 allocatable-stmt -> ALLOCATABLE [::] allocatable-decl-list`.
  **L849 CN**: 注释说明附近代码的逻辑、意图或元数据：`R829 allocatable-stmt -> ALLOCATABLE [::] allocatable-decl-list`。
- **L850 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L850 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L851 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L851 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L852 EN**: Continues the surrounding expression or declaration: `"expected object declarations"_err_en_US, Parser<ObjectDecl>{})))`.
  **L852 CN**: 继续构造周围的表达式或声明：`"expected object declarations"_err_en_US, Parser<ObjectDecl>{})))`。
- **L853 EN**: Blank line separating nearby declarations or logic blocks.
  **L853 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L854 EN**: Comment explains nearby logic, intent, or metadata: `R830 allocatable-decl ->`.
  **L854 CN**: 注释说明附近代码的逻辑、意图或元数据：`R830 allocatable-decl ->`。
- **L855 EN**: Comment explains nearby logic, intent, or metadata: `object-name [( array-spec )] [lbracket coarray-spec rbracket]`.
  **L855 CN**: 注释说明附近代码的逻辑、意图或元数据：`object-name [( array-spec )] [lbracket coarray-spec rbracket]`。
- **L856 EN**: Comment explains nearby logic, intent, or metadata: `R860 target-decl ->`.
  **L856 CN**: 注释说明附近代码的逻辑、意图或元数据：`R860 target-decl ->`。
- **L857 EN**: Comment explains nearby logic, intent, or metadata: `object-name [( array-spec )] [lbracket coarray-spec rbracket]`.
  **L857 CN**: 注释说明附近代码的逻辑、意图或元数据：`object-name [( array-spec )] [lbracket coarray-spec rbracket]`。
- **L858 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L858 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L859 EN**: Continues logic associated with callable symbol `construct<ObjectDecl>`.
  **L859 CN**: 继续与可调用符号 `construct<ObjectDecl>` 相关的逻辑。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L861 EN**: Comment explains nearby logic, intent, or metadata: `R831 asynchronous-stmt -> ASYNCHRONOUS [::] object-name-list`.
  **L861 CN**: 注释说明附近代码的逻辑、意图或元数据：`R831 asynchronous-stmt -> ASYNCHRONOUS [::] object-name-list`。
- **L862 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L862 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L863 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L863 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L864 EN**: Blank line separating nearby declarations or logic blocks.
  **L864 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 865-888

````cpp
// R832 bind-stmt -> language-binding-spec [::] bind-entity-list
TYPE_PARSER(construct<BindStmt>(languageBindingSpec / maybe("::"_tok),
    nonemptyList("expected bind entities"_err_en_US, Parser<BindEntity>{})))

// R833 bind-entity -> entity-name | / common-block-name /
TYPE_PARSER(construct<BindEntity>(pure(BindEntity::Kind::Object), name) ||
    construct<BindEntity>("/" >> pure(BindEntity::Kind::Common), name / "/"))

// R834 codimension-stmt -> CODIMENSION [::] codimension-decl-list
TYPE_PARSER(construct<CodimensionStmt>("CODIMENSION" >> maybe("::"_tok) >>
    nonemptyList("expected codimension declarations"_err_en_US,
        Parser<CodimensionDecl>{})))

// R835 codimension-decl -> coarray-name lbracket coarray-spec rbracket
TYPE_PARSER(construct<CodimensionDecl>(name, coarraySpec))

// R836 contiguous-stmt -> CONTIGUOUS [::] object-name-list
TYPE_PARSER(construct<ContiguousStmt>("CONTIGUOUS" >> maybe("::"_tok) >>
    nonemptyList("expected object names"_err_en_US, objectName)))

// R837 data-stmt -> DATA data-stmt-set [[,] data-stmt-set]...
TYPE_CONTEXT_PARSER("DATA statement"_en_US,
    construct<DataStmt>(
        "DATA" >> nonemptySeparated(Parser<DataStmtSet>{}, maybe(","_tok))))
````
- **L865 EN**: Comment explains nearby logic, intent, or metadata: `R832 bind-stmt -> language-binding-spec [::] bind-entity-list`.
  **L865 CN**: 注释说明附近代码的逻辑、意图或元数据：`R832 bind-stmt -> language-binding-spec [::] bind-entity-list`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<BindStmt>(languageBindingSpec / maybe("::"_tok),`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<BindStmt>(languageBindingSpec / maybe("::"_tok),`。
- **L867 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L867 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L869 EN**: Comment explains nearby logic, intent, or metadata: `R833 bind-entity -> entity-name | / common-block-name`.
  **L869 CN**: 注释说明附近代码的逻辑、意图或元数据：`R833 bind-entity -> entity-name | / common-block-name`。
- **L870 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L870 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L871 EN**: Continues logic associated with callable symbol `construct<BindEntity>`.
  **L871 CN**: 继续与可调用符号 `construct<BindEntity>` 相关的逻辑。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L873 EN**: Comment explains nearby logic, intent, or metadata: `R834 codimension-stmt -> CODIMENSION [::] codimension-decl-list`.
  **L873 CN**: 注释说明附近代码的逻辑、意图或元数据：`R834 codimension-stmt -> CODIMENSION [::] codimension-decl-list`。
- **L874 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L874 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected codimension declarations"_err_en_US,`.
  **L875 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected codimension declarations"_err_en_US,`。
- **L876 EN**: Continues the surrounding expression or declaration: `Parser<CodimensionDecl>{})))`.
  **L876 CN**: 继续构造周围的表达式或声明：`Parser<CodimensionDecl>{})))`。
- **L877 EN**: Blank line separating nearby declarations or logic blocks.
  **L877 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L878 EN**: Comment explains nearby logic, intent, or metadata: `R835 codimension-decl -> coarray-name lbracket coarray-spec rbracket`.
  **L878 CN**: 注释说明附近代码的逻辑、意图或元数据：`R835 codimension-decl -> coarray-name lbracket coarray-spec rbracket`。
- **L879 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L879 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L881 EN**: Comment explains nearby logic, intent, or metadata: `R836 contiguous-stmt -> CONTIGUOUS [::] object-name-list`.
  **L881 CN**: 注释说明附近代码的逻辑、意图或元数据：`R836 contiguous-stmt -> CONTIGUOUS [::] object-name-list`。
- **L882 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L882 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L883 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L883 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L884 EN**: Blank line separating nearby declarations or logic blocks.
  **L884 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L885 EN**: Comment explains nearby logic, intent, or metadata: `R837 data-stmt -> DATA data-stmt-set [[,] data-stmt-set]...`.
  **L885 CN**: 注释说明附近代码的逻辑、意图或元数据：`R837 data-stmt -> DATA data-stmt-set [[,] data-stmt-set]...`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("DATA statement"_en_US,`.
  **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("DATA statement"_en_US,`。
- **L887 EN**: Continues logic associated with callable symbol `construct<DataStmt>`.
  **L887 CN**: 继续与可调用符号 `construct<DataStmt>` 相关的逻辑。
- **L888 EN**: Continues logic associated with callable symbol `nonemptySeparated`.
  **L888 CN**: 继续与可调用符号 `nonemptySeparated` 相关的逻辑。

### Lines 889-912

````cpp

// R838 data-stmt-set -> data-stmt-object-list / data-stmt-value-list /
TYPE_PARSER(construct<DataStmtSet>(
    nonemptyList(
        "expected DATA statement objects"_err_en_US, Parser<DataStmtObject>{}),
    withMessage("expected DATA statement value list"_err_en_US,
        "/"_tok >> nonemptyList("expected DATA statement values"_err_en_US,
                       Parser<DataStmtValue>{})) /
        "/"))

// R839 data-stmt-object -> variable | data-implied-do
TYPE_PARSER(construct<DataStmtObject>(indirect(variable)) ||
    construct<DataStmtObject>(dataImpliedDo))

// R840 data-implied-do ->
//        ( data-i-do-object-list , [integer-type-spec ::] data-i-do-variable
//        = scalar-int-constant-expr , scalar-int-constant-expr
//        [, scalar-int-constant-expr] )
// R842 data-i-do-variable -> do-variable
TYPE_PARSER(parenthesized(construct<DataImpliedDo>(
    nonemptyList(Parser<DataIDoObject>{} / lookAhead(","_tok)) / ",",
    maybe(integerTypeSpec / "::"), loopBounds(scalarIntConstantExpr))))

// R841 data-i-do-object ->
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L890 EN**: Comment explains nearby logic, intent, or metadata: `R838 data-stmt-set -> data-stmt-object-list / data-stmt-value-list`.
  **L890 CN**: 注释说明附近代码的逻辑、意图或元数据：`R838 data-stmt-set -> data-stmt-object-list / data-stmt-value-list`。
- **L891 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L891 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L892 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L892 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected DATA statement objects"_err_en_US, Parser<DataStmtObject>{}),`.
  **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected DATA statement objects"_err_en_US, Parser<DataStmtObject>{}),`。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `withMessage("expected DATA statement value list"_err_en_US,`.
  **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`withMessage("expected DATA statement value list"_err_en_US,`。
- **L895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"/"_tok >> nonemptyList("expected DATA statement values"_err_en_US,`.
  **L895 CN**: 继续一个多行参数列表、初始化器或聚合项：`"/"_tok >> nonemptyList("expected DATA statement values"_err_en_US,`。
- **L896 EN**: Continues the surrounding expression or declaration: `Parser<DataStmtValue>{})) /`.
  **L896 CN**: 继续构造周围的表达式或声明：`Parser<DataStmtValue>{})) /`。
- **L897 EN**: Continues the surrounding expression or declaration: `"/"))`.
  **L897 CN**: 继续构造周围的表达式或声明：`"/"))`。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L899 EN**: Comment explains nearby logic, intent, or metadata: `R839 data-stmt-object -> variable | data-implied-do`.
  **L899 CN**: 注释说明附近代码的逻辑、意图或元数据：`R839 data-stmt-object -> variable | data-implied-do`。
- **L900 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L900 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L901 EN**: Continues logic associated with callable symbol `construct<DataStmtObject>`.
  **L901 CN**: 继续与可调用符号 `construct<DataStmtObject>` 相关的逻辑。
- **L902 EN**: Blank line separating nearby declarations or logic blocks.
  **L902 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L903 EN**: Comment explains nearby logic, intent, or metadata: `R840 data-implied-do ->`.
  **L903 CN**: 注释说明附近代码的逻辑、意图或元数据：`R840 data-implied-do ->`。
- **L904 EN**: Comment explains nearby logic, intent, or metadata: `( data-i-do-object-list , [integer-type-spec ::] data-i-do-variable`.
  **L904 CN**: 注释说明附近代码的逻辑、意图或元数据：`( data-i-do-object-list , [integer-type-spec ::] data-i-do-variable`。
- **L905 EN**: Comment explains nearby logic, intent, or metadata: `= scalar-int-constant-expr , scalar-int-constant-expr`.
  **L905 CN**: 注释说明附近代码的逻辑、意图或元数据：`= scalar-int-constant-expr , scalar-int-constant-expr`。
- **L906 EN**: Comment explains nearby logic, intent, or metadata: `[, scalar-int-constant-expr] )`.
  **L906 CN**: 注释说明附近代码的逻辑、意图或元数据：`[, scalar-int-constant-expr] )`。
- **L907 EN**: Comment explains nearby logic, intent, or metadata: `R842 data-i-do-variable -> do-variable`.
  **L907 CN**: 注释说明附近代码的逻辑、意图或元数据：`R842 data-i-do-variable -> do-variable`。
- **L908 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L908 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList(Parser<DataIDoObject>{} / lookAhead(","_tok)) / ",",`.
  **L909 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList(Parser<DataIDoObject>{} / lookAhead(","_tok)) / ",",`。
- **L910 EN**: Continues logic associated with callable symbol `maybe`.
  **L910 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L911 EN**: Blank line separating nearby declarations or logic blocks.
  **L911 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L912 EN**: Comment explains nearby logic, intent, or metadata: `R841 data-i-do-object ->`.
  **L912 CN**: 注释说明附近代码的逻辑、意图或元数据：`R841 data-i-do-object ->`。

### Lines 913-936

````cpp
//        array-element | scalar-structure-component | data-implied-do
TYPE_PARSER(construct<DataIDoObject>(scalar(indirect(designator))) ||
    construct<DataIDoObject>(indirect(dataImpliedDo)))

// R843 data-stmt-value -> [data-stmt-repeat *] data-stmt-constant
TYPE_PARSER(construct<DataStmtValue>(
    maybe(Parser<DataStmtRepeat>{} / "*"), Parser<DataStmtConstant>{}))

// R847 constant-subobject -> designator
// R846 int-constant-subobject -> constant-subobject
constexpr auto constantSubobject{constant(indirect(designator))};

// R844 data-stmt-repeat -> scalar-int-constant | scalar-int-constant-subobject
// R607 int-constant -> constant
// Factored into: constant -> literal-constant -> int-literal-constant
// The named-constant alternative of constant is subsumed by constant-subobject
TYPE_PARSER(construct<DataStmtRepeat>(intLiteralConstant) ||
    construct<DataStmtRepeat>(scalar(integer(constantSubobject))))

// R845 data-stmt-constant ->
//        scalar-constant | scalar-constant-subobject |
//        signed-int-literal-constant | signed-real-literal-constant |
//        null-init | initial-data-target |
//        constant-structure-constructor
````
- **L913 EN**: Comment explains nearby logic, intent, or metadata: `array-element | scalar-structure-component | data-implied-do`.
  **L913 CN**: 注释说明附近代码的逻辑、意图或元数据：`array-element | scalar-structure-component | data-implied-do`。
- **L914 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L914 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L915 EN**: Continues logic associated with callable symbol `construct<DataIDoObject>`.
  **L915 CN**: 继续与可调用符号 `construct<DataIDoObject>` 相关的逻辑。
- **L916 EN**: Blank line separating nearby declarations or logic blocks.
  **L916 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L917 EN**: Comment explains nearby logic, intent, or metadata: `R843 data-stmt-value -> [data-stmt-repeat *] data-stmt-constant`.
  **L917 CN**: 注释说明附近代码的逻辑、意图或元数据：`R843 data-stmt-value -> [data-stmt-repeat *] data-stmt-constant`。
- **L918 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L918 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L919 EN**: Continues logic associated with callable symbol `maybe`.
  **L919 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L921 EN**: Comment explains nearby logic, intent, or metadata: `R847 constant-subobject -> designator`.
  **L921 CN**: 注释说明附近代码的逻辑、意图或元数据：`R847 constant-subobject -> designator`。
- **L922 EN**: Comment explains nearby logic, intent, or metadata: `R846 int-constant-subobject -> constant-subobject`.
  **L922 CN**: 注释说明附近代码的逻辑、意图或元数据：`R846 int-constant-subobject -> constant-subobject`。
- **L923 EN**: Executes a call or declaration centered on `constantSubobject{constant`.
  **L923 CN**: 执行以 `constantSubobject{constant` 为核心的调用或声明。
- **L924 EN**: Blank line separating nearby declarations or logic blocks.
  **L924 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L925 EN**: Comment explains nearby logic, intent, or metadata: `R844 data-stmt-repeat -> scalar-int-constant | scalar-int-constant-subobject`.
  **L925 CN**: 注释说明附近代码的逻辑、意图或元数据：`R844 data-stmt-repeat -> scalar-int-constant | scalar-int-constant-subobject`。
- **L926 EN**: Comment explains nearby logic, intent, or metadata: `R607 int-constant -> constant`.
  **L926 CN**: 注释说明附近代码的逻辑、意图或元数据：`R607 int-constant -> constant`。
- **L927 EN**: Comment explains nearby logic, intent, or metadata: `Factored into: constant -> literal-constant -> int-literal-constant`.
  **L927 CN**: 注释说明附近代码的逻辑、意图或元数据：`Factored into: constant -> literal-constant -> int-literal-constant`。
- **L928 EN**: Comment explains nearby logic, intent, or metadata: `The named-constant alternative of constant is subsumed by constant-subobject`.
  **L928 CN**: 注释说明附近代码的逻辑、意图或元数据：`The named-constant alternative of constant is subsumed by constant-subobject`。
- **L929 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L929 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L930 EN**: Continues logic associated with callable symbol `construct<DataStmtRepeat>`.
  **L930 CN**: 继续与可调用符号 `construct<DataStmtRepeat>` 相关的逻辑。
- **L931 EN**: Blank line separating nearby declarations or logic blocks.
  **L931 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L932 EN**: Comment explains nearby logic, intent, or metadata: `R845 data-stmt-constant ->`.
  **L932 CN**: 注释说明附近代码的逻辑、意图或元数据：`R845 data-stmt-constant ->`。
- **L933 EN**: Comment explains nearby logic, intent, or metadata: `scalar-constant | scalar-constant-subobject |`.
  **L933 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar-constant | scalar-constant-subobject |`。
- **L934 EN**: Comment explains nearby logic, intent, or metadata: `signed-int-literal-constant | signed-real-literal-constant |`.
  **L934 CN**: 注释说明附近代码的逻辑、意图或元数据：`signed-int-literal-constant | signed-real-literal-constant |`。
- **L935 EN**: Comment explains nearby logic, intent, or metadata: `null-init | initial-data-target |`.
  **L935 CN**: 注释说明附近代码的逻辑、意图或元数据：`null-init | initial-data-target |`。
- **L936 EN**: Comment explains nearby logic, intent, or metadata: `constant-structure-constructor`.
  **L936 CN**: 注释说明附近代码的逻辑、意图或元数据：`constant-structure-constructor`。

### Lines 937-960

````cpp
// N.B. scalar-constant and scalar-constant-subobject are ambiguous with
// initial-data-target; null-init and structure-constructor are ambiguous
// in the absence of parameters and components; structure-constructor with
// components can be ambiguous with a scalar-constant-subobject.
// So we parse literal constants, designator, null-init, and
// structure-constructor, so that semantics can figure things out later
// with the symbol table.  A literal constant substring must be attempted
// first to avoid a partial match with a literal constant.
TYPE_PARSER(sourced(first(
    construct<DataStmtConstant>(indirect(charLiteralConstantSubstring)),
    construct<DataStmtConstant>(literalConstant),
    construct<DataStmtConstant>(signedRealLiteralConstant),
    construct<DataStmtConstant>(signedIntLiteralConstant),
    extension<LanguageFeature::SignedComplexLiteral>(
        "nonstandard usage: signed COMPLEX literal"_port_en_US,
        construct<DataStmtConstant>(Parser<SignedComplexLiteralConstant>{})),
    construct<DataStmtConstant>(nullInit),
    construct<DataStmtConstant>(indirect(designator) / !"("_tok),
    construct<DataStmtConstant>(Parser<StructureConstructor>{}))))

// R848 dimension-stmt ->
//        DIMENSION [::] array-name ( array-spec )
//        [, array-name ( array-spec )]...
TYPE_CONTEXT_PARSER("DIMENSION statement"_en_US,
````
- **L937 EN**: Comment explains nearby logic, intent, or metadata: `N.B. scalar-constant and scalar-constant-subobject are ambiguous with`.
  **L937 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. scalar-constant and scalar-constant-subobject are ambiguous with`。
- **L938 EN**: Comment explains nearby logic, intent, or metadata: `initial-data-target; null-init and structure-constructor are ambiguous`.
  **L938 CN**: 注释说明附近代码的逻辑、意图或元数据：`initial-data-target; null-init and structure-constructor are ambiguous`。
- **L939 EN**: Comment explains nearby logic, intent, or metadata: `in the absence of parameters and components; structure-constructor with`.
  **L939 CN**: 注释说明附近代码的逻辑、意图或元数据：`in the absence of parameters and components; structure-constructor with`。
- **L940 EN**: Comment explains nearby logic, intent, or metadata: `components can be ambiguous with a scalar-constant-subobject.`.
  **L940 CN**: 注释说明附近代码的逻辑、意图或元数据：`components can be ambiguous with a scalar-constant-subobject.`。
- **L941 EN**: Comment explains nearby logic, intent, or metadata: `So we parse literal constants, designator, null-init, and`.
  **L941 CN**: 注释说明附近代码的逻辑、意图或元数据：`So we parse literal constants, designator, null-init, and`。
- **L942 EN**: Comment explains nearby logic, intent, or metadata: `structure-constructor, so that semantics can figure things out later`.
  **L942 CN**: 注释说明附近代码的逻辑、意图或元数据：`structure-constructor, so that semantics can figure things out later`。
- **L943 EN**: Comment explains nearby logic, intent, or metadata: `with the symbol table.  A literal constant substring must be attempted`.
  **L943 CN**: 注释说明附近代码的逻辑、意图或元数据：`with the symbol table.  A literal constant substring must be attempted`。
- **L944 EN**: Comment explains nearby logic, intent, or metadata: `first to avoid a partial match with a literal constant.`.
  **L944 CN**: 注释说明附近代码的逻辑、意图或元数据：`first to avoid a partial match with a literal constant.`。
- **L945 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L945 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DataStmtConstant>(indirect(charLiteralConstantSubstring)),`.
  **L946 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DataStmtConstant>(indirect(charLiteralConstantSubstring)),`。
- **L947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DataStmtConstant>(literalConstant),`.
  **L947 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DataStmtConstant>(literalConstant),`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DataStmtConstant>(signedRealLiteralConstant),`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DataStmtConstant>(signedRealLiteralConstant),`。
- **L949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DataStmtConstant>(signedIntLiteralConstant),`.
  **L949 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DataStmtConstant>(signedIntLiteralConstant),`。
- **L950 EN**: Continues logic associated with callable symbol `SignedComplexLiteral>`.
  **L950 CN**: 继续与可调用符号 `SignedComplexLiteral>` 相关的逻辑。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: signed COMPLEX literal"_port_en_US,`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: signed COMPLEX literal"_port_en_US,`。
- **L952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DataStmtConstant>(Parser<SignedComplexLiteralConstant>{})),`.
  **L952 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DataStmtConstant>(Parser<SignedComplexLiteralConstant>{})),`。
- **L953 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DataStmtConstant>(nullInit),`.
  **L953 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DataStmtConstant>(nullInit),`。
- **L954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DataStmtConstant>(indirect(designator) / !"("_tok),`.
  **L954 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DataStmtConstant>(indirect(designator) / !"("_tok),`。
- **L955 EN**: Continues logic associated with callable symbol `construct<DataStmtConstant>`.
  **L955 CN**: 继续与可调用符号 `construct<DataStmtConstant>` 相关的逻辑。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L957 EN**: Comment explains nearby logic, intent, or metadata: `R848 dimension-stmt ->`.
  **L957 CN**: 注释说明附近代码的逻辑、意图或元数据：`R848 dimension-stmt ->`。
- **L958 EN**: Comment explains nearby logic, intent, or metadata: `DIMENSION [::] array-name ( array-spec )`.
  **L958 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIMENSION [::] array-name ( array-spec )`。
- **L959 EN**: Comment explains nearby logic, intent, or metadata: `[, array-name ( array-spec )]...`.
  **L959 CN**: 注释说明附近代码的逻辑、意图或元数据：`[, array-name ( array-spec )]...`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("DIMENSION statement"_en_US,`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("DIMENSION statement"_en_US,`。

### Lines 961-984

````cpp
    construct<DimensionStmt>("DIMENSION" >> maybe("::"_tok) >>
        nonemptyList("expected array specifications"_err_en_US,
            construct<DimensionStmt::Declaration>(name, arraySpec))))

// R849 intent-stmt -> INTENT ( intent-spec ) [::] dummy-arg-name-list
TYPE_CONTEXT_PARSER("INTENT statement"_en_US,
    construct<IntentStmt>(
        "INTENT" >> parenthesized(intentSpec) / maybe("::"_tok), listOfNames))

// R850 optional-stmt -> OPTIONAL [::] dummy-arg-name-list
TYPE_PARSER(
    construct<OptionalStmt>("OPTIONAL" >> maybe("::"_tok) >> listOfNames))

// R851 parameter-stmt -> PARAMETER ( named-constant-def-list )
// Legacy extension: omitted parentheses, no implicit typing from names
TYPE_CONTEXT_PARSER("PARAMETER statement"_en_US,
    construct<ParameterStmt>(
        "PARAMETER" >> parenthesized(nonemptyList(Parser<NamedConstantDef>{}))))
TYPE_CONTEXT_PARSER("old style PARAMETER statement"_en_US,
    extension<LanguageFeature::OldStyleParameter>(
        "nonstandard usage: PARAMETER without parentheses"_port_en_US,
        construct<OldParameterStmt>(
            "PARAMETER" >> nonemptyList(Parser<NamedConstantDef>{}))))

````
- **L961 EN**: Continues logic associated with callable symbol `construct<DimensionStmt>`.
  **L961 CN**: 继续与可调用符号 `construct<DimensionStmt>` 相关的逻辑。
- **L962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected array specifications"_err_en_US,`.
  **L962 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected array specifications"_err_en_US,`。
- **L963 EN**: Continues logic associated with callable symbol `Declaration>`.
  **L963 CN**: 继续与可调用符号 `Declaration>` 相关的逻辑。
- **L964 EN**: Blank line separating nearby declarations or logic blocks.
  **L964 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L965 EN**: Comment explains nearby logic, intent, or metadata: `R849 intent-stmt -> INTENT ( intent-spec ) [::] dummy-arg-name-list`.
  **L965 CN**: 注释说明附近代码的逻辑、意图或元数据：`R849 intent-stmt -> INTENT ( intent-spec ) [::] dummy-arg-name-list`。
- **L966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("INTENT statement"_en_US,`.
  **L966 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("INTENT statement"_en_US,`。
- **L967 EN**: Continues logic associated with callable symbol `construct<IntentStmt>`.
  **L967 CN**: 继续与可调用符号 `construct<IntentStmt>` 相关的逻辑。
- **L968 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L968 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L969 EN**: Blank line separating nearby declarations or logic blocks.
  **L969 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L970 EN**: Comment explains nearby logic, intent, or metadata: `R850 optional-stmt -> OPTIONAL [::] dummy-arg-name-list`.
  **L970 CN**: 注释说明附近代码的逻辑、意图或元数据：`R850 optional-stmt -> OPTIONAL [::] dummy-arg-name-list`。
- **L971 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L971 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L972 EN**: Continues logic associated with callable symbol `construct<OptionalStmt>`.
  **L972 CN**: 继续与可调用符号 `construct<OptionalStmt>` 相关的逻辑。
- **L973 EN**: Blank line separating nearby declarations or logic blocks.
  **L973 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L974 EN**: Comment explains nearby logic, intent, or metadata: `R851 parameter-stmt -> PARAMETER ( named-constant-def-list )`.
  **L974 CN**: 注释说明附近代码的逻辑、意图或元数据：`R851 parameter-stmt -> PARAMETER ( named-constant-def-list )`。
- **L975 EN**: Comment explains nearby logic, intent, or metadata: `Legacy extension: omitted parentheses, no implicit typing from names`.
  **L975 CN**: 注释说明附近代码的逻辑、意图或元数据：`Legacy extension: omitted parentheses, no implicit typing from names`。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("PARAMETER statement"_en_US,`.
  **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("PARAMETER statement"_en_US,`。
- **L977 EN**: Continues logic associated with callable symbol `construct<ParameterStmt>`.
  **L977 CN**: 继续与可调用符号 `construct<ParameterStmt>` 相关的逻辑。
- **L978 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L978 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("old style PARAMETER statement"_en_US,`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("old style PARAMETER statement"_en_US,`。
- **L980 EN**: Continues logic associated with callable symbol `OldStyleParameter>`.
  **L980 CN**: 继续与可调用符号 `OldStyleParameter>` 相关的逻辑。
- **L981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: PARAMETER without parentheses"_port_en_US,`.
  **L981 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: PARAMETER without parentheses"_port_en_US,`。
- **L982 EN**: Continues logic associated with callable symbol `construct<OldParameterStmt>`.
  **L982 CN**: 继续与可调用符号 `construct<OldParameterStmt>` 相关的逻辑。
- **L983 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L983 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L984 EN**: Blank line separating nearby declarations or logic blocks.
  **L984 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 985-1008

````cpp
// R852 named-constant-def -> named-constant = constant-expr
TYPE_PARSER(construct<NamedConstantDef>(namedConstant, "=" >> constantExpr))

// R853 pointer-stmt -> POINTER [::] pointer-decl-list
TYPE_PARSER(construct<PointerStmt>("POINTER" >> maybe("::"_tok) >>
    nonemptyList(
        "expected pointer declarations"_err_en_US, Parser<PointerDecl>{})))

// R854 pointer-decl ->
//        object-name [( deferred-shape-spec-list )] | proc-entity-name
TYPE_PARSER(
    construct<PointerDecl>(name, maybe(parenthesized(deferredShapeSpecList))))

// R855 protected-stmt -> PROTECTED [::] entity-name-list
TYPE_PARSER(
    construct<ProtectedStmt>("PROTECTED" >> maybe("::"_tok) >> listOfNames))

// R856 save-stmt -> SAVE [[::] saved-entity-list]
TYPE_PARSER(construct<SaveStmt>(
    "SAVE" >> defaulted(maybe("::"_tok) >>
                  nonemptyList("expected SAVE entities"_err_en_US,
                      Parser<SavedEntity>{}))))

// R857 saved-entity -> object-name | proc-pointer-name | / common-block-name /
````
- **L985 EN**: Comment explains nearby logic, intent, or metadata: `R852 named-constant-def -> named-constant = constant-expr`.
  **L985 CN**: 注释说明附近代码的逻辑、意图或元数据：`R852 named-constant-def -> named-constant = constant-expr`。
- **L986 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L986 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L987 EN**: Blank line separating nearby declarations or logic blocks.
  **L987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L988 EN**: Comment explains nearby logic, intent, or metadata: `R853 pointer-stmt -> POINTER [::] pointer-decl-list`.
  **L988 CN**: 注释说明附近代码的逻辑、意图或元数据：`R853 pointer-stmt -> POINTER [::] pointer-decl-list`。
- **L989 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L989 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L990 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L990 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L991 EN**: Continues the surrounding expression or declaration: `"expected pointer declarations"_err_en_US, Parser<PointerDecl>{})))`.
  **L991 CN**: 继续构造周围的表达式或声明：`"expected pointer declarations"_err_en_US, Parser<PointerDecl>{})))`。
- **L992 EN**: Blank line separating nearby declarations or logic blocks.
  **L992 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L993 EN**: Comment explains nearby logic, intent, or metadata: `R854 pointer-decl ->`.
  **L993 CN**: 注释说明附近代码的逻辑、意图或元数据：`R854 pointer-decl ->`。
- **L994 EN**: Comment explains nearby logic, intent, or metadata: `object-name [( deferred-shape-spec-list )] | proc-entity-name`.
  **L994 CN**: 注释说明附近代码的逻辑、意图或元数据：`object-name [( deferred-shape-spec-list )] | proc-entity-name`。
- **L995 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L995 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L996 EN**: Continues logic associated with callable symbol `construct<PointerDecl>`.
  **L996 CN**: 继续与可调用符号 `construct<PointerDecl>` 相关的逻辑。
- **L997 EN**: Blank line separating nearby declarations or logic blocks.
  **L997 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L998 EN**: Comment explains nearby logic, intent, or metadata: `R855 protected-stmt -> PROTECTED [::] entity-name-list`.
  **L998 CN**: 注释说明附近代码的逻辑、意图或元数据：`R855 protected-stmt -> PROTECTED [::] entity-name-list`。
- **L999 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L999 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1000 EN**: Continues logic associated with callable symbol `construct<ProtectedStmt>`.
  **L1000 CN**: 继续与可调用符号 `construct<ProtectedStmt>` 相关的逻辑。
- **L1001 EN**: Blank line separating nearby declarations or logic blocks.
  **L1001 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1002 EN**: Comment explains nearby logic, intent, or metadata: `R856 save-stmt -> SAVE [[::] saved-entity-list]`.
  **L1002 CN**: 注释说明附近代码的逻辑、意图或元数据：`R856 save-stmt -> SAVE [[::] saved-entity-list]`。
- **L1003 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1003 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1004 EN**: Continues logic associated with callable symbol `defaulted`.
  **L1004 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L1005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected SAVE entities"_err_en_US,`.
  **L1005 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected SAVE entities"_err_en_US,`。
- **L1006 EN**: Continues the surrounding expression or declaration: `Parser<SavedEntity>{}))))`.
  **L1006 CN**: 继续构造周围的表达式或声明：`Parser<SavedEntity>{}))))`。
- **L1007 EN**: Blank line separating nearby declarations or logic blocks.
  **L1007 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1008 EN**: Comment explains nearby logic, intent, or metadata: `R857 saved-entity -> object-name | proc-pointer-name | / common-block-name`.
  **L1008 CN**: 注释说明附近代码的逻辑、意图或元数据：`R857 saved-entity -> object-name | proc-pointer-name | / common-block-name`。

### Lines 1009-1032

````cpp
// R858 proc-pointer-name -> name
TYPE_PARSER(construct<SavedEntity>(pure(SavedEntity::Kind::Entity), name) ||
    construct<SavedEntity>("/" >> pure(SavedEntity::Kind::Common), name / "/"))

// R859 target-stmt -> TARGET [::] target-decl-list
TYPE_PARSER(construct<TargetStmt>("TARGET" >> maybe("::"_tok) >>
    nonemptyList("expected objects"_err_en_US, Parser<ObjectDecl>{})))

// R861 value-stmt -> VALUE [::] dummy-arg-name-list
TYPE_PARSER(construct<ValueStmt>("VALUE" >> maybe("::"_tok) >> listOfNames))

// R862 volatile-stmt -> VOLATILE [::] object-name-list
TYPE_PARSER(construct<VolatileStmt>("VOLATILE" >> maybe("::"_tok) >>
    nonemptyList("expected object names"_err_en_US, objectName)))

// R866 implicit-name-spec -> EXTERNAL | TYPE
constexpr auto implicitNameSpec{
    "EXTERNAL" >> pure(ImplicitStmt::ImplicitNoneNameSpec::External) ||
    "TYPE" >> pure(ImplicitStmt::ImplicitNoneNameSpec::Type)};

// R863 implicit-stmt ->
//        IMPLICIT implicit-spec-list |
//        IMPLICIT NONE [( [implicit-name-spec-list] )]
TYPE_CONTEXT_PARSER("IMPLICIT statement"_en_US,
````
- **L1009 EN**: Comment explains nearby logic, intent, or metadata: `R858 proc-pointer-name -> name`.
  **L1009 CN**: 注释说明附近代码的逻辑、意图或元数据：`R858 proc-pointer-name -> name`。
- **L1010 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1010 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1011 EN**: Continues logic associated with callable symbol `construct<SavedEntity>`.
  **L1011 CN**: 继续与可调用符号 `construct<SavedEntity>` 相关的逻辑。
- **L1012 EN**: Blank line separating nearby declarations or logic blocks.
  **L1012 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1013 EN**: Comment explains nearby logic, intent, or metadata: `R859 target-stmt -> TARGET [::] target-decl-list`.
  **L1013 CN**: 注释说明附近代码的逻辑、意图或元数据：`R859 target-stmt -> TARGET [::] target-decl-list`。
- **L1014 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1014 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1015 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L1015 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L1016 EN**: Blank line separating nearby declarations or logic blocks.
  **L1016 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1017 EN**: Comment explains nearby logic, intent, or metadata: `R861 value-stmt -> VALUE [::] dummy-arg-name-list`.
  **L1017 CN**: 注释说明附近代码的逻辑、意图或元数据：`R861 value-stmt -> VALUE [::] dummy-arg-name-list`。
- **L1018 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1018 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1019 EN**: Blank line separating nearby declarations or logic blocks.
  **L1019 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1020 EN**: Comment explains nearby logic, intent, or metadata: `R862 volatile-stmt -> VOLATILE [::] object-name-list`.
  **L1020 CN**: 注释说明附近代码的逻辑、意图或元数据：`R862 volatile-stmt -> VOLATILE [::] object-name-list`。
- **L1021 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1021 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1022 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L1022 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L1023 EN**: Blank line separating nearby declarations or logic blocks.
  **L1023 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1024 EN**: Comment explains nearby logic, intent, or metadata: `R866 implicit-name-spec -> EXTERNAL | TYPE`.
  **L1024 CN**: 注释说明附近代码的逻辑、意图或元数据：`R866 implicit-name-spec -> EXTERNAL | TYPE`。
- **L1025 EN**: Continues the surrounding expression or declaration: `constexpr auto implicitNameSpec{`.
  **L1025 CN**: 继续构造周围的表达式或声明：`constexpr auto implicitNameSpec{`。
- **L1026 EN**: Continues logic associated with callable symbol `pure`.
  **L1026 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L1027 EN**: Executes a call or declaration centered on `pure`.
  **L1027 CN**: 执行以 `pure` 为核心的调用或声明。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1029 EN**: Comment explains nearby logic, intent, or metadata: `R863 implicit-stmt ->`.
  **L1029 CN**: 注释说明附近代码的逻辑、意图或元数据：`R863 implicit-stmt ->`。
- **L1030 EN**: Comment explains nearby logic, intent, or metadata: `IMPLICIT implicit-spec-list |`.
  **L1030 CN**: 注释说明附近代码的逻辑、意图或元数据：`IMPLICIT implicit-spec-list |`。
- **L1031 EN**: Comment explains nearby logic, intent, or metadata: `IMPLICIT NONE [( [implicit-name-spec-list] )]`.
  **L1031 CN**: 注释说明附近代码的逻辑、意图或元数据：`IMPLICIT NONE [( [implicit-name-spec-list] )]`。
- **L1032 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("IMPLICIT statement"_en_US,`.
  **L1032 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("IMPLICIT statement"_en_US,`。

### Lines 1033-1056

````cpp
    construct<ImplicitStmt>(
        "IMPLICIT" >> nonemptyList("expected IMPLICIT specifications"_err_en_US,
                          Parser<ImplicitSpec>{})) ||
        construct<ImplicitStmt>("IMPLICIT NONE"_sptok >>
            defaulted(parenthesized(optionalList(implicitNameSpec)))))

// R864 implicit-spec -> declaration-type-spec ( letter-spec-list )
// The variant form of declarationTypeSpec is meant to avoid misrecognition
// of a letter-spec as a simple parenthesized expression for kind or character
// length, e.g., PARAMETER(I=5,N=1); IMPLICIT REAL(I-N)(O-Z) vs.
// IMPLICIT REAL(I-N).  The variant form needs to attempt to reparse only
// types with optional parenthesized kind/length expressions, so derived
// type specs, DOUBLE PRECISION, and DOUBLE COMPLEX need not be considered.
constexpr auto noKindSelector{construct<std::optional<KindSelector>>()};
constexpr auto implicitSpecDeclarationTypeSpecRetry{
    construct<DeclarationTypeSpec>(first(
        construct<IntrinsicTypeSpec>(
            construct<IntegerTypeSpec>("INTEGER" >> noKindSelector)),
        construct<IntrinsicTypeSpec>(
            construct<IntrinsicTypeSpec::Real>("REAL" >> noKindSelector)),
        construct<IntrinsicTypeSpec>(
            construct<IntrinsicTypeSpec::Complex>("COMPLEX" >> noKindSelector)),
        construct<IntrinsicTypeSpec>(construct<IntrinsicTypeSpec::Character>(
            "CHARACTER" >> construct<std::optional<CharSelector>>())),
````
- **L1033 EN**: Continues logic associated with callable symbol `construct<ImplicitStmt>`.
  **L1033 CN**: 继续与可调用符号 `construct<ImplicitStmt>` 相关的逻辑。
- **L1034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"IMPLICIT" >> nonemptyList("expected IMPLICIT specifications"_err_en_US,`.
  **L1034 CN**: 继续一个多行参数列表、初始化器或聚合项：`"IMPLICIT" >> nonemptyList("expected IMPLICIT specifications"_err_en_US,`。
- **L1035 EN**: Continues the surrounding expression or declaration: `Parser<ImplicitSpec>{})) ||`.
  **L1035 CN**: 继续构造周围的表达式或声明：`Parser<ImplicitSpec>{})) ||`。
- **L1036 EN**: Continues logic associated with callable symbol `construct<ImplicitStmt>`.
  **L1036 CN**: 继续与可调用符号 `construct<ImplicitStmt>` 相关的逻辑。
- **L1037 EN**: Continues logic associated with callable symbol `defaulted`.
  **L1037 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1039 EN**: Comment explains nearby logic, intent, or metadata: `R864 implicit-spec -> declaration-type-spec ( letter-spec-list )`.
  **L1039 CN**: 注释说明附近代码的逻辑、意图或元数据：`R864 implicit-spec -> declaration-type-spec ( letter-spec-list )`。
- **L1040 EN**: Comment explains nearby logic, intent, or metadata: `The variant form of declarationTypeSpec is meant to avoid misrecognition`.
  **L1040 CN**: 注释说明附近代码的逻辑、意图或元数据：`The variant form of declarationTypeSpec is meant to avoid misrecognition`。
- **L1041 EN**: Comment explains nearby logic, intent, or metadata: `of a letter-spec as a simple parenthesized expression for kind or character`.
  **L1041 CN**: 注释说明附近代码的逻辑、意图或元数据：`of a letter-spec as a simple parenthesized expression for kind or character`。
- **L1042 EN**: Comment explains nearby logic, intent, or metadata: `length, e.g., PARAMETER(I=5,N=1); IMPLICIT REAL(I-N)(O-Z) vs.`.
  **L1042 CN**: 注释说明附近代码的逻辑、意图或元数据：`length, e.g., PARAMETER(I=5,N=1); IMPLICIT REAL(I-N)(O-Z) vs.`。
- **L1043 EN**: Comment explains nearby logic, intent, or metadata: `IMPLICIT REAL(I-N).  The variant form needs to attempt to reparse only`.
  **L1043 CN**: 注释说明附近代码的逻辑、意图或元数据：`IMPLICIT REAL(I-N).  The variant form needs to attempt to reparse only`。
- **L1044 EN**: Comment explains nearby logic, intent, or metadata: `types with optional parenthesized kind/length expressions, so derived`.
  **L1044 CN**: 注释说明附近代码的逻辑、意图或元数据：`types with optional parenthesized kind/length expressions, so derived`。
- **L1045 EN**: Comment explains nearby logic, intent, or metadata: `type specs, DOUBLE PRECISION, and DOUBLE COMPLEX need not be considered.`.
  **L1045 CN**: 注释说明附近代码的逻辑、意图或元数据：`type specs, DOUBLE PRECISION, and DOUBLE COMPLEX need not be considered.`。
- **L1046 EN**: Executes a call or declaration centered on `noKindSelector{construct<std::optional<KindSelector>>`.
  **L1046 CN**: 执行以 `noKindSelector{construct<std::optional<KindSelector>>` 为核心的调用或声明。
- **L1047 EN**: Continues the surrounding expression or declaration: `constexpr auto implicitSpecDeclarationTypeSpecRetry{`.
  **L1047 CN**: 继续构造周围的表达式或声明：`constexpr auto implicitSpecDeclarationTypeSpecRetry{`。
- **L1048 EN**: Continues logic associated with callable symbol `construct<DeclarationTypeSpec>`.
  **L1048 CN**: 继续与可调用符号 `construct<DeclarationTypeSpec>` 相关的逻辑。
- **L1049 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L1049 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L1050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<IntegerTypeSpec>("INTEGER" >> noKindSelector)),`.
  **L1050 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<IntegerTypeSpec>("INTEGER" >> noKindSelector)),`。
- **L1051 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L1051 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L1052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<IntrinsicTypeSpec::Real>("REAL" >> noKindSelector)),`.
  **L1052 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<IntrinsicTypeSpec::Real>("REAL" >> noKindSelector)),`。
- **L1053 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L1053 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L1054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<IntrinsicTypeSpec::Complex>("COMPLEX" >> noKindSelector)),`.
  **L1054 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<IntrinsicTypeSpec::Complex>("COMPLEX" >> noKindSelector)),`。
- **L1055 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L1055 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L1056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CHARACTER" >> construct<std::optional<CharSelector>>())),`.
  **L1056 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CHARACTER" >> construct<std::optional<CharSelector>>())),`。

### Lines 1057-1080

````cpp
        construct<IntrinsicTypeSpec>(
            construct<IntrinsicTypeSpec::Logical>("LOGICAL" >> noKindSelector)),
        construct<IntrinsicTypeSpec>(
            construct<UnsignedTypeSpec>("UNSIGNED" >> noKindSelector))))};

TYPE_PARSER(construct<ImplicitSpec>(declarationTypeSpec,
                parenthesized(nonemptyList(Parser<LetterSpec>{}))) ||
    construct<ImplicitSpec>(implicitSpecDeclarationTypeSpecRetry,
        parenthesized(nonemptyList(Parser<LetterSpec>{}))))

// R865 letter-spec -> letter [- letter]
TYPE_PARSER(space >> (construct<LetterSpec>(letter, maybe("-" >> letter)) ||
                         construct<LetterSpec>(otherIdChar,
                             construct<std::optional<const char *>>())))

// R867 import-stmt ->
//        IMPORT [[::] import-name-list] |
//        IMPORT , ONLY : import-name-list | IMPORT , NONE | IMPORT , ALL
TYPE_CONTEXT_PARSER("IMPORT statement"_en_US,
    construct<ImportStmt>(
        "IMPORT , ONLY :" >> pure(common::ImportKind::Only), listOfNames) ||
        construct<ImportStmt>(
            "IMPORT , NONE" >> pure(common::ImportKind::None)) ||
        construct<ImportStmt>(
````
- **L1057 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L1057 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L1058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<IntrinsicTypeSpec::Logical>("LOGICAL" >> noKindSelector)),`.
  **L1058 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<IntrinsicTypeSpec::Logical>("LOGICAL" >> noKindSelector)),`。
- **L1059 EN**: Continues logic associated with callable symbol `construct<IntrinsicTypeSpec>`.
  **L1059 CN**: 继续与可调用符号 `construct<IntrinsicTypeSpec>` 相关的逻辑。
- **L1060 EN**: Executes a call or declaration centered on `construct<UnsignedTypeSpec>`.
  **L1060 CN**: 执行以 `construct<UnsignedTypeSpec>` 为核心的调用或声明。
- **L1061 EN**: Blank line separating nearby declarations or logic blocks.
  **L1061 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<ImplicitSpec>(declarationTypeSpec,`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<ImplicitSpec>(declarationTypeSpec,`。
- **L1063 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L1063 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L1064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ImplicitSpec>(implicitSpecDeclarationTypeSpecRetry,`.
  **L1064 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ImplicitSpec>(implicitSpecDeclarationTypeSpecRetry,`。
- **L1065 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L1065 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L1066 EN**: Blank line separating nearby declarations or logic blocks.
  **L1066 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1067 EN**: Comment explains nearby logic, intent, or metadata: `R865 letter-spec -> letter [- letter]`.
  **L1067 CN**: 注释说明附近代码的逻辑、意图或元数据：`R865 letter-spec -> letter [- letter]`。
- **L1068 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1068 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<LetterSpec>(otherIdChar,`.
  **L1069 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<LetterSpec>(otherIdChar,`。
- **L1070 EN**: Continues the surrounding expression or declaration: `construct<std::optional<const char *>>())))`.
  **L1070 CN**: 继续构造周围的表达式或声明：`construct<std::optional<const char *>>())))`。
- **L1071 EN**: Blank line separating nearby declarations or logic blocks.
  **L1071 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1072 EN**: Comment explains nearby logic, intent, or metadata: `R867 import-stmt ->`.
  **L1072 CN**: 注释说明附近代码的逻辑、意图或元数据：`R867 import-stmt ->`。
- **L1073 EN**: Comment explains nearby logic, intent, or metadata: `IMPORT [[::] import-name-list] |`.
  **L1073 CN**: 注释说明附近代码的逻辑、意图或元数据：`IMPORT [[::] import-name-list] |`。
- **L1074 EN**: Comment explains nearby logic, intent, or metadata: `IMPORT , ONLY : import-name-list | IMPORT , NONE | IMPORT , ALL`.
  **L1074 CN**: 注释说明附近代码的逻辑、意图或元数据：`IMPORT , ONLY : import-name-list | IMPORT , NONE | IMPORT , ALL`。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("IMPORT statement"_en_US,`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("IMPORT statement"_en_US,`。
- **L1076 EN**: Continues logic associated with callable symbol `construct<ImportStmt>`.
  **L1076 CN**: 继续与可调用符号 `construct<ImportStmt>` 相关的逻辑。
- **L1077 EN**: Continues logic associated with callable symbol `pure`.
  **L1077 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L1078 EN**: Continues logic associated with callable symbol `construct<ImportStmt>`.
  **L1078 CN**: 继续与可调用符号 `construct<ImportStmt>` 相关的逻辑。
- **L1079 EN**: Continues logic associated with callable symbol `pure`.
  **L1079 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L1080 EN**: Continues logic associated with callable symbol `construct<ImportStmt>`.
  **L1080 CN**: 继续与可调用符号 `construct<ImportStmt>` 相关的逻辑。

### Lines 1081-1104

````cpp
            "IMPORT , ALL" >> pure(common::ImportKind::All)) ||
        construct<ImportStmt>(
            "IMPORT" >> maybe("::"_tok) >> optionalList(name)))

// R868 namelist-stmt ->
//        NAMELIST / namelist-group-name / namelist-group-object-list
//        [[,] / namelist-group-name / namelist-group-object-list]...
// R869 namelist-group-object -> variable-name
TYPE_PARSER(construct<NamelistStmt>("NAMELIST" >>
    nonemptySeparated(
        construct<NamelistStmt::Group>("/" >> name / "/", listOfNames),
        maybe(","_tok))))

// R870 equivalence-stmt -> EQUIVALENCE equivalence-set-list
// R871 equivalence-set -> ( equivalence-object , equivalence-object-list )
TYPE_PARSER(construct<EquivalenceStmt>("EQUIVALENCE" >>
    nonemptyList(
        parenthesized(nonemptyList("expected EQUIVALENCE objects"_err_en_US,
            Parser<EquivalenceObject>{})))))

// R872 equivalence-object -> variable-name | array-element | substring
TYPE_PARSER(construct<EquivalenceObject>(indirect(designator)))

// R873 common-stmt ->
````
- **L1081 EN**: Continues logic associated with callable symbol `pure`.
  **L1081 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L1082 EN**: Continues logic associated with callable symbol `construct<ImportStmt>`.
  **L1082 CN**: 继续与可调用符号 `construct<ImportStmt>` 相关的逻辑。
- **L1083 EN**: Continues logic associated with callable symbol `maybe`.
  **L1083 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1085 EN**: Comment explains nearby logic, intent, or metadata: `R868 namelist-stmt ->`.
  **L1085 CN**: 注释说明附近代码的逻辑、意图或元数据：`R868 namelist-stmt ->`。
- **L1086 EN**: Comment explains nearby logic, intent, or metadata: `NAMELIST / namelist-group-name / namelist-group-object-list`.
  **L1086 CN**: 注释说明附近代码的逻辑、意图或元数据：`NAMELIST / namelist-group-name / namelist-group-object-list`。
- **L1087 EN**: Comment explains nearby logic, intent, or metadata: `[[,] / namelist-group-name / namelist-group-object-list]...`.
  **L1087 CN**: 注释说明附近代码的逻辑、意图或元数据：`[[,] / namelist-group-name / namelist-group-object-list]...`。
- **L1088 EN**: Comment explains nearby logic, intent, or metadata: `R869 namelist-group-object -> variable-name`.
  **L1088 CN**: 注释说明附近代码的逻辑、意图或元数据：`R869 namelist-group-object -> variable-name`。
- **L1089 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1089 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1090 EN**: Continues logic associated with callable symbol `nonemptySeparated`.
  **L1090 CN**: 继续与可调用符号 `nonemptySeparated` 相关的逻辑。
- **L1091 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<NamelistStmt::Group>("/" >> name / "/", listOfNames),`.
  **L1091 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<NamelistStmt::Group>("/" >> name / "/", listOfNames),`。
- **L1092 EN**: Continues logic associated with callable symbol `maybe`.
  **L1092 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1093 EN**: Blank line separating nearby declarations or logic blocks.
  **L1093 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1094 EN**: Comment explains nearby logic, intent, or metadata: `R870 equivalence-stmt -> EQUIVALENCE equivalence-set-list`.
  **L1094 CN**: 注释说明附近代码的逻辑、意图或元数据：`R870 equivalence-stmt -> EQUIVALENCE equivalence-set-list`。
- **L1095 EN**: Comment explains nearby logic, intent, or metadata: `R871 equivalence-set -> ( equivalence-object , equivalence-object-list )`.
  **L1095 CN**: 注释说明附近代码的逻辑、意图或元数据：`R871 equivalence-set -> ( equivalence-object , equivalence-object-list )`。
- **L1096 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1096 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1097 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L1097 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L1098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parenthesized(nonemptyList("expected EQUIVALENCE objects"_err_en_US,`.
  **L1098 CN**: 继续一个多行参数列表、初始化器或聚合项：`parenthesized(nonemptyList("expected EQUIVALENCE objects"_err_en_US,`。
- **L1099 EN**: Continues the surrounding expression or declaration: `Parser<EquivalenceObject>{})))))`.
  **L1099 CN**: 继续构造周围的表达式或声明：`Parser<EquivalenceObject>{})))))`。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1101 EN**: Comment explains nearby logic, intent, or metadata: `R872 equivalence-object -> variable-name | array-element | substring`.
  **L1101 CN**: 注释说明附近代码的逻辑、意图或元数据：`R872 equivalence-object -> variable-name | array-element | substring`。
- **L1102 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1102 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1103 EN**: Blank line separating nearby declarations or logic blocks.
  **L1103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1104 EN**: Comment explains nearby logic, intent, or metadata: `R873 common-stmt ->`.
  **L1104 CN**: 注释说明附近代码的逻辑、意图或元数据：`R873 common-stmt ->`。

### Lines 1105-1128

````cpp
//        COMMON [/ [common-block-name] /] common-block-object-list
//        [[,] / [common-block-name] / common-block-object-list]...
TYPE_PARSER(sourced(
    construct<CommonStmt>("COMMON" >> defaulted("/" >> maybe(name) / "/"),
        nonemptyList("expected COMMON block objects"_err_en_US,
            Parser<CommonBlockObject>{}),
        many(maybe(","_tok) >>
            construct<CommonStmt::Block>("/" >> maybe(name) / "/",
                nonemptyList("expected COMMON block objects"_err_en_US,
                    Parser<CommonBlockObject>{}))))))

// R874 common-block-object -> variable-name [( array-spec )]
TYPE_PARSER(construct<CommonBlockObject>(name, maybe(arraySpec)))

// R901 designator -> object-name | array-element | array-section |
//                    coindexed-named-object | complex-part-designator |
//                    structure-component | substring
// The Standard's productions for designator and its alternatives are
// ambiguous without recourse to a symbol table.  Many of the alternatives
// for designator (viz., array-element, coindexed-named-object,
// and structure-component) are all syntactically just data-ref.
// What designator boils down to is this:
//  It starts with either a name or a character literal.
//  If it starts with a character literal, it must be a substring.
````
- **L1105 EN**: Comment explains nearby logic, intent, or metadata: `COMMON [/ [common-block-name] /] common-block-object-list`.
  **L1105 CN**: 注释说明附近代码的逻辑、意图或元数据：`COMMON [/ [common-block-name] /] common-block-object-list`。
- **L1106 EN**: Comment explains nearby logic, intent, or metadata: `[[,] / [common-block-name] / common-block-object-list]...`.
  **L1106 CN**: 注释说明附近代码的逻辑、意图或元数据：`[[,] / [common-block-name] / common-block-object-list]...`。
- **L1107 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1107 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<CommonStmt>("COMMON" >> defaulted("/" >> maybe(name) / "/"),`.
  **L1108 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<CommonStmt>("COMMON" >> defaulted("/" >> maybe(name) / "/"),`。
- **L1109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected COMMON block objects"_err_en_US,`.
  **L1109 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected COMMON block objects"_err_en_US,`。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Parser<CommonBlockObject>{}),`.
  **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`Parser<CommonBlockObject>{}),`。
- **L1111 EN**: Continues logic associated with callable symbol `many`.
  **L1111 CN**: 继续与可调用符号 `many` 相关的逻辑。
- **L1112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<CommonStmt::Block>("/" >> maybe(name) / "/",`.
  **L1112 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<CommonStmt::Block>("/" >> maybe(name) / "/",`。
- **L1113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList("expected COMMON block objects"_err_en_US,`.
  **L1113 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList("expected COMMON block objects"_err_en_US,`。
- **L1114 EN**: Continues the surrounding expression or declaration: `Parser<CommonBlockObject>{}))))))`.
  **L1114 CN**: 继续构造周围的表达式或声明：`Parser<CommonBlockObject>{}))))))`。
- **L1115 EN**: Blank line separating nearby declarations or logic blocks.
  **L1115 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1116 EN**: Comment explains nearby logic, intent, or metadata: `R874 common-block-object -> variable-name [( array-spec )]`.
  **L1116 CN**: 注释说明附近代码的逻辑、意图或元数据：`R874 common-block-object -> variable-name [( array-spec )]`。
- **L1117 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1117 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1118 EN**: Blank line separating nearby declarations or logic blocks.
  **L1118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1119 EN**: Comment explains nearby logic, intent, or metadata: `R901 designator -> object-name | array-element | array-section |`.
  **L1119 CN**: 注释说明附近代码的逻辑、意图或元数据：`R901 designator -> object-name | array-element | array-section |`。
- **L1120 EN**: Comment explains nearby logic, intent, or metadata: `coindexed-named-object | complex-part-designator |`.
  **L1120 CN**: 注释说明附近代码的逻辑、意图或元数据：`coindexed-named-object | complex-part-designator |`。
- **L1121 EN**: Comment explains nearby logic, intent, or metadata: `structure-component | substring`.
  **L1121 CN**: 注释说明附近代码的逻辑、意图或元数据：`structure-component | substring`。
- **L1122 EN**: Comment explains nearby logic, intent, or metadata: `The Standard's productions for designator and its alternatives are`.
  **L1122 CN**: 注释说明附近代码的逻辑、意图或元数据：`The Standard's productions for designator and its alternatives are`。
- **L1123 EN**: Comment explains nearby logic, intent, or metadata: `ambiguous without recourse to a symbol table.  Many of the alternatives`.
  **L1123 CN**: 注释说明附近代码的逻辑、意图或元数据：`ambiguous without recourse to a symbol table.  Many of the alternatives`。
- **L1124 EN**: Comment explains nearby logic, intent, or metadata: `for designator (viz., array-element, coindexed-named-object,`.
  **L1124 CN**: 注释说明附近代码的逻辑、意图或元数据：`for designator (viz., array-element, coindexed-named-object,`。
- **L1125 EN**: Comment explains nearby logic, intent, or metadata: `and structure-component) are all syntactically just data-ref.`.
  **L1125 CN**: 注释说明附近代码的逻辑、意图或元数据：`and structure-component) are all syntactically just data-ref.`。
- **L1126 EN**: Comment explains nearby logic, intent, or metadata: `What designator boils down to is this:`.
  **L1126 CN**: 注释说明附近代码的逻辑、意图或元数据：`What designator boils down to is this:`。
- **L1127 EN**: Comment explains nearby logic, intent, or metadata: `It starts with either a name or a character literal.`.
  **L1127 CN**: 注释说明附近代码的逻辑、意图或元数据：`It starts with either a name or a character literal.`。
- **L1128 EN**: Comment explains nearby logic, intent, or metadata: `If it starts with a character literal, it must be a substring.`.
  **L1128 CN**: 注释说明附近代码的逻辑、意图或元数据：`If it starts with a character literal, it must be a substring.`。

### Lines 1129-1152

````cpp
//  If it starts with a name, it's a sequence of %-separated parts;
//  each part is a name, maybe a (section-subscript-list), and
//  maybe an [image-selector].
//  If it's a substring, it ends with (substring-range).
TYPE_CONTEXT_PARSER("designator"_en_US,
    sourced(construct<Designator>(substring) || construct<Designator>(dataRef)))

constexpr auto percentOrDot{"%"_tok ||
    // legacy VAX extension for RECORD field access
    extension<LanguageFeature::DECStructures>(
        "nonstandard usage: component access with '.' in place of '%'"_port_en_US,
        "."_tok / lookAhead(OldStructureComponentName{}))};

// R902 variable -> designator | function-reference
// This production appears to be left-recursive in the grammar via
//   function-reference ->  procedure-designator -> proc-component-ref ->
//     scalar-variable
// and would be so if we were to allow functions to be called via procedure
// pointer components within derived type results of other function references
// (a reasonable extension, esp. in the case of procedure pointer components
// that are NOPASS).  However, Fortran constrains the use of a variable in a
// proc-component-ref to be a data-ref without coindices (C1027).
// Some array element references will be misrecognized as function references.
constexpr auto noMoreAddressing{!"("_tok >> !"["_tok >> !percentOrDot};
````
- **L1129 EN**: Comment explains nearby logic, intent, or metadata: `If it starts with a name, it's a sequence of %-separated parts;`.
  **L1129 CN**: 注释说明附近代码的逻辑、意图或元数据：`If it starts with a name, it's a sequence of %-separated parts;`。
- **L1130 EN**: Comment explains nearby logic, intent, or metadata: `each part is a name, maybe a (section-subscript-list), and`.
  **L1130 CN**: 注释说明附近代码的逻辑、意图或元数据：`each part is a name, maybe a (section-subscript-list), and`。
- **L1131 EN**: Comment explains nearby logic, intent, or metadata: `maybe an [image-selector].`.
  **L1131 CN**: 注释说明附近代码的逻辑、意图或元数据：`maybe an [image-selector].`。
- **L1132 EN**: Comment explains nearby logic, intent, or metadata: `If it's a substring, it ends with (substring-range).`.
  **L1132 CN**: 注释说明附近代码的逻辑、意图或元数据：`If it's a substring, it ends with (substring-range).`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("designator"_en_US,`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("designator"_en_US,`。
- **L1134 EN**: Continues logic associated with callable symbol `sourced`.
  **L1134 CN**: 继续与可调用符号 `sourced` 相关的逻辑。
- **L1135 EN**: Blank line separating nearby declarations or logic blocks.
  **L1135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1136 EN**: Continues the surrounding expression or declaration: `constexpr auto percentOrDot{"%"_tok ||`.
  **L1136 CN**: 继续构造周围的表达式或声明：`constexpr auto percentOrDot{"%"_tok ||`。
- **L1137 EN**: Comment explains nearby logic, intent, or metadata: `legacy VAX extension for RECORD field access`.
  **L1137 CN**: 注释说明附近代码的逻辑、意图或元数据：`legacy VAX extension for RECORD field access`。
- **L1138 EN**: Continues logic associated with callable symbol `DECStructures>`.
  **L1138 CN**: 继续与可调用符号 `DECStructures>` 相关的逻辑。
- **L1139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: component access with '.' in place of '%'"_port_en_US,`.
  **L1139 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: component access with '.' in place of '%'"_port_en_US,`。
- **L1140 EN**: Executes a call or declaration centered on `lookAhead`.
  **L1140 CN**: 执行以 `lookAhead` 为核心的调用或声明。
- **L1141 EN**: Blank line separating nearby declarations or logic blocks.
  **L1141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1142 EN**: Comment explains nearby logic, intent, or metadata: `R902 variable -> designator | function-reference`.
  **L1142 CN**: 注释说明附近代码的逻辑、意图或元数据：`R902 variable -> designator | function-reference`。
- **L1143 EN**: Comment explains nearby logic, intent, or metadata: `This production appears to be left-recursive in the grammar via`.
  **L1143 CN**: 注释说明附近代码的逻辑、意图或元数据：`This production appears to be left-recursive in the grammar via`。
- **L1144 EN**: Comment explains nearby logic, intent, or metadata: `function-reference ->  procedure-designator -> proc-component-ref ->`.
  **L1144 CN**: 注释说明附近代码的逻辑、意图或元数据：`function-reference ->  procedure-designator -> proc-component-ref ->`。
- **L1145 EN**: Comment explains nearby logic, intent, or metadata: `scalar-variable`.
  **L1145 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar-variable`。
- **L1146 EN**: Comment explains nearby logic, intent, or metadata: `and would be so if we were to allow functions to be called via procedure`.
  **L1146 CN**: 注释说明附近代码的逻辑、意图或元数据：`and would be so if we were to allow functions to be called via procedure`。
- **L1147 EN**: Comment explains nearby logic, intent, or metadata: `pointer components within derived type results of other function references`.
  **L1147 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointer components within derived type results of other function references`。
- **L1148 EN**: Comment explains nearby logic, intent, or metadata: `(a reasonable extension, esp. in the case of procedure pointer components`.
  **L1148 CN**: 注释说明附近代码的逻辑、意图或元数据：`(a reasonable extension, esp. in the case of procedure pointer components`。
- **L1149 EN**: Comment explains nearby logic, intent, or metadata: `that are NOPASS).  However, Fortran constrains the use of a variable in a`.
  **L1149 CN**: 注释说明附近代码的逻辑、意图或元数据：`that are NOPASS).  However, Fortran constrains the use of a variable in a`。
- **L1150 EN**: Comment explains nearby logic, intent, or metadata: `proc-component-ref to be a data-ref without coindices (C1027).`.
  **L1150 CN**: 注释说明附近代码的逻辑、意图或元数据：`proc-component-ref to be a data-ref without coindices (C1027).`。
- **L1151 EN**: Comment explains nearby logic, intent, or metadata: `Some array element references will be misrecognized as function references.`.
  **L1151 CN**: 注释说明附近代码的逻辑、意图或元数据：`Some array element references will be misrecognized as function references.`。
- **L1152 EN**: Executes a call or declaration centered on `noMoreAddressing{!"`.
  **L1152 CN**: 执行以 `noMoreAddressing{!"` 为核心的调用或声明。

### Lines 1153-1176

````cpp
TYPE_CONTEXT_PARSER("variable"_en_US,
    construct<Variable>(indirect(functionReference / noMoreAddressing)) ||
        construct<Variable>(indirect(designator)))

// R908 substring -> parent-string ( substring-range )
// R909 parent-string ->
//        scalar-variable-name | array-element | coindexed-named-object |
//        scalar-structure-component | scalar-char-literal-constant |
//        scalar-named-constant
TYPE_PARSER(
    construct<Substring>(dataRef, parenthesized(Parser<SubstringRange>{})))

TYPE_PARSER(construct<CharLiteralConstantSubstring>(
    charLiteralConstant, parenthesized(Parser<SubstringRange>{})))

TYPE_PARSER(sourced(construct<SubstringInquiry>(Parser<Substring>{}) /
    ("%LEN"_tok || "%KIND"_tok)))

// R910 substring-range -> [scalar-int-expr] : [scalar-int-expr]
TYPE_PARSER(construct<SubstringRange>(
    maybe(scalarIntExpr), ":" >> maybe(scalarIntExpr)))

// R911 data-ref -> part-ref [% part-ref]...
// R914 coindexed-named-object -> data-ref
````
- **L1153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("variable"_en_US,`.
  **L1153 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("variable"_en_US,`。
- **L1154 EN**: Continues logic associated with callable symbol `construct<Variable>`.
  **L1154 CN**: 继续与可调用符号 `construct<Variable>` 相关的逻辑。
- **L1155 EN**: Continues logic associated with callable symbol `construct<Variable>`.
  **L1155 CN**: 继续与可调用符号 `construct<Variable>` 相关的逻辑。
- **L1156 EN**: Blank line separating nearby declarations or logic blocks.
  **L1156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1157 EN**: Comment explains nearby logic, intent, or metadata: `R908 substring -> parent-string ( substring-range )`.
  **L1157 CN**: 注释说明附近代码的逻辑、意图或元数据：`R908 substring -> parent-string ( substring-range )`。
- **L1158 EN**: Comment explains nearby logic, intent, or metadata: `R909 parent-string ->`.
  **L1158 CN**: 注释说明附近代码的逻辑、意图或元数据：`R909 parent-string ->`。
- **L1159 EN**: Comment explains nearby logic, intent, or metadata: `scalar-variable-name | array-element | coindexed-named-object |`.
  **L1159 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar-variable-name | array-element | coindexed-named-object |`。
- **L1160 EN**: Comment explains nearby logic, intent, or metadata: `scalar-structure-component | scalar-char-literal-constant |`.
  **L1160 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar-structure-component | scalar-char-literal-constant |`。
- **L1161 EN**: Comment explains nearby logic, intent, or metadata: `scalar-named-constant`.
  **L1161 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar-named-constant`。
- **L1162 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1162 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1163 EN**: Continues logic associated with callable symbol `construct<Substring>`.
  **L1163 CN**: 继续与可调用符号 `construct<Substring>` 相关的逻辑。
- **L1164 EN**: Blank line separating nearby declarations or logic blocks.
  **L1164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1165 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1165 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1166 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L1166 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L1167 EN**: Blank line separating nearby declarations or logic blocks.
  **L1167 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1168 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1168 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1169 EN**: Continues the surrounding expression or declaration: `("%LEN"_tok || "%KIND"_tok)))`.
  **L1169 CN**: 继续构造周围的表达式或声明：`("%LEN"_tok || "%KIND"_tok)))`。
- **L1170 EN**: Blank line separating nearby declarations or logic blocks.
  **L1170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1171 EN**: Comment explains nearby logic, intent, or metadata: `R910 substring-range -> [scalar-int-expr] : [scalar-int-expr]`.
  **L1171 CN**: 注释说明附近代码的逻辑、意图或元数据：`R910 substring-range -> [scalar-int-expr] : [scalar-int-expr]`。
- **L1172 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1172 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1173 EN**: Continues logic associated with callable symbol `maybe`.
  **L1173 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1174 EN**: Blank line separating nearby declarations or logic blocks.
  **L1174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1175 EN**: Comment explains nearby logic, intent, or metadata: `R911 data-ref -> part-ref [% part-ref]...`.
  **L1175 CN**: 注释说明附近代码的逻辑、意图或元数据：`R911 data-ref -> part-ref [% part-ref]...`。
- **L1176 EN**: Comment explains nearby logic, intent, or metadata: `R914 coindexed-named-object -> data-ref`.
  **L1176 CN**: 注释说明附近代码的逻辑、意图或元数据：`R914 coindexed-named-object -> data-ref`。

### Lines 1177-1200

````cpp
// R917 array-element -> data-ref
TYPE_PARSER(
    construct<DataRef>(nonemptySeparated(Parser<PartRef>{}, percentOrDot)))

// R912 part-ref -> part-name [( section-subscript-list )] [image-selector]
TYPE_PARSER(construct<PartRef>(name,
    defaulted(
        parenthesized(nonemptyList(Parser<SectionSubscript>{})) / !"=>"_tok),
    maybe(Parser<ImageSelector>{})))

// R913 structure-component -> data-ref
// The final part-ref in the data-ref is not allowed to have subscripts.
TYPE_CONTEXT_PARSER("component"_en_US,
    construct<StructureComponent>(
        construct<DataRef>(some(Parser<PartRef>{} / percentOrDot)), name))

// R919 subscript -> scalar-int-expr
constexpr auto subscript{scalarIntExpr};

// R920 section-subscript -> subscript | subscript-triplet | vector-subscript
// R923 vector-subscript -> int-expr
// N.B. The distinction that needs to be made between "subscript" and
// "vector-subscript" is deferred to semantic analysis.
TYPE_PARSER(construct<SectionSubscript>(Parser<SubscriptTriplet>{}) ||
````
- **L1177 EN**: Comment explains nearby logic, intent, or metadata: `R917 array-element -> data-ref`.
  **L1177 CN**: 注释说明附近代码的逻辑、意图或元数据：`R917 array-element -> data-ref`。
- **L1178 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1178 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1179 EN**: Continues logic associated with callable symbol `construct<DataRef>`.
  **L1179 CN**: 继续与可调用符号 `construct<DataRef>` 相关的逻辑。
- **L1180 EN**: Blank line separating nearby declarations or logic blocks.
  **L1180 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1181 EN**: Comment explains nearby logic, intent, or metadata: `R912 part-ref -> part-name [( section-subscript-list )] [image-selector]`.
  **L1181 CN**: 注释说明附近代码的逻辑、意图或元数据：`R912 part-ref -> part-name [( section-subscript-list )] [image-selector]`。
- **L1182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<PartRef>(name,`.
  **L1182 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<PartRef>(name,`。
- **L1183 EN**: Continues logic associated with callable symbol `defaulted`.
  **L1183 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parenthesized(nonemptyList(Parser<SectionSubscript>{})) / !"=>"_tok),`.
  **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`parenthesized(nonemptyList(Parser<SectionSubscript>{})) / !"=>"_tok),`。
- **L1185 EN**: Continues logic associated with callable symbol `maybe`.
  **L1185 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1186 EN**: Blank line separating nearby declarations or logic blocks.
  **L1186 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1187 EN**: Comment explains nearby logic, intent, or metadata: `R913 structure-component -> data-ref`.
  **L1187 CN**: 注释说明附近代码的逻辑、意图或元数据：`R913 structure-component -> data-ref`。
- **L1188 EN**: Comment explains nearby logic, intent, or metadata: `The final part-ref in the data-ref is not allowed to have subscripts.`.
  **L1188 CN**: 注释说明附近代码的逻辑、意图或元数据：`The final part-ref in the data-ref is not allowed to have subscripts.`。
- **L1189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("component"_en_US,`.
  **L1189 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("component"_en_US,`。
- **L1190 EN**: Continues logic associated with callable symbol `construct<StructureComponent>`.
  **L1190 CN**: 继续与可调用符号 `construct<StructureComponent>` 相关的逻辑。
- **L1191 EN**: Continues logic associated with callable symbol `construct<DataRef>`.
  **L1191 CN**: 继续与可调用符号 `construct<DataRef>` 相关的逻辑。
- **L1192 EN**: Blank line separating nearby declarations or logic blocks.
  **L1192 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1193 EN**: Comment explains nearby logic, intent, or metadata: `R919 subscript -> scalar-int-expr`.
  **L1193 CN**: 注释说明附近代码的逻辑、意图或元数据：`R919 subscript -> scalar-int-expr`。
- **L1194 EN**: Executes a standalone statement or declaration: `constexpr auto subscript{scalarIntExpr};`.
  **L1194 CN**: 执行一条独立语句或声明：`constexpr auto subscript{scalarIntExpr};`。
- **L1195 EN**: Blank line separating nearby declarations or logic blocks.
  **L1195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1196 EN**: Comment explains nearby logic, intent, or metadata: `R920 section-subscript -> subscript | subscript-triplet | vector-subscript`.
  **L1196 CN**: 注释说明附近代码的逻辑、意图或元数据：`R920 section-subscript -> subscript | subscript-triplet | vector-subscript`。
- **L1197 EN**: Comment explains nearby logic, intent, or metadata: `R923 vector-subscript -> int-expr`.
  **L1197 CN**: 注释说明附近代码的逻辑、意图或元数据：`R923 vector-subscript -> int-expr`。
- **L1198 EN**: Comment explains nearby logic, intent, or metadata: `N.B. The distinction that needs to be made between "subscript" and`.
  **L1198 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. The distinction that needs to be made between "subscript" and`。
- **L1199 EN**: Comment explains nearby logic, intent, or metadata: `"vector-subscript" is deferred to semantic analysis.`.
  **L1199 CN**: 注释说明附近代码的逻辑、意图或元数据：`"vector-subscript" is deferred to semantic analysis.`。
- **L1200 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1200 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。

### Lines 1201-1224

````cpp
    construct<SectionSubscript>(intExpr))

// R921 subscript-triplet -> [subscript] : [subscript] [: stride]
TYPE_PARSER(construct<SubscriptTriplet>(
    maybe(subscript), ":" >> maybe(subscript), maybe(":" >> subscript)))

// R925 cosubscript -> scalar-int-expr
constexpr auto cosubscript{scalarIntExpr};

// R924 image-selector ->
//        lbracket cosubscript-list [, image-selector-spec-list] rbracket
TYPE_CONTEXT_PARSER("image selector"_en_US,
    construct<ImageSelector>(
        "[" >> nonemptyList(cosubscript / lookAhead(space / ",]"_ch)),
        defaulted("," >> nonemptyList(Parser<ImageSelectorSpec>{})) / "]"))

// R926 image-selector-spec ->
//        STAT = stat-variable | TEAM = team-value |
//        TEAM_NUMBER = scalar-int-expr |
//        NOTIFY = notify-variable
TYPE_PARSER(construct<ImageSelectorSpec>(construct<ImageSelectorSpec::Stat>(
                "STAT =" >> scalar(integer(indirect(variable))))) ||
    construct<ImageSelectorSpec>(construct<TeamValue>("TEAM =" >> teamValue)) ||
    construct<ImageSelectorSpec>(construct<ImageSelectorSpec::Team_Number>(
````
- **L1201 EN**: Continues logic associated with callable symbol `construct<SectionSubscript>`.
  **L1201 CN**: 继续与可调用符号 `construct<SectionSubscript>` 相关的逻辑。
- **L1202 EN**: Blank line separating nearby declarations or logic blocks.
  **L1202 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1203 EN**: Comment explains nearby logic, intent, or metadata: `R921 subscript-triplet -> [subscript] : [subscript] [: stride]`.
  **L1203 CN**: 注释说明附近代码的逻辑、意图或元数据：`R921 subscript-triplet -> [subscript] : [subscript] [: stride]`。
- **L1204 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1204 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1205 EN**: Continues logic associated with callable symbol `maybe`.
  **L1205 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1206 EN**: Blank line separating nearby declarations or logic blocks.
  **L1206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1207 EN**: Comment explains nearby logic, intent, or metadata: `R925 cosubscript -> scalar-int-expr`.
  **L1207 CN**: 注释说明附近代码的逻辑、意图或元数据：`R925 cosubscript -> scalar-int-expr`。
- **L1208 EN**: Executes a standalone statement or declaration: `constexpr auto cosubscript{scalarIntExpr};`.
  **L1208 CN**: 执行一条独立语句或声明：`constexpr auto cosubscript{scalarIntExpr};`。
- **L1209 EN**: Blank line separating nearby declarations or logic blocks.
  **L1209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1210 EN**: Comment explains nearby logic, intent, or metadata: `R924 image-selector ->`.
  **L1210 CN**: 注释说明附近代码的逻辑、意图或元数据：`R924 image-selector ->`。
- **L1211 EN**: Comment explains nearby logic, intent, or metadata: `lbracket cosubscript-list [, image-selector-spec-list] rbracket`.
  **L1211 CN**: 注释说明附近代码的逻辑、意图或元数据：`lbracket cosubscript-list [, image-selector-spec-list] rbracket`。
- **L1212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("image selector"_en_US,`.
  **L1212 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("image selector"_en_US,`。
- **L1213 EN**: Continues logic associated with callable symbol `construct<ImageSelector>`.
  **L1213 CN**: 继续与可调用符号 `construct<ImageSelector>` 相关的逻辑。
- **L1214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"[" >> nonemptyList(cosubscript / lookAhead(space / ",]"_ch)),`.
  **L1214 CN**: 继续一个多行参数列表、初始化器或聚合项：`"[" >> nonemptyList(cosubscript / lookAhead(space / ",]"_ch)),`。
- **L1215 EN**: Continues logic associated with callable symbol `defaulted`.
  **L1215 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L1216 EN**: Blank line separating nearby declarations or logic blocks.
  **L1216 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1217 EN**: Comment explains nearby logic, intent, or metadata: `R926 image-selector-spec ->`.
  **L1217 CN**: 注释说明附近代码的逻辑、意图或元数据：`R926 image-selector-spec ->`。
- **L1218 EN**: Comment explains nearby logic, intent, or metadata: `STAT = stat-variable | TEAM = team-value |`.
  **L1218 CN**: 注释说明附近代码的逻辑、意图或元数据：`STAT = stat-variable | TEAM = team-value |`。
- **L1219 EN**: Comment explains nearby logic, intent, or metadata: `TEAM_NUMBER = scalar-int-expr |`.
  **L1219 CN**: 注释说明附近代码的逻辑、意图或元数据：`TEAM_NUMBER = scalar-int-expr |`。
- **L1220 EN**: Comment explains nearby logic, intent, or metadata: `NOTIFY = notify-variable`.
  **L1220 CN**: 注释说明附近代码的逻辑、意图或元数据：`NOTIFY = notify-variable`。
- **L1221 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1221 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1222 EN**: Continues logic associated with callable symbol `scalar`.
  **L1222 CN**: 继续与可调用符号 `scalar` 相关的逻辑。
- **L1223 EN**: Continues logic associated with callable symbol `construct<ImageSelectorSpec>`.
  **L1223 CN**: 继续与可调用符号 `construct<ImageSelectorSpec>` 相关的逻辑。
- **L1224 EN**: Continues logic associated with callable symbol `construct<ImageSelectorSpec>`.
  **L1224 CN**: 继续与可调用符号 `construct<ImageSelectorSpec>` 相关的逻辑。

### Lines 1225-1248

````cpp
        "TEAM_NUMBER =" >> scalarIntExpr)) ||
    construct<ImageSelectorSpec>(construct<ImageSelectorSpec::Notify>(
        "NOTIFY =" >> scalar(indirect(variable)))))

// R927 allocate-stmt ->
//        ALLOCATE ( [type-spec ::] allocation-list [, alloc-opt-list] )
TYPE_CONTEXT_PARSER("ALLOCATE statement"_en_US,
    construct<AllocateStmt>("ALLOCATE (" >> maybe(typeSpec / "::"),
        nonemptyList(Parser<Allocation>{}),
        defaulted("," >> nonemptyList(Parser<AllocOpt>{})) / ")"))

// R928 alloc-opt ->
//        ERRMSG = errmsg-variable | MOLD = source-expr |
//        SOURCE = source-expr | STAT = stat-variable |
// (CUDA) STREAM = scalar-int-expr
//        PINNED = scalar-logical-variable
// R931 source-expr -> expr
TYPE_PARSER(construct<AllocOpt>(
                construct<AllocOpt::Mold>("MOLD =" >> indirect(expr))) ||
    construct<AllocOpt>(
        construct<AllocOpt::Source>("SOURCE =" >> indirect(expr))) ||
    construct<AllocOpt>(statOrErrmsg) ||
    extension<LanguageFeature::CUDA>(
        construct<AllocOpt>(construct<AllocOpt::Stream>(
````
- **L1225 EN**: Continues the surrounding expression or declaration: `"TEAM_NUMBER =" >> scalarIntExpr)) ||`.
  **L1225 CN**: 继续构造周围的表达式或声明：`"TEAM_NUMBER =" >> scalarIntExpr)) ||`。
- **L1226 EN**: Continues logic associated with callable symbol `construct<ImageSelectorSpec>`.
  **L1226 CN**: 继续与可调用符号 `construct<ImageSelectorSpec>` 相关的逻辑。
- **L1227 EN**: Continues logic associated with callable symbol `scalar`.
  **L1227 CN**: 继续与可调用符号 `scalar` 相关的逻辑。
- **L1228 EN**: Blank line separating nearby declarations or logic blocks.
  **L1228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1229 EN**: Comment explains nearby logic, intent, or metadata: `R927 allocate-stmt ->`.
  **L1229 CN**: 注释说明附近代码的逻辑、意图或元数据：`R927 allocate-stmt ->`。
- **L1230 EN**: Comment explains nearby logic, intent, or metadata: `ALLOCATE ( [type-spec ::] allocation-list [, alloc-opt-list] )`.
  **L1230 CN**: 注释说明附近代码的逻辑、意图或元数据：`ALLOCATE ( [type-spec ::] allocation-list [, alloc-opt-list] )`。
- **L1231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("ALLOCATE statement"_en_US,`.
  **L1231 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("ALLOCATE statement"_en_US,`。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<AllocateStmt>("ALLOCATE (" >> maybe(typeSpec / "::"),`.
  **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<AllocateStmt>("ALLOCATE (" >> maybe(typeSpec / "::"),`。
- **L1233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nonemptyList(Parser<Allocation>{}),`.
  **L1233 CN**: 继续一个多行参数列表、初始化器或聚合项：`nonemptyList(Parser<Allocation>{}),`。
- **L1234 EN**: Continues logic associated with callable symbol `defaulted`.
  **L1234 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L1235 EN**: Blank line separating nearby declarations or logic blocks.
  **L1235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1236 EN**: Comment explains nearby logic, intent, or metadata: `R928 alloc-opt ->`.
  **L1236 CN**: 注释说明附近代码的逻辑、意图或元数据：`R928 alloc-opt ->`。
- **L1237 EN**: Comment explains nearby logic, intent, or metadata: `ERRMSG = errmsg-variable | MOLD = source-expr |`.
  **L1237 CN**: 注释说明附近代码的逻辑、意图或元数据：`ERRMSG = errmsg-variable | MOLD = source-expr |`。
- **L1238 EN**: Comment explains nearby logic, intent, or metadata: `SOURCE = source-expr | STAT = stat-variable |`.
  **L1238 CN**: 注释说明附近代码的逻辑、意图或元数据：`SOURCE = source-expr | STAT = stat-variable |`。
- **L1239 EN**: Comment explains nearby logic, intent, or metadata: `(CUDA) STREAM = scalar-int-expr`.
  **L1239 CN**: 注释说明附近代码的逻辑、意图或元数据：`(CUDA) STREAM = scalar-int-expr`。
- **L1240 EN**: Comment explains nearby logic, intent, or metadata: `PINNED = scalar-logical-variable`.
  **L1240 CN**: 注释说明附近代码的逻辑、意图或元数据：`PINNED = scalar-logical-variable`。
- **L1241 EN**: Comment explains nearby logic, intent, or metadata: `R931 source-expr -> expr`.
  **L1241 CN**: 注释说明附近代码的逻辑、意图或元数据：`R931 source-expr -> expr`。
- **L1242 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1242 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1243 EN**: Continues logic associated with callable symbol `Mold>`.
  **L1243 CN**: 继续与可调用符号 `Mold>` 相关的逻辑。
- **L1244 EN**: Continues logic associated with callable symbol `construct<AllocOpt>`.
  **L1244 CN**: 继续与可调用符号 `construct<AllocOpt>` 相关的逻辑。
- **L1245 EN**: Continues logic associated with callable symbol `Source>`.
  **L1245 CN**: 继续与可调用符号 `Source>` 相关的逻辑。
- **L1246 EN**: Continues logic associated with callable symbol `construct<AllocOpt>`.
  **L1246 CN**: 继续与可调用符号 `construct<AllocOpt>` 相关的逻辑。
- **L1247 EN**: Continues logic associated with callable symbol `CUDA>`.
  **L1247 CN**: 继续与可调用符号 `CUDA>` 相关的逻辑。
- **L1248 EN**: Continues logic associated with callable symbol `construct<AllocOpt>`.
  **L1248 CN**: 继续与可调用符号 `construct<AllocOpt>` 相关的逻辑。

### Lines 1249-1272

````cpp
            "STREAM =" >> indirect(scalarIntExpr))) ||
        construct<AllocOpt>(construct<AllocOpt::Pinned>(
            "PINNED =" >> indirect(scalarLogicalVariable)))))

// R929 stat-variable -> scalar-int-variable
TYPE_PARSER(construct<StatVariable>(scalar(integer(variable))))

// R932 allocation ->
//        allocate-object [( allocate-shape-spec-list )]
//        [lbracket allocate-coarray-spec rbracket]
TYPE_PARSER(construct<Allocation>(Parser<AllocateObject>{},
    defaulted(parenthesized(nonemptyList(Parser<AllocateShapeSpec>{}))),
    maybe(bracketed(Parser<AllocateCoarraySpec>{}))))

// R933 allocate-object -> variable-name | structure-component
TYPE_PARSER(construct<AllocateObject>(structureComponent) ||
    construct<AllocateObject>(name / !"="_tok))

// R934 allocate-shape-spec -> [lower-bound-expr :] upper-bound-expr
// R938 allocate-coshape-spec -> [lower-bound-expr :] upper-bound-expr
TYPE_PARSER(construct<AllocateShapeSpec>(maybe(boundExpr / ":"), boundExpr))

// R937 allocate-coarray-spec ->
//      [allocate-coshape-spec-list ,] [lower-bound-expr :] *
````
- **L1249 EN**: Continues logic associated with callable symbol `indirect`.
  **L1249 CN**: 继续与可调用符号 `indirect` 相关的逻辑。
- **L1250 EN**: Continues logic associated with callable symbol `construct<AllocOpt>`.
  **L1250 CN**: 继续与可调用符号 `construct<AllocOpt>` 相关的逻辑。
- **L1251 EN**: Continues logic associated with callable symbol `indirect`.
  **L1251 CN**: 继续与可调用符号 `indirect` 相关的逻辑。
- **L1252 EN**: Blank line separating nearby declarations or logic blocks.
  **L1252 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1253 EN**: Comment explains nearby logic, intent, or metadata: `R929 stat-variable -> scalar-int-variable`.
  **L1253 CN**: 注释说明附近代码的逻辑、意图或元数据：`R929 stat-variable -> scalar-int-variable`。
- **L1254 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1254 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1255 EN**: Blank line separating nearby declarations or logic blocks.
  **L1255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1256 EN**: Comment explains nearby logic, intent, or metadata: `R932 allocation ->`.
  **L1256 CN**: 注释说明附近代码的逻辑、意图或元数据：`R932 allocation ->`。
- **L1257 EN**: Comment explains nearby logic, intent, or metadata: `allocate-object [( allocate-shape-spec-list )]`.
  **L1257 CN**: 注释说明附近代码的逻辑、意图或元数据：`allocate-object [( allocate-shape-spec-list )]`。
- **L1258 EN**: Comment explains nearby logic, intent, or metadata: `[lbracket allocate-coarray-spec rbracket]`.
  **L1258 CN**: 注释说明附近代码的逻辑、意图或元数据：`[lbracket allocate-coarray-spec rbracket]`。
- **L1259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<Allocation>(Parser<AllocateObject>{},`.
  **L1259 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<Allocation>(Parser<AllocateObject>{},`。
- **L1260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defaulted(parenthesized(nonemptyList(Parser<AllocateShapeSpec>{}))),`.
  **L1260 CN**: 继续一个多行参数列表、初始化器或聚合项：`defaulted(parenthesized(nonemptyList(Parser<AllocateShapeSpec>{}))),`。
- **L1261 EN**: Continues logic associated with callable symbol `maybe`.
  **L1261 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1262 EN**: Blank line separating nearby declarations or logic blocks.
  **L1262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1263 EN**: Comment explains nearby logic, intent, or metadata: `R933 allocate-object -> variable-name | structure-component`.
  **L1263 CN**: 注释说明附近代码的逻辑、意图或元数据：`R933 allocate-object -> variable-name | structure-component`。
- **L1264 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1264 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1265 EN**: Continues logic associated with callable symbol `construct<AllocateObject>`.
  **L1265 CN**: 继续与可调用符号 `construct<AllocateObject>` 相关的逻辑。
- **L1266 EN**: Blank line separating nearby declarations or logic blocks.
  **L1266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1267 EN**: Comment explains nearby logic, intent, or metadata: `R934 allocate-shape-spec -> [lower-bound-expr :] upper-bound-expr`.
  **L1267 CN**: 注释说明附近代码的逻辑、意图或元数据：`R934 allocate-shape-spec -> [lower-bound-expr :] upper-bound-expr`。
- **L1268 EN**: Comment explains nearby logic, intent, or metadata: `R938 allocate-coshape-spec -> [lower-bound-expr :] upper-bound-expr`.
  **L1268 CN**: 注释说明附近代码的逻辑、意图或元数据：`R938 allocate-coshape-spec -> [lower-bound-expr :] upper-bound-expr`。
- **L1269 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1269 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1270 EN**: Blank line separating nearby declarations or logic blocks.
  **L1270 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1271 EN**: Comment explains nearby logic, intent, or metadata: `R937 allocate-coarray-spec ->`.
  **L1271 CN**: 注释说明附近代码的逻辑、意图或元数据：`R937 allocate-coarray-spec ->`。
- **L1272 EN**: Comment explains nearby logic, intent, or metadata: `[allocate-coshape-spec-list ,] [lower-bound-expr :]`.
  **L1272 CN**: 注释说明附近代码的逻辑、意图或元数据：`[allocate-coshape-spec-list ,] [lower-bound-expr :]`。

### Lines 1273-1296

````cpp
TYPE_PARSER(construct<AllocateCoarraySpec>(
    defaulted(nonemptyList(Parser<AllocateShapeSpec>{}) / ","),
    maybe(boundExpr / ":") / "*"))

// R939 nullify-stmt -> NULLIFY ( pointer-object-list )
TYPE_CONTEXT_PARSER("NULLIFY statement"_en_US,
    "NULLIFY" >> parenthesized(construct<NullifyStmt>(
                     nonemptyList(Parser<PointerObject>{}))))

// R940 pointer-object ->
//        variable-name | structure-component | proc-pointer-name
TYPE_PARSER(construct<PointerObject>(structureComponent) ||
    construct<PointerObject>(name))

// R941 deallocate-stmt ->
//        DEALLOCATE ( allocate-object-list [, dealloc-opt-list] )
TYPE_CONTEXT_PARSER("DEALLOCATE statement"_en_US,
    construct<DeallocateStmt>(
        "DEALLOCATE (" >> nonemptyList(Parser<AllocateObject>{}),
        defaulted("," >> nonemptyList(statOrErrmsg)) / ")"))

// R942 dealloc-opt -> STAT = stat-variable | ERRMSG = errmsg-variable
// R1165 sync-stat -> STAT = stat-variable | ERRMSG = errmsg-variable
TYPE_PARSER(construct<StatOrErrmsg>("STAT =" >> statVariable) ||
````
- **L1273 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1273 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defaulted(nonemptyList(Parser<AllocateShapeSpec>{}) / ","),`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`defaulted(nonemptyList(Parser<AllocateShapeSpec>{}) / ","),`。
- **L1275 EN**: Continues logic associated with callable symbol `maybe`.
  **L1275 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1276 EN**: Blank line separating nearby declarations or logic blocks.
  **L1276 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1277 EN**: Comment explains nearby logic, intent, or metadata: `R939 nullify-stmt -> NULLIFY ( pointer-object-list )`.
  **L1277 CN**: 注释说明附近代码的逻辑、意图或元数据：`R939 nullify-stmt -> NULLIFY ( pointer-object-list )`。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("NULLIFY statement"_en_US,`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("NULLIFY statement"_en_US,`。
- **L1279 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L1279 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L1280 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L1280 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L1281 EN**: Blank line separating nearby declarations or logic blocks.
  **L1281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1282 EN**: Comment explains nearby logic, intent, or metadata: `R940 pointer-object ->`.
  **L1282 CN**: 注释说明附近代码的逻辑、意图或元数据：`R940 pointer-object ->`。
- **L1283 EN**: Comment explains nearby logic, intent, or metadata: `variable-name | structure-component | proc-pointer-name`.
  **L1283 CN**: 注释说明附近代码的逻辑、意图或元数据：`variable-name | structure-component | proc-pointer-name`。
- **L1284 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1284 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1285 EN**: Continues logic associated with callable symbol `construct<PointerObject>`.
  **L1285 CN**: 继续与可调用符号 `construct<PointerObject>` 相关的逻辑。
- **L1286 EN**: Blank line separating nearby declarations or logic blocks.
  **L1286 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1287 EN**: Comment explains nearby logic, intent, or metadata: `R941 deallocate-stmt ->`.
  **L1287 CN**: 注释说明附近代码的逻辑、意图或元数据：`R941 deallocate-stmt ->`。
- **L1288 EN**: Comment explains nearby logic, intent, or metadata: `DEALLOCATE ( allocate-object-list [, dealloc-opt-list] )`.
  **L1288 CN**: 注释说明附近代码的逻辑、意图或元数据：`DEALLOCATE ( allocate-object-list [, dealloc-opt-list] )`。
- **L1289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("DEALLOCATE statement"_en_US,`.
  **L1289 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("DEALLOCATE statement"_en_US,`。
- **L1290 EN**: Continues logic associated with callable symbol `construct<DeallocateStmt>`.
  **L1290 CN**: 继续与可调用符号 `construct<DeallocateStmt>` 相关的逻辑。
- **L1291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"DEALLOCATE (" >> nonemptyList(Parser<AllocateObject>{}),`.
  **L1291 CN**: 继续一个多行参数列表、初始化器或聚合项：`"DEALLOCATE (" >> nonemptyList(Parser<AllocateObject>{}),`。
- **L1292 EN**: Continues logic associated with callable symbol `defaulted`.
  **L1292 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L1293 EN**: Blank line separating nearby declarations or logic blocks.
  **L1293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1294 EN**: Comment explains nearby logic, intent, or metadata: `R942 dealloc-opt -> STAT = stat-variable | ERRMSG = errmsg-variable`.
  **L1294 CN**: 注释说明附近代码的逻辑、意图或元数据：`R942 dealloc-opt -> STAT = stat-variable | ERRMSG = errmsg-variable`。
- **L1295 EN**: Comment explains nearby logic, intent, or metadata: `R1165 sync-stat -> STAT = stat-variable | ERRMSG = errmsg-variable`.
  **L1295 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1165 sync-stat -> STAT = stat-variable | ERRMSG = errmsg-variable`。
- **L1296 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1296 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。

### Lines 1297-1320

````cpp
    construct<StatOrErrmsg>("ERRMSG =" >> msgVariable))

// Directives, extensions, and deprecated statements
// !DIR$ IGNORE_TKR [ [(tkrdmac...)] name ]...
// !DIR$ LOOP COUNT (n1[, n2]...)
// !DIR$ VECTOR VECTORLENGTH ({FIXED|SCALABLE|<num>|<num>,FIXED|<num>,SCALABLE})
// !DIR$ name[=value] [, name[=value]]...
// !DIR$ UNROLL [n]
// !DIR$ PREFETCH designator[, designator]...
// !DIR$ IVDEP
// !DIR$ <anything else>
constexpr auto ignore_tkr{
    "IGNORE_TKR" >> optionalList(construct<CompilerDirective::IgnoreTKR>(
                        maybe(parenthesized(many(letter))), name))};
constexpr auto loopCount{
    "LOOP COUNT" >> construct<CompilerDirective::LoopCount>(
                        parenthesized(nonemptyList(digitString64)))};
constexpr auto assumeAligned{"ASSUME_ALIGNED" >>
    optionalList(construct<CompilerDirective::AssumeAligned>(
        indirect(designator), ":"_tok >> digitString64))};
constexpr auto vectorAlways{
    "VECTOR ALWAYS" >> construct<CompilerDirective::VectorAlways>()};
constexpr auto vectorLengthKind{
    "FIXED" >> pure(CompilerDirective::VectorLength::Kind::Fixed) ||
````
- **L1297 EN**: Continues logic associated with callable symbol `construct<StatOrErrmsg>`.
  **L1297 CN**: 继续与可调用符号 `construct<StatOrErrmsg>` 相关的逻辑。
- **L1298 EN**: Blank line separating nearby declarations or logic blocks.
  **L1298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1299 EN**: Comment explains nearby logic, intent, or metadata: `Directives, extensions, and deprecated statements`.
  **L1299 CN**: 注释说明附近代码的逻辑、意图或元数据：`Directives, extensions, and deprecated statements`。
- **L1300 EN**: Comment explains nearby logic, intent, or metadata: `DIR$ IGNORE_TKR [ [(tkrdmac...)] name ]...`.
  **L1300 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIR$ IGNORE_TKR [ [(tkrdmac...)] name ]...`。
- **L1301 EN**: Comment explains nearby logic, intent, or metadata: `DIR$ LOOP COUNT (n1[, n2]...)`.
  **L1301 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIR$ LOOP COUNT (n1[, n2]...)`。
- **L1302 EN**: Comment explains nearby logic, intent, or metadata: `DIR$ VECTOR VECTORLENGTH ({FIXED|SCALABLE|<num>|<num>,FIXED|<num>,SCALABLE})`.
  **L1302 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIR$ VECTOR VECTORLENGTH ({FIXED|SCALABLE|<num>|<num>,FIXED|<num>,SCALABLE})`。
- **L1303 EN**: Comment explains nearby logic, intent, or metadata: `DIR$ name[=value] [, name[=value]]...`.
  **L1303 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIR$ name[=value] [, name[=value]]...`。
- **L1304 EN**: Comment explains nearby logic, intent, or metadata: `DIR$ UNROLL [n]`.
  **L1304 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIR$ UNROLL [n]`。
- **L1305 EN**: Comment explains nearby logic, intent, or metadata: `DIR$ PREFETCH designator[, designator]...`.
  **L1305 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIR$ PREFETCH designator[, designator]...`。
- **L1306 EN**: Comment explains nearby logic, intent, or metadata: `DIR$ IVDEP`.
  **L1306 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIR$ IVDEP`。
- **L1307 EN**: Comment explains nearby logic, intent, or metadata: `DIR$ <anything else>`.
  **L1307 CN**: 注释说明附近代码的逻辑、意图或元数据：`DIR$ <anything else>`。
- **L1308 EN**: Continues the surrounding expression or declaration: `constexpr auto ignore_tkr{`.
  **L1308 CN**: 继续构造周围的表达式或声明：`constexpr auto ignore_tkr{`。
- **L1309 EN**: Continues logic associated with callable symbol `optionalList`.
  **L1309 CN**: 继续与可调用符号 `optionalList` 相关的逻辑。
- **L1310 EN**: Executes a call or declaration centered on `maybe`.
  **L1310 CN**: 执行以 `maybe` 为核心的调用或声明。
- **L1311 EN**: Continues the surrounding expression or declaration: `constexpr auto loopCount{`.
  **L1311 CN**: 继续构造周围的表达式或声明：`constexpr auto loopCount{`。
- **L1312 EN**: Continues logic associated with callable symbol `LoopCount>`.
  **L1312 CN**: 继续与可调用符号 `LoopCount>` 相关的逻辑。
- **L1313 EN**: Executes a call or declaration centered on `parenthesized`.
  **L1313 CN**: 执行以 `parenthesized` 为核心的调用或声明。
- **L1314 EN**: Continues the surrounding expression or declaration: `constexpr auto assumeAligned{"ASSUME_ALIGNED" >>`.
  **L1314 CN**: 继续构造周围的表达式或声明：`constexpr auto assumeAligned{"ASSUME_ALIGNED" >>`。
- **L1315 EN**: Continues logic associated with callable symbol `optionalList`.
  **L1315 CN**: 继续与可调用符号 `optionalList` 相关的逻辑。
- **L1316 EN**: Executes a call or declaration centered on `indirect`.
  **L1316 CN**: 执行以 `indirect` 为核心的调用或声明。
- **L1317 EN**: Continues the surrounding expression or declaration: `constexpr auto vectorAlways{`.
  **L1317 CN**: 继续构造周围的表达式或声明：`constexpr auto vectorAlways{`。
- **L1318 EN**: Executes a call or declaration centered on `construct<CompilerDirective::VectorAlways>`.
  **L1318 CN**: 执行以 `construct<CompilerDirective::VectorAlways>` 为核心的调用或声明。
- **L1319 EN**: Continues the surrounding expression or declaration: `constexpr auto vectorLengthKind{`.
  **L1319 CN**: 继续构造周围的表达式或声明：`constexpr auto vectorLengthKind{`。
- **L1320 EN**: Continues logic associated with callable symbol `pure`.
  **L1320 CN**: 继续与可调用符号 `pure` 相关的逻辑。

### Lines 1321-1344

````cpp
    "SCALABLE" >> pure(CompilerDirective::VectorLength::Kind::Scalable)};
constexpr auto vectorLength{"VECTOR VECTORLENGTH" >>
    parenthesized(construct<CompilerDirective::VectorLength>(
                      digitString64, ","_tok >> vectorLengthKind) ||
        construct<CompilerDirective::VectorLength>(pure(0), vectorLengthKind) ||
        construct<CompilerDirective::VectorLength>(
            digitString64, pure(CompilerDirective::VectorLength::Kind::Auto)))};
constexpr auto unroll{
    "UNROLL" >> construct<CompilerDirective::Unroll>(maybe(digitString64))};
constexpr auto prefetch{"PREFETCH" >>
    construct<CompilerDirective::Prefetch>(nonemptyList(indirect(designator)))};
constexpr auto unrollAndJam{"UNROLL_AND_JAM" >>
    construct<CompilerDirective::UnrollAndJam>(maybe(digitString64))};
constexpr auto novector{"NOVECTOR" >> construct<CompilerDirective::NoVector>()};
constexpr auto nounroll{"NOUNROLL" >> construct<CompilerDirective::NoUnroll>()};
constexpr auto nounrollAndJam{
    "NOUNROLL_AND_JAM" >> construct<CompilerDirective::NoUnrollAndJam>()};
constexpr auto forceinlineDir{
    "FORCEINLINE" >> construct<CompilerDirective::ForceInline>()};
constexpr auto noinlineDir{
    "NOINLINE" >> construct<CompilerDirective::NoInline>()};
constexpr auto inlinealwaysDir{
    "INLINEALWAYS" >> construct<CompilerDirective::InlineAlways>(maybe(name))};
constexpr auto inlineDir{"INLINE" >> construct<CompilerDirective::Inline>()};
````
- **L1321 EN**: Executes a call or declaration centered on `pure`.
  **L1321 CN**: 执行以 `pure` 为核心的调用或声明。
- **L1322 EN**: Continues the surrounding expression or declaration: `constexpr auto vectorLength{"VECTOR VECTORLENGTH" >>`.
  **L1322 CN**: 继续构造周围的表达式或声明：`constexpr auto vectorLength{"VECTOR VECTORLENGTH" >>`。
- **L1323 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L1323 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L1324 EN**: Continues the surrounding expression or declaration: `digitString64, ","_tok >> vectorLengthKind) ||`.
  **L1324 CN**: 继续构造周围的表达式或声明：`digitString64, ","_tok >> vectorLengthKind) ||`。
- **L1325 EN**: Continues logic associated with callable symbol `VectorLength>`.
  **L1325 CN**: 继续与可调用符号 `VectorLength>` 相关的逻辑。
- **L1326 EN**: Continues logic associated with callable symbol `VectorLength>`.
  **L1326 CN**: 继续与可调用符号 `VectorLength>` 相关的逻辑。
- **L1327 EN**: Executes a call or declaration centered on `pure`.
  **L1327 CN**: 执行以 `pure` 为核心的调用或声明。
- **L1328 EN**: Continues the surrounding expression or declaration: `constexpr auto unroll{`.
  **L1328 CN**: 继续构造周围的表达式或声明：`constexpr auto unroll{`。
- **L1329 EN**: Executes a call or declaration centered on `construct<CompilerDirective::Unroll>`.
  **L1329 CN**: 执行以 `construct<CompilerDirective::Unroll>` 为核心的调用或声明。
- **L1330 EN**: Continues the surrounding expression or declaration: `constexpr auto prefetch{"PREFETCH" >>`.
  **L1330 CN**: 继续构造周围的表达式或声明：`constexpr auto prefetch{"PREFETCH" >>`。
- **L1331 EN**: Executes a call or declaration centered on `construct<CompilerDirective::Prefetch>`.
  **L1331 CN**: 执行以 `construct<CompilerDirective::Prefetch>` 为核心的调用或声明。
- **L1332 EN**: Continues the surrounding expression or declaration: `constexpr auto unrollAndJam{"UNROLL_AND_JAM" >>`.
  **L1332 CN**: 继续构造周围的表达式或声明：`constexpr auto unrollAndJam{"UNROLL_AND_JAM" >>`。
- **L1333 EN**: Executes a call or declaration centered on `construct<CompilerDirective::UnrollAndJam>`.
  **L1333 CN**: 执行以 `construct<CompilerDirective::UnrollAndJam>` 为核心的调用或声明。
- **L1334 EN**: Executes a call or declaration centered on `construct<CompilerDirective::NoVector>`.
  **L1334 CN**: 执行以 `construct<CompilerDirective::NoVector>` 为核心的调用或声明。
- **L1335 EN**: Executes a call or declaration centered on `construct<CompilerDirective::NoUnroll>`.
  **L1335 CN**: 执行以 `construct<CompilerDirective::NoUnroll>` 为核心的调用或声明。
- **L1336 EN**: Continues the surrounding expression or declaration: `constexpr auto nounrollAndJam{`.
  **L1336 CN**: 继续构造周围的表达式或声明：`constexpr auto nounrollAndJam{`。
- **L1337 EN**: Executes a call or declaration centered on `construct<CompilerDirective::NoUnrollAndJam>`.
  **L1337 CN**: 执行以 `construct<CompilerDirective::NoUnrollAndJam>` 为核心的调用或声明。
- **L1338 EN**: Continues the surrounding expression or declaration: `constexpr auto forceinlineDir{`.
  **L1338 CN**: 继续构造周围的表达式或声明：`constexpr auto forceinlineDir{`。
- **L1339 EN**: Executes a call or declaration centered on `construct<CompilerDirective::ForceInline>`.
  **L1339 CN**: 执行以 `construct<CompilerDirective::ForceInline>` 为核心的调用或声明。
- **L1340 EN**: Continues the surrounding expression or declaration: `constexpr auto noinlineDir{`.
  **L1340 CN**: 继续构造周围的表达式或声明：`constexpr auto noinlineDir{`。
- **L1341 EN**: Executes a call or declaration centered on `construct<CompilerDirective::NoInline>`.
  **L1341 CN**: 执行以 `construct<CompilerDirective::NoInline>` 为核心的调用或声明。
- **L1342 EN**: Continues the surrounding expression or declaration: `constexpr auto inlinealwaysDir{`.
  **L1342 CN**: 继续构造周围的表达式或声明：`constexpr auto inlinealwaysDir{`。
- **L1343 EN**: Executes a call or declaration centered on `construct<CompilerDirective::InlineAlways>`.
  **L1343 CN**: 执行以 `construct<CompilerDirective::InlineAlways>` 为核心的调用或声明。
- **L1344 EN**: Executes a call or declaration centered on `construct<CompilerDirective::Inline>`.
  **L1344 CN**: 执行以 `construct<CompilerDirective::Inline>` 为核心的调用或声明。

### Lines 1345-1368

````cpp
constexpr auto ivdep{"IVDEP" >> construct<CompilerDirective::IVDep>()};
constexpr auto simd{"SIMD" >> construct<CompilerDirective::Simd>()};
TYPE_PARSER(beginDirective >> some(letter) >> "$ "_tok >>
    sourced((construct<CompilerDirective>(ignore_tkr) ||
                construct<CompilerDirective>(loopCount) ||
                construct<CompilerDirective>(assumeAligned) ||
                construct<CompilerDirective>(vectorAlways) ||
                construct<CompilerDirective>(vectorLength) ||
                construct<CompilerDirective>(unrollAndJam) ||
                construct<CompilerDirective>(unroll) ||
                construct<CompilerDirective>(prefetch) ||
                construct<CompilerDirective>(novector) ||
                construct<CompilerDirective>(nounrollAndJam) ||
                construct<CompilerDirective>(nounroll) ||
                construct<CompilerDirective>(noinlineDir) ||
                construct<CompilerDirective>(forceinlineDir) ||
                construct<CompilerDirective>(inlinealwaysDir) ||
                construct<CompilerDirective>(inlineDir) ||
                construct<CompilerDirective>(simd) ||
                construct<CompilerDirective>(ivdep) ||
                construct<CompilerDirective>(
                    many(construct<CompilerDirective::NameValue>(
                        name, maybe(("="_tok || ":"_tok) >> digitString64))))) /
            endOfStmt ||
````
- **L1345 EN**: Executes a call or declaration centered on `construct<CompilerDirective::IVDep>`.
  **L1345 CN**: 执行以 `construct<CompilerDirective::IVDep>` 为核心的调用或声明。
- **L1346 EN**: Executes a call or declaration centered on `construct<CompilerDirective::Simd>`.
  **L1346 CN**: 执行以 `construct<CompilerDirective::Simd>` 为核心的调用或声明。
- **L1347 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1347 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1348 EN**: Continues logic associated with callable symbol `sourced`.
  **L1348 CN**: 继续与可调用符号 `sourced` 相关的逻辑。
- **L1349 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1349 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1350 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1350 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1351 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1351 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1352 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1352 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1353 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1353 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1354 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1354 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1355 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1355 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1356 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1356 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1357 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1357 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1358 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1358 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1359 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1359 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1360 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1360 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1361 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1361 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1362 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1362 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1363 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1363 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1364 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1364 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1365 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1365 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1366 EN**: Continues logic associated with callable symbol `many`.
  **L1366 CN**: 继续与可调用符号 `many` 相关的逻辑。
- **L1367 EN**: Continues logic associated with callable symbol `maybe`.
  **L1367 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1368 EN**: Continues the surrounding expression or declaration: `endOfStmt ||`.
  **L1368 CN**: 继续构造周围的表达式或声明：`endOfStmt ||`。

### Lines 1369-1392

````cpp
        construct<CompilerDirective>(pure<CompilerDirective::Unrecognized>()) /
            SkipTo<'\n'>{}))

TYPE_PARSER(extension<LanguageFeature::CrayPointer>(
    "nonstandard usage: based POINTER"_port_en_US,
    construct<BasedPointerStmt>(
        "POINTER" >> nonemptyList("expected POINTER associations"_err_en_US,
                         construct<BasedPointer>("(" >> objectName / ",",
                             objectName, maybe(Parser<ArraySpec>{}) / ")")))))

// CUDA-attributes-stmt -> ATTRIBUTES (CUDA-data-attr) [::] name-list
TYPE_PARSER(extension<LanguageFeature::CUDA>(construct<CUDAAttributesStmt>(
    "ATTRIBUTES" >> parenthesized(Parser<common::CUDADataAttr>{}),
    defaulted(
        maybe("::"_tok) >> nonemptyList("expected names"_err_en_US, name)))))

// Subtle: A structure's name includes the surrounding slashes, which avoids
// clashes with other uses of the name in the same scope.
constexpr auto structureName{maybe(sourced("/" >> name / "/"))};

// Note that Parser<StructureStmt>{} has a mandatory list of entity-decls
// and is used only by NestedStructureStmt{}.Parse() in user-state.cpp.
TYPE_PARSER(construct<StructureStmt>("STRUCTURE" >> structureName,
    localRecovery(
````
- **L1369 EN**: Continues logic associated with callable symbol `construct<CompilerDirective>`.
  **L1369 CN**: 继续与可调用符号 `construct<CompilerDirective>` 相关的逻辑。
- **L1370 EN**: Continues the surrounding expression or declaration: `SkipTo<'\n'>{}))`.
  **L1370 CN**: 继续构造周围的表达式或声明：`SkipTo<'\n'>{}))`。
- **L1371 EN**: Blank line separating nearby declarations or logic blocks.
  **L1371 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1372 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1372 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: based POINTER"_port_en_US,`.
  **L1373 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: based POINTER"_port_en_US,`。
- **L1374 EN**: Continues logic associated with callable symbol `construct<BasedPointerStmt>`.
  **L1374 CN**: 继续与可调用符号 `construct<BasedPointerStmt>` 相关的逻辑。
- **L1375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"POINTER" >> nonemptyList("expected POINTER associations"_err_en_US,`.
  **L1375 CN**: 继续一个多行参数列表、初始化器或聚合项：`"POINTER" >> nonemptyList("expected POINTER associations"_err_en_US,`。
- **L1376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<BasedPointer>("(" >> objectName / ",",`.
  **L1376 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<BasedPointer>("(" >> objectName / ",",`。
- **L1377 EN**: Continues logic associated with callable symbol `maybe`.
  **L1377 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1378 EN**: Blank line separating nearby declarations or logic blocks.
  **L1378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1379 EN**: Comment explains nearby logic, intent, or metadata: `CUDA-attributes-stmt -> ATTRIBUTES (CUDA-data-attr) [::] name-list`.
  **L1379 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUDA-attributes-stmt -> ATTRIBUTES (CUDA-data-attr) [::] name-list`。
- **L1380 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1380 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ATTRIBUTES" >> parenthesized(Parser<common::CUDADataAttr>{}),`.
  **L1381 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ATTRIBUTES" >> parenthesized(Parser<common::CUDADataAttr>{}),`。
- **L1382 EN**: Continues logic associated with callable symbol `defaulted`.
  **L1382 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L1383 EN**: Continues logic associated with callable symbol `maybe`.
  **L1383 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L1384 EN**: Blank line separating nearby declarations or logic blocks.
  **L1384 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1385 EN**: Comment explains nearby logic, intent, or metadata: `Subtle: A structure's name includes the surrounding slashes, which avoids`.
  **L1385 CN**: 注释说明附近代码的逻辑、意图或元数据：`Subtle: A structure's name includes the surrounding slashes, which avoids`。
- **L1386 EN**: Comment explains nearby logic, intent, or metadata: `clashes with other uses of the name in the same scope.`.
  **L1386 CN**: 注释说明附近代码的逻辑、意图或元数据：`clashes with other uses of the name in the same scope.`。
- **L1387 EN**: Executes a call or declaration centered on `structureName{maybe`.
  **L1387 CN**: 执行以 `structureName{maybe` 为核心的调用或声明。
- **L1388 EN**: Blank line separating nearby declarations or logic blocks.
  **L1388 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1389 EN**: Comment explains nearby logic, intent, or metadata: `Note that Parser<StructureStmt>{} has a mandatory list of entity-decls`.
  **L1389 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that Parser<StructureStmt>{} has a mandatory list of entity-decls`。
- **L1390 EN**: Comment explains nearby logic, intent, or metadata: `and is used only by NestedStructureStmt{}.Parse() in user-state.cpp.`.
  **L1390 CN**: 注释说明附近代码的逻辑、意图或元数据：`and is used only by NestedStructureStmt{}.Parse() in user-state.cpp.`。
- **L1391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<StructureStmt>("STRUCTURE" >> structureName,`.
  **L1391 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<StructureStmt>("STRUCTURE" >> structureName,`。
- **L1392 EN**: Continues logic associated with callable symbol `localRecovery`.
  **L1392 CN**: 继续与可调用符号 `localRecovery` 相关的逻辑。

### Lines 1393-1416

````cpp
        "entity declarations are required on a nested structure"_err_en_US,
        nonemptyList(entityDecl), ok)))

constexpr auto nestedStructureDef{
    CONTEXT_PARSER("nested STRUCTURE definition"_en_US,
        construct<StructureDef>(statement(NestedStructureStmt{}),
            many(Parser<StructureField>{}),
            statement(construct<StructureDef::EndStructureStmt>(
                "END STRUCTURE"_tok))))};

TYPE_PARSER(construct<StructureField>(statement(StructureComponents{})) ||
    construct<StructureField>(indirect(Parser<Union>{})) ||
    construct<StructureField>(indirect(nestedStructureDef)))

TYPE_CONTEXT_PARSER("STRUCTURE definition"_en_US,
    extension<LanguageFeature::DECStructures>(
        "nonstandard usage: STRUCTURE"_port_en_US,
        construct<StructureDef>(
            statement(construct<StructureStmt>(
                "STRUCTURE" >> structureName, optionalList(entityDecl))),
            many(Parser<StructureField>{}),
            statement(construct<StructureDef::EndStructureStmt>(
                "END STRUCTURE"_tok)))))

````
- **L1393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"entity declarations are required on a nested structure"_err_en_US,`.
  **L1393 CN**: 继续一个多行参数列表、初始化器或聚合项：`"entity declarations are required on a nested structure"_err_en_US,`。
- **L1394 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L1394 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L1395 EN**: Blank line separating nearby declarations or logic blocks.
  **L1395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1396 EN**: Continues the surrounding expression or declaration: `constexpr auto nestedStructureDef{`.
  **L1396 CN**: 继续构造周围的表达式或声明：`constexpr auto nestedStructureDef{`。
- **L1397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CONTEXT_PARSER("nested STRUCTURE definition"_en_US,`.
  **L1397 CN**: 继续一个多行参数列表、初始化器或聚合项：`CONTEXT_PARSER("nested STRUCTURE definition"_en_US,`。
- **L1398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<StructureDef>(statement(NestedStructureStmt{}),`.
  **L1398 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<StructureDef>(statement(NestedStructureStmt{}),`。
- **L1399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(Parser<StructureField>{}),`.
  **L1399 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(Parser<StructureField>{}),`。
- **L1400 EN**: Continues logic associated with callable symbol `statement`.
  **L1400 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L1401 EN**: Executes a standalone statement or declaration: `"END STRUCTURE"_tok))))};`.
  **L1401 CN**: 执行一条独立语句或声明：`"END STRUCTURE"_tok))))};`。
- **L1402 EN**: Blank line separating nearby declarations or logic blocks.
  **L1402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1403 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L1403 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L1404 EN**: Continues logic associated with callable symbol `construct<StructureField>`.
  **L1404 CN**: 继续与可调用符号 `construct<StructureField>` 相关的逻辑。
- **L1405 EN**: Continues logic associated with callable symbol `construct<StructureField>`.
  **L1405 CN**: 继续与可调用符号 `construct<StructureField>` 相关的逻辑。
- **L1406 EN**: Blank line separating nearby declarations or logic blocks.
  **L1406 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("STRUCTURE definition"_en_US,`.
  **L1407 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("STRUCTURE definition"_en_US,`。
- **L1408 EN**: Continues logic associated with callable symbol `DECStructures>`.
  **L1408 CN**: 继续与可调用符号 `DECStructures>` 相关的逻辑。
- **L1409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: STRUCTURE"_port_en_US,`.
  **L1409 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: STRUCTURE"_port_en_US,`。
- **L1410 EN**: Continues logic associated with callable symbol `construct<StructureDef>`.
  **L1410 CN**: 继续与可调用符号 `construct<StructureDef>` 相关的逻辑。
- **L1411 EN**: Continues logic associated with callable symbol `statement`.
  **L1411 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L1412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"STRUCTURE" >> structureName, optionalList(entityDecl))),`.
  **L1412 CN**: 继续一个多行参数列表、初始化器或聚合项：`"STRUCTURE" >> structureName, optionalList(entityDecl))),`。
- **L1413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(Parser<StructureField>{}),`.
  **L1413 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(Parser<StructureField>{}),`。
- **L1414 EN**: Continues logic associated with callable symbol `statement`.
  **L1414 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L1415 EN**: Continues the surrounding expression or declaration: `"END STRUCTURE"_tok)))))`.
  **L1415 CN**: 继续构造周围的表达式或声明：`"END STRUCTURE"_tok)))))`。
- **L1416 EN**: Blank line separating nearby declarations or logic blocks.
  **L1416 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1417-1440

````cpp
TYPE_CONTEXT_PARSER("UNION definition"_en_US,
    construct<Union>(statement(construct<Union::UnionStmt>("UNION"_tok)),
        many(Parser<Map>{}),
        statement(construct<Union::EndUnionStmt>("END UNION"_tok))))

TYPE_CONTEXT_PARSER("MAP definition"_en_US,
    construct<Map>(statement(construct<Map::MapStmt>("MAP"_tok)),
        many(Parser<StructureField>{}),
        statement(construct<Map::EndMapStmt>("END MAP"_tok))))

TYPE_CONTEXT_PARSER("arithmetic IF statement"_en_US,
    deprecated<LanguageFeature::ArithmeticIF>(construct<ArithmeticIfStmt>(
        "IF" >> parenthesized(expr), label / ",", label / ",", label)))

TYPE_CONTEXT_PARSER("ASSIGN statement"_en_US,
    deprecated<LanguageFeature::Assign>(
        construct<AssignStmt>("ASSIGN" >> label, "TO" >> name)))

TYPE_CONTEXT_PARSER("assigned GOTO statement"_en_US,
    deprecated<LanguageFeature::AssignedGOTO>(construct<AssignedGotoStmt>(
        "GO TO" >> name,
        defaulted(maybe(","_tok) >>
            parenthesized(nonemptyList("expected labels"_err_en_US, label))))))

````
- **L1417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("UNION definition"_en_US,`.
  **L1417 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("UNION definition"_en_US,`。
- **L1418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<Union>(statement(construct<Union::UnionStmt>("UNION"_tok)),`.
  **L1418 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<Union>(statement(construct<Union::UnionStmt>("UNION"_tok)),`。
- **L1419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(Parser<Map>{}),`.
  **L1419 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(Parser<Map>{}),`。
- **L1420 EN**: Continues logic associated with callable symbol `statement`.
  **L1420 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L1421 EN**: Blank line separating nearby declarations or logic blocks.
  **L1421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("MAP definition"_en_US,`.
  **L1422 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("MAP definition"_en_US,`。
- **L1423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<Map>(statement(construct<Map::MapStmt>("MAP"_tok)),`.
  **L1423 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<Map>(statement(construct<Map::MapStmt>("MAP"_tok)),`。
- **L1424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(Parser<StructureField>{}),`.
  **L1424 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(Parser<StructureField>{}),`。
- **L1425 EN**: Continues logic associated with callable symbol `statement`.
  **L1425 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L1426 EN**: Blank line separating nearby declarations or logic blocks.
  **L1426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("arithmetic IF statement"_en_US,`.
  **L1427 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("arithmetic IF statement"_en_US,`。
- **L1428 EN**: Continues logic associated with callable symbol `ArithmeticIF>`.
  **L1428 CN**: 继续与可调用符号 `ArithmeticIF>` 相关的逻辑。
- **L1429 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L1429 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("ASSIGN statement"_en_US,`.
  **L1431 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("ASSIGN statement"_en_US,`。
- **L1432 EN**: Continues logic associated with callable symbol `Assign>`.
  **L1432 CN**: 继续与可调用符号 `Assign>` 相关的逻辑。
- **L1433 EN**: Continues logic associated with callable symbol `construct<AssignStmt>`.
  **L1433 CN**: 继续与可调用符号 `construct<AssignStmt>` 相关的逻辑。
- **L1434 EN**: Blank line separating nearby declarations or logic blocks.
  **L1434 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("assigned GOTO statement"_en_US,`.
  **L1435 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("assigned GOTO statement"_en_US,`。
- **L1436 EN**: Continues logic associated with callable symbol `AssignedGOTO>`.
  **L1436 CN**: 继续与可调用符号 `AssignedGOTO>` 相关的逻辑。
- **L1437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"GO TO" >> name,`.
  **L1437 CN**: 继续一个多行参数列表、初始化器或聚合项：`"GO TO" >> name,`。
- **L1438 EN**: Continues logic associated with callable symbol `defaulted`.
  **L1438 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L1439 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L1439 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L1440 EN**: Blank line separating nearby declarations or logic blocks.
  **L1440 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1441-1464

````cpp
TYPE_CONTEXT_PARSER("PAUSE statement"_en_US,
    deprecated<LanguageFeature::Pause>(
        construct<PauseStmt>("PAUSE" >> maybe(Parser<StopCode>{}))))

// These requirement productions are defined by the Fortran standard but never
// used directly by the grammar:
//   R620 delimiter -> ( | ) | / | [ | ] | (/ | /)
//   R1027 numeric-expr -> expr
//   R1031 int-constant-expr -> int-expr
//   R1221 dtv-type-spec -> TYPE ( derived-type-spec ) |
//           CLASS ( derived-type-spec )
//
// These requirement productions are defined and used, but need not be
// defined independently here in this file:
//   R771 lbracket -> [
//   R772 rbracket -> ]
//
// Further note that:
//   R607 int-constant -> constant
//     is used only once via R844 scalar-int-constant
//   R904 logical-variable -> variable
//     is used only via scalar-logical-variable
//   R906 default-char-variable -> variable
//     is used only via scalar-default-char-variable
````
- **L1441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("PAUSE statement"_en_US,`.
  **L1441 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("PAUSE statement"_en_US,`。
- **L1442 EN**: Continues logic associated with callable symbol `Pause>`.
  **L1442 CN**: 继续与可调用符号 `Pause>` 相关的逻辑。
- **L1443 EN**: Continues logic associated with callable symbol `construct<PauseStmt>`.
  **L1443 CN**: 继续与可调用符号 `construct<PauseStmt>` 相关的逻辑。
- **L1444 EN**: Blank line separating nearby declarations or logic blocks.
  **L1444 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1445 EN**: Comment explains nearby logic, intent, or metadata: `These requirement productions are defined by the Fortran standard but never`.
  **L1445 CN**: 注释说明附近代码的逻辑、意图或元数据：`These requirement productions are defined by the Fortran standard but never`。
- **L1446 EN**: Comment explains nearby logic, intent, or metadata: `used directly by the grammar:`.
  **L1446 CN**: 注释说明附近代码的逻辑、意图或元数据：`used directly by the grammar:`。
- **L1447 EN**: Comment explains nearby logic, intent, or metadata: `R620 delimiter -> ( | ) | / | [ | ] | (/ | /)`.
  **L1447 CN**: 注释说明附近代码的逻辑、意图或元数据：`R620 delimiter -> ( | ) | / | [ | ] | (/ | /)`。
- **L1448 EN**: Comment explains nearby logic, intent, or metadata: `R1027 numeric-expr -> expr`.
  **L1448 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1027 numeric-expr -> expr`。
- **L1449 EN**: Comment explains nearby logic, intent, or metadata: `R1031 int-constant-expr -> int-expr`.
  **L1449 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1031 int-constant-expr -> int-expr`。
- **L1450 EN**: Comment explains nearby logic, intent, or metadata: `R1221 dtv-type-spec -> TYPE ( derived-type-spec ) |`.
  **L1450 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1221 dtv-type-spec -> TYPE ( derived-type-spec ) |`。
- **L1451 EN**: Comment explains nearby logic, intent, or metadata: `CLASS ( derived-type-spec )`.
  **L1451 CN**: 注释说明附近代码的逻辑、意图或元数据：`CLASS ( derived-type-spec )`。
- **L1452 EN**: Separator comment used for visual grouping.
  **L1452 CN**: 用于视觉分组的分隔注释。
- **L1453 EN**: Comment explains nearby logic, intent, or metadata: `These requirement productions are defined and used, but need not be`.
  **L1453 CN**: 注释说明附近代码的逻辑、意图或元数据：`These requirement productions are defined and used, but need not be`。
- **L1454 EN**: Comment explains nearby logic, intent, or metadata: `defined independently here in this file:`.
  **L1454 CN**: 注释说明附近代码的逻辑、意图或元数据：`defined independently here in this file:`。
- **L1455 EN**: Comment explains nearby logic, intent, or metadata: `R771 lbracket -> [`.
  **L1455 CN**: 注释说明附近代码的逻辑、意图或元数据：`R771 lbracket -> [`。
- **L1456 EN**: Comment explains nearby logic, intent, or metadata: `R772 rbracket -> ]`.
  **L1456 CN**: 注释说明附近代码的逻辑、意图或元数据：`R772 rbracket -> ]`。
- **L1457 EN**: Separator comment used for visual grouping.
  **L1457 CN**: 用于视觉分组的分隔注释。
- **L1458 EN**: Comment explains nearby logic, intent, or metadata: `Further note that:`.
  **L1458 CN**: 注释说明附近代码的逻辑、意图或元数据：`Further note that:`。
- **L1459 EN**: Comment explains nearby logic, intent, or metadata: `R607 int-constant -> constant`.
  **L1459 CN**: 注释说明附近代码的逻辑、意图或元数据：`R607 int-constant -> constant`。
- **L1460 EN**: Comment explains nearby logic, intent, or metadata: `is used only once via R844 scalar-int-constant`.
  **L1460 CN**: 注释说明附近代码的逻辑、意图或元数据：`is used only once via R844 scalar-int-constant`。
- **L1461 EN**: Comment explains nearby logic, intent, or metadata: `R904 logical-variable -> variable`.
  **L1461 CN**: 注释说明附近代码的逻辑、意图或元数据：`R904 logical-variable -> variable`。
- **L1462 EN**: Comment explains nearby logic, intent, or metadata: `is used only via scalar-logical-variable`.
  **L1462 CN**: 注释说明附近代码的逻辑、意图或元数据：`is used only via scalar-logical-variable`。
- **L1463 EN**: Comment explains nearby logic, intent, or metadata: `R906 default-char-variable -> variable`.
  **L1463 CN**: 注释说明附近代码的逻辑、意图或元数据：`R906 default-char-variable -> variable`。
- **L1464 EN**: Comment explains nearby logic, intent, or metadata: `is used only via scalar-default-char-variable`.
  **L1464 CN**: 注释说明附近代码的逻辑、意图或元数据：`is used only via scalar-default-char-variable`。

### Lines 1465-1476

````cpp
//   R907 int-variable -> variable
//     is used only via scalar-int-variable
//   R915 complex-part-designator -> designator % RE | designator % IM
//     %RE and %IM are initially recognized as structure components
//   R916 type-param-inquiry -> designator % type-param-name
//     is occulted by structure component designators
//   R918 array-section ->
//        data-ref [( substring-range )] | complex-part-designator
//     is not used because parsing is not sensitive to rank
//   R1030 default-char-constant-expr -> default-char-expr
//     is only used via scalar-default-char-constant-expr
} // namespace Fortran::parser
````
- **L1465 EN**: Comment explains nearby logic, intent, or metadata: `R907 int-variable -> variable`.
  **L1465 CN**: 注释说明附近代码的逻辑、意图或元数据：`R907 int-variable -> variable`。
- **L1466 EN**: Comment explains nearby logic, intent, or metadata: `is used only via scalar-int-variable`.
  **L1466 CN**: 注释说明附近代码的逻辑、意图或元数据：`is used only via scalar-int-variable`。
- **L1467 EN**: Comment explains nearby logic, intent, or metadata: `R915 complex-part-designator -> designator % RE | designator % IM`.
  **L1467 CN**: 注释说明附近代码的逻辑、意图或元数据：`R915 complex-part-designator -> designator % RE | designator % IM`。
- **L1468 EN**: Comment explains nearby logic, intent, or metadata: `%RE and %IM are initially recognized as structure components`.
  **L1468 CN**: 注释说明附近代码的逻辑、意图或元数据：`%RE and %IM are initially recognized as structure components`。
- **L1469 EN**: Comment explains nearby logic, intent, or metadata: `R916 type-param-inquiry -> designator % type-param-name`.
  **L1469 CN**: 注释说明附近代码的逻辑、意图或元数据：`R916 type-param-inquiry -> designator % type-param-name`。
- **L1470 EN**: Comment explains nearby logic, intent, or metadata: `is occulted by structure component designators`.
  **L1470 CN**: 注释说明附近代码的逻辑、意图或元数据：`is occulted by structure component designators`。
- **L1471 EN**: Comment explains nearby logic, intent, or metadata: `R918 array-section ->`.
  **L1471 CN**: 注释说明附近代码的逻辑、意图或元数据：`R918 array-section ->`。
- **L1472 EN**: Comment explains nearby logic, intent, or metadata: `data-ref [( substring-range )] | complex-part-designator`.
  **L1472 CN**: 注释说明附近代码的逻辑、意图或元数据：`data-ref [( substring-range )] | complex-part-designator`。
- **L1473 EN**: Comment explains nearby logic, intent, or metadata: `is not used because parsing is not sensitive to rank`.
  **L1473 CN**: 注释说明附近代码的逻辑、意图或元数据：`is not used because parsing is not sensitive to rank`。
- **L1474 EN**: Comment explains nearby logic, intent, or metadata: `R1030 default-char-constant-expr -> default-char-expr`.
  **L1474 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1030 default-char-constant-expr -> default-char-expr`。
- **L1475 EN**: Comment explains nearby logic, intent, or metadata: `is only used via scalar-default-char-constant-expr`.
  **L1475 CN**: 注释说明附近代码的逻辑、意图或元数据：`is only used via scalar-default-char-constant-expr`。
- **L1476 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::parser`.
  **L1476 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::parser`。

## Key Concepts / 关键概念

- **Parse-tree construction and source handling / 语法树构建与源码处理**
- **Fortran parse tree handling / Fortran 语法树处理**
- **OpenMP handling / OpenMP 处理**
- **CUDA-specific lowering or runtime handling / CUDA 专用 lowering 或运行时处理**

## Dependencies / 依赖关系

- `basic-parsers.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `expr-parsers.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `misc-parsers.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `stmt-parser.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `token-parsers.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `type-parser-implementation.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/user-state.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
