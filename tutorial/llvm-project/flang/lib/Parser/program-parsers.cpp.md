# program-parsers.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Parser/program-parsers.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Per-type parsers for program units.
- **Purpose (CN)**: 实现 program parsers 相关的解析、语法树支持或源码级处理流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- lib/Parser/program-parsers.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// Per-type parsers for program units

#include "basic-parsers.h"
#include "expr-parsers.h"
#include "misc-parsers.h"
#include "stmt-parser.h"
#include "token-parsers.h"
#include "type-parser-implementation.h"
#include "flang/Parser/characters.h"
#include "flang/Parser/parse-tree.h"

namespace Fortran::parser {
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
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `Per-type parsers for program units`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`Per-type parsers for program units`。
- **L10 EN**: Blank line separating nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L11 EN**: Includes "basic-parsers.h" to access local declarations paired with this implementation.
  **L11 CN**: 引入 "basic-parsers.h" 以使用与该实现配套的本地声明。
- **L12 EN**: Includes "expr-parsers.h" to access local declarations paired with this implementation.
  **L12 CN**: 引入 "expr-parsers.h" 以使用与该实现配套的本地声明。
- **L13 EN**: Includes "misc-parsers.h" to access local declarations paired with this implementation.
  **L13 CN**: 引入 "misc-parsers.h" 以使用与该实现配套的本地声明。
- **L14 EN**: Includes "stmt-parser.h" to access local declarations paired with this implementation.
  **L14 CN**: 引入 "stmt-parser.h" 以使用与该实现配套的本地声明。
- **L15 EN**: Includes "token-parsers.h" to access local declarations paired with this implementation.
  **L15 CN**: 引入 "token-parsers.h" 以使用与该实现配套的本地声明。
- **L16 EN**: Includes "type-parser-implementation.h" to access local declarations paired with this implementation.
  **L16 CN**: 引入 "type-parser-implementation.h" 以使用与该实现配套的本地声明。
- **L17 EN**: Includes "flang/Parser/characters.h" to access parse-tree, token, or source representation support.
  **L17 CN**: 引入 "flang/Parser/characters.h" 以使用语法树、词法单元或源码表示支持。
- **L18 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L18 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Opens namespace scope `Fortran::parser`.
  **L20 CN**: 打开命名空间作用域 `Fortran::parser`。

### Lines 21-40

````cpp

// R1530 function-stmt ->
//         [prefix] FUNCTION function-name ( [dummy-arg-name-list] ) [suffix]
// R1526 prefix -> prefix-spec [prefix-spec]...
// R1531 dummy-arg-name -> name

static constexpr auto validFunctionStmt{
    construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name,
        parenthesized(optionalList(name)), maybe(suffix)) /
        atEndOfStmt ||
    construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name / atEndOfStmt,
        // PGI & Intel accept "FUNCTION F"
        extension<LanguageFeature::OmitFunctionDummies>(
            "nonstandard usage: FUNCTION statement without dummy argument list"_port_en_US,
            pure<std::list<Name>>()),
        pure<std::optional<Suffix>>())};

// function-stmt with error recovery -- used in interfaces and internal
// subprograms, but not at the top level, where REALFUNCTIONF and
// INTEGERPUREELEMENTALFUNCTIONG(10) might appear as the first statement
````
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Comment explains nearby logic, intent, or metadata: `R1530 function-stmt ->`.
  **L22 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1530 function-stmt ->`。
- **L23 EN**: Comment explains nearby logic, intent, or metadata: `[prefix] FUNCTION function-name ( [dummy-arg-name-list] ) [suffix]`.
  **L23 CN**: 注释说明附近代码的逻辑、意图或元数据：`[prefix] FUNCTION function-name ( [dummy-arg-name-list] ) [suffix]`。
- **L24 EN**: Comment explains nearby logic, intent, or metadata: `R1526 prefix -> prefix-spec [prefix-spec]...`.
  **L24 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1526 prefix -> prefix-spec [prefix-spec]...`。
- **L25 EN**: Comment explains nearby logic, intent, or metadata: `R1531 dummy-arg-name -> name`.
  **L25 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1531 dummy-arg-name -> name`。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Continues the surrounding expression or declaration: `static constexpr auto validFunctionStmt{`.
  **L27 CN**: 继续构造周围的表达式或声明：`static constexpr auto validFunctionStmt{`。
- **L28 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name,`.
  **L28 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name,`。
- **L29 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L29 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L30 EN**: Continues the surrounding expression or declaration: `atEndOfStmt ||`.
  **L30 CN**: 继续构造周围的表达式或声明：`atEndOfStmt ||`。
- **L31 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name / atEndOfStmt,`.
  **L31 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name / atEndOfStmt,`。
- **L32 EN**: Comment explains nearby logic, intent, or metadata: `PGI & Intel accept "FUNCTION F"`.
  **L32 CN**: 注释说明附近代码的逻辑、意图或元数据：`PGI & Intel accept "FUNCTION F"`。
- **L33 EN**: Continues logic associated with callable symbol `OmitFunctionDummies>`.
  **L33 CN**: 继续与可调用符号 `OmitFunctionDummies>` 相关的逻辑。
- **L34 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: FUNCTION statement without dummy argument list"_port_en_US,`.
  **L34 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: FUNCTION statement without dummy argument list"_port_en_US,`。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `pure<std::list<Name>>()),`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`pure<std::list<Name>>()),`。
- **L36 EN**: Executes a call or declaration centered on `pure<std::optional<Suffix>>`.
  **L36 CN**: 执行以 `pure<std::optional<Suffix>>` 为核心的调用或声明。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Comment explains nearby logic, intent, or metadata: `function-stmt with error recovery -- used in interfaces and internal`.
  **L38 CN**: 注释说明附近代码的逻辑、意图或元数据：`function-stmt with error recovery -- used in interfaces and internal`。
- **L39 EN**: Comment explains nearby logic, intent, or metadata: `subprograms, but not at the top level, where REALFUNCTIONF and`.
  **L39 CN**: 注释说明附近代码的逻辑、意图或元数据：`subprograms, but not at the top level, where REALFUNCTIONF and`。
- **L40 EN**: Comment explains nearby logic, intent, or metadata: `INTEGERPUREELEMENTALFUNCTIONG(10) might appear as the first statement`.
  **L40 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTEGERPUREELEMENTALFUNCTIONG(10) might appear as the first statement`。

### Lines 41-60

````cpp
// of a main program.
TYPE_PARSER(validFunctionStmt ||
    construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name,
        defaulted(parenthesized(optionalList(name))), maybe(suffix)) /
        checkEndOfKnownStmt)

// R502 program-unit ->
//        main-program | external-subprogram | module | submodule | block-data
// R503 external-subprogram -> function-subprogram | subroutine-subprogram
// N.B. "module" must precede "external-subprogram" in this sequence of
// alternatives to avoid ambiguity with the MODULE keyword prefix that
// they recognize.  I.e., "modulesubroutinefoo" should start a module
// "subroutinefoo", not a subroutine "foo" with the MODULE prefix.  The
// ambiguity is exacerbated by the extension that accepts a function
// statement without an otherwise empty list of dummy arguments.  That
// MODULE prefix is disallowed by a constraint (C1547) in this context,
// so the standard language is not ambiguous, but disabling its misrecognition
// here would require context-sensitive keyword recognition or variant parsers
// for several productions; giving the "module" production priority here is a
// cleaner solution, though regrettably subtle.
````
- **L41 EN**: Comment explains nearby logic, intent, or metadata: `of a main program.`.
  **L41 CN**: 注释说明附近代码的逻辑、意图或元数据：`of a main program.`。
- **L42 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L42 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L43 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name,`.
  **L43 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<FunctionStmt>(many(prefixSpec), "FUNCTION" >> name,`。
- **L44 EN**: Continues logic associated with callable symbol `defaulted`.
  **L44 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L45 EN**: Continues the surrounding expression or declaration: `checkEndOfKnownStmt)`.
  **L45 CN**: 继续构造周围的表达式或声明：`checkEndOfKnownStmt)`。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Comment explains nearby logic, intent, or metadata: `R502 program-unit ->`.
  **L47 CN**: 注释说明附近代码的逻辑、意图或元数据：`R502 program-unit ->`。
- **L48 EN**: Comment explains nearby logic, intent, or metadata: `main-program | external-subprogram | module | submodule | block-data`.
  **L48 CN**: 注释说明附近代码的逻辑、意图或元数据：`main-program | external-subprogram | module | submodule | block-data`。
- **L49 EN**: Comment explains nearby logic, intent, or metadata: `R503 external-subprogram -> function-subprogram | subroutine-subprogram`.
  **L49 CN**: 注释说明附近代码的逻辑、意图或元数据：`R503 external-subprogram -> function-subprogram | subroutine-subprogram`。
- **L50 EN**: Comment explains nearby logic, intent, or metadata: `N.B. "module" must precede "external-subprogram" in this sequence of`.
  **L50 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. "module" must precede "external-subprogram" in this sequence of`。
- **L51 EN**: Comment explains nearby logic, intent, or metadata: `alternatives to avoid ambiguity with the MODULE keyword prefix that`.
  **L51 CN**: 注释说明附近代码的逻辑、意图或元数据：`alternatives to avoid ambiguity with the MODULE keyword prefix that`。
- **L52 EN**: Comment explains nearby logic, intent, or metadata: `they recognize.  I.e., "modulesubroutinefoo" should start a module`.
  **L52 CN**: 注释说明附近代码的逻辑、意图或元数据：`they recognize.  I.e., "modulesubroutinefoo" should start a module`。
- **L53 EN**: Comment explains nearby logic, intent, or metadata: `"subroutinefoo", not a subroutine "foo" with the MODULE prefix.  The`.
  **L53 CN**: 注释说明附近代码的逻辑、意图或元数据：`"subroutinefoo", not a subroutine "foo" with the MODULE prefix.  The`。
- **L54 EN**: Comment explains nearby logic, intent, or metadata: `ambiguity is exacerbated by the extension that accepts a function`.
  **L54 CN**: 注释说明附近代码的逻辑、意图或元数据：`ambiguity is exacerbated by the extension that accepts a function`。
- **L55 EN**: Comment explains nearby logic, intent, or metadata: `statement without an otherwise empty list of dummy arguments.  That`.
  **L55 CN**: 注释说明附近代码的逻辑、意图或元数据：`statement without an otherwise empty list of dummy arguments.  That`。
- **L56 EN**: Comment explains nearby logic, intent, or metadata: `MODULE prefix is disallowed by a constraint (C1547) in this context,`.
  **L56 CN**: 注释说明附近代码的逻辑、意图或元数据：`MODULE prefix is disallowed by a constraint (C1547) in this context,`。
- **L57 EN**: Comment explains nearby logic, intent, or metadata: `so the standard language is not ambiguous, but disabling its misrecognition`.
  **L57 CN**: 注释说明附近代码的逻辑、意图或元数据：`so the standard language is not ambiguous, but disabling its misrecognition`。
- **L58 EN**: Comment explains nearby logic, intent, or metadata: `here would require context-sensitive keyword recognition or variant parsers`.
  **L58 CN**: 注释说明附近代码的逻辑、意图或元数据：`here would require context-sensitive keyword recognition or variant parsers`。
- **L59 EN**: Comment explains nearby logic, intent, or metadata: `for several productions; giving the "module" production priority here is a`.
  **L59 CN**: 注释说明附近代码的逻辑、意图或元数据：`for several productions; giving the "module" production priority here is a`。
- **L60 EN**: Comment explains nearby logic, intent, or metadata: `cleaner solution, though regrettably subtle.`.
  **L60 CN**: 注释说明附近代码的逻辑、意图或元数据：`cleaner solution, though regrettably subtle.`。

### Lines 61-80

````cpp
// Enforcing C1547 is done in semantics.
static constexpr auto programUnit{
    construct<ProgramUnit>(indirect(Parser<Module>{})) ||
    construct<ProgramUnit>(indirect(subroutineSubprogram)) ||
    construct<ProgramUnit>(indirect(Parser<Submodule>{})) ||
    construct<ProgramUnit>(indirect(Parser<BlockData>{})) ||
    lookAhead(maybe(label) >> validFunctionStmt) >>
        construct<ProgramUnit>(indirect(functionSubprogram)) ||
    construct<ProgramUnit>(indirect(Parser<MainProgram>{}))};

// Note, F'23 6.3.1 states that "A Fortran program unit is a sequence of one or
// more lines, organized as Fortran statements, comments, and INCLUDE lines."
// which could be interpreted as implying program units must exist on mutually
// exclusive lines. Nag interprets it this way. We have an extension to allow
// multiple program units on the same line.
static constexpr auto normalProgramUnit{
    !consumedAllInput >> StartNewSubprogram{} >> programUnit /
        recovery((maybe(semicolons) >> endOfLine) ||
                (extension<LanguageFeature::MultipleProgramUnitsOnSameLine>(
                    "nonstandard usage: end of program unit not terminated by new line"_port_en_US,
````
- **L61 EN**: Comment explains nearby logic, intent, or metadata: `Enforcing C1547 is done in semantics.`.
  **L61 CN**: 注释说明附近代码的逻辑、意图或元数据：`Enforcing C1547 is done in semantics.`。
- **L62 EN**: Continues the surrounding expression or declaration: `static constexpr auto programUnit{`.
  **L62 CN**: 继续构造周围的表达式或声明：`static constexpr auto programUnit{`。
- **L63 EN**: Continues logic associated with callable symbol `construct<ProgramUnit>`.
  **L63 CN**: 继续与可调用符号 `construct<ProgramUnit>` 相关的逻辑。
- **L64 EN**: Continues logic associated with callable symbol `construct<ProgramUnit>`.
  **L64 CN**: 继续与可调用符号 `construct<ProgramUnit>` 相关的逻辑。
- **L65 EN**: Continues logic associated with callable symbol `construct<ProgramUnit>`.
  **L65 CN**: 继续与可调用符号 `construct<ProgramUnit>` 相关的逻辑。
- **L66 EN**: Continues logic associated with callable symbol `construct<ProgramUnit>`.
  **L66 CN**: 继续与可调用符号 `construct<ProgramUnit>` 相关的逻辑。
- **L67 EN**: Continues logic associated with callable symbol `lookAhead`.
  **L67 CN**: 继续与可调用符号 `lookAhead` 相关的逻辑。
- **L68 EN**: Continues logic associated with callable symbol `construct<ProgramUnit>`.
  **L68 CN**: 继续与可调用符号 `construct<ProgramUnit>` 相关的逻辑。
- **L69 EN**: Executes a call or declaration centered on `construct<ProgramUnit>`.
  **L69 CN**: 执行以 `construct<ProgramUnit>` 为核心的调用或声明。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L71 EN**: Comment explains nearby logic, intent, or metadata: `Note, F'23 6.3.1 states that "A Fortran program unit is a sequence of one or`.
  **L71 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note, F'23 6.3.1 states that "A Fortran program unit is a sequence of one or`。
- **L72 EN**: Comment explains nearby logic, intent, or metadata: `more lines, organized as Fortran statements, comments, and INCLUDE lines."`.
  **L72 CN**: 注释说明附近代码的逻辑、意图或元数据：`more lines, organized as Fortran statements, comments, and INCLUDE lines."`。
- **L73 EN**: Comment explains nearby logic, intent, or metadata: `which could be interpreted as implying program units must exist on mutually`.
  **L73 CN**: 注释说明附近代码的逻辑、意图或元数据：`which could be interpreted as implying program units must exist on mutually`。
- **L74 EN**: Comment explains nearby logic, intent, or metadata: `exclusive lines. Nag interprets it this way. We have an extension to allow`.
  **L74 CN**: 注释说明附近代码的逻辑、意图或元数据：`exclusive lines. Nag interprets it this way. We have an extension to allow`。
- **L75 EN**: Comment explains nearby logic, intent, or metadata: `multiple program units on the same line.`.
  **L75 CN**: 注释说明附近代码的逻辑、意图或元数据：`multiple program units on the same line.`。
- **L76 EN**: Continues the surrounding expression or declaration: `static constexpr auto normalProgramUnit{`.
  **L76 CN**: 继续构造周围的表达式或声明：`static constexpr auto normalProgramUnit{`。
- **L77 EN**: Continues the surrounding expression or declaration: `!consumedAllInput >> StartNewSubprogram{} >> programUnit /`.
  **L77 CN**: 继续构造周围的表达式或声明：`!consumedAllInput >> StartNewSubprogram{} >> programUnit /`。
- **L78 EN**: Continues logic associated with callable symbol `recovery`.
  **L78 CN**: 继续与可调用符号 `recovery` 相关的逻辑。
- **L79 EN**: Continues logic associated with callable symbol `MultipleProgramUnitsOnSameLine>`.
  **L79 CN**: 继续与可调用符号 `MultipleProgramUnitsOnSameLine>` 相关的逻辑。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: end of program unit not terminated by new line"_port_en_US,`.
  **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: end of program unit not terminated by new line"_port_en_US,`。

### Lines 81-100

````cpp
                    semicolons >> not(endOfLine))),
            skipToNextLineIfAny)};

static constexpr auto globalCompilerDirective{
    construct<ProgramUnit>(indirect(compilerDirective))};

static constexpr auto globalOpenACCCompilerDirective{
    construct<ProgramUnit>(indirect(skipStuffBeforeStatement >>
        "!$ACC "_sptok >> Parser<OpenACCRoutineConstruct>{} / endOfLine))};

// R501 program -> program-unit [program-unit]...
// This is the top-level production for the Fortran language.
TYPE_PARSER(construct<Program>(skipStuffBeforeStatement >>
    (extension<LanguageFeature::EmptySourceFile>(
         "nonstandard usage: empty source file"_port_en_US,
         consumedAllInput >> pure<std::list<ProgramUnit>>()) ||
        some(skipStuffBeforeStatement >> (globalCompilerDirective ||
                                             globalOpenACCCompilerDirective ||
                                             normalProgramUnit)) /
            skipStuffBeforeStatement)))
````
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `semicolons >> not(endOfLine))),`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`semicolons >> not(endOfLine))),`。
- **L82 EN**: Executes a standalone statement or declaration: `skipToNextLineIfAny)};`.
  **L82 CN**: 执行一条独立语句或声明：`skipToNextLineIfAny)};`。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L84 EN**: Continues the surrounding expression or declaration: `static constexpr auto globalCompilerDirective{`.
  **L84 CN**: 继续构造周围的表达式或声明：`static constexpr auto globalCompilerDirective{`。
- **L85 EN**: Executes a call or declaration centered on `construct<ProgramUnit>`.
  **L85 CN**: 执行以 `construct<ProgramUnit>` 为核心的调用或声明。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L87 EN**: Continues the surrounding expression or declaration: `static constexpr auto globalOpenACCCompilerDirective{`.
  **L87 CN**: 继续构造周围的表达式或声明：`static constexpr auto globalOpenACCCompilerDirective{`。
- **L88 EN**: Continues logic associated with callable symbol `construct<ProgramUnit>`.
  **L88 CN**: 继续与可调用符号 `construct<ProgramUnit>` 相关的逻辑。
- **L89 EN**: Executes a standalone statement or declaration: `"!$ACC "_sptok >> Parser<OpenACCRoutineConstruct>{} / endOfLine))};`.
  **L89 CN**: 执行一条独立语句或声明：`"!$ACC "_sptok >> Parser<OpenACCRoutineConstruct>{} / endOfLine))};`。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Comment explains nearby logic, intent, or metadata: `R501 program -> program-unit [program-unit]...`.
  **L91 CN**: 注释说明附近代码的逻辑、意图或元数据：`R501 program -> program-unit [program-unit]...`。
- **L92 EN**: Comment explains nearby logic, intent, or metadata: `This is the top-level production for the Fortran language.`.
  **L92 CN**: 注释说明附近代码的逻辑、意图或元数据：`This is the top-level production for the Fortran language.`。
- **L93 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L93 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L94 EN**: Continues logic associated with callable symbol `EmptySourceFile>`.
  **L94 CN**: 继续与可调用符号 `EmptySourceFile>` 相关的逻辑。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: empty source file"_port_en_US,`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: empty source file"_port_en_US,`。
- **L96 EN**: Continues logic associated with callable symbol `list<ProgramUnit>>`.
  **L96 CN**: 继续与可调用符号 `list<ProgramUnit>>` 相关的逻辑。
- **L97 EN**: Continues logic associated with callable symbol `some`.
  **L97 CN**: 继续与可调用符号 `some` 相关的逻辑。
- **L98 EN**: Continues the surrounding expression or declaration: `globalOpenACCCompilerDirective ||`.
  **L98 CN**: 继续构造周围的表达式或声明：`globalOpenACCCompilerDirective ||`。
- **L99 EN**: Continues the surrounding expression or declaration: `normalProgramUnit)) /`.
  **L99 CN**: 继续构造周围的表达式或声明：`normalProgramUnit)) /`。
- **L100 EN**: Continues the surrounding expression or declaration: `skipStuffBeforeStatement)))`.
  **L100 CN**: 继续构造周围的表达式或声明：`skipStuffBeforeStatement)))`。

### Lines 101-120

````cpp

// R507 declaration-construct ->
//        specification-construct | data-stmt | format-stmt |
//        entry-stmt | stmt-function-stmt
// N.B. These parsers incorporate recognition of some other statements that
// may have been misplaced in the sequence of statements that are acceptable
// as a specification part in order to improve error recovery.
// Also note that many instances of specification-part in the standard grammar
// are in contexts that impose constraints on the kinds of statements that
// are allowed, and so we have a variant production for declaration-construct
// that implements those constraints.
constexpr auto actionStmtLookAhead{first(actionStmt >> ok,
    // Also accept apparent action statements with errors if they might be
    // first in the execution part
    "ALLOCATE ("_tok, "CALL" >> name >> "("_tok, "GO TO"_tok, "OPEN ("_tok,
    "PRINT"_tok / space / !"("_tok, "READ ("_tok, "WRITE ("_tok)};
constexpr auto execPartLookAhead{first(actionStmtLookAhead,
    openaccConstruct >> ok, openmpExecDirective >> ok, "ASSOCIATE ("_tok,
    "BLOCK"_tok, "SELECT"_tok, "CHANGE TEAM"_sptok, "CRITICAL"_tok, "DO"_tok,
    "IF ("_tok, "WHERE ("_tok, "FORALL ("_tok, "!$CUF"_tok)};
````
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L102 EN**: Comment explains nearby logic, intent, or metadata: `R507 declaration-construct ->`.
  **L102 CN**: 注释说明附近代码的逻辑、意图或元数据：`R507 declaration-construct ->`。
- **L103 EN**: Comment explains nearby logic, intent, or metadata: `specification-construct | data-stmt | format-stmt |`.
  **L103 CN**: 注释说明附近代码的逻辑、意图或元数据：`specification-construct | data-stmt | format-stmt |`。
- **L104 EN**: Comment explains nearby logic, intent, or metadata: `entry-stmt | stmt-function-stmt`.
  **L104 CN**: 注释说明附近代码的逻辑、意图或元数据：`entry-stmt | stmt-function-stmt`。
- **L105 EN**: Comment explains nearby logic, intent, or metadata: `N.B. These parsers incorporate recognition of some other statements that`.
  **L105 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. These parsers incorporate recognition of some other statements that`。
