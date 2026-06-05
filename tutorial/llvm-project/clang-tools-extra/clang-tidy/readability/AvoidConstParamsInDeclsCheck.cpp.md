# AvoidConstParamsInDeclsCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/readability/AvoidConstParamsInDeclsCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `AvoidConstParamsInDeclsCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `AvoidConstParamsInDeclsCheck`，包括 AST 匹配、诊断与自动修复行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
 1 | //===----------------------------------------------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | 
 9 | #include "AvoidConstParamsInDeclsCheck.h"
10 | #include "../utils/LexerUtils.h"
11 | #include "clang/ASTMatchers/ASTMatchFinder.h"
12 | #include "clang/ASTMatchers/ASTMatchers.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "AvoidConstParamsInDeclsCheck.h" to access local declarations from the current tool or check. / 引入 "AvoidConstParamsInDeclsCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Includes "../utils/LexerUtils.h" to access shared clang-tidy utility helpers. / 引入 "../utils/LexerUtils.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L11**: Includes "clang/ASTMatchers/ASTMatchFinder.h" to access AST matcher construction helpers. / 引入 "clang/ASTMatchers/ASTMatchFinder.h" 以使用AST 匹配器构造辅助逻辑。
- **L12**: Includes "clang/ASTMatchers/ASTMatchers.h" to access AST matcher construction helpers. / 引入 "clang/ASTMatchers/ASTMatchers.h" 以使用AST 匹配器构造辅助逻辑。

### Lines 13-24 / 第 13-24 行

```cpp
13 | #include "clang/Lex/Lexer.h"
14 | 
15 | using namespace clang::ast_matchers;
16 | 
17 | namespace clang::tidy::readability {
18 | 
19 | static SourceRange getTypeRange(const ParmVarDecl &Param) {
20 |   return {Param.getBeginLoc(), Param.getLocation().getLocWithOffset(-1)};
21 | }
22 | 
23 | // Finds the location of the qualifying `const` token in the `ParmValDecl`'s
24 | // return type. Returns `std::nullopt` when the parm type is not
```

- **L13**: Includes "clang/Lex/Lexer.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/Lexer.h" 以使用词法分析器与预处理器接口。
- **L14**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L15**: Brings namespace `clang::ast_matchers` into the local scope. / 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **L16**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L17**: Opens namespace scope `clang::tidy::readability`. / 打开命名空间作用域 `clang::tidy::readability`。
- **L18**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L19**: Starts a function, method, lambda, or structured scope: `static SourceRange getTypeRange(const ParmVarDecl &Param) {`. / 开始一个函数、方法、lambda 或结构化作用域：`static SourceRange getTypeRange(const ParmVarDecl &Param) {`。
- **L20**: Returns from the current function with `{Param.getBeginLoc(), Param.getLocation().getLocWithOffset(-1)}`. / 以 `{Param.getBeginLoc(), Param.getLocation().getLocWithOffset(-1)}` 从当前函数返回。
- **L21**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L22**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L23**: Comment explains nearby logic, intent, or usage: `Finds the location of the qualifying \`const\` token in the \`ParmValDecl\`'s`. / 注释说明了附近代码的逻辑、意图或用法：`Finds the location of the qualifying \`const\` token in the \`ParmValDecl\`'s`。
- **L24**: Comment explains nearby logic, intent, or usage: `return type. Returns \`std::nullopt\` when the parm type is not`. / 注释说明了附近代码的逻辑、意图或用法：`return type. Returns \`std::nullopt\` when the parm type is not`。

### Lines 25-36 / 第 25-36 行

```cpp
25 | // `const`-qualified like when the type is an alias or a macro.
26 | static std::optional<Token>
27 | findConstToRemove(const ParmVarDecl &Param,
28 |                   const MatchFinder::MatchResult &Result) {
29 |   const CharSourceRange FileRange = Lexer::makeFileCharRange(
30 |       CharSourceRange::getTokenRange(getTypeRange(Param)),
31 |       *Result.SourceManager, Result.Context->getLangOpts());
32 | 
33 |   if (FileRange.isInvalid())
34 |     return std::nullopt;
35 | 
36 |   return tidy::utils::lexer::getQualifyingToken(
```

