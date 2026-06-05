# UseStdFormatCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/modernize/UseStdFormatCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `UseStdFormatCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `UseStdFormatCheck`，包括 AST 匹配、诊断与自动修复行为。

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
 9 | #include "UseStdFormatCheck.h"
10 | #include "../utils/FormatStringConverter.h"
11 | #include "../utils/Matchers.h"
12 | #include "../utils/OptionsUtils.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "UseStdFormatCheck.h" to access local declarations from the current tool or check. / 引入 "UseStdFormatCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Includes "../utils/FormatStringConverter.h" to access shared clang-tidy utility helpers. / 引入 "../utils/FormatStringConverter.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L11**: Includes "../utils/Matchers.h" to access shared clang-tidy utility helpers. / 引入 "../utils/Matchers.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L12**: Includes "../utils/OptionsUtils.h" to access shared clang-tidy utility helpers. / 引入 "../utils/OptionsUtils.h" 以使用共享 clang-tidy 工具辅助逻辑。

### Lines 13-24 / 第 13-24 行

```cpp
13 | #include "clang/ASTMatchers/ASTMatchFinder.h"
14 | #include "clang/Lex/Lexer.h"
15 | 
16 | using namespace clang::ast_matchers;
17 | 
18 | namespace clang::tidy::modernize {
19 | 
20 | namespace {
21 | AST_MATCHER(StringLiteral, isOrdinary) { return Node.isOrdinary(); }
22 | } // namespace
23 | 
24 | UseStdFormatCheck::UseStdFormatCheck(StringRef Name, ClangTidyContext *Context)
```

- **L13**: Includes "clang/ASTMatchers/ASTMatchFinder.h" to access AST matcher construction helpers. / 引入 "clang/ASTMatchers/ASTMatchFinder.h" 以使用AST 匹配器构造辅助逻辑。
- **L14**: Includes "clang/Lex/Lexer.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/Lexer.h" 以使用词法分析器与预处理器接口。
- **L15**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L16**: Brings namespace `clang::ast_matchers` into the local scope. / 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **L17**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L18**: Opens namespace scope `clang::tidy::modernize`. / 打开命名空间作用域 `clang::tidy::modernize`。
- **L19**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L20**: Opens an anonymous namespace for internal linkage. / 打开匿名命名空间以提供内部链接。
- **L21**: Defines a reusable AST matcher helper for later pattern matching logic. / 定义一个可复用的 AST 匹配器辅助组件，供后续模式匹配逻辑使用。
- **L22**: Closes a namespace scope while preserving the trailing comment: `} // namespace`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L23**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L24**: Continues logic associated with callable symbol `UseStdFormatCheck`. / 继续与可调用符号 `UseStdFormatCheck` 相关的逻辑。

### Lines 25-36 / 第 25-36 行

```cpp
25 |     : ClangTidyCheck(Name, Context),
26 |       StrictMode(Options.get("StrictMode", false)),
27 |       StrFormatLikeFunctions(utils::options::parseStringList(
28 |           Options.get("StrFormatLikeFunctions", ""))),
29 |       ReplacementFormatFunction(
30 |           Options.get("ReplacementFormatFunction", "std::format")),
31 |       IncludeInserter(Options.getLocalOrGlobal("IncludeStyle",
32 |                                                utils::IncludeSorter::IS_LLVM),
33 |                       areDiagsSelfContained()),
34 |       MaybeHeaderToInclude(Options.get("FormatHeader")) {
35 |   if (StrFormatLikeFunctions.empty())
36 |     StrFormatLikeFunctions.emplace_back("absl::StrFormat");
```