- **L106 EN**: Comment explains nearby logic, intent, or metadata: `may have been misplaced in the sequence of statements that are acceptable`.
  **L106 CN**: 注释说明附近代码的逻辑、意图或元数据：`may have been misplaced in the sequence of statements that are acceptable`。
- **L107 EN**: Comment explains nearby logic, intent, or metadata: `as a specification part in order to improve error recovery.`.
  **L107 CN**: 注释说明附近代码的逻辑、意图或元数据：`as a specification part in order to improve error recovery.`。
- **L108 EN**: Comment explains nearby logic, intent, or metadata: `Also note that many instances of specification-part in the standard grammar`.
  **L108 CN**: 注释说明附近代码的逻辑、意图或元数据：`Also note that many instances of specification-part in the standard grammar`。
- **L109 EN**: Comment explains nearby logic, intent, or metadata: `are in contexts that impose constraints on the kinds of statements that`.
  **L109 CN**: 注释说明附近代码的逻辑、意图或元数据：`are in contexts that impose constraints on the kinds of statements that`。
- **L110 EN**: Comment explains nearby logic, intent, or metadata: `are allowed, and so we have a variant production for declaration-construct`.
  **L110 CN**: 注释说明附近代码的逻辑、意图或元数据：`are allowed, and so we have a variant production for declaration-construct`。
- **L111 EN**: Comment explains nearby logic, intent, or metadata: `that implements those constraints.`.
  **L111 CN**: 注释说明附近代码的逻辑、意图或元数据：`that implements those constraints.`。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `constexpr auto actionStmtLookAhead{first(actionStmt >> ok,`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`constexpr auto actionStmtLookAhead{first(actionStmt >> ok,`。
- **L113 EN**: Comment explains nearby logic, intent, or metadata: `Also accept apparent action statements with errors if they might be`.
  **L113 CN**: 注释说明附近代码的逻辑、意图或元数据：`Also accept apparent action statements with errors if they might be`。