- **L25**: Comment explains nearby logic, intent, or usage: `\`const\`-qualified like when the type is an alias or a macro.`. / 注释说明了附近代码的逻辑、意图或用法：`\`const\`-qualified like when the type is an alias or a macro.`。
- **L26**: Continues the surrounding expression or declaration: `static std::optional<Token>`. / 继续构造周围的表达式或声明：`static std::optional<Token>`。
- **L27**: Continues a multi-line argument list, initializer, or aggregate entry: `findConstToRemove(const ParmVarDecl &Param,`. / 继续一个多行参数列表、初始化器或聚合项：`findConstToRemove(const ParmVarDecl &Param,`。
- **L28**: Continues the surrounding expression or declaration: `const MatchFinder::MatchResult &Result) {`. / 继续构造周围的表达式或声明：`const MatchFinder::MatchResult &Result) {`。
- **L29**: Continues logic associated with callable symbol `makeFileCharRange`. / 继续与可调用符号 `makeFileCharRange` 相关的逻辑。
- **L30**: Continues a multi-line argument list, initializer, or aggregate entry: `CharSourceRange::getTokenRange(getTypeRange(Param)),`. / 继续一个多行参数列表、初始化器或聚合项：`CharSourceRange::getTokenRange(getTypeRange(Param)),`。
- **L31**: Comment explains nearby logic, intent, or usage: `Result.SourceManager, Result.Context->getLangOpts());`. / 注释说明了附近代码的逻辑、意图或用法：`Result.SourceManager, Result.Context->getLangOpts());`。
- **L32**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L33**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L34**: Returns from the current function with `std::nullopt`. / 以 `std::nullopt` 从当前函数返回。
- **L35**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L36**: Returns from the current function with `tidy::utils::lexer::getQualifyingToken(`. / 以 `tidy::utils::lexer::getQualifyingToken(` 从当前函数返回。

### Lines 37-48 / 第 37-48 行

```cpp
37 |       tok::kw_const, FileRange, *Result.Context, *Result.SourceManager);
38 | }
39 | 
40 | void AvoidConstParamsInDeclsCheck::storeOptions(
41 |     ClangTidyOptions::OptionMap &Opts) {
42 |   Options.store(Opts, "IgnoreMacros", IgnoreMacros);
43 | }
44 | 
45 | void AvoidConstParamsInDeclsCheck::registerMatchers(MatchFinder *Finder) {
46 |   const auto ConstParamDecl =
47 |       parmVarDecl(hasType(qualType(isConstQualified()))).bind("param");
48 |   Finder->addMatcher(functionDecl(unless(isDefinition()),
```

- **L37**: Executes a standalone statement or declaration: `tok::kw_const, FileRange, *Result.Context, *Result.SourceManager);`. / 执行一条独立语句或声明：`tok::kw_const, FileRange, *Result.Context, *Result.SourceManager);`。
- **L38**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L39**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L40**: Continues logic associated with callable symbol `storeOptions`. / 继续与可调用符号 `storeOptions` 相关的逻辑。
- **L41**: Continues the surrounding expression or declaration: `ClangTidyOptions::OptionMap &Opts) {`. / 继续构造周围的表达式或声明：`ClangTidyOptions::OptionMap &Opts) {`。
- **L42**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L43**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L44**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L45**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L46**: Continues the surrounding expression or declaration: `const auto ConstParamDecl =`. / 继续构造周围的表达式或声明：`const auto ConstParamDecl =`。
- **L47**: Executes a call or declaration centered on `parmVarDecl`. / 执行以 `parmVarDecl` 为核心的调用或声明。
- **L48**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。