- **L25**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangTidyCheck(Name, Context),`. / 继续一个多行参数列表、初始化器或聚合项：`: ClangTidyCheck(Name, Context),`。
- **L26**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L27**: Continues logic associated with callable symbol `StrFormatLikeFunctions`. / 继续与可调用符号 `StrFormatLikeFunctions` 相关的逻辑。
- **L28**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L29**: Continues logic associated with callable symbol `ReplacementFormatFunction`. / 继续与可调用符号 `ReplacementFormatFunction` 相关的逻辑。
- **L30**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L31**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L32**: Continues a multi-line argument list, initializer, or aggregate entry: `utils::IncludeSorter::IS_LLVM),`. / 继续一个多行参数列表、初始化器或聚合项：`utils::IncludeSorter::IS_LLVM),`。
- **L33**: Continues a multi-line argument list, initializer, or aggregate entry: `areDiagsSelfContained()),`. / 继续一个多行参数列表、初始化器或聚合项：`areDiagsSelfContained()),`。
- **L34**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L35**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L36**: Executes a call or declaration centered on `StrFormatLikeFunctions.emplace_back`. / 执行以 `StrFormatLikeFunctions.emplace_back` 为核心的调用或声明。

### Lines 37-48 / 第 37-48 行

```cpp
37 | 
38 |   if (!MaybeHeaderToInclude && ReplacementFormatFunction == "std::format")
39 |     MaybeHeaderToInclude = "<format>";
40 | }
41 | 
42 | void UseStdFormatCheck::registerPPCallbacks(const SourceManager &SM,
43 |                                             Preprocessor *PP,
44 |                                             Preprocessor *ModuleExpanderPP) {
45 |   IncludeInserter.registerPreprocessor(PP);
46 |   this->PP = PP;
47 | }
48 | 
```

- **L37**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L38**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L39**: Assigns new state to `MaybeHeaderToInclude` for later logic. / 为后续逻辑给 `MaybeHeaderToInclude` 赋予新状态。
- **L40**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L41**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L42**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L43**: Continues a multi-line argument list, initializer, or aggregate entry: `Preprocessor *PP,`. / 继续一个多行参数列表、初始化器或聚合项：`Preprocessor *PP,`。
- **L44**: Continues the surrounding expression or declaration: `Preprocessor *ModuleExpanderPP) {`. / 继续构造周围的表达式或声明：`Preprocessor *ModuleExpanderPP) {`。
- **L45**: Uses include-insertion support to keep rewritten code compilable. / 使用头文件插入支持，以保持重写后代码可编译。
- **L46**: Executes a standalone statement or declaration: `this->PP = PP;`. / 执行一条独立语句或声明：`this->PP = PP;`。
- **L47**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L48**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 49-60 / 第 49-60 行

```cpp
49 | void UseStdFormatCheck::registerMatchers(MatchFinder *Finder) {
50 |   Finder->addMatcher(
51 |       callExpr(argumentCountAtLeast(1),
52 |                hasArgument(0, stringLiteral(isOrdinary())),
53 |                callee(functionDecl(matchers::matchesAnyListedRegexName(
54 |                                        StrFormatLikeFunctions))
55 |                           .bind("func_decl")))
56 |           .bind("strformat"),
57 |       this);
58 | }
59 | 
60 | void UseStdFormatCheck::storeOptions(ClangTidyOptions::OptionMap &Opts) {
```

- **L49**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L50**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L51**: Continues a multi-line argument list, initializer, or aggregate entry: `callExpr(argumentCountAtLeast(1),`. / 继续一个多行参数列表、初始化器或聚合项：`callExpr(argumentCountAtLeast(1),`。
- **L52**: Continues a multi-line argument list, initializer, or aggregate entry: `hasArgument(0, stringLiteral(isOrdinary())),`. / 继续一个多行参数列表、初始化器或聚合项：`hasArgument(0, stringLiteral(isOrdinary())),`。
- **L53**: Continues logic associated with callable symbol `callee`. / 继续与可调用符号 `callee` 相关的逻辑。
- **L54**: Continues the surrounding expression or declaration: `StrFormatLikeFunctions))`. / 继续构造周围的表达式或声明：`StrFormatLikeFunctions))`。
- **L55**: Continues logic associated with callable symbol `bind`. / 继续与可调用符号 `bind` 相关的逻辑。
- **L56**: Continues a multi-line argument list, initializer, or aggregate entry: `.bind("strformat"),`. / 继续一个多行参数列表、初始化器或聚合项：`.bind("strformat"),`。
- **L57**: Executes a standalone statement or declaration: `this);`. / 执行一条独立语句或声明：`this);`。
- **L58**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L59**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L60**: Starts a function, method, lambda, or structured scope: `void UseStdFormatCheck::storeOptions(ClangTidyOptions::OptionMap &Opts) {`. / 开始一个函数、方法、lambda 或结构化作用域：`void UseStdFormatCheck::storeOptions(ClangTidyOptions::OptionMap &Opts) {`。

### Lines 61-72 / 第 61-72 行

```cpp
61 |   using utils::options::serializeStringList;
62 |   Options.store(Opts, "StrictMode", StrictMode);
63 |   Options.store(Opts, "StrFormatLikeFunctions",
64 |                 serializeStringList(StrFormatLikeFunctions));
65 |   Options.store(Opts, "ReplacementFormatFunction", ReplacementFormatFunction);
66 |   Options.store(Opts, "IncludeStyle", IncludeInserter.getStyle());
67 |   if (MaybeHeaderToInclude)
68 |     Options.store(Opts, "FormatHeader", *MaybeHeaderToInclude);
69 | }
70 | 
71 | void UseStdFormatCheck::check(const MatchFinder::MatchResult &Result) {
72 |   const unsigned FormatArgOffset = 0;
```

- **L61**: Introduces a using declaration or alias: `using utils::options::serializeStringList;`. / 引入一条 using 声明或别名：`using utils::options::serializeStringList;`。
- **L62**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L63**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L64**: Executes a call or declaration centered on `serializeStringList`. / 执行以 `serializeStringList` 为核心的调用或声明。
- **L65**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L66**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L67**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L68**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L69**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L70**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L71**: Starts a function, method, lambda, or structured scope: `void UseStdFormatCheck::check(const MatchFinder::MatchResult &Result) {`. / 开始一个函数、方法、lambda 或结构化作用域：`void UseStdFormatCheck::check(const MatchFinder::MatchResult &Result) {`。
- **L72**: Initializes variable `FormatArgOffset` from the right-hand expression. / 使用右侧表达式初始化变量 `FormatArgOffset`。

### Lines 73-84 / 第 73-84 行

```cpp
73 |   const auto *OldFunction = Result.Nodes.getNodeAs<FunctionDecl>("func_decl");
74 |   const auto *StrFormat = Result.Nodes.getNodeAs<CallExpr>("strformat");
75 | 
76 |   utils::FormatStringConverter::Configuration ConverterConfig;
77 |   ConverterConfig.StrictMode = StrictMode;
78 |   utils::FormatStringConverter Converter(
79 |       Result.Context, StrFormat, FormatArgOffset, ConverterConfig,
80 |       getLangOpts(), *Result.SourceManager, *PP);
81 |   const Expr *StrFormatCall = StrFormat->getCallee();
82 |   if (!Converter.canApply()) {
83 |     diag(StrFormat->getBeginLoc(),
84 |          "unable to use '%0' instead of %1 because %2")
```

- **L73**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<FunctionDecl>`. / 执行以 `Result.Nodes.getNodeAs<FunctionDecl>` 为核心的调用或声明。
- **L74**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<CallExpr>`. / 执行以 `Result.Nodes.getNodeAs<CallExpr>` 为核心的调用或声明。
- **L75**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L76**: Executes a standalone statement or declaration: `utils::FormatStringConverter::Configuration ConverterConfig;`. / 执行一条独立语句或声明：`utils::FormatStringConverter::Configuration ConverterConfig;`。
- **L77**: Executes a standalone statement or declaration: `ConverterConfig.StrictMode = StrictMode;`. / 执行一条独立语句或声明：`ConverterConfig.StrictMode = StrictMode;`。
- **L78**: Continues logic associated with callable symbol `Converter`. / 继续与可调用符号 `Converter` 相关的逻辑。
- **L79**: Continues a multi-line argument list, initializer, or aggregate entry: `Result.Context, StrFormat, FormatArgOffset, ConverterConfig,`. / 继续一个多行参数列表、初始化器或聚合项：`Result.Context, StrFormat, FormatArgOffset, ConverterConfig,`。
- **L80**: Executes a call or declaration centered on `getLangOpts`. / 执行以 `getLangOpts` 为核心的调用或声明。
- **L81**: Executes a call or declaration centered on `StrFormat->getCallee`. / 执行以 `StrFormat->getCallee` 为核心的调用或声明。
- **L82**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L83**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L84**: Continues the surrounding expression or declaration: `"unable to use '%0' instead of %1 because %2")`. / 继续构造周围的表达式或声明：`"unable to use '%0' instead of %1 because %2")`。

### Lines 85-96 / 第 85-96 行

```cpp
85 |         << StrFormatCall->getSourceRange() << ReplacementFormatFunction
86 |         << OldFunction->getIdentifier()
87 |         << Converter.conversionNotPossibleReason();
88 |     return;
89 |   }
90 | 
91 |   DiagnosticBuilder Diag =
92 |       diag(StrFormatCall->getBeginLoc(), "use '%0' instead of %1")
93 |       << ReplacementFormatFunction << OldFunction->getIdentifier();
94 |   Diag << FixItHint::CreateReplacement(
95 |       CharSourceRange::getTokenRange(StrFormatCall->getExprLoc(),
96 |                                      StrFormatCall->getEndLoc()),
```

- **L85**: Continues logic associated with callable symbol `getSourceRange`. / 继续与可调用符号 `getSourceRange` 相关的逻辑。
- **L86**: Continues logic associated with callable symbol `getIdentifier`. / 继续与可调用符号 `getIdentifier` 相关的逻辑。
- **L87**: Executes a call or declaration centered on `Converter.conversionNotPossibleReason`. / 执行以 `Converter.conversionNotPossibleReason` 为核心的调用或声明。
- **L88**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L89**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L90**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L91**: Continues the surrounding expression or declaration: `DiagnosticBuilder Diag =`. / 继续构造周围的表达式或声明：`DiagnosticBuilder Diag =`。
- **L92**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L93**: Executes a call or declaration centered on `OldFunction->getIdentifier`. / 执行以 `OldFunction->getIdentifier` 为核心的调用或声明。
- **L94**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L95**: Continues a multi-line argument list, initializer, or aggregate entry: `CharSourceRange::getTokenRange(StrFormatCall->getExprLoc(),`. / 继续一个多行参数列表、初始化器或聚合项：`CharSourceRange::getTokenRange(StrFormatCall->getExprLoc(),`。
- **L96**: Continues a multi-line argument list, initializer, or aggregate entry: `StrFormatCall->getEndLoc()),`. / 继续一个多行参数列表、初始化器或聚合项：`StrFormatCall->getEndLoc()),`。

### Lines 97-107 / 第 97-107 行

```cpp
 97 |       ReplacementFormatFunction);
 98 |   Converter.applyFixes(Diag, *Result.SourceManager);
 99 | 
100 |   if (MaybeHeaderToInclude)
101 |     Diag << IncludeInserter.createIncludeInsertion(
102 |         Result.SourceManager->getFileID(Result.SourceManager->getExpansionLoc(
103 |             StrFormatCall->getBeginLoc())),
104 |         *MaybeHeaderToInclude);
105 | }
106 | 
107 | } // namespace clang::tidy::modernize
```

- **L97**: Executes a standalone statement or declaration: `ReplacementFormatFunction);`. / 执行一条独立语句或声明：`ReplacementFormatFunction);`。
- **L98**: Executes a call or declaration centered on `Converter.applyFixes`. / 执行以 `Converter.applyFixes` 为核心的调用或声明。
- **L99**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L100**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L101**: Uses include-insertion support to keep rewritten code compilable. / 使用头文件插入支持，以保持重写后代码可编译。
- **L102**: Continues logic associated with callable symbol `getFileID`. / 继续与可调用符号 `getFileID` 相关的逻辑。
- **L103**: Continues a multi-line argument list, initializer, or aggregate entry: `StrFormatCall->getBeginLoc())),`. / 继续一个多行参数列表、初始化器或聚合项：`StrFormatCall->getBeginLoc())),`。
- **L104**: Comment explains nearby logic, intent, or usage: `MaybeHeaderToInclude);`. / 注释说明了附近代码的逻辑、意图或用法：`MaybeHeaderToInclude);`。
- **L105**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L106**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L107**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::modernize`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::modernize`。

## Key Concepts / 关键概念

- **Modernization refactoring / 现代化重构**:
  - **EN**: Moves source code toward newer library facilities and safer modern idioms.
  - **CN**: 把源码迁移到更新的库设施与更安全的现代惯用法。
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
- **Preprocessor callbacks / 预处理器回调**:
  - **EN**: Listens to macro and include events before the AST is fully formed.
  - **CN**: 在 AST 完全形成前监听宏与包含事件。
- **Preprocessor integration / 预处理器集成**:
  - **EN**: Uses Clang preprocessor state to reason about directives and tokens.
  - **CN**: 利用 Clang 预处理器状态推断指令与记号。
- **Include management / 头文件管理**:
  - **EN**: Inserts headers when a rewrite depends on newly referenced library facilities.
  - **CN**: 当重写依赖新的库设施时插入相应头文件。

## Dependencies / 依赖关系

- `UseStdFormatCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `../utils/FormatStringConverter.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `../utils/Matchers.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `../utils/OptionsUtils.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `clang/ASTMatchers/ASTMatchFinder.h`: Provides AST matcher construction helpers. / 提供AST 匹配器构造辅助逻辑。
- `clang/Lex/Lexer.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