- **L114 EN**: Comment explains nearby logic, intent, or metadata: `first in the execution part`.
  **L114 CN**: 注释说明附近代码的逻辑、意图或元数据：`first in the execution part`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ALLOCATE ("_tok, "CALL" >> name >> "("_tok, "GO TO"_tok, "OPEN ("_tok,`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ALLOCATE ("_tok, "CALL" >> name >> "("_tok, "GO TO"_tok, "OPEN ("_tok,`。
- **L116 EN**: Executes a call or declaration centered on `!"`.
  **L116 CN**: 执行以 `!"` 为核心的调用或声明。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `constexpr auto execPartLookAhead{first(actionStmtLookAhead,`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`constexpr auto execPartLookAhead{first(actionStmtLookAhead,`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `openaccConstruct >> ok, openmpExecDirective >> ok, "ASSOCIATE ("_tok,`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`openaccConstruct >> ok, openmpExecDirective >> ok, "ASSOCIATE ("_tok,`。
- **L119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"BLOCK"_tok, "SELECT"_tok, "CHANGE TEAM"_sptok, "CRITICAL"_tok, "DO"_tok,`.
  **L119 CN**: 继续一个多行参数列表、初始化器或聚合项：`"BLOCK"_tok, "SELECT"_tok, "CHANGE TEAM"_sptok, "CRITICAL"_tok, "DO"_tok,`。
- **L120 EN**: Executes a call or declaration centered on `"IF`.
  **L120 CN**: 执行以 `"IF` 为核心的调用或声明。

### Lines 121-140

````cpp
constexpr auto declErrorRecovery{
    stmtErrorRecoveryStart >> !execPartLookAhead >> skipStmtErrorRecovery};
constexpr auto misplacedSpecificationStmt{Parser<UseStmt>{} >>
        fail<DeclarationConstruct>("misplaced USE statement"_err_en_US) ||
    Parser<ImportStmt>{} >>
        fail<DeclarationConstruct>(
            "IMPORT statements must follow any USE statements and precede all other declarations"_err_en_US) ||
    Parser<ImplicitStmt>{} >>
        fail<DeclarationConstruct>(
            "IMPLICIT statements must follow USE and IMPORT and precede all other declarations"_err_en_US)};

TYPE_CONTEXT_PARSER("declaration construct"_en_US,
    first(construct<DeclarationConstruct>(specificationConstruct),
        construct<DeclarationConstruct>(statement(indirect(dataStmt))),
        construct<DeclarationConstruct>(statement(indirect(formatStmt))),
        construct<DeclarationConstruct>(statement(indirect(entryStmt))),
        construct<DeclarationConstruct>(
            statement(indirect(Parser<StmtFunctionStmt>{}))),
        misplacedSpecificationStmt))

````
- **L121 EN**: Continues the surrounding expression or declaration: `constexpr auto declErrorRecovery{`.
  **L121 CN**: 继续构造周围的表达式或声明：`constexpr auto declErrorRecovery{`。
- **L122 EN**: Executes a standalone statement or declaration: `stmtErrorRecoveryStart >> !execPartLookAhead >> skipStmtErrorRecovery};`.
  **L122 CN**: 执行一条独立语句或声明：`stmtErrorRecoveryStart >> !execPartLookAhead >> skipStmtErrorRecovery};`。
- **L123 EN**: Continues the surrounding expression or declaration: `constexpr auto misplacedSpecificationStmt{Parser<UseStmt>{} >>`.
  **L123 CN**: 继续构造周围的表达式或声明：`constexpr auto misplacedSpecificationStmt{Parser<UseStmt>{} >>`。
- **L124 EN**: Continues logic associated with callable symbol `fail<DeclarationConstruct>`.
  **L124 CN**: 继续与可调用符号 `fail<DeclarationConstruct>` 相关的逻辑。
- **L125 EN**: Continues the surrounding expression or declaration: `Parser<ImportStmt>{} >>`.
  **L125 CN**: 继续构造周围的表达式或声明：`Parser<ImportStmt>{} >>`。
- **L126 EN**: Continues logic associated with callable symbol `fail<DeclarationConstruct>`.
  **L126 CN**: 继续与可调用符号 `fail<DeclarationConstruct>` 相关的逻辑。
- **L127 EN**: Continues the surrounding expression or declaration: `"IMPORT statements must follow any USE statements and precede all other declarations"_err_en_US) ||`.
  **L127 CN**: 继续构造周围的表达式或声明：`"IMPORT statements must follow any USE statements and precede all other declarations"_err_en_US) ||`。
- **L128 EN**: Continues the surrounding expression or declaration: `Parser<ImplicitStmt>{} >>`.
  **L128 CN**: 继续构造周围的表达式或声明：`Parser<ImplicitStmt>{} >>`。
- **L129 EN**: Continues logic associated with callable symbol `fail<DeclarationConstruct>`.
  **L129 CN**: 继续与可调用符号 `fail<DeclarationConstruct>` 相关的逻辑。
- **L130 EN**: Executes a standalone statement or declaration: `"IMPLICIT statements must follow USE and IMPORT and precede all other declarations"_err_en_US)};`.
  **L130 CN**: 执行一条独立语句或声明：`"IMPLICIT statements must follow USE and IMPORT and precede all other declarations"_err_en_US)};`。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("declaration construct"_en_US,`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("declaration construct"_en_US,`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first(construct<DeclarationConstruct>(specificationConstruct),`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`first(construct<DeclarationConstruct>(specificationConstruct),`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DeclarationConstruct>(statement(indirect(dataStmt))),`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DeclarationConstruct>(statement(indirect(dataStmt))),`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DeclarationConstruct>(statement(indirect(formatStmt))),`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DeclarationConstruct>(statement(indirect(formatStmt))),`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DeclarationConstruct>(statement(indirect(entryStmt))),`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DeclarationConstruct>(statement(indirect(entryStmt))),`。
- **L137 EN**: Continues logic associated with callable symbol `construct<DeclarationConstruct>`.
  **L137 CN**: 继续与可调用符号 `construct<DeclarationConstruct>` 相关的逻辑。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `statement(indirect(Parser<StmtFunctionStmt>{}))),`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`statement(indirect(Parser<StmtFunctionStmt>{}))),`。
- **L139 EN**: Continues the surrounding expression or declaration: `misplacedSpecificationStmt))`.
  **L139 CN**: 继续构造周围的表达式或声明：`misplacedSpecificationStmt))`。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 141-160

````cpp
constexpr auto recoveredDeclarationConstruct{
    recovery(withMessage("expected declaration construct"_err_en_US,
                 declarationConstruct),
        construct<DeclarationConstruct>(declErrorRecovery))};

// R504 specification-part ->
//         [use-stmt]... [import-stmt]... [implicit-part]
//         [declaration-construct]...
TYPE_CONTEXT_PARSER("specification part"_en_US,
    construct<SpecificationPart>(many(openaccDeclarativeConstruct),
        many(openmpDeclarativeConstruct), many(indirect(compilerDirective)),
        many(statement(indirect(Parser<UseStmt>{}))),
        many(unambiguousStatement(indirect(Parser<ImportStmt>{}))),
        implicitPart, many(recoveredDeclarationConstruct)))

// R507 variant of declaration-construct for use in limitedSpecificationPart.
constexpr auto invalidDeclarationStmt{formatStmt >>
        fail<DeclarationConstruct>(
            "FORMAT statements are not permitted in this specification part"_err_en_US) ||
    entryStmt >>
````
- **L141 EN**: Continues the surrounding expression or declaration: `constexpr auto recoveredDeclarationConstruct{`.
  **L141 CN**: 继续构造周围的表达式或声明：`constexpr auto recoveredDeclarationConstruct{`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `recovery(withMessage("expected declaration construct"_err_en_US,`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`recovery(withMessage("expected declaration construct"_err_en_US,`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `declarationConstruct),`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`declarationConstruct),`。
- **L144 EN**: Executes a call or declaration centered on `construct<DeclarationConstruct>`.
  **L144 CN**: 执行以 `construct<DeclarationConstruct>` 为核心的调用或声明。
- **L145 EN**: Blank line separating nearby declarations or logic blocks.
  **L145 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L146 EN**: Comment explains nearby logic, intent, or metadata: `R504 specification-part ->`.
  **L146 CN**: 注释说明附近代码的逻辑、意图或元数据：`R504 specification-part ->`。
- **L147 EN**: Comment explains nearby logic, intent, or metadata: `[use-stmt]... [import-stmt]... [implicit-part]`.
  **L147 CN**: 注释说明附近代码的逻辑、意图或元数据：`[use-stmt]... [import-stmt]... [implicit-part]`。
- **L148 EN**: Comment explains nearby logic, intent, or metadata: `[declaration-construct]...`.
  **L148 CN**: 注释说明附近代码的逻辑、意图或元数据：`[declaration-construct]...`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("specification part"_en_US,`.
  **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("specification part"_en_US,`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SpecificationPart>(many(openaccDeclarativeConstruct),`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SpecificationPart>(many(openaccDeclarativeConstruct),`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(openmpDeclarativeConstruct), many(indirect(compilerDirective)),`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(openmpDeclarativeConstruct), many(indirect(compilerDirective)),`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(statement(indirect(Parser<UseStmt>{}))),`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(statement(indirect(Parser<UseStmt>{}))),`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(unambiguousStatement(indirect(Parser<ImportStmt>{}))),`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(unambiguousStatement(indirect(Parser<ImportStmt>{}))),`。
- **L154 EN**: Continues logic associated with callable symbol `many`.
  **L154 CN**: 继续与可调用符号 `many` 相关的逻辑。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L156 EN**: Comment explains nearby logic, intent, or metadata: `R507 variant of declaration-construct for use in limitedSpecificationPart.`.
  **L156 CN**: 注释说明附近代码的逻辑、意图或元数据：`R507 variant of declaration-construct for use in limitedSpecificationPart.`。
- **L157 EN**: Continues the surrounding expression or declaration: `constexpr auto invalidDeclarationStmt{formatStmt >>`.
  **L157 CN**: 继续构造周围的表达式或声明：`constexpr auto invalidDeclarationStmt{formatStmt >>`。
- **L158 EN**: Continues logic associated with callable symbol `fail<DeclarationConstruct>`.
  **L158 CN**: 继续与可调用符号 `fail<DeclarationConstruct>` 相关的逻辑。
- **L159 EN**: Continues the surrounding expression or declaration: `"FORMAT statements are not permitted in this specification part"_err_en_US) ||`.
  **L159 CN**: 继续构造周围的表达式或声明：`"FORMAT statements are not permitted in this specification part"_err_en_US) ||`。
- **L160 EN**: Continues the surrounding expression or declaration: `entryStmt >>`.
  **L160 CN**: 继续构造周围的表达式或声明：`entryStmt >>`。

### Lines 161-180

````cpp
        fail<DeclarationConstruct>(
            "ENTRY statements are not permitted in this specification part"_err_en_US)};

constexpr auto limitedDeclarationConstruct{recovery(
    withMessage("expected declaration construct"_err_en_US,
        inContext("declaration construct"_en_US,
            first(construct<DeclarationConstruct>(specificationConstruct),
                construct<DeclarationConstruct>(statement(indirect(dataStmt))),
                misplacedSpecificationStmt, invalidDeclarationStmt))),
    construct<DeclarationConstruct>(
        stmtErrorRecoveryStart >> skipStmtErrorRecovery))};

// R504 variant for many contexts (modules, submodules, BLOCK DATA subprograms,
// and interfaces) which have constraints on their specification parts that
// preclude FORMAT, ENTRY, and statement functions, and benefit from
// specialized error recovery in the event of a spurious executable
// statement.
constexpr auto limitedSpecificationPart{inContext("specification part"_en_US,
    construct<SpecificationPart>(many(openaccDeclarativeConstruct),
        many(openmpDeclarativeConstruct), many(indirect(compilerDirective)),
````
- **L161 EN**: Continues logic associated with callable symbol `fail<DeclarationConstruct>`.
  **L161 CN**: 继续与可调用符号 `fail<DeclarationConstruct>` 相关的逻辑。
- **L162 EN**: Executes a standalone statement or declaration: `"ENTRY statements are not permitted in this specification part"_err_en_US)};`.
  **L162 CN**: 执行一条独立语句或声明：`"ENTRY statements are not permitted in this specification part"_err_en_US)};`。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Continues logic associated with callable symbol `recovery`.
  **L164 CN**: 继续与可调用符号 `recovery` 相关的逻辑。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `withMessage("expected declaration construct"_err_en_US,`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`withMessage("expected declaration construct"_err_en_US,`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `inContext("declaration construct"_en_US,`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`inContext("declaration construct"_en_US,`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first(construct<DeclarationConstruct>(specificationConstruct),`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`first(construct<DeclarationConstruct>(specificationConstruct),`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<DeclarationConstruct>(statement(indirect(dataStmt))),`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<DeclarationConstruct>(statement(indirect(dataStmt))),`。
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `misplacedSpecificationStmt, invalidDeclarationStmt))),`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`misplacedSpecificationStmt, invalidDeclarationStmt))),`。
- **L170 EN**: Continues logic associated with callable symbol `construct<DeclarationConstruct>`.
  **L170 CN**: 继续与可调用符号 `construct<DeclarationConstruct>` 相关的逻辑。
- **L171 EN**: Executes a standalone statement or declaration: `stmtErrorRecoveryStart >> skipStmtErrorRecovery))};`.
  **L171 CN**: 执行一条独立语句或声明：`stmtErrorRecoveryStart >> skipStmtErrorRecovery))};`。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L173 EN**: Comment explains nearby logic, intent, or metadata: `R504 variant for many contexts (modules, submodules, BLOCK DATA subprograms,`.
  **L173 CN**: 注释说明附近代码的逻辑、意图或元数据：`R504 variant for many contexts (modules, submodules, BLOCK DATA subprograms,`。
- **L174 EN**: Comment explains nearby logic, intent, or metadata: `and interfaces) which have constraints on their specification parts that`.
  **L174 CN**: 注释说明附近代码的逻辑、意图或元数据：`and interfaces) which have constraints on their specification parts that`。
- **L175 EN**: Comment explains nearby logic, intent, or metadata: `preclude FORMAT, ENTRY, and statement functions, and benefit from`.
  **L175 CN**: 注释说明附近代码的逻辑、意图或元数据：`preclude FORMAT, ENTRY, and statement functions, and benefit from`。
- **L176 EN**: Comment explains nearby logic, intent, or metadata: `specialized error recovery in the event of a spurious executable`.
  **L176 CN**: 注释说明附近代码的逻辑、意图或元数据：`specialized error recovery in the event of a spurious executable`。
- **L177 EN**: Comment explains nearby logic, intent, or metadata: `statement.`.
  **L177 CN**: 注释说明附近代码的逻辑、意图或元数据：`statement.`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `constexpr auto limitedSpecificationPart{inContext("specification part"_en_US,`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`constexpr auto limitedSpecificationPart{inContext("specification part"_en_US,`。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SpecificationPart>(many(openaccDeclarativeConstruct),`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SpecificationPart>(many(openaccDeclarativeConstruct),`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(openmpDeclarativeConstruct), many(indirect(compilerDirective)),`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(openmpDeclarativeConstruct), many(indirect(compilerDirective)),`。

### Lines 181-200

````cpp
        many(statement(indirect(Parser<UseStmt>{}))),
        many(unambiguousStatement(indirect(Parser<ImportStmt>{}))),
        implicitPart, many(limitedDeclarationConstruct)))};

// R508 specification-construct ->
//        derived-type-def | enum-def | generic-stmt | interface-block |
//        parameter-stmt | procedure-declaration-stmt |
//        other-specification-stmt | type-declaration-stmt
TYPE_CONTEXT_PARSER("specification construct"_en_US,
    first(construct<SpecificationConstruct>(indirect(Parser<DerivedTypeDef>{})),
        construct<SpecificationConstruct>(indirect(Parser<EnumDef>{})),
        construct<SpecificationConstruct>(
            statement(indirect(Parser<GenericStmt>{}))),
        construct<SpecificationConstruct>(indirect(interfaceBlock)),
        construct<SpecificationConstruct>(statement(indirect(parameterStmt))),
        construct<SpecificationConstruct>(
            statement(indirect(oldParameterStmt))),
        construct<SpecificationConstruct>(
            statement(indirect(Parser<ProcedureDeclarationStmt>{}))),
        construct<SpecificationConstruct>(
````
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(statement(indirect(Parser<UseStmt>{}))),`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(statement(indirect(Parser<UseStmt>{}))),`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(unambiguousStatement(indirect(Parser<ImportStmt>{}))),`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(unambiguousStatement(indirect(Parser<ImportStmt>{}))),`。
- **L183 EN**: Executes a call or declaration centered on `many`.
  **L183 CN**: 执行以 `many` 为核心的调用或声明。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L185 EN**: Comment explains nearby logic, intent, or metadata: `R508 specification-construct ->`.
  **L185 CN**: 注释说明附近代码的逻辑、意图或元数据：`R508 specification-construct ->`。
- **L186 EN**: Comment explains nearby logic, intent, or metadata: `derived-type-def | enum-def | generic-stmt | interface-block |`.
  **L186 CN**: 注释说明附近代码的逻辑、意图或元数据：`derived-type-def | enum-def | generic-stmt | interface-block |`。
- **L187 EN**: Comment explains nearby logic, intent, or metadata: `parameter-stmt | procedure-declaration-stmt |`.
  **L187 CN**: 注释说明附近代码的逻辑、意图或元数据：`parameter-stmt | procedure-declaration-stmt |`。
- **L188 EN**: Comment explains nearby logic, intent, or metadata: `other-specification-stmt | type-declaration-stmt`.
  **L188 CN**: 注释说明附近代码的逻辑、意图或元数据：`other-specification-stmt | type-declaration-stmt`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("specification construct"_en_US,`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("specification construct"_en_US,`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first(construct<SpecificationConstruct>(indirect(Parser<DerivedTypeDef>{})),`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`first(construct<SpecificationConstruct>(indirect(Parser<DerivedTypeDef>{})),`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SpecificationConstruct>(indirect(Parser<EnumDef>{})),`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SpecificationConstruct>(indirect(Parser<EnumDef>{})),`。
- **L192 EN**: Continues logic associated with callable symbol `construct<SpecificationConstruct>`.
  **L192 CN**: 继续与可调用符号 `construct<SpecificationConstruct>` 相关的逻辑。
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `statement(indirect(Parser<GenericStmt>{}))),`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`statement(indirect(Parser<GenericStmt>{}))),`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SpecificationConstruct>(indirect(interfaceBlock)),`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SpecificationConstruct>(indirect(interfaceBlock)),`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SpecificationConstruct>(statement(indirect(parameterStmt))),`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SpecificationConstruct>(statement(indirect(parameterStmt))),`。
- **L196 EN**: Continues logic associated with callable symbol `construct<SpecificationConstruct>`.
  **L196 CN**: 继续与可调用符号 `construct<SpecificationConstruct>` 相关的逻辑。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `statement(indirect(oldParameterStmt))),`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`statement(indirect(oldParameterStmt))),`。
- **L198 EN**: Continues logic associated with callable symbol `construct<SpecificationConstruct>`.
  **L198 CN**: 继续与可调用符号 `construct<SpecificationConstruct>` 相关的逻辑。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `statement(indirect(Parser<ProcedureDeclarationStmt>{}))),`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`statement(indirect(Parser<ProcedureDeclarationStmt>{}))),`。
- **L200 EN**: Continues logic associated with callable symbol `construct<SpecificationConstruct>`.
  **L200 CN**: 继续与可调用符号 `construct<SpecificationConstruct>` 相关的逻辑。

### Lines 201-220

````cpp
            statement(Parser<OtherSpecificationStmt>{})),
        construct<SpecificationConstruct>(
            statement(indirect(typeDeclarationStmt))),
        construct<SpecificationConstruct>(indirect(Parser<StructureDef>{})),
        construct<SpecificationConstruct>(
            indirect(openaccDeclarativeConstruct)),
        construct<SpecificationConstruct>(indirect(openmpDeclarativeConstruct)),
        construct<SpecificationConstruct>(
            indirect(openmpMisplacedEndDirective)),
        construct<SpecificationConstruct>(indirect(openmpInvalidDirective)),
        construct<SpecificationConstruct>(indirect(compilerDirective))))

// R513 other-specification-stmt ->
//        access-stmt | allocatable-stmt | asynchronous-stmt | bind-stmt |
//        codimension-stmt | contiguous-stmt | dimension-stmt | external-stmt |
//        intent-stmt | intrinsic-stmt | namelist-stmt | optional-stmt |
//        pointer-stmt | protected-stmt | save-stmt | target-stmt |
//        volatile-stmt | value-stmt | common-stmt | equivalence-stmt |
// (CUDA) CUDA-attributes-stmt
TYPE_PARSER(first(
````
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `statement(Parser<OtherSpecificationStmt>{})),`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`statement(Parser<OtherSpecificationStmt>{})),`。
- **L202 EN**: Continues logic associated with callable symbol `construct<SpecificationConstruct>`.
  **L202 CN**: 继续与可调用符号 `construct<SpecificationConstruct>` 相关的逻辑。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `statement(indirect(typeDeclarationStmt))),`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`statement(indirect(typeDeclarationStmt))),`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SpecificationConstruct>(indirect(Parser<StructureDef>{})),`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SpecificationConstruct>(indirect(Parser<StructureDef>{})),`。
- **L205 EN**: Continues logic associated with callable symbol `construct<SpecificationConstruct>`.
  **L205 CN**: 继续与可调用符号 `construct<SpecificationConstruct>` 相关的逻辑。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `indirect(openaccDeclarativeConstruct)),`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`indirect(openaccDeclarativeConstruct)),`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SpecificationConstruct>(indirect(openmpDeclarativeConstruct)),`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SpecificationConstruct>(indirect(openmpDeclarativeConstruct)),`。
- **L208 EN**: Continues logic associated with callable symbol `construct<SpecificationConstruct>`.
  **L208 CN**: 继续与可调用符号 `construct<SpecificationConstruct>` 相关的逻辑。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `indirect(openmpMisplacedEndDirective)),`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`indirect(openmpMisplacedEndDirective)),`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SpecificationConstruct>(indirect(openmpInvalidDirective)),`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SpecificationConstruct>(indirect(openmpInvalidDirective)),`。
- **L211 EN**: Continues logic associated with callable symbol `construct<SpecificationConstruct>`.
  **L211 CN**: 继续与可调用符号 `construct<SpecificationConstruct>` 相关的逻辑。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L213 EN**: Comment explains nearby logic, intent, or metadata: `R513 other-specification-stmt ->`.
  **L213 CN**: 注释说明附近代码的逻辑、意图或元数据：`R513 other-specification-stmt ->`。
- **L214 EN**: Comment explains nearby logic, intent, or metadata: `access-stmt | allocatable-stmt | asynchronous-stmt | bind-stmt |`.
  **L214 CN**: 注释说明附近代码的逻辑、意图或元数据：`access-stmt | allocatable-stmt | asynchronous-stmt | bind-stmt |`。
- **L215 EN**: Comment explains nearby logic, intent, or metadata: `codimension-stmt | contiguous-stmt | dimension-stmt | external-stmt |`.
  **L215 CN**: 注释说明附近代码的逻辑、意图或元数据：`codimension-stmt | contiguous-stmt | dimension-stmt | external-stmt |`。
- **L216 EN**: Comment explains nearby logic, intent, or metadata: `intent-stmt | intrinsic-stmt | namelist-stmt | optional-stmt |`.
  **L216 CN**: 注释说明附近代码的逻辑、意图或元数据：`intent-stmt | intrinsic-stmt | namelist-stmt | optional-stmt |`。
- **L217 EN**: Comment explains nearby logic, intent, or metadata: `pointer-stmt | protected-stmt | save-stmt | target-stmt |`.
  **L217 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointer-stmt | protected-stmt | save-stmt | target-stmt |`。
- **L218 EN**: Comment explains nearby logic, intent, or metadata: `volatile-stmt | value-stmt | common-stmt | equivalence-stmt |`.
  **L218 CN**: 注释说明附近代码的逻辑、意图或元数据：`volatile-stmt | value-stmt | common-stmt | equivalence-stmt |`。
- **L219 EN**: Comment explains nearby logic, intent, or metadata: `(CUDA) CUDA-attributes-stmt`.
  **L219 CN**: 注释说明附近代码的逻辑、意图或元数据：`(CUDA) CUDA-attributes-stmt`。
- **L220 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L220 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。

### Lines 221-240

````cpp
    construct<OtherSpecificationStmt>(indirect(Parser<AccessStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<AllocatableStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<AsynchronousStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<BindStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<CodimensionStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<ContiguousStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<DimensionStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<ExternalStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<IntentStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<IntrinsicStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<NamelistStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<OptionalStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<PointerStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<ProtectedStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<SaveStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<TargetStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<ValueStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<VolatileStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<CommonStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<EquivalenceStmt>{})),
````
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<AccessStmt>{})),`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<AccessStmt>{})),`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<AllocatableStmt>{})),`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<AllocatableStmt>{})),`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<AsynchronousStmt>{})),`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<AsynchronousStmt>{})),`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<BindStmt>{})),`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<BindStmt>{})),`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<CodimensionStmt>{})),`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<CodimensionStmt>{})),`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<ContiguousStmt>{})),`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<ContiguousStmt>{})),`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<DimensionStmt>{})),`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<DimensionStmt>{})),`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<ExternalStmt>{})),`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<ExternalStmt>{})),`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<IntentStmt>{})),`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<IntentStmt>{})),`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<IntrinsicStmt>{})),`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<IntrinsicStmt>{})),`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<NamelistStmt>{})),`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<NamelistStmt>{})),`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<OptionalStmt>{})),`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<OptionalStmt>{})),`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<PointerStmt>{})),`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<PointerStmt>{})),`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<ProtectedStmt>{})),`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<ProtectedStmt>{})),`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<SaveStmt>{})),`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<SaveStmt>{})),`。
- **L236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<TargetStmt>{})),`.
  **L236 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<TargetStmt>{})),`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<ValueStmt>{})),`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<ValueStmt>{})),`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<VolatileStmt>{})),`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<VolatileStmt>{})),`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<CommonStmt>{})),`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<CommonStmt>{})),`。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<EquivalenceStmt>{})),`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<EquivalenceStmt>{})),`。

### Lines 241-260

````cpp
    construct<OtherSpecificationStmt>(indirect(Parser<BasedPointerStmt>{})),
    construct<OtherSpecificationStmt>(indirect(Parser<CUDAAttributesStmt>{}))))

// R1401 main-program ->
//         [program-stmt] [specification-part] [execution-part]
//         [internal-subprogram-part] end-program-stmt
TYPE_CONTEXT_PARSER("main program"_en_US,
    construct<MainProgram>(maybe(statement(Parser<ProgramStmt>{})),
        specificationPart, executionPart, maybe(internalSubprogramPart),
        unterminatedStatement(Parser<EndProgramStmt>{})))

// R1402 program-stmt -> PROGRAM program-name
// PGI allows empty parentheses after the name.
TYPE_CONTEXT_PARSER("PROGRAM statement"_en_US,
    construct<ProgramStmt>("PROGRAM" >> name /
            maybe(extension<LanguageFeature::ProgramParentheses>(
                "nonstandard usage: parentheses in PROGRAM statement"_port_en_US,
                parenthesized(ok)))))

// R1403 end-program-stmt -> END [PROGRAM [program-name]]
````
- **L241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<OtherSpecificationStmt>(indirect(Parser<BasedPointerStmt>{})),`.
  **L241 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<OtherSpecificationStmt>(indirect(Parser<BasedPointerStmt>{})),`。
- **L242 EN**: Continues logic associated with callable symbol `construct<OtherSpecificationStmt>`.
  **L242 CN**: 继续与可调用符号 `construct<OtherSpecificationStmt>` 相关的逻辑。