### Lines 49-60 / 第 49-60 行

```cpp
49 |                                   has(typeLoc(forEach(ConstParamDecl))))
50 |                          .bind("func"),
51 |                      this);
52 | }
53 | 
54 | void AvoidConstParamsInDeclsCheck::check(
55 |     const MatchFinder::MatchResult &Result) {
56 |   const auto *Func = Result.Nodes.getNodeAs<FunctionDecl>("func");
57 |   const auto *Param = Result.Nodes.getNodeAs<ParmVarDecl>("param");
58 | 
59 |   if (!Param->getType().isLocalConstQualified())
60 |     return;
```

- **L49**: Continues logic associated with callable symbol `has`. / 继续与可调用符号 `has` 相关的逻辑。
- **L50**: Continues a multi-line argument list, initializer, or aggregate entry: `.bind("func"),`. / 继续一个多行参数列表、初始化器或聚合项：`.bind("func"),`。
- **L51**: Executes a standalone statement or declaration: `this);`. / 执行一条独立语句或声明：`this);`。
- **L52**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L53**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L54**: Continues logic associated with callable symbol `check`. / 继续与可调用符号 `check` 相关的逻辑。
- **L55**: Continues the surrounding expression or declaration: `const MatchFinder::MatchResult &Result) {`. / 继续构造周围的表达式或声明：`const MatchFinder::MatchResult &Result) {`。
- **L56**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<FunctionDecl>`. / 执行以 `Result.Nodes.getNodeAs<FunctionDecl>` 为核心的调用或声明。
- **L57**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<ParmVarDecl>`. / 执行以 `Result.Nodes.getNodeAs<ParmVarDecl>` 为核心的调用或声明。
- **L58**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L59**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L60**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。

### Lines 61-72 / 第 61-72 行

```cpp
61 | 
62 |   if (IgnoreMacros &&
63 |       (Param->getBeginLoc().isMacroID() || Param->getEndLoc().isMacroID())) {
64 |     // Suppress the check if macros are involved.
65 |     return;
66 |   }
67 | 
68 |   const auto Tok = findConstToRemove(*Param, Result);
69 |   const auto ConstLocation = Tok ? Tok->getLocation() : Param->getBeginLoc();
70 | 
71 |   auto Diag = diag(ConstLocation,
72 |                    "parameter %0 is const-qualified in the function "
```

- **L61**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L62**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L63**: Starts a function, method, lambda, or structured scope: `(Param->getBeginLoc().isMacroID() || Param->getEndLoc().isMacroID())) {`. / 开始一个函数、方法、lambda 或结构化作用域：`(Param->getBeginLoc().isMacroID() || Param->getEndLoc().isMacroID())) {`。
- **L64**: Comment explains nearby logic, intent, or usage: `Suppress the check if macros are involved.`. / 注释说明了附近代码的逻辑、意图或用法：`Suppress the check if macros are involved.`。
- **L65**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L66**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L67**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L68**: Initializes variable `Tok` from the right-hand expression. / 使用右侧表达式初始化变量 `Tok`。
- **L69**: Initializes variable `ConstLocation` from the right-hand expression. / 使用右侧表达式初始化变量 `ConstLocation`。
- **L70**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L71**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L72**: Continues the surrounding expression or declaration: `"parameter %0 is const-qualified in the function "`. / 继续构造周围的表达式或声明：`"parameter %0 is const-qualified in the function "`。

### Lines 73-84 / 第 73-84 行

```cpp
73 |                    "declaration; const-qualification of parameters only has an "
74 |                    "effect in function definitions");
75 |   if (Param->getName().empty()) {
76 |     for (unsigned int I = 0; I < Func->getNumParams(); ++I) {
77 |       if (Param == Func->getParamDecl(I)) {
78 |         Diag << (I + 1);
79 |         break;
80 |       }
81 |     }
82 |   } else {
83 |     Diag << Param;
84 |   }
```