- **L243 EN**: Blank line separating nearby declarations or logic blocks.
  **L243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L244 EN**: Comment explains nearby logic, intent, or metadata: `R1401 main-program ->`.
  **L244 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1401 main-program ->`。
- **L245 EN**: Comment explains nearby logic, intent, or metadata: `[program-stmt] [specification-part] [execution-part]`.
  **L245 CN**: 注释说明附近代码的逻辑、意图或元数据：`[program-stmt] [specification-part] [execution-part]`。
- **L246 EN**: Comment explains nearby logic, intent, or metadata: `[internal-subprogram-part] end-program-stmt`.
  **L246 CN**: 注释说明附近代码的逻辑、意图或元数据：`[internal-subprogram-part] end-program-stmt`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("main program"_en_US,`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("main program"_en_US,`。
- **L248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<MainProgram>(maybe(statement(Parser<ProgramStmt>{})),`.
  **L248 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<MainProgram>(maybe(statement(Parser<ProgramStmt>{})),`。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specificationPart, executionPart, maybe(internalSubprogramPart),`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`specificationPart, executionPart, maybe(internalSubprogramPart),`。
- **L250 EN**: Continues logic associated with callable symbol `unterminatedStatement`.
  **L250 CN**: 继续与可调用符号 `unterminatedStatement` 相关的逻辑。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L252 EN**: Comment explains nearby logic, intent, or metadata: `R1402 program-stmt -> PROGRAM program-name`.
  **L252 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1402 program-stmt -> PROGRAM program-name`。
- **L253 EN**: Comment explains nearby logic, intent, or metadata: `PGI allows empty parentheses after the name.`.
  **L253 CN**: 注释说明附近代码的逻辑、意图或元数据：`PGI allows empty parentheses after the name.`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("PROGRAM statement"_en_US,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("PROGRAM statement"_en_US,`。
- **L255 EN**: Continues logic associated with callable symbol `construct<ProgramStmt>`.
  **L255 CN**: 继续与可调用符号 `construct<ProgramStmt>` 相关的逻辑。
- **L256 EN**: Continues logic associated with callable symbol `maybe`.
  **L256 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: parentheses in PROGRAM statement"_port_en_US,`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: parentheses in PROGRAM statement"_port_en_US,`。
- **L258 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L258 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L260 EN**: Comment explains nearby logic, intent, or metadata: `R1403 end-program-stmt -> END [PROGRAM [program-name]]`.
  **L260 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1403 end-program-stmt -> END [PROGRAM [program-name]]`。

### Lines 261-280

````cpp
TYPE_CONTEXT_PARSER("END PROGRAM statement"_en_US,
    construct<EndProgramStmt>(
        recovery("END " >> defaulted("PROGRAM" >> maybe(name)) / atEndOfStmt,
            progUnitEndStmtErrorRecovery)))

// R1404 module ->
//         module-stmt [specification-part] [module-subprogram-part]
//         end-module-stmt
TYPE_CONTEXT_PARSER("module"_en_US,
    construct<Module>(statement(Parser<ModuleStmt>{}), limitedSpecificationPart,
        maybe(Parser<ModuleSubprogramPart>{}),
        unterminatedStatement(Parser<EndModuleStmt>{})))

// R1405 module-stmt -> MODULE module-name
TYPE_CONTEXT_PARSER(
    "MODULE statement"_en_US, construct<ModuleStmt>("MODULE" >> name))

// R1406 end-module-stmt -> END [MODULE [module-name]]
TYPE_CONTEXT_PARSER("END MODULE statement"_en_US,
    construct<EndModuleStmt>(
````
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("END PROGRAM statement"_en_US,`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("END PROGRAM statement"_en_US,`。
- **L262 EN**: Continues logic associated with callable symbol `construct<EndProgramStmt>`.
  **L262 CN**: 继续与可调用符号 `construct<EndProgramStmt>` 相关的逻辑。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `recovery("END " >> defaulted("PROGRAM" >> maybe(name)) / atEndOfStmt,`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`recovery("END " >> defaulted("PROGRAM" >> maybe(name)) / atEndOfStmt,`。
- **L264 EN**: Continues the surrounding expression or declaration: `progUnitEndStmtErrorRecovery)))`.
  **L264 CN**: 继续构造周围的表达式或声明：`progUnitEndStmtErrorRecovery)))`。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Comment explains nearby logic, intent, or metadata: `R1404 module ->`.
  **L266 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1404 module ->`。
- **L267 EN**: Comment explains nearby logic, intent, or metadata: `module-stmt [specification-part] [module-subprogram-part]`.
  **L267 CN**: 注释说明附近代码的逻辑、意图或元数据：`module-stmt [specification-part] [module-subprogram-part]`。
- **L268 EN**: Comment explains nearby logic, intent, or metadata: `end-module-stmt`.
  **L268 CN**: 注释说明附近代码的逻辑、意图或元数据：`end-module-stmt`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("module"_en_US,`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("module"_en_US,`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<Module>(statement(Parser<ModuleStmt>{}), limitedSpecificationPart,`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<Module>(statement(Parser<ModuleStmt>{}), limitedSpecificationPart,`。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `maybe(Parser<ModuleSubprogramPart>{}),`.
  **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`maybe(Parser<ModuleSubprogramPart>{}),`。
- **L272 EN**: Continues logic associated with callable symbol `unterminatedStatement`.
  **L272 CN**: 继续与可调用符号 `unterminatedStatement` 相关的逻辑。
- **L273 EN**: Blank line separating nearby declarations or logic blocks.
  **L273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L274 EN**: Comment explains nearby logic, intent, or metadata: `R1405 module-stmt -> MODULE module-name`.
  **L274 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1405 module-stmt -> MODULE module-name`。
- **L275 EN**: Continues logic associated with callable symbol `TYPE_CONTEXT_PARSER`.
  **L275 CN**: 继续与可调用符号 `TYPE_CONTEXT_PARSER` 相关的逻辑。
- **L276 EN**: Continues logic associated with callable symbol `construct<ModuleStmt>`.
  **L276 CN**: 继续与可调用符号 `construct<ModuleStmt>` 相关的逻辑。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L278 EN**: Comment explains nearby logic, intent, or metadata: `R1406 end-module-stmt -> END [MODULE [module-name]]`.
  **L278 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1406 end-module-stmt -> END [MODULE [module-name]]`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("END MODULE statement"_en_US,`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("END MODULE statement"_en_US,`。
- **L280 EN**: Continues logic associated with callable symbol `construct<EndModuleStmt>`.
  **L280 CN**: 继续与可调用符号 `construct<EndModuleStmt>` 相关的逻辑。

### Lines 281-300

````cpp
        recovery("END " >> defaulted("MODULE" >> maybe(name)) / atEndOfStmt,
            progUnitEndStmtErrorRecovery)))

// R1407 module-subprogram-part -> contains-stmt [module-subprogram]...
TYPE_CONTEXT_PARSER("module subprogram part"_en_US,
    construct<ModuleSubprogramPart>(statement(containsStmt),
        many(StartNewSubprogram{} >> Parser<ModuleSubprogram>{})))

// R1408 module-subprogram ->
//         function-subprogram | subroutine-subprogram |
//         separate-module-subprogram
TYPE_PARSER(construct<ModuleSubprogram>(indirect(functionSubprogram)) ||
    construct<ModuleSubprogram>(indirect(subroutineSubprogram)) ||
    construct<ModuleSubprogram>(indirect(Parser<SeparateModuleSubprogram>{})) ||
    construct<ModuleSubprogram>(indirect(compilerDirective)))

// R1410 module-nature -> INTRINSIC | NON_INTRINSIC
constexpr auto moduleNature{
    "INTRINSIC" >> pure(UseStmt::ModuleNature::Intrinsic) ||
    "NON_INTRINSIC" >> pure(UseStmt::ModuleNature::Non_Intrinsic)};
````
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `recovery("END " >> defaulted("MODULE" >> maybe(name)) / atEndOfStmt,`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`recovery("END " >> defaulted("MODULE" >> maybe(name)) / atEndOfStmt,`。
- **L282 EN**: Continues the surrounding expression or declaration: `progUnitEndStmtErrorRecovery)))`.
  **L282 CN**: 继续构造周围的表达式或声明：`progUnitEndStmtErrorRecovery)))`。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L284 EN**: Comment explains nearby logic, intent, or metadata: `R1407 module-subprogram-part -> contains-stmt [module-subprogram]...`.
  **L284 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1407 module-subprogram-part -> contains-stmt [module-subprogram]...`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("module subprogram part"_en_US,`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("module subprogram part"_en_US,`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ModuleSubprogramPart>(statement(containsStmt),`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ModuleSubprogramPart>(statement(containsStmt),`。
- **L287 EN**: Continues logic associated with callable symbol `many`.
  **L287 CN**: 继续与可调用符号 `many` 相关的逻辑。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L289 EN**: Comment explains nearby logic, intent, or metadata: `R1408 module-subprogram ->`.
  **L289 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1408 module-subprogram ->`。
- **L290 EN**: Comment explains nearby logic, intent, or metadata: `function-subprogram | subroutine-subprogram |`.
  **L290 CN**: 注释说明附近代码的逻辑、意图或元数据：`function-subprogram | subroutine-subprogram |`。
- **L291 EN**: Comment explains nearby logic, intent, or metadata: `separate-module-subprogram`.
  **L291 CN**: 注释说明附近代码的逻辑、意图或元数据：`separate-module-subprogram`。
- **L292 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L292 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L293 EN**: Continues logic associated with callable symbol `construct<ModuleSubprogram>`.
  **L293 CN**: 继续与可调用符号 `construct<ModuleSubprogram>` 相关的逻辑。
- **L294 EN**: Continues logic associated with callable symbol `construct<ModuleSubprogram>`.
  **L294 CN**: 继续与可调用符号 `construct<ModuleSubprogram>` 相关的逻辑。
- **L295 EN**: Continues logic associated with callable symbol `construct<ModuleSubprogram>`.
  **L295 CN**: 继续与可调用符号 `construct<ModuleSubprogram>` 相关的逻辑。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L297 EN**: Comment explains nearby logic, intent, or metadata: `R1410 module-nature -> INTRINSIC | NON_INTRINSIC`.
  **L297 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1410 module-nature -> INTRINSIC | NON_INTRINSIC`。
- **L298 EN**: Continues the surrounding expression or declaration: `constexpr auto moduleNature{`.
  **L298 CN**: 继续构造周围的表达式或声明：`constexpr auto moduleNature{`。
- **L299 EN**: Continues logic associated with callable symbol `pure`.
  **L299 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L300 EN**: Executes a call or declaration centered on `pure`.
  **L300 CN**: 执行以 `pure` 为核心的调用或声明。

### Lines 301-320

````cpp

// R1409 use-stmt ->
//         USE [[, module-nature] ::] module-name [, rename-list] |
//         USE [[, module-nature] ::] module-name , ONLY : [only-list]
// N.B. Lookahead to the end of the statement is necessary to resolve
// ambiguity with assignments and statement function definitions that
// begin with the letters "USE".
TYPE_PARSER(construct<UseStmt>("USE" >> optionalBeforeColons(moduleNature),
                name, ", ONLY :" >> optionalList(Parser<Only>{})) ||
    construct<UseStmt>("USE" >> optionalBeforeColons(moduleNature), name,
        defaulted("," >>
            nonemptyList("expected renamings"_err_en_US, Parser<Rename>{})) /
            lookAhead(endOfStmt)))

// R1411 rename ->
//         local-name => use-name |
//         OPERATOR ( local-defined-operator ) =>
//           OPERATOR ( use-defined-operator )
TYPE_PARSER(construct<Rename>("OPERATOR (" >>
                construct<Rename::Operators>(
````
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L302 EN**: Comment explains nearby logic, intent, or metadata: `R1409 use-stmt ->`.
  **L302 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1409 use-stmt ->`。
- **L303 EN**: Comment explains nearby logic, intent, or metadata: `USE [[, module-nature] ::] module-name [, rename-list] |`.
  **L303 CN**: 注释说明附近代码的逻辑、意图或元数据：`USE [[, module-nature] ::] module-name [, rename-list] |`。
- **L304 EN**: Comment explains nearby logic, intent, or metadata: `USE [[, module-nature] ::] module-name , ONLY : [only-list]`.
  **L304 CN**: 注释说明附近代码的逻辑、意图或元数据：`USE [[, module-nature] ::] module-name , ONLY : [only-list]`。
- **L305 EN**: Comment explains nearby logic, intent, or metadata: `N.B. Lookahead to the end of the statement is necessary to resolve`.
  **L305 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. Lookahead to the end of the statement is necessary to resolve`。
- **L306 EN**: Comment explains nearby logic, intent, or metadata: `ambiguity with assignments and statement function definitions that`.
  **L306 CN**: 注释说明附近代码的逻辑、意图或元数据：`ambiguity with assignments and statement function definitions that`。
- **L307 EN**: Comment explains nearby logic, intent, or metadata: `begin with the letters "USE".`.
  **L307 CN**: 注释说明附近代码的逻辑、意图或元数据：`begin with the letters "USE".`。
- **L308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<UseStmt>("USE" >> optionalBeforeColons(moduleNature),`.
  **L308 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<UseStmt>("USE" >> optionalBeforeColons(moduleNature),`。
- **L309 EN**: Continues logic associated with callable symbol `optionalList`.
  **L309 CN**: 继续与可调用符号 `optionalList` 相关的逻辑。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<UseStmt>("USE" >> optionalBeforeColons(moduleNature), name,`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<UseStmt>("USE" >> optionalBeforeColons(moduleNature), name,`。
- **L311 EN**: Continues logic associated with callable symbol `defaulted`.
  **L311 CN**: 继续与可调用符号 `defaulted` 相关的逻辑。
- **L312 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L312 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L313 EN**: Continues logic associated with callable symbol `lookAhead`.
  **L313 CN**: 继续与可调用符号 `lookAhead` 相关的逻辑。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L315 EN**: Comment explains nearby logic, intent, or metadata: `R1411 rename ->`.
  **L315 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1411 rename ->`。
- **L316 EN**: Comment explains nearby logic, intent, or metadata: `local-name => use-name |`.
  **L316 CN**: 注释说明附近代码的逻辑、意图或元数据：`local-name => use-name |`。
- **L317 EN**: Comment explains nearby logic, intent, or metadata: `OPERATOR ( local-defined-operator ) =>`.
  **L317 CN**: 注释说明附近代码的逻辑、意图或元数据：`OPERATOR ( local-defined-operator ) =>`。
- **L318 EN**: Comment explains nearby logic, intent, or metadata: `OPERATOR ( use-defined-operator )`.
  **L318 CN**: 注释说明附近代码的逻辑、意图或元数据：`OPERATOR ( use-defined-operator )`。
- **L319 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L319 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L320 EN**: Continues logic associated with callable symbol `Operators>`.
  **L320 CN**: 继续与可调用符号 `Operators>` 相关的逻辑。

### Lines 321-340

````cpp
                    definedOpName / ") => OPERATOR (", definedOpName / ")")) ||
    construct<Rename>(construct<Rename::Names>(name, "=>" >> name)))

// R1412 only -> generic-spec | only-use-name | rename
// R1413 only-use-name -> use-name
// N.B. generic-spec and only-use-name are ambiguous; resolved with symbols
TYPE_PARSER(construct<Only>(Parser<Rename>{}) ||
    construct<Only>(indirect(genericSpec)) || construct<Only>(name))

// R1416 submodule ->
//         submodule-stmt [specification-part] [module-subprogram-part]
//         end-submodule-stmt
TYPE_CONTEXT_PARSER("submodule"_en_US,
    construct<Submodule>(statement(Parser<SubmoduleStmt>{}),
        limitedSpecificationPart, maybe(Parser<ModuleSubprogramPart>{}),
        unterminatedStatement(Parser<EndSubmoduleStmt>{})))

// R1417 submodule-stmt -> SUBMODULE ( parent-identifier ) submodule-name
TYPE_CONTEXT_PARSER("SUBMODULE statement"_en_US,
    construct<SubmoduleStmt>(
````
- **L321 EN**: Continues logic associated with callable symbol `OPERATOR`.
  **L321 CN**: 继续与可调用符号 `OPERATOR` 相关的逻辑。
- **L322 EN**: Continues logic associated with callable symbol `construct<Rename>`.
  **L322 CN**: 继续与可调用符号 `construct<Rename>` 相关的逻辑。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L324 EN**: Comment explains nearby logic, intent, or metadata: `R1412 only -> generic-spec | only-use-name | rename`.
  **L324 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1412 only -> generic-spec | only-use-name | rename`。
- **L325 EN**: Comment explains nearby logic, intent, or metadata: `R1413 only-use-name -> use-name`.
  **L325 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1413 only-use-name -> use-name`。
- **L326 EN**: Comment explains nearby logic, intent, or metadata: `N.B. generic-spec and only-use-name are ambiguous; resolved with symbols`.
  **L326 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. generic-spec and only-use-name are ambiguous; resolved with symbols`。
- **L327 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L327 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L328 EN**: Continues logic associated with callable symbol `construct<Only>`.
  **L328 CN**: 继续与可调用符号 `construct<Only>` 相关的逻辑。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L330 EN**: Comment explains nearby logic, intent, or metadata: `R1416 submodule ->`.
  **L330 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1416 submodule ->`。
- **L331 EN**: Comment explains nearby logic, intent, or metadata: `submodule-stmt [specification-part] [module-subprogram-part]`.
  **L331 CN**: 注释说明附近代码的逻辑、意图或元数据：`submodule-stmt [specification-part] [module-subprogram-part]`。
- **L332 EN**: Comment explains nearby logic, intent, or metadata: `end-submodule-stmt`.
  **L332 CN**: 注释说明附近代码的逻辑、意图或元数据：`end-submodule-stmt`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("submodule"_en_US,`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("submodule"_en_US,`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<Submodule>(statement(Parser<SubmoduleStmt>{}),`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<Submodule>(statement(Parser<SubmoduleStmt>{}),`。
- **L335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `limitedSpecificationPart, maybe(Parser<ModuleSubprogramPart>{}),`.
  **L335 CN**: 继续一个多行参数列表、初始化器或聚合项：`limitedSpecificationPart, maybe(Parser<ModuleSubprogramPart>{}),`。
- **L336 EN**: Continues logic associated with callable symbol `unterminatedStatement`.
  **L336 CN**: 继续与可调用符号 `unterminatedStatement` 相关的逻辑。
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L338 EN**: Comment explains nearby logic, intent, or metadata: `R1417 submodule-stmt -> SUBMODULE ( parent-identifier ) submodule-name`.
  **L338 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1417 submodule-stmt -> SUBMODULE ( parent-identifier ) submodule-name`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("SUBMODULE statement"_en_US,`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("SUBMODULE statement"_en_US,`。
- **L340 EN**: Continues logic associated with callable symbol `construct<SubmoduleStmt>`.
  **L340 CN**: 继续与可调用符号 `construct<SubmoduleStmt>` 相关的逻辑。

### Lines 341-360

````cpp
        "SUBMODULE" >> parenthesized(Parser<ParentIdentifier>{}), name))

// R1418 parent-identifier -> ancestor-module-name [: parent-submodule-name]
TYPE_PARSER(construct<ParentIdentifier>(name, maybe(":" >> name)))

// R1419 end-submodule-stmt -> END [SUBMODULE [submodule-name]]
TYPE_CONTEXT_PARSER("END SUBMODULE statement"_en_US,
    construct<EndSubmoduleStmt>(
        recovery("END " >> defaulted("SUBMODULE" >> maybe(name)) / atEndOfStmt,
            progUnitEndStmtErrorRecovery)))

// R1420 block-data -> block-data-stmt [specification-part] end-block-data-stmt
TYPE_CONTEXT_PARSER("BLOCK DATA subprogram"_en_US,
    construct<BlockData>(statement(Parser<BlockDataStmt>{}),
        limitedSpecificationPart,
        unterminatedStatement(Parser<EndBlockDataStmt>{})))

// R1421 block-data-stmt -> BLOCK DATA [block-data-name]
TYPE_CONTEXT_PARSER("BLOCK DATA statement"_en_US,
    construct<BlockDataStmt>("BLOCK DATA" >> maybe(name)))
````
- **L341 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L341 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L343 EN**: Comment explains nearby logic, intent, or metadata: `R1418 parent-identifier -> ancestor-module-name [: parent-submodule-name]`.
  **L343 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1418 parent-identifier -> ancestor-module-name [: parent-submodule-name]`。
- **L344 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L344 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L346 EN**: Comment explains nearby logic, intent, or metadata: `R1419 end-submodule-stmt -> END [SUBMODULE [submodule-name]]`.
  **L346 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1419 end-submodule-stmt -> END [SUBMODULE [submodule-name]]`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("END SUBMODULE statement"_en_US,`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("END SUBMODULE statement"_en_US,`。
- **L348 EN**: Continues logic associated with callable symbol `construct<EndSubmoduleStmt>`.
  **L348 CN**: 继续与可调用符号 `construct<EndSubmoduleStmt>` 相关的逻辑。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `recovery("END " >> defaulted("SUBMODULE" >> maybe(name)) / atEndOfStmt,`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`recovery("END " >> defaulted("SUBMODULE" >> maybe(name)) / atEndOfStmt,`。
- **L350 EN**: Continues the surrounding expression or declaration: `progUnitEndStmtErrorRecovery)))`.
  **L350 CN**: 继续构造周围的表达式或声明：`progUnitEndStmtErrorRecovery)))`。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L352 EN**: Comment explains nearby logic, intent, or metadata: `R1420 block-data -> block-data-stmt [specification-part] end-block-data-stmt`.
  **L352 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1420 block-data -> block-data-stmt [specification-part] end-block-data-stmt`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("BLOCK DATA subprogram"_en_US,`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("BLOCK DATA subprogram"_en_US,`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<BlockData>(statement(Parser<BlockDataStmt>{}),`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<BlockData>(statement(Parser<BlockDataStmt>{}),`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `limitedSpecificationPart,`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`limitedSpecificationPart,`。
- **L356 EN**: Continues logic associated with callable symbol `unterminatedStatement`.
  **L356 CN**: 继续与可调用符号 `unterminatedStatement` 相关的逻辑。
- **L357 EN**: Blank line separating nearby declarations or logic blocks.
  **L357 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L358 EN**: Comment explains nearby logic, intent, or metadata: `R1421 block-data-stmt -> BLOCK DATA [block-data-name]`.
  **L358 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1421 block-data-stmt -> BLOCK DATA [block-data-name]`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("BLOCK DATA statement"_en_US,`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("BLOCK DATA statement"_en_US,`。
- **L360 EN**: Continues logic associated with callable symbol `construct<BlockDataStmt>`.
  **L360 CN**: 继续与可调用符号 `construct<BlockDataStmt>` 相关的逻辑。

### Lines 361-380

````cpp

// R1422 end-block-data-stmt -> END [BLOCK DATA [block-data-name]]
TYPE_CONTEXT_PARSER("END BLOCK DATA statement"_en_US,
    construct<EndBlockDataStmt>(
        recovery("END " >> defaulted("BLOCK DATA" >> maybe(name)) / atEndOfStmt,
            progUnitEndStmtErrorRecovery)))

// R1501 interface-block ->
//         interface-stmt [interface-specification]... end-interface-stmt
TYPE_PARSER(construct<InterfaceBlock>(statement(Parser<InterfaceStmt>{}),
    many(Parser<InterfaceSpecification>{}),
    statement(Parser<EndInterfaceStmt>{})))

// R1502 interface-specification -> interface-body | procedure-stmt
TYPE_PARSER(construct<InterfaceSpecification>(Parser<InterfaceBody>{}) ||
    construct<InterfaceSpecification>(statement(Parser<ProcedureStmt>{})))

// R1503 interface-stmt -> INTERFACE [generic-spec] | ABSTRACT INTERFACE
TYPE_PARSER(construct<InterfaceStmt>("INTERFACE" >> maybe(genericSpec)) ||
    construct<InterfaceStmt>(construct<Abstract>("ABSTRACT INTERFACE"_sptok)))
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L362 EN**: Comment explains nearby logic, intent, or metadata: `R1422 end-block-data-stmt -> END [BLOCK DATA [block-data-name]]`.
  **L362 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1422 end-block-data-stmt -> END [BLOCK DATA [block-data-name]]`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("END BLOCK DATA statement"_en_US,`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("END BLOCK DATA statement"_en_US,`。
- **L364 EN**: Continues logic associated with callable symbol `construct<EndBlockDataStmt>`.
  **L364 CN**: 继续与可调用符号 `construct<EndBlockDataStmt>` 相关的逻辑。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `recovery("END " >> defaulted("BLOCK DATA" >> maybe(name)) / atEndOfStmt,`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`recovery("END " >> defaulted("BLOCK DATA" >> maybe(name)) / atEndOfStmt,`。
- **L366 EN**: Continues the surrounding expression or declaration: `progUnitEndStmtErrorRecovery)))`.
  **L366 CN**: 继续构造周围的表达式或声明：`progUnitEndStmtErrorRecovery)))`。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L368 EN**: Comment explains nearby logic, intent, or metadata: `R1501 interface-block ->`.
  **L368 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1501 interface-block ->`。
- **L369 EN**: Comment explains nearby logic, intent, or metadata: `interface-stmt [interface-specification]... end-interface-stmt`.
  **L369 CN**: 注释说明附近代码的逻辑、意图或元数据：`interface-stmt [interface-specification]... end-interface-stmt`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<InterfaceBlock>(statement(Parser<InterfaceStmt>{}),`.
  **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<InterfaceBlock>(statement(Parser<InterfaceStmt>{}),`。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `many(Parser<InterfaceSpecification>{}),`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`many(Parser<InterfaceSpecification>{}),`。
- **L372 EN**: Continues logic associated with callable symbol `statement`.
  **L372 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L373 EN**: Blank line separating nearby declarations or logic blocks.
  **L373 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L374 EN**: Comment explains nearby logic, intent, or metadata: `R1502 interface-specification -> interface-body | procedure-stmt`.
  **L374 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1502 interface-specification -> interface-body | procedure-stmt`。
- **L375 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L375 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L376 EN**: Continues logic associated with callable symbol `construct<InterfaceSpecification>`.
  **L376 CN**: 继续与可调用符号 `construct<InterfaceSpecification>` 相关的逻辑。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L378 EN**: Comment explains nearby logic, intent, or metadata: `R1503 interface-stmt -> INTERFACE [generic-spec] | ABSTRACT INTERFACE`.
  **L378 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1503 interface-stmt -> INTERFACE [generic-spec] | ABSTRACT INTERFACE`。
- **L379 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L379 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L380 EN**: Continues logic associated with callable symbol `construct<InterfaceStmt>`.
  **L380 CN**: 继续与可调用符号 `construct<InterfaceStmt>` 相关的逻辑。

### Lines 381-400

````cpp

// R1504 end-interface-stmt -> END INTERFACE [generic-spec]
TYPE_PARSER(
    construct<EndInterfaceStmt>(recovery("END INTERFACE" >> maybe(genericSpec),
        constructEndStmtErrorRecovery >> pure<std::optional<GenericSpec>>())))

// R1505 interface-body ->
//         function-stmt [specification-part] end-function-stmt |
//         subroutine-stmt [specification-part] end-subroutine-stmt
TYPE_CONTEXT_PARSER("interface body"_en_US,
    construct<InterfaceBody>(
        construct<InterfaceBody::Function>(statement(functionStmt),
            indirect(limitedSpecificationPart), statement(endFunctionStmt))) ||
        construct<InterfaceBody>(construct<InterfaceBody::Subroutine>(
            statement(subroutineStmt), indirect(limitedSpecificationPart),
            statement(endSubroutineStmt))))

// R1507 specific-procedure -> procedure-name
constexpr auto specificProcedures{
    nonemptyList("expected specific procedure names"_err_en_US, name)};
````
- **L381 EN**: Blank line separating nearby declarations or logic blocks.
  **L381 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L382 EN**: Comment explains nearby logic, intent, or metadata: `R1504 end-interface-stmt -> END INTERFACE [generic-spec]`.
  **L382 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1504 end-interface-stmt -> END INTERFACE [generic-spec]`。
- **L383 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L383 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<EndInterfaceStmt>(recovery("END INTERFACE" >> maybe(genericSpec),`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<EndInterfaceStmt>(recovery("END INTERFACE" >> maybe(genericSpec),`。
- **L385 EN**: Continues logic associated with callable symbol `optional<GenericSpec>>`.
  **L385 CN**: 继续与可调用符号 `optional<GenericSpec>>` 相关的逻辑。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L387 EN**: Comment explains nearby logic, intent, or metadata: `R1505 interface-body ->`.
  **L387 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1505 interface-body ->`。
- **L388 EN**: Comment explains nearby logic, intent, or metadata: `function-stmt [specification-part] end-function-stmt |`.
  **L388 CN**: 注释说明附近代码的逻辑、意图或元数据：`function-stmt [specification-part] end-function-stmt |`。
- **L389 EN**: Comment explains nearby logic, intent, or metadata: `subroutine-stmt [specification-part] end-subroutine-stmt`.
  **L389 CN**: 注释说明附近代码的逻辑、意图或元数据：`subroutine-stmt [specification-part] end-subroutine-stmt`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("interface body"_en_US,`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("interface body"_en_US,`。
- **L391 EN**: Continues logic associated with callable symbol `construct<InterfaceBody>`.
  **L391 CN**: 继续与可调用符号 `construct<InterfaceBody>` 相关的逻辑。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<InterfaceBody::Function>(statement(functionStmt),`.
  **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<InterfaceBody::Function>(statement(functionStmt),`。
- **L393 EN**: Continues logic associated with callable symbol `indirect`.
  **L393 CN**: 继续与可调用符号 `indirect` 相关的逻辑。
- **L394 EN**: Continues logic associated with callable symbol `construct<InterfaceBody>`.
  **L394 CN**: 继续与可调用符号 `construct<InterfaceBody>` 相关的逻辑。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `statement(subroutineStmt), indirect(limitedSpecificationPart),`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`statement(subroutineStmt), indirect(limitedSpecificationPart),`。
- **L396 EN**: Continues logic associated with callable symbol `statement`.
  **L396 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L398 EN**: Comment explains nearby logic, intent, or metadata: `R1507 specific-procedure -> procedure-name`.
  **L398 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1507 specific-procedure -> procedure-name`。
- **L399 EN**: Continues the surrounding expression or declaration: `constexpr auto specificProcedures{`.
  **L399 CN**: 继续构造周围的表达式或声明：`constexpr auto specificProcedures{`。
- **L400 EN**: Executes a call or declaration centered on `nonemptyList`.
  **L400 CN**: 执行以 `nonemptyList` 为核心的调用或声明。

### Lines 401-420

````cpp

// R1506 procedure-stmt -> [MODULE] PROCEDURE [::] specific-procedure-list
TYPE_PARSER(construct<ProcedureStmt>("MODULE PROCEDURE"_sptok >>
                    pure(ProcedureStmt::Kind::ModuleProcedure),
                maybe("::"_tok) >> specificProcedures) ||
    construct<ProcedureStmt>(
        "PROCEDURE" >> pure(ProcedureStmt::Kind::Procedure),
        maybe("::"_tok) >> specificProcedures))

// R1508 generic-spec ->
//         generic-name | OPERATOR ( defined-operator ) |
//         ASSIGNMENT ( = ) | defined-io-generic-spec
// R1509 defined-io-generic-spec ->
//         READ ( FORMATTED ) | READ ( UNFORMATTED ) |
//         WRITE ( FORMATTED ) | WRITE ( UNFORMATTED )
TYPE_PARSER(sourced(first(construct<GenericSpec>("OPERATOR" >>
                              parenthesized(Parser<DefinedOperator>{})),
    construct<GenericSpec>(
        construct<GenericSpec::Assignment>("ASSIGNMENT ( = )"_tok)),
    construct<GenericSpec>(
````
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L402 EN**: Comment explains nearby logic, intent, or metadata: `R1506 procedure-stmt -> [MODULE] PROCEDURE [::] specific-procedure-list`.
  **L402 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1506 procedure-stmt -> [MODULE] PROCEDURE [::] specific-procedure-list`。
- **L403 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L403 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `pure(ProcedureStmt::Kind::ModuleProcedure),`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`pure(ProcedureStmt::Kind::ModuleProcedure),`。
- **L405 EN**: Continues logic associated with callable symbol `maybe`.
  **L405 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L406 EN**: Continues logic associated with callable symbol `construct<ProcedureStmt>`.
  **L406 CN**: 继续与可调用符号 `construct<ProcedureStmt>` 相关的逻辑。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"PROCEDURE" >> pure(ProcedureStmt::Kind::Procedure),`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`"PROCEDURE" >> pure(ProcedureStmt::Kind::Procedure),`。
- **L408 EN**: Continues logic associated with callable symbol `maybe`.
  **L408 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L410 EN**: Comment explains nearby logic, intent, or metadata: `R1508 generic-spec ->`.
  **L410 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1508 generic-spec ->`。
- **L411 EN**: Comment explains nearby logic, intent, or metadata: `generic-name | OPERATOR ( defined-operator ) |`.
  **L411 CN**: 注释说明附近代码的逻辑、意图或元数据：`generic-name | OPERATOR ( defined-operator ) |`。
- **L412 EN**: Comment explains nearby logic, intent, or metadata: `ASSIGNMENT ( = ) | defined-io-generic-spec`.
  **L412 CN**: 注释说明附近代码的逻辑、意图或元数据：`ASSIGNMENT ( = ) | defined-io-generic-spec`。
- **L413 EN**: Comment explains nearby logic, intent, or metadata: `R1509 defined-io-generic-spec ->`.
  **L413 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1509 defined-io-generic-spec ->`。
- **L414 EN**: Comment explains nearby logic, intent, or metadata: `READ ( FORMATTED ) | READ ( UNFORMATTED ) |`.
  **L414 CN**: 注释说明附近代码的逻辑、意图或元数据：`READ ( FORMATTED ) | READ ( UNFORMATTED ) |`。
- **L415 EN**: Comment explains nearby logic, intent, or metadata: `WRITE ( FORMATTED ) | WRITE ( UNFORMATTED )`.
  **L415 CN**: 注释说明附近代码的逻辑、意图或元数据：`WRITE ( FORMATTED ) | WRITE ( UNFORMATTED )`。
- **L416 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L416 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parenthesized(Parser<DefinedOperator>{})),`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`parenthesized(Parser<DefinedOperator>{})),`。
- **L418 EN**: Continues logic associated with callable symbol `construct<GenericSpec>`.
  **L418 CN**: 继续与可调用符号 `construct<GenericSpec>` 相关的逻辑。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<GenericSpec::Assignment>("ASSIGNMENT ( = )"_tok)),`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<GenericSpec::Assignment>("ASSIGNMENT ( = )"_tok)),`。
- **L420 EN**: Continues logic associated with callable symbol `construct<GenericSpec>`.
  **L420 CN**: 继续与可调用符号 `construct<GenericSpec>` 相关的逻辑。

### Lines 421-440

````cpp
        construct<GenericSpec::ReadFormatted>("READ ( FORMATTED )"_tok)),
    construct<GenericSpec>(
        construct<GenericSpec::ReadUnformatted>("READ ( UNFORMATTED )"_tok)),
    construct<GenericSpec>(
        construct<GenericSpec::WriteFormatted>("WRITE ( FORMATTED )"_tok)),
    construct<GenericSpec>(
        construct<GenericSpec::WriteUnformatted>("WRITE ( UNFORMATTED )"_tok)),
    construct<GenericSpec>(name))))

// R1510 generic-stmt ->
//         GENERIC [, access-spec] :: generic-spec => specific-procedure-list
TYPE_PARSER(construct<GenericStmt>("GENERIC" >> maybe("," >> accessSpec),
    "::" >> genericSpec, "=>" >> specificProcedures))

// R1511 external-stmt -> EXTERNAL [::] external-name-list
TYPE_PARSER(
    "EXTERNAL" >> maybe("::"_tok) >> construct<ExternalStmt>(listOfNames))

// R1512 procedure-declaration-stmt ->
//         PROCEDURE ( [proc-interface] ) [[, proc-attr-spec]... ::]
````
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<GenericSpec::ReadFormatted>("READ ( FORMATTED )"_tok)),`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<GenericSpec::ReadFormatted>("READ ( FORMATTED )"_tok)),`。
- **L422 EN**: Continues logic associated with callable symbol `construct<GenericSpec>`.
  **L422 CN**: 继续与可调用符号 `construct<GenericSpec>` 相关的逻辑。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<GenericSpec::ReadUnformatted>("READ ( UNFORMATTED )"_tok)),`.
  **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<GenericSpec::ReadUnformatted>("READ ( UNFORMATTED )"_tok)),`。
- **L424 EN**: Continues logic associated with callable symbol `construct<GenericSpec>`.
  **L424 CN**: 继续与可调用符号 `construct<GenericSpec>` 相关的逻辑。
- **L425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<GenericSpec::WriteFormatted>("WRITE ( FORMATTED )"_tok)),`.
  **L425 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<GenericSpec::WriteFormatted>("WRITE ( FORMATTED )"_tok)),`。
- **L426 EN**: Continues logic associated with callable symbol `construct<GenericSpec>`.
  **L426 CN**: 继续与可调用符号 `construct<GenericSpec>` 相关的逻辑。
- **L427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<GenericSpec::WriteUnformatted>("WRITE ( UNFORMATTED )"_tok)),`.
  **L427 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<GenericSpec::WriteUnformatted>("WRITE ( UNFORMATTED )"_tok)),`。
- **L428 EN**: Continues logic associated with callable symbol `construct<GenericSpec>`.
  **L428 CN**: 继续与可调用符号 `construct<GenericSpec>` 相关的逻辑。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L430 EN**: Comment explains nearby logic, intent, or metadata: `R1510 generic-stmt ->`.
  **L430 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1510 generic-stmt ->`。
- **L431 EN**: Comment explains nearby logic, intent, or metadata: `GENERIC [, access-spec] :: generic-spec => specific-procedure-list`.
  **L431 CN**: 注释说明附近代码的逻辑、意图或元数据：`GENERIC [, access-spec] :: generic-spec => specific-procedure-list`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<GenericStmt>("GENERIC" >> maybe("," >> accessSpec),`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<GenericStmt>("GENERIC" >> maybe("," >> accessSpec),`。
- **L433 EN**: Continues the surrounding expression or declaration: `"::" >> genericSpec, "=>" >> specificProcedures))`.
  **L433 CN**: 继续构造周围的表达式或声明：`"::" >> genericSpec, "=>" >> specificProcedures))`。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L435 EN**: Comment explains nearby logic, intent, or metadata: `R1511 external-stmt -> EXTERNAL [::] external-name-list`.
  **L435 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1511 external-stmt -> EXTERNAL [::] external-name-list`。
- **L436 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L436 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L437 EN**: Continues logic associated with callable symbol `maybe`.
  **L437 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L439 EN**: Comment explains nearby logic, intent, or metadata: `R1512 procedure-declaration-stmt ->`.
  **L439 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1512 procedure-declaration-stmt ->`。
- **L440 EN**: Comment explains nearby logic, intent, or metadata: `PROCEDURE ( [proc-interface] ) [[, proc-attr-spec]... ::]`.
  **L440 CN**: 注释说明附近代码的逻辑、意图或元数据：`PROCEDURE ( [proc-interface] ) [[, proc-attr-spec]... ::]`。

### Lines 441-460

````cpp
//         proc-decl-list
TYPE_PARSER("PROCEDURE" >>
    construct<ProcedureDeclarationStmt>(parenthesized(maybe(procInterface)),
        optionalListBeforeColons(Parser<ProcAttrSpec>{}),
        nonemptyList("expected procedure declarations"_err_en_US, procDecl)))

// R1513 proc-interface -> interface-name | declaration-type-spec
// R1516 interface-name -> name
// N.B. Simple names of intrinsic types (e.g., "REAL") are not
// ambiguous here - they take precedence over derived type names
// thanks to C1516.
TYPE_PARSER(
    construct<ProcInterface>(declarationTypeSpec / lookAhead(")"_tok)) ||
    construct<ProcInterface>(name))

// R1514 proc-attr-spec ->
//         access-spec | proc-language-binding-spec | INTENT ( intent-spec ) |
//         OPTIONAL | POINTER | PROTECTED | SAVE
TYPE_PARSER(construct<ProcAttrSpec>(accessSpec) ||
    construct<ProcAttrSpec>(languageBindingSpec) ||
````
- **L441 EN**: Comment explains nearby logic, intent, or metadata: `proc-decl-list`.
  **L441 CN**: 注释说明附近代码的逻辑、意图或元数据：`proc-decl-list`。
- **L442 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L442 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<ProcedureDeclarationStmt>(parenthesized(maybe(procInterface)),`.
  **L443 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<ProcedureDeclarationStmt>(parenthesized(maybe(procInterface)),`。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `optionalListBeforeColons(Parser<ProcAttrSpec>{}),`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`optionalListBeforeColons(Parser<ProcAttrSpec>{}),`。
- **L445 EN**: Continues logic associated with callable symbol `nonemptyList`.
  **L445 CN**: 继续与可调用符号 `nonemptyList` 相关的逻辑。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L447 EN**: Comment explains nearby logic, intent, or metadata: `R1513 proc-interface -> interface-name | declaration-type-spec`.
  **L447 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1513 proc-interface -> interface-name | declaration-type-spec`。
- **L448 EN**: Comment explains nearby logic, intent, or metadata: `R1516 interface-name -> name`.
  **L448 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1516 interface-name -> name`。
- **L449 EN**: Comment explains nearby logic, intent, or metadata: `N.B. Simple names of intrinsic types (e.g., "REAL") are not`.
  **L449 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. Simple names of intrinsic types (e.g., "REAL") are not`。
- **L450 EN**: Comment explains nearby logic, intent, or metadata: `ambiguous here - they take precedence over derived type names`.
  **L450 CN**: 注释说明附近代码的逻辑、意图或元数据：`ambiguous here - they take precedence over derived type names`。
- **L451 EN**: Comment explains nearby logic, intent, or metadata: `thanks to C1516.`.
  **L451 CN**: 注释说明附近代码的逻辑、意图或元数据：`thanks to C1516.`。
- **L452 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L452 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L453 EN**: Continues logic associated with callable symbol `construct<ProcInterface>`.
  **L453 CN**: 继续与可调用符号 `construct<ProcInterface>` 相关的逻辑。
- **L454 EN**: Continues logic associated with callable symbol `construct<ProcInterface>`.
  **L454 CN**: 继续与可调用符号 `construct<ProcInterface>` 相关的逻辑。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L456 EN**: Comment explains nearby logic, intent, or metadata: `R1514 proc-attr-spec ->`.
  **L456 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1514 proc-attr-spec ->`。
- **L457 EN**: Comment explains nearby logic, intent, or metadata: `access-spec | proc-language-binding-spec | INTENT ( intent-spec ) |`.
  **L457 CN**: 注释说明附近代码的逻辑、意图或元数据：`access-spec | proc-language-binding-spec | INTENT ( intent-spec ) |`。
- **L458 EN**: Comment explains nearby logic, intent, or metadata: `OPTIONAL | POINTER | PROTECTED | SAVE`.
  **L458 CN**: 注释说明附近代码的逻辑、意图或元数据：`OPTIONAL | POINTER | PROTECTED | SAVE`。
- **L459 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L459 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L460 EN**: Continues logic associated with callable symbol `construct<ProcAttrSpec>`.
  **L460 CN**: 继续与可调用符号 `construct<ProcAttrSpec>` 相关的逻辑。

### Lines 461-480

````cpp
    construct<ProcAttrSpec>("INTENT" >> parenthesized(intentSpec)) ||
    construct<ProcAttrSpec>(optional) || construct<ProcAttrSpec>(pointer) ||
    construct<ProcAttrSpec>(protectedAttr) || construct<ProcAttrSpec>(save))

// R1515 proc-decl -> procedure-entity-name [=> proc-pointer-init]
TYPE_PARSER(construct<ProcDecl>(name, maybe("=>" >> Parser<ProcPointerInit>{})))

// R1517 proc-pointer-init -> null-init | initial-proc-target
// R1518 initial-proc-target -> procedure-name
TYPE_PARSER(
    construct<ProcPointerInit>(nullInit) || construct<ProcPointerInit>(name))

// R1519 intrinsic-stmt -> INTRINSIC [::] intrinsic-procedure-name-list
TYPE_PARSER(
    "INTRINSIC" >> maybe("::"_tok) >> construct<IntrinsicStmt>(listOfNames))

// R1520 function-reference -> procedure-designator
//                               ( [actual-arg-spec-list] )
TYPE_CONTEXT_PARSER("function reference"_en_US,
    sourced(construct<FunctionReference>(
````
- **L461 EN**: Continues logic associated with callable symbol `construct<ProcAttrSpec>`.
  **L461 CN**: 继续与可调用符号 `construct<ProcAttrSpec>` 相关的逻辑。
- **L462 EN**: Continues logic associated with callable symbol `construct<ProcAttrSpec>`.
  **L462 CN**: 继续与可调用符号 `construct<ProcAttrSpec>` 相关的逻辑。
- **L463 EN**: Continues logic associated with callable symbol `construct<ProcAttrSpec>`.
  **L463 CN**: 继续与可调用符号 `construct<ProcAttrSpec>` 相关的逻辑。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L465 EN**: Comment explains nearby logic, intent, or metadata: `R1515 proc-decl -> procedure-entity-name [=> proc-pointer-init]`.
  **L465 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1515 proc-decl -> procedure-entity-name [=> proc-pointer-init]`。
- **L466 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L466 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L468 EN**: Comment explains nearby logic, intent, or metadata: `R1517 proc-pointer-init -> null-init | initial-proc-target`.
  **L468 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1517 proc-pointer-init -> null-init | initial-proc-target`。
- **L469 EN**: Comment explains nearby logic, intent, or metadata: `R1518 initial-proc-target -> procedure-name`.
  **L469 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1518 initial-proc-target -> procedure-name`。
- **L470 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L470 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L471 EN**: Continues logic associated with callable symbol `construct<ProcPointerInit>`.
  **L471 CN**: 继续与可调用符号 `construct<ProcPointerInit>` 相关的逻辑。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L473 EN**: Comment explains nearby logic, intent, or metadata: `R1519 intrinsic-stmt -> INTRINSIC [::] intrinsic-procedure-name-list`.
  **L473 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1519 intrinsic-stmt -> INTRINSIC [::] intrinsic-procedure-name-list`。
- **L474 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L474 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L475 EN**: Continues logic associated with callable symbol `maybe`.
  **L475 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L477 EN**: Comment explains nearby logic, intent, or metadata: `R1520 function-reference -> procedure-designator`.
  **L477 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1520 function-reference -> procedure-designator`。
- **L478 EN**: Comment explains nearby logic, intent, or metadata: `( [actual-arg-spec-list] )`.
  **L478 CN**: 注释说明附近代码的逻辑、意图或元数据：`( [actual-arg-spec-list] )`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("function reference"_en_US,`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("function reference"_en_US,`。
- **L480 EN**: Continues logic associated with callable symbol `sourced`.
  **L480 CN**: 继续与可调用符号 `sourced` 相关的逻辑。

### Lines 481-500

````cpp
        construct<Call>(Parser<ProcedureDesignator>{},
            parenthesized(optionalList(actualArgSpec))))) /
        !"["_tok)

// R1521 call-stmt -> CALL procedure-designator [chevrons]
///                          [( [actual-arg-spec-list] )]
// (CUDA) chevrons -> <<< * | scalar-expr, scalar-expr [, scalar-int-expr
//                      [, scalar-int-expr ] ] >>>
constexpr auto starOrExpr{
    construct<CallStmt::StarOrExpr>("*" >> pure<std::optional<ScalarExpr>>() ||
        applyFunction(presentOptional<ScalarExpr>, scalarExpr))};
TYPE_PARSER(extension<LanguageFeature::CUDA>(
    "<<<" >> construct<CallStmt::Chevrons>(starOrExpr, ", " >> scalarExpr,
                 maybe("," >> scalarExpr), maybe("," >> scalarIntExpr)) /
        ">>>"))
constexpr auto actualArgSpecList{optionalList(actualArgSpec)};
TYPE_CONTEXT_PARSER("CALL statement"_en_US,
    construct<CallStmt>(
        sourced(construct<CallStmt>("CALL" >> Parser<ProcedureDesignator>{},
            maybe(Parser<CallStmt::Chevrons>{}) / space,
````
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<Call>(Parser<ProcedureDesignator>{},`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<Call>(Parser<ProcedureDesignator>{},`。
- **L482 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L482 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L483 EN**: Continues the surrounding expression or declaration: `!"["_tok)`.
  **L483 CN**: 继续构造周围的表达式或声明：`!"["_tok)`。
- **L484 EN**: Blank line separating nearby declarations or logic blocks.
  **L484 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L485 EN**: Comment explains nearby logic, intent, or metadata: `R1521 call-stmt -> CALL procedure-designator [chevrons]`.
  **L485 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1521 call-stmt -> CALL procedure-designator [chevrons]`。
- **L486 EN**: Comment explains nearby logic, intent, or metadata: `[( [actual-arg-spec-list] )]`.
  **L486 CN**: 注释说明附近代码的逻辑、意图或元数据：`[( [actual-arg-spec-list] )]`。
- **L487 EN**: Comment explains nearby logic, intent, or metadata: `(CUDA) chevrons -> <<< * | scalar-expr, scalar-expr [, scalar-int-expr`.
  **L487 CN**: 注释说明附近代码的逻辑、意图或元数据：`(CUDA) chevrons -> <<< * | scalar-expr, scalar-expr [, scalar-int-expr`。
- **L488 EN**: Comment explains nearby logic, intent, or metadata: `[, scalar-int-expr ] ] >>>`.
  **L488 CN**: 注释说明附近代码的逻辑、意图或元数据：`[, scalar-int-expr ] ] >>>`。
- **L489 EN**: Continues the surrounding expression or declaration: `constexpr auto starOrExpr{`.
  **L489 CN**: 继续构造周围的表达式或声明：`constexpr auto starOrExpr{`。
- **L490 EN**: Continues logic associated with callable symbol `StarOrExpr>`.
  **L490 CN**: 继续与可调用符号 `StarOrExpr>` 相关的逻辑。
- **L491 EN**: Executes a call or declaration centered on `applyFunction`.
  **L491 CN**: 执行以 `applyFunction` 为核心的调用或声明。
- **L492 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L492 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"<<<" >> construct<CallStmt::Chevrons>(starOrExpr, ", " >> scalarExpr,`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`"<<<" >> construct<CallStmt::Chevrons>(starOrExpr, ", " >> scalarExpr,`。
- **L494 EN**: Continues logic associated with callable symbol `maybe`.
  **L494 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L495 EN**: Continues the surrounding expression or declaration: `">>>"))`.
  **L495 CN**: 继续构造周围的表达式或声明：`">>>"))`。
- **L496 EN**: Executes a call or declaration centered on `actualArgSpecList{optionalList`.
  **L496 CN**: 执行以 `actualArgSpecList{optionalList` 为核心的调用或声明。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("CALL statement"_en_US,`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("CALL statement"_en_US,`。
- **L498 EN**: Continues logic associated with callable symbol `construct<CallStmt>`.
  **L498 CN**: 继续与可调用符号 `construct<CallStmt>` 相关的逻辑。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `sourced(construct<CallStmt>("CALL" >> Parser<ProcedureDesignator>{},`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`sourced(construct<CallStmt>("CALL" >> Parser<ProcedureDesignator>{},`。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `maybe(Parser<CallStmt::Chevrons>{}) / space,`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`maybe(Parser<CallStmt::Chevrons>{}) / space,`。

### Lines 501-520

````cpp
            "(" >> actualArgSpecList / ")" ||
                lookAhead(endOfStmt) >> defaulted(actualArgSpecList)))))

// R1522 procedure-designator ->
//         procedure-name | proc-component-ref | data-ref % binding-name
TYPE_PARSER(construct<ProcedureDesignator>(Parser<ProcComponentRef>{}) ||
    construct<ProcedureDesignator>(name))

// R1523 actual-arg-spec -> [keyword =] actual-arg
TYPE_PARSER(construct<ActualArgSpec>(
    maybe(keyword / "=" / !"="_ch), Parser<ActualArg>{}))

// F2023 R1527 consequent -> consequent-arg | .NIL.
// F2023 R1528 consequent-arg -> expr | variable
// N.B. "variable" is subsumed by "expr" in the parser;
// semantics determines the distinction.
constexpr auto consequent{construct<ConditionalArg::Consequent>(
                              ".NIL." >> construct<ConditionalArgNil>()) ||
    construct<ConditionalArg::Consequent>(indirect(expr))};

````
- **L501 EN**: Continues the surrounding expression or declaration: `"(" >> actualArgSpecList / ")" ||`.
  **L501 CN**: 继续构造周围的表达式或声明：`"(" >> actualArgSpecList / ")" ||`。
- **L502 EN**: Continues logic associated with callable symbol `lookAhead`.
  **L502 CN**: 继续与可调用符号 `lookAhead` 相关的逻辑。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L504 EN**: Comment explains nearby logic, intent, or metadata: `R1522 procedure-designator ->`.
  **L504 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1522 procedure-designator ->`。
- **L505 EN**: Comment explains nearby logic, intent, or metadata: `procedure-name | proc-component-ref | data-ref % binding-name`.
  **L505 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure-name | proc-component-ref | data-ref % binding-name`。
- **L506 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L506 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L507 EN**: Continues logic associated with callable symbol `construct<ProcedureDesignator>`.
  **L507 CN**: 继续与可调用符号 `construct<ProcedureDesignator>` 相关的逻辑。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L509 EN**: Comment explains nearby logic, intent, or metadata: `R1523 actual-arg-spec -> [keyword =] actual-arg`.
  **L509 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1523 actual-arg-spec -> [keyword =] actual-arg`。
- **L510 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L510 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L511 EN**: Continues logic associated with callable symbol `maybe`.
  **L511 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L513 EN**: Comment explains nearby logic, intent, or metadata: `F2023 R1527 consequent -> consequent-arg | .NIL.`.
  **L513 CN**: 注释说明附近代码的逻辑、意图或元数据：`F2023 R1527 consequent -> consequent-arg | .NIL.`。
- **L514 EN**: Comment explains nearby logic, intent, or metadata: `F2023 R1528 consequent-arg -> expr | variable`.
  **L514 CN**: 注释说明附近代码的逻辑、意图或元数据：`F2023 R1528 consequent-arg -> expr | variable`。
- **L515 EN**: Comment explains nearby logic, intent, or metadata: `N.B. "variable" is subsumed by "expr" in the parser;`.
  **L515 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. "variable" is subsumed by "expr" in the parser;`。
- **L516 EN**: Comment explains nearby logic, intent, or metadata: `semantics determines the distinction.`.
  **L516 CN**: 注释说明附近代码的逻辑、意图或元数据：`semantics determines the distinction.`。
- **L517 EN**: Continues logic associated with callable symbol `Consequent>`.
  **L517 CN**: 继续与可调用符号 `Consequent>` 相关的逻辑。
- **L518 EN**: Continues logic associated with callable symbol `construct<ConditionalArgNil>`.
  **L518 CN**: 继续与可调用符号 `construct<ConditionalArgNil>` 相关的逻辑。
- **L519 EN**: Executes a call or declaration centered on `construct<ConditionalArg::Consequent>`.
  **L519 CN**: 执行以 `construct<ConditionalArg::Consequent>` 为核心的调用或声明。
- **L520 EN**: Blank line separating nearby declarations or logic blocks.
  **L520 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 521-540

````cpp
// F2023 R1526 conditional-arg ->
//   scalar-logical-expr ? consequent : conditional-arg-or-consequent
// The outer parentheses are added at the ActualArg level.
TYPE_PARSER(construct<ConditionalArg>(scalarLogicalExpr / "?", consequent / ":",
    indirect(Parser<ConditionalArgTail>{})))

// conditional-arg-part-or-consequent -> conditional-arg | consequent
TYPE_PARSER(construct<ConditionalArgTail>(Parser<ConditionalArg>{}) ||
    construct<ConditionalArgTail>(consequent))

// R1524 actual-arg ->
//         expr | variable | procedure-name | proc-component-ref |
//         alt-return-spec | conditional-arg
// N.B. the "procedure-name" and "proc-component-ref" alternatives can't
// yet be distinguished from "variable", many instances of which can't be
// distinguished from "expr" anyway (to do so would misparse structure
// constructors and function calls as array elements).
// Semantics sorts it all out later.
TYPE_PARSER(construct<ActualArg>(parenthesized(Parser<ConditionalArg>{})) ||
    construct<ActualArg>(expr) ||
````
- **L521 EN**: Comment explains nearby logic, intent, or metadata: `F2023 R1526 conditional-arg ->`.
  **L521 CN**: 注释说明附近代码的逻辑、意图或元数据：`F2023 R1526 conditional-arg ->`。
- **L522 EN**: Comment explains nearby logic, intent, or metadata: `scalar-logical-expr ? consequent : conditional-arg-or-consequent`.
  **L522 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar-logical-expr ? consequent : conditional-arg-or-consequent`。
- **L523 EN**: Comment explains nearby logic, intent, or metadata: `The outer parentheses are added at the ActualArg level.`.
  **L523 CN**: 注释说明附近代码的逻辑、意图或元数据：`The outer parentheses are added at the ActualArg level.`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(construct<ConditionalArg>(scalarLogicalExpr / "?", consequent / ":",`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(construct<ConditionalArg>(scalarLogicalExpr / "?", consequent / ":",`。
- **L525 EN**: Continues logic associated with callable symbol `indirect`.
  **L525 CN**: 继续与可调用符号 `indirect` 相关的逻辑。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L527 EN**: Comment explains nearby logic, intent, or metadata: `conditional-arg-part-or-consequent -> conditional-arg | consequent`.
  **L527 CN**: 注释说明附近代码的逻辑、意图或元数据：`conditional-arg-part-or-consequent -> conditional-arg | consequent`。
- **L528 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L528 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L529 EN**: Continues logic associated with callable symbol `construct<ConditionalArgTail>`.
  **L529 CN**: 继续与可调用符号 `construct<ConditionalArgTail>` 相关的逻辑。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L531 EN**: Comment explains nearby logic, intent, or metadata: `R1524 actual-arg ->`.
  **L531 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1524 actual-arg ->`。
- **L532 EN**: Comment explains nearby logic, intent, or metadata: `expr | variable | procedure-name | proc-component-ref |`.
  **L532 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr | variable | procedure-name | proc-component-ref |`。
- **L533 EN**: Comment explains nearby logic, intent, or metadata: `alt-return-spec | conditional-arg`.
  **L533 CN**: 注释说明附近代码的逻辑、意图或元数据：`alt-return-spec | conditional-arg`。
- **L534 EN**: Comment explains nearby logic, intent, or metadata: `N.B. the "procedure-name" and "proc-component-ref" alternatives can't`.
  **L534 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. the "procedure-name" and "proc-component-ref" alternatives can't`。
- **L535 EN**: Comment explains nearby logic, intent, or metadata: `yet be distinguished from "variable", many instances of which can't be`.
  **L535 CN**: 注释说明附近代码的逻辑、意图或元数据：`yet be distinguished from "variable", many instances of which can't be`。
- **L536 EN**: Comment explains nearby logic, intent, or metadata: `distinguished from "expr" anyway (to do so would misparse structure`.
  **L536 CN**: 注释说明附近代码的逻辑、意图或元数据：`distinguished from "expr" anyway (to do so would misparse structure`。
- **L537 EN**: Comment explains nearby logic, intent, or metadata: `constructors and function calls as array elements).`.
  **L537 CN**: 注释说明附近代码的逻辑、意图或元数据：`constructors and function calls as array elements).`。
- **L538 EN**: Comment explains nearby logic, intent, or metadata: `Semantics sorts it all out later.`.
  **L538 CN**: 注释说明附近代码的逻辑、意图或元数据：`Semantics sorts it all out later.`。
- **L539 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L539 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L540 EN**: Continues logic associated with callable symbol `construct<ActualArg>`.
  **L540 CN**: 继续与可调用符号 `construct<ActualArg>` 相关的逻辑。

### Lines 541-560

````cpp
    construct<ActualArg>(Parser<AltReturnSpec>{}) ||
    extension<LanguageFeature::PercentRefAndVal>(
        "nonstandard usage: %REF"_port_en_US,
        construct<ActualArg>(
            construct<ActualArg::PercentRef>("%REF" >> parenthesized(expr)))) ||
    extension<LanguageFeature::PercentRefAndVal>(
        "nonstandard usage: %VAL"_port_en_US,
        construct<ActualArg>(
            construct<ActualArg::PercentVal>("%VAL" >> parenthesized(expr)))))

// R1525 alt-return-spec -> * label
TYPE_PARSER(construct<AltReturnSpec>(star >> label))

// R1527 prefix-spec ->
//         declaration-type-spec | ELEMENTAL | IMPURE | MODULE |
//         NON_RECURSIVE | PURE | RECURSIVE | SIMPLE |
// (CUDA)  ATTRIBUTES ( (DEVICE | GLOBAL | GRID_GLOBAL | HOST)... ) |
//         LAUNCH_BOUNDS(expr-list) | CLUSTER_DIMS(expr-list)
TYPE_PARSER(withMessage(
    "expected DEVICE, GLOBAL, GRID_GLOBAL, or HOST attribute"_err_en_US,
````
- **L541 EN**: Continues logic associated with callable symbol `construct<ActualArg>`.
  **L541 CN**: 继续与可调用符号 `construct<ActualArg>` 相关的逻辑。
- **L542 EN**: Continues logic associated with callable symbol `PercentRefAndVal>`.
  **L542 CN**: 继续与可调用符号 `PercentRefAndVal>` 相关的逻辑。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: %REF"_port_en_US,`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: %REF"_port_en_US,`。
- **L544 EN**: Continues logic associated with callable symbol `construct<ActualArg>`.
  **L544 CN**: 继续与可调用符号 `construct<ActualArg>` 相关的逻辑。
- **L545 EN**: Continues logic associated with callable symbol `PercentRef>`.
  **L545 CN**: 继续与可调用符号 `PercentRef>` 相关的逻辑。
- **L546 EN**: Continues logic associated with callable symbol `PercentRefAndVal>`.
  **L546 CN**: 继续与可调用符号 `PercentRefAndVal>` 相关的逻辑。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nonstandard usage: %VAL"_port_en_US,`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nonstandard usage: %VAL"_port_en_US,`。
- **L548 EN**: Continues logic associated with callable symbol `construct<ActualArg>`.
  **L548 CN**: 继续与可调用符号 `construct<ActualArg>` 相关的逻辑。
- **L549 EN**: Continues logic associated with callable symbol `PercentVal>`.
  **L549 CN**: 继续与可调用符号 `PercentVal>` 相关的逻辑。
- **L550 EN**: Blank line separating nearby declarations or logic blocks.
  **L550 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L551 EN**: Comment explains nearby logic, intent, or metadata: `R1525 alt-return-spec -> * label`.
  **L551 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1525 alt-return-spec -> * label`。
- **L552 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L552 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L554 EN**: Comment explains nearby logic, intent, or metadata: `R1527 prefix-spec ->`.
  **L554 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1527 prefix-spec ->`。
- **L555 EN**: Comment explains nearby logic, intent, or metadata: `declaration-type-spec | ELEMENTAL | IMPURE | MODULE |`.
  **L555 CN**: 注释说明附近代码的逻辑、意图或元数据：`declaration-type-spec | ELEMENTAL | IMPURE | MODULE |`。
- **L556 EN**: Comment explains nearby logic, intent, or metadata: `NON_RECURSIVE | PURE | RECURSIVE | SIMPLE |`.
  **L556 CN**: 注释说明附近代码的逻辑、意图或元数据：`NON_RECURSIVE | PURE | RECURSIVE | SIMPLE |`。
- **L557 EN**: Comment explains nearby logic, intent, or metadata: `(CUDA)  ATTRIBUTES ( (DEVICE | GLOBAL | GRID_GLOBAL | HOST)... ) |`.
  **L557 CN**: 注释说明附近代码的逻辑、意图或元数据：`(CUDA)  ATTRIBUTES ( (DEVICE | GLOBAL | GRID_GLOBAL | HOST)... ) |`。
- **L558 EN**: Comment explains nearby logic, intent, or metadata: `LAUNCH_BOUNDS(expr-list) | CLUSTER_DIMS(expr-list)`.
  **L558 CN**: 注释说明附近代码的逻辑、意图或元数据：`LAUNCH_BOUNDS(expr-list) | CLUSTER_DIMS(expr-list)`。
- **L559 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L559 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected DEVICE, GLOBAL, GRID_GLOBAL, or HOST attribute"_err_en_US,`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected DEVICE, GLOBAL, GRID_GLOBAL, or HOST attribute"_err_en_US,`。

### Lines 561-580

````cpp
    first("DEVICE" >> pure(common::CUDASubprogramAttrs::Device),
        "GLOBAL" >> pure(common::CUDASubprogramAttrs::Global),
        "GRID_GLOBAL" >> pure(common::CUDASubprogramAttrs::Grid_Global),
        "HOST" >> pure(common::CUDASubprogramAttrs::Host))))
TYPE_PARSER(first(construct<PrefixSpec>(declarationTypeSpec),
    construct<PrefixSpec>(construct<PrefixSpec::Elemental>("ELEMENTAL"_tok)),
    construct<PrefixSpec>(construct<PrefixSpec::Impure>("IMPURE"_tok)),
    construct<PrefixSpec>(construct<PrefixSpec::Module>("MODULE"_tok)),
    construct<PrefixSpec>(
        construct<PrefixSpec::Non_Recursive>("NON_RECURSIVE"_tok)),
    construct<PrefixSpec>(construct<PrefixSpec::Pure>("PURE"_tok)),
    construct<PrefixSpec>(construct<PrefixSpec::Recursive>("RECURSIVE"_tok)),
    construct<PrefixSpec>(construct<PrefixSpec::Simple>("SIMPLE"_tok)),
    extension<LanguageFeature::CUDA>(
        construct<PrefixSpec>(construct<PrefixSpec::Attributes>(
            localRecovery("expected valid ATTRIBUTES specification"_err_en_US,
                "ATTRIBUTES" >> parenthesized(nonemptyList(
                                    Parser<common::CUDASubprogramAttrs>{})),
                "ATTRIBUTES" >> SkipTo<')'>{} >> ")"_ch >>
                    pure<std::list<common::CUDASubprogramAttrs>>())))),
````
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `first("DEVICE" >> pure(common::CUDASubprogramAttrs::Device),`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`first("DEVICE" >> pure(common::CUDASubprogramAttrs::Device),`。
- **L562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"GLOBAL" >> pure(common::CUDASubprogramAttrs::Global),`.
  **L562 CN**: 继续一个多行参数列表、初始化器或聚合项：`"GLOBAL" >> pure(common::CUDASubprogramAttrs::Global),`。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"GRID_GLOBAL" >> pure(common::CUDASubprogramAttrs::Grid_Global),`.
  **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`"GRID_GLOBAL" >> pure(common::CUDASubprogramAttrs::Grid_Global),`。
- **L564 EN**: Continues logic associated with callable symbol `pure`.
  **L564 CN**: 继续与可调用符号 `pure` 相关的逻辑。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_PARSER(first(construct<PrefixSpec>(declarationTypeSpec),`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_PARSER(first(construct<PrefixSpec>(declarationTypeSpec),`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<PrefixSpec>(construct<PrefixSpec::Elemental>("ELEMENTAL"_tok)),`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<PrefixSpec>(construct<PrefixSpec::Elemental>("ELEMENTAL"_tok)),`。
- **L567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<PrefixSpec>(construct<PrefixSpec::Impure>("IMPURE"_tok)),`.
  **L567 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<PrefixSpec>(construct<PrefixSpec::Impure>("IMPURE"_tok)),`。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<PrefixSpec>(construct<PrefixSpec::Module>("MODULE"_tok)),`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<PrefixSpec>(construct<PrefixSpec::Module>("MODULE"_tok)),`。
- **L569 EN**: Continues logic associated with callable symbol `construct<PrefixSpec>`.
  **L569 CN**: 继续与可调用符号 `construct<PrefixSpec>` 相关的逻辑。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<PrefixSpec::Non_Recursive>("NON_RECURSIVE"_tok)),`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<PrefixSpec::Non_Recursive>("NON_RECURSIVE"_tok)),`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<PrefixSpec>(construct<PrefixSpec::Pure>("PURE"_tok)),`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<PrefixSpec>(construct<PrefixSpec::Pure>("PURE"_tok)),`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<PrefixSpec>(construct<PrefixSpec::Recursive>("RECURSIVE"_tok)),`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<PrefixSpec>(construct<PrefixSpec::Recursive>("RECURSIVE"_tok)),`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<PrefixSpec>(construct<PrefixSpec::Simple>("SIMPLE"_tok)),`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<PrefixSpec>(construct<PrefixSpec::Simple>("SIMPLE"_tok)),`。
- **L574 EN**: Continues logic associated with callable symbol `CUDA>`.
  **L574 CN**: 继续与可调用符号 `CUDA>` 相关的逻辑。
- **L575 EN**: Continues logic associated with callable symbol `construct<PrefixSpec>`.
  **L575 CN**: 继续与可调用符号 `construct<PrefixSpec>` 相关的逻辑。
- **L576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `localRecovery("expected valid ATTRIBUTES specification"_err_en_US,`.
  **L576 CN**: 继续一个多行参数列表、初始化器或聚合项：`localRecovery("expected valid ATTRIBUTES specification"_err_en_US,`。
- **L577 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L577 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Parser<common::CUDASubprogramAttrs>{})),`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`Parser<common::CUDASubprogramAttrs>{})),`。
- **L579 EN**: Continues the surrounding expression or declaration: `"ATTRIBUTES" >> SkipTo<')'>{} >> ")"_ch >>`.
  **L579 CN**: 继续构造周围的表达式或声明：`"ATTRIBUTES" >> SkipTo<')'>{} >> ")"_ch >>`。
- **L580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `pure<std::list<common::CUDASubprogramAttrs>>())))),`.
  **L580 CN**: 继续一个多行参数列表、初始化器或聚合项：`pure<std::list<common::CUDASubprogramAttrs>>())))),`。

### Lines 581-600

````cpp
    extension<LanguageFeature::CUDA>(construct<PrefixSpec>(
        construct<PrefixSpec::Launch_Bounds>("LAUNCH_BOUNDS" >>
            parenthesized(nonemptyList(
                "expected launch bounds"_err_en_US, scalarIntConstantExpr))))),
    extension<LanguageFeature::CUDA>(construct<PrefixSpec>(
        construct<PrefixSpec::Cluster_Dims>("CLUSTER_DIMS" >>
            parenthesized(nonemptyList("expected cluster dimensions"_err_en_US,
                scalarIntConstantExpr)))))))

// R1529 function-subprogram ->
//         function-stmt [specification-part] [execution-part]
//         [internal-subprogram-part] end-function-stmt
TYPE_CONTEXT_PARSER("FUNCTION subprogram"_en_US,
    construct<FunctionSubprogram>(statement(functionStmt), specificationPart,
        executionPart, maybe(internalSubprogramPart),
        unterminatedStatement(endFunctionStmt)))

// R1532 suffix ->
//         proc-language-binding-spec [RESULT ( result-name )] |
//         RESULT ( result-name ) [proc-language-binding-spec]
````
- **L581 EN**: Continues logic associated with callable symbol `CUDA>`.
  **L581 CN**: 继续与可调用符号 `CUDA>` 相关的逻辑。
- **L582 EN**: Continues logic associated with callable symbol `Launch_Bounds>`.
  **L582 CN**: 继续与可调用符号 `Launch_Bounds>` 相关的逻辑。
- **L583 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L583 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected launch bounds"_err_en_US, scalarIntConstantExpr))))),`.
  **L584 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected launch bounds"_err_en_US, scalarIntConstantExpr))))),`。
- **L585 EN**: Continues logic associated with callable symbol `CUDA>`.
  **L585 CN**: 继续与可调用符号 `CUDA>` 相关的逻辑。
- **L586 EN**: Continues logic associated with callable symbol `Cluster_Dims>`.
  **L586 CN**: 继续与可调用符号 `Cluster_Dims>` 相关的逻辑。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parenthesized(nonemptyList("expected cluster dimensions"_err_en_US,`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`parenthesized(nonemptyList("expected cluster dimensions"_err_en_US,`。
- **L588 EN**: Continues the surrounding expression or declaration: `scalarIntConstantExpr)))))))`.
  **L588 CN**: 继续构造周围的表达式或声明：`scalarIntConstantExpr)))))))`。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L590 EN**: Comment explains nearby logic, intent, or metadata: `R1529 function-subprogram ->`.
  **L590 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1529 function-subprogram ->`。
- **L591 EN**: Comment explains nearby logic, intent, or metadata: `function-stmt [specification-part] [execution-part]`.
  **L591 CN**: 注释说明附近代码的逻辑、意图或元数据：`function-stmt [specification-part] [execution-part]`。
- **L592 EN**: Comment explains nearby logic, intent, or metadata: `[internal-subprogram-part] end-function-stmt`.
  **L592 CN**: 注释说明附近代码的逻辑、意图或元数据：`[internal-subprogram-part] end-function-stmt`。
- **L593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("FUNCTION subprogram"_en_US,`.
  **L593 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("FUNCTION subprogram"_en_US,`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<FunctionSubprogram>(statement(functionStmt), specificationPart,`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<FunctionSubprogram>(statement(functionStmt), specificationPart,`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `executionPart, maybe(internalSubprogramPart),`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`executionPart, maybe(internalSubprogramPart),`。
- **L596 EN**: Continues logic associated with callable symbol `unterminatedStatement`.
  **L596 CN**: 继续与可调用符号 `unterminatedStatement` 相关的逻辑。
- **L597 EN**: Blank line separating nearby declarations or logic blocks.
  **L597 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L598 EN**: Comment explains nearby logic, intent, or metadata: `R1532 suffix ->`.
  **L598 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1532 suffix ->`。
- **L599 EN**: Comment explains nearby logic, intent, or metadata: `proc-language-binding-spec [RESULT ( result-name )] |`.
  **L599 CN**: 注释说明附近代码的逻辑、意图或元数据：`proc-language-binding-spec [RESULT ( result-name )] |`。
- **L600 EN**: Comment explains nearby logic, intent, or metadata: `RESULT ( result-name ) [proc-language-binding-spec]`.
  **L600 CN**: 注释说明附近代码的逻辑、意图或元数据：`RESULT ( result-name ) [proc-language-binding-spec]`。

### Lines 601-620

````cpp
TYPE_PARSER(construct<Suffix>(
                languageBindingSpec, maybe("RESULT" >> parenthesized(name))) ||
    construct<Suffix>(
        "RESULT" >> parenthesized(name), maybe(languageBindingSpec)))

// R1533 end-function-stmt -> END [FUNCTION [function-name]]
TYPE_PARSER(construct<EndFunctionStmt>(
    recovery("END " >> defaulted("FUNCTION" >> maybe(name)) / atEndOfStmt,
        progUnitEndStmtErrorRecovery)))

// R1534 subroutine-subprogram ->
//         subroutine-stmt [specification-part] [execution-part]
//         [internal-subprogram-part] end-subroutine-stmt
TYPE_CONTEXT_PARSER("SUBROUTINE subprogram"_en_US,
    construct<SubroutineSubprogram>(statement(subroutineStmt),
        specificationPart, executionPart, maybe(internalSubprogramPart),
        unterminatedStatement(endSubroutineStmt)))

// R1535 subroutine-stmt ->
//         [prefix] SUBROUTINE subroutine-name [( [dummy-arg-list] )
````
- **L601 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L601 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L602 EN**: Continues logic associated with callable symbol `maybe`.
  **L602 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L603 EN**: Continues logic associated with callable symbol `construct<Suffix>`.
  **L603 CN**: 继续与可调用符号 `construct<Suffix>` 相关的逻辑。
- **L604 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L604 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L605 EN**: Blank line separating nearby declarations or logic blocks.
  **L605 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L606 EN**: Comment explains nearby logic, intent, or metadata: `R1533 end-function-stmt -> END [FUNCTION [function-name]]`.
  **L606 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1533 end-function-stmt -> END [FUNCTION [function-name]]`。
- **L607 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L607 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `recovery("END " >> defaulted("FUNCTION" >> maybe(name)) / atEndOfStmt,`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`recovery("END " >> defaulted("FUNCTION" >> maybe(name)) / atEndOfStmt,`。
- **L609 EN**: Continues the surrounding expression or declaration: `progUnitEndStmtErrorRecovery)))`.
  **L609 CN**: 继续构造周围的表达式或声明：`progUnitEndStmtErrorRecovery)))`。
- **L610 EN**: Blank line separating nearby declarations or logic blocks.
  **L610 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L611 EN**: Comment explains nearby logic, intent, or metadata: `R1534 subroutine-subprogram ->`.
  **L611 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1534 subroutine-subprogram ->`。
- **L612 EN**: Comment explains nearby logic, intent, or metadata: `subroutine-stmt [specification-part] [execution-part]`.
  **L612 CN**: 注释说明附近代码的逻辑、意图或元数据：`subroutine-stmt [specification-part] [execution-part]`。
- **L613 EN**: Comment explains nearby logic, intent, or metadata: `[internal-subprogram-part] end-subroutine-stmt`.
  **L613 CN**: 注释说明附近代码的逻辑、意图或元数据：`[internal-subprogram-part] end-subroutine-stmt`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("SUBROUTINE subprogram"_en_US,`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("SUBROUTINE subprogram"_en_US,`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SubroutineSubprogram>(statement(subroutineStmt),`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SubroutineSubprogram>(statement(subroutineStmt),`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specificationPart, executionPart, maybe(internalSubprogramPart),`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`specificationPart, executionPart, maybe(internalSubprogramPart),`。
- **L617 EN**: Continues logic associated with callable symbol `unterminatedStatement`.
  **L617 CN**: 继续与可调用符号 `unterminatedStatement` 相关的逻辑。
- **L618 EN**: Blank line separating nearby declarations or logic blocks.
  **L618 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L619 EN**: Comment explains nearby logic, intent, or metadata: `R1535 subroutine-stmt ->`.
  **L619 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1535 subroutine-stmt ->`。
- **L620 EN**: Comment explains nearby logic, intent, or metadata: `[prefix] SUBROUTINE subroutine-name [( [dummy-arg-list] )`.
  **L620 CN**: 注释说明附近代码的逻辑、意图或元数据：`[prefix] SUBROUTINE subroutine-name [( [dummy-arg-list] )`。

### Lines 621-640

````cpp
//         [proc-language-binding-spec]]
TYPE_PARSER(
    (construct<SubroutineStmt>(many(prefixSpec), "SUBROUTINE" >> name,
         !"("_tok >> pure<std::list<DummyArg>>(),
         pure<std::optional<LanguageBindingSpec>>()) ||
        construct<SubroutineStmt>(many(prefixSpec), "SUBROUTINE" >> name,
            defaulted(parenthesized(optionalList(dummyArg))),
            maybe(languageBindingSpec))) /
    checkEndOfKnownStmt)

// R1536 dummy-arg -> dummy-arg-name | *
TYPE_PARSER(construct<DummyArg>(name) || construct<DummyArg>(star))

// R1537 end-subroutine-stmt -> END [SUBROUTINE [subroutine-name]]
TYPE_PARSER(construct<EndSubroutineStmt>(
    recovery("END " >> defaulted("SUBROUTINE" >> maybe(name)) / atEndOfStmt,
        progUnitEndStmtErrorRecovery)))

// R1538 separate-module-subprogram ->
//         mp-subprogram-stmt [specification-part] [execution-part]
````
- **L621 EN**: Comment explains nearby logic, intent, or metadata: `[proc-language-binding-spec]]`.
  **L621 CN**: 注释说明附近代码的逻辑、意图或元数据：`[proc-language-binding-spec]]`。
- **L622 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L622 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(construct<SubroutineStmt>(many(prefixSpec), "SUBROUTINE" >> name,`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`(construct<SubroutineStmt>(many(prefixSpec), "SUBROUTINE" >> name,`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!"("_tok >> pure<std::list<DummyArg>>(),`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`!"("_tok >> pure<std::list<DummyArg>>(),`。
- **L625 EN**: Continues logic associated with callable symbol `optional<LanguageBindingSpec>>`.
  **L625 CN**: 继续与可调用符号 `optional<LanguageBindingSpec>>` 相关的逻辑。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SubroutineStmt>(many(prefixSpec), "SUBROUTINE" >> name,`.
  **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SubroutineStmt>(many(prefixSpec), "SUBROUTINE" >> name,`。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defaulted(parenthesized(optionalList(dummyArg))),`.
  **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`defaulted(parenthesized(optionalList(dummyArg))),`。
- **L628 EN**: Continues logic associated with callable symbol `maybe`.
  **L628 CN**: 继续与可调用符号 `maybe` 相关的逻辑。
- **L629 EN**: Continues the surrounding expression or declaration: `checkEndOfKnownStmt)`.
  **L629 CN**: 继续构造周围的表达式或声明：`checkEndOfKnownStmt)`。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L631 EN**: Comment explains nearby logic, intent, or metadata: `R1536 dummy-arg -> dummy-arg-name |`.
  **L631 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1536 dummy-arg -> dummy-arg-name |`。
- **L632 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L632 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L633 EN**: Blank line separating nearby declarations or logic blocks.
  **L633 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L634 EN**: Comment explains nearby logic, intent, or metadata: `R1537 end-subroutine-stmt -> END [SUBROUTINE [subroutine-name]]`.
  **L634 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1537 end-subroutine-stmt -> END [SUBROUTINE [subroutine-name]]`。
- **L635 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L635 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `recovery("END " >> defaulted("SUBROUTINE" >> maybe(name)) / atEndOfStmt,`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`recovery("END " >> defaulted("SUBROUTINE" >> maybe(name)) / atEndOfStmt,`。
- **L637 EN**: Continues the surrounding expression or declaration: `progUnitEndStmtErrorRecovery)))`.
  **L637 CN**: 继续构造周围的表达式或声明：`progUnitEndStmtErrorRecovery)))`。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L639 EN**: Comment explains nearby logic, intent, or metadata: `R1538 separate-module-subprogram ->`.
  **L639 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1538 separate-module-subprogram ->`。
- **L640 EN**: Comment explains nearby logic, intent, or metadata: `mp-subprogram-stmt [specification-part] [execution-part]`.
  **L640 CN**: 注释说明附近代码的逻辑、意图或元数据：`mp-subprogram-stmt [specification-part] [execution-part]`。

### Lines 641-660

````cpp
//         [internal-subprogram-part] end-mp-subprogram-stmt
TYPE_CONTEXT_PARSER("separate module subprogram"_en_US,
    construct<SeparateModuleSubprogram>(statement(Parser<MpSubprogramStmt>{}),
        specificationPart, executionPart, maybe(internalSubprogramPart),
        statement(Parser<EndMpSubprogramStmt>{})))

// R1539 mp-subprogram-stmt -> MODULE PROCEDURE procedure-name
TYPE_CONTEXT_PARSER("MODULE PROCEDURE statement"_en_US,
    construct<MpSubprogramStmt>("MODULE PROCEDURE"_sptok >> name))

// R1540 end-mp-subprogram-stmt -> END [PROCEDURE [procedure-name]]
TYPE_CONTEXT_PARSER("END PROCEDURE statement"_en_US,
    construct<EndMpSubprogramStmt>(
        recovery("END " >> defaulted("PROCEDURE" >> maybe(name)) / atEndOfStmt,
            progUnitEndStmtErrorRecovery)))

// R1541 entry-stmt -> ENTRY entry-name [( [dummy-arg-list] ) [suffix]]
TYPE_PARSER(
    "ENTRY" >> (construct<EntryStmt>(name,
                    parenthesized(optionalList(dummyArg)), maybe(suffix)) ||
````
- **L641 EN**: Comment explains nearby logic, intent, or metadata: `[internal-subprogram-part] end-mp-subprogram-stmt`.
  **L641 CN**: 注释说明附近代码的逻辑、意图或元数据：`[internal-subprogram-part] end-mp-subprogram-stmt`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("separate module subprogram"_en_US,`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("separate module subprogram"_en_US,`。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<SeparateModuleSubprogram>(statement(Parser<MpSubprogramStmt>{}),`.
  **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<SeparateModuleSubprogram>(statement(Parser<MpSubprogramStmt>{}),`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specificationPart, executionPart, maybe(internalSubprogramPart),`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`specificationPart, executionPart, maybe(internalSubprogramPart),`。
- **L645 EN**: Continues logic associated with callable symbol `statement`.
  **L645 CN**: 继续与可调用符号 `statement` 相关的逻辑。
- **L646 EN**: Blank line separating nearby declarations or logic blocks.
  **L646 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L647 EN**: Comment explains nearby logic, intent, or metadata: `R1539 mp-subprogram-stmt -> MODULE PROCEDURE procedure-name`.
  **L647 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1539 mp-subprogram-stmt -> MODULE PROCEDURE procedure-name`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("MODULE PROCEDURE statement"_en_US,`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("MODULE PROCEDURE statement"_en_US,`。
- **L649 EN**: Continues logic associated with callable symbol `construct<MpSubprogramStmt>`.
  **L649 CN**: 继续与可调用符号 `construct<MpSubprogramStmt>` 相关的逻辑。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L651 EN**: Comment explains nearby logic, intent, or metadata: `R1540 end-mp-subprogram-stmt -> END [PROCEDURE [procedure-name]]`.
  **L651 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1540 end-mp-subprogram-stmt -> END [PROCEDURE [procedure-name]]`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("END PROCEDURE statement"_en_US,`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("END PROCEDURE statement"_en_US,`。
- **L653 EN**: Continues logic associated with callable symbol `construct<EndMpSubprogramStmt>`.
  **L653 CN**: 继续与可调用符号 `construct<EndMpSubprogramStmt>` 相关的逻辑。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `recovery("END " >> defaulted("PROCEDURE" >> maybe(name)) / atEndOfStmt,`.
  **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`recovery("END " >> defaulted("PROCEDURE" >> maybe(name)) / atEndOfStmt,`。
- **L655 EN**: Continues the surrounding expression or declaration: `progUnitEndStmtErrorRecovery)))`.
  **L655 CN**: 继续构造周围的表达式或声明：`progUnitEndStmtErrorRecovery)))`。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L657 EN**: Comment explains nearby logic, intent, or metadata: `R1541 entry-stmt -> ENTRY entry-name [( [dummy-arg-list] ) [suffix]]`.
  **L657 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1541 entry-stmt -> ENTRY entry-name [( [dummy-arg-list] ) [suffix]]`。
- **L658 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L658 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ENTRY" >> (construct<EntryStmt>(name,`.
  **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ENTRY" >> (construct<EntryStmt>(name,`。
- **L660 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L660 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。

### Lines 661-676

````cpp
                   construct<EntryStmt>(name, construct<std::list<DummyArg>>(),
                       construct<std::optional<Suffix>>())))

// R1542 return-stmt -> RETURN [scalar-int-expr]
TYPE_CONTEXT_PARSER("RETURN statement"_en_US,
    construct<ReturnStmt>("RETURN" >> maybe(scalarIntExpr)))

// R1543 contains-stmt -> CONTAINS
TYPE_PARSER(construct<ContainsStmt>("CONTAINS"_tok))

// R1544 stmt-function-stmt ->
//         function-name ( [dummy-arg-name-list] ) = scalar-expr
TYPE_CONTEXT_PARSER("statement function definition"_en_US,
    construct<StmtFunctionStmt>(
        name, parenthesized(optionalList(name)), "=" >> scalar(expr)))
} // namespace Fortran::parser
````
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `construct<EntryStmt>(name, construct<std::list<DummyArg>>(),`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`construct<EntryStmt>(name, construct<std::list<DummyArg>>(),`。
- **L662 EN**: Continues logic associated with callable symbol `optional<Suffix>>`.
  **L662 CN**: 继续与可调用符号 `optional<Suffix>>` 相关的逻辑。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L664 EN**: Comment explains nearby logic, intent, or metadata: `R1542 return-stmt -> RETURN [scalar-int-expr]`.
  **L664 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1542 return-stmt -> RETURN [scalar-int-expr]`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("RETURN statement"_en_US,`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("RETURN statement"_en_US,`。
- **L666 EN**: Continues logic associated with callable symbol `construct<ReturnStmt>`.
  **L666 CN**: 继续与可调用符号 `construct<ReturnStmt>` 相关的逻辑。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L668 EN**: Comment explains nearby logic, intent, or metadata: `R1543 contains-stmt -> CONTAINS`.
  **L668 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1543 contains-stmt -> CONTAINS`。
- **L669 EN**: Continues logic associated with callable symbol `TYPE_PARSER`.
  **L669 CN**: 继续与可调用符号 `TYPE_PARSER` 相关的逻辑。
- **L670 EN**: Blank line separating nearby declarations or logic blocks.
  **L670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L671 EN**: Comment explains nearby logic, intent, or metadata: `R1544 stmt-function-stmt ->`.
  **L671 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1544 stmt-function-stmt ->`。
- **L672 EN**: Comment explains nearby logic, intent, or metadata: `function-name ( [dummy-arg-name-list] ) = scalar-expr`.
  **L672 CN**: 注释说明附近代码的逻辑、意图或元数据：`function-name ( [dummy-arg-name-list] ) = scalar-expr`。
- **L673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TYPE_CONTEXT_PARSER("statement function definition"_en_US,`.
  **L673 CN**: 继续一个多行参数列表、初始化器或聚合项：`TYPE_CONTEXT_PARSER("statement function definition"_en_US,`。
- **L674 EN**: Continues logic associated with callable symbol `construct<StmtFunctionStmt>`.
  **L674 CN**: 继续与可调用符号 `construct<StmtFunctionStmt>` 相关的逻辑。
- **L675 EN**: Continues logic associated with callable symbol `parenthesized`.
  **L675 CN**: 继续与可调用符号 `parenthesized` 相关的逻辑。
- **L676 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::parser`.
  **L676 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::parser`。

## Key Concepts / 关键概念

- **Parse-tree construction and source handling / 语法树构建与源码处理**
- **Fortran parse tree handling / Fortran 语法树处理**
- **OpenACC handling / OpenACC 处理**
- **CUDA-specific lowering or runtime handling / CUDA 专用 lowering 或运行时处理**

## Dependencies / 依赖关系

- `basic-parsers.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `expr-parsers.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `misc-parsers.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `stmt-parser.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `token-parsers.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `type-parser-implementation.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Parser/characters.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