- **L73**: Continues the surrounding expression or declaration: `"declaration; const-qualification of parameters only has an "`. / 继续构造周围的表达式或声明：`"declaration; const-qualification of parameters only has an "`。
- **L74**: Executes a standalone statement or declaration: `"effect in function definitions");`. / 执行一条独立语句或声明：`"effect in function definitions");`。
- **L75**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L76**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L77**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L78**: Executes a call or declaration centered on `<<`. / 执行以 `<<` 为核心的调用或声明。
- **L79**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L80**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L81**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L82**: Continues the surrounding expression or declaration: `} else {`. / 继续构造周围的表达式或声明：`} else {`。
- **L83**: Executes a standalone statement or declaration: `Diag << Param;`. / 执行一条独立语句或声明：`Diag << Param;`。
- **L84**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 85-96 / 第 85-96 行

```cpp
85 | 
86 |   if (Param->getBeginLoc().isMacroID() != Param->getEndLoc().isMacroID()) {
87 |     // Do not offer a suggestion if the part of the variable declaration comes
88 |     // from a macro.
89 |     return;
90 |   }
91 |   if (!Tok)
92 |     return;
93 | 
94 |   Diag << FixItHint::CreateRemoval(
95 |       CharSourceRange::getTokenRange(Tok->getLocation(), Tok->getLocation()));
96 | }
```

- **L85**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L86**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L87**: Comment explains nearby logic, intent, or usage: `Do not offer a suggestion if the part of the variable declaration comes`. / 注释说明了附近代码的逻辑、意图或用法：`Do not offer a suggestion if the part of the variable declaration comes`。
- **L88**: Comment explains nearby logic, intent, or usage: `from a macro.`. / 注释说明了附近代码的逻辑、意图或用法：`from a macro.`。
- **L89**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L90**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L91**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L92**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L93**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L94**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L95**: Executes a call or declaration centered on `CharSourceRange::getTokenRange`. / 执行以 `CharSourceRange::getTokenRange` 为核心的调用或声明。
- **L96**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 97-98 / 第 97-98 行

```cpp
97 | 
98 | } // namespace clang::tidy::readability
```

- **L97**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L98**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::readability`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::readability`。

## Key Concepts / 关键概念

- **Readability checking / 可读性检查**:
  - **EN**: Encourages clearer control flow, naming, and source structure.
  - **CN**: 鼓励更清晰的控制流、命名与源码结构。
- **AST matcher registration / AST 匹配器注册**:
  - **EN**: Builds matcher trees that let the check find relevant AST patterns.
  - **CN**: 构建匹配器树，使检查能够找到相关 AST 模式。
- **AST matcher callbacks / AST 匹配回调**:
  - **EN**: Routes matched AST nodes into check-specific callback logic.
  - **CN**: 把匹配到的 AST 节点路由到检查特定的回调逻辑。
- **Diagnostic emission / 诊断信息发出**:
  - **EN**: Reports issues at source locations with message parameters.
  - **CN**: 在源码位置上携带消息参数报告问题。
- **Automated fix-its / 自动修复提示**:
  - **EN**: Attaches machine-applicable edits to diagnostics whenever the transformation is safe.
  - **CN**: 在转换安全时为诊断附加可由机器应用的编辑。
- **Persistent options / 持久化选项**:
  - **EN**: Saves configurable behavior so checks can be tuned from .clang-tidy.
  - **CN**: 保存可配置行为，以便从 .clang-tidy 调整检查。

## Dependencies / 依赖关系

- `AvoidConstParamsInDeclsCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `../utils/LexerUtils.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `clang/ASTMatchers/ASTMatchFinder.h`: Provides AST matcher construction helpers. / 提供AST 匹配器构造辅助逻辑。
- `clang/ASTMatchers/ASTMatchers.h`: Provides AST matcher construction helpers. / 提供AST 匹配器构造辅助逻辑。
- `clang/Lex/Lexer.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
