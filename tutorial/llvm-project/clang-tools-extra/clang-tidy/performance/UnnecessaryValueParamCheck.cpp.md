# UnnecessaryValueParamCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/performance/UnnecessaryValueParamCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `UnnecessaryValueParamCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `UnnecessaryValueParamCheck`，包括 AST 匹配、诊断与自动修复行为。

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
 9 | #include "UnnecessaryValueParamCheck.h"
10 | #include "../utils/DeclRefExprUtils.h"
11 | #include "../utils/FixItHintUtils.h"
12 | #include "../utils/Matchers.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "UnnecessaryValueParamCheck.h" to access local declarations from the current tool or check. / 引入 "UnnecessaryValueParamCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Includes "../utils/DeclRefExprUtils.h" to access shared clang-tidy utility helpers. / 引入 "../utils/DeclRefExprUtils.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L11**: Includes "../utils/FixItHintUtils.h" to access shared clang-tidy utility helpers. / 引入 "../utils/FixItHintUtils.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L12**: Includes "../utils/Matchers.h" to access shared clang-tidy utility helpers. / 引入 "../utils/Matchers.h" 以使用共享 clang-tidy 工具辅助逻辑。

### Lines 13-24 / 第 13-24 行

```cpp
13 | #include "../utils/OptionsUtils.h"
14 | #include "../utils/TypeTraits.h"
15 | #include "clang/Frontend/CompilerInstance.h"
16 | #include "clang/Lex/Lexer.h"
17 | #include "clang/Lex/Preprocessor.h"
18 | #include <optional>
19 | 
20 | using namespace clang::ast_matchers;
21 | 
22 | namespace clang::tidy::performance {
23 | 
24 | static std::string paramNameOrIndex(StringRef Name, size_t Index) {
```

- **L13**: Includes "../utils/OptionsUtils.h" to access shared clang-tidy utility helpers. / 引入 "../utils/OptionsUtils.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L14**: Includes "../utils/TypeTraits.h" to access shared clang-tidy utility helpers. / 引入 "../utils/TypeTraits.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L15**: Includes "clang/Frontend/CompilerInstance.h" to access frontend action and compiler-instance APIs. / 引入 "clang/Frontend/CompilerInstance.h" 以使用前端动作与编译器实例 API。
- **L16**: Includes "clang/Lex/Lexer.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/Lexer.h" 以使用词法分析器与预处理器接口。
- **L17**: Includes "clang/Lex/Preprocessor.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/Preprocessor.h" 以使用词法分析器与预处理器接口。
- **L18**: Includes <optional> to access C or C++ standard library facilities. / 引入 <optional> 以使用C 或 C++ 标准库设施。
- **L19**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L20**: Brings namespace `clang::ast_matchers` into the local scope. / 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **L21**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L22**: Opens namespace scope `clang::tidy::performance`. / 打开命名空间作用域 `clang::tidy::performance`。
- **L23**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L24**: Starts a function, method, lambda, or structured scope: `static std::string paramNameOrIndex(StringRef Name, size_t Index) {`. / 开始一个函数、方法、lambda 或结构化作用域：`static std::string paramNameOrIndex(StringRef Name, size_t Index) {`。

### Lines 25-36 / 第 25-36 行

```cpp
25 |   return (Name.empty() ? llvm::Twine('#') + llvm::Twine(Index + 1)
26 |                        : llvm::Twine('\'') + Name + llvm::Twine('\''))
27 |       .str();
28 | }
29 | 
30 | static bool hasLoopStmtAncestor(const DeclRefExpr &DeclRef, const Decl &Decl,
31 |                                 ASTContext &Context) {
32 |   auto Matches = match(
33 |       traverse(TK_AsIs,
34 |                decl(forEachDescendant(declRefExpr(
35 |                    equalsNode(&DeclRef),
36 |                    unless(hasAncestor(stmt(anyOf(forStmt(), cxxForRangeStmt(),
```

- **L25**: Returns from the current function with `(Name.empty() ? llvm::Twine('#') + llvm::Twine(Index + 1)`. / 以 `(Name.empty() ? llvm::Twine('#') + llvm::Twine(Index + 1)` 从当前函数返回。
- **L26**: Continues logic associated with callable symbol `Twine`. / 继续与可调用符号 `Twine` 相关的逻辑。
- **L27**: Executes a call or declaration centered on `.str`. / 执行以 `.str` 为核心的调用或声明。
- **L28**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L29**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L30**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool hasLoopStmtAncestor(const DeclRefExpr &DeclRef, const Decl &Decl,`. / 继续一个多行参数列表、初始化器或聚合项：`static bool hasLoopStmtAncestor(const DeclRefExpr &DeclRef, const Decl &Decl,`。
- **L31**: Continues the surrounding expression or declaration: `ASTContext &Context) {`. / 继续构造周围的表达式或声明：`ASTContext &Context) {`。
- **L32**: Continues logic associated with callable symbol `match`. / 继续与可调用符号 `match` 相关的逻辑。
- **L33**: Continues a multi-line argument list, initializer, or aggregate entry: `traverse(TK_AsIs,`. / 继续一个多行参数列表、初始化器或聚合项：`traverse(TK_AsIs,`。
- **L34**: Continues logic associated with callable symbol `decl`. / 继续与可调用符号 `decl` 相关的逻辑。
- **L35**: Continues a multi-line argument list, initializer, or aggregate entry: `equalsNode(&DeclRef),`. / 继续一个多行参数列表、初始化器或聚合项：`equalsNode(&DeclRef),`。
- **L36**: Continues a multi-line argument list, initializer, or aggregate entry: `unless(hasAncestor(stmt(anyOf(forStmt(), cxxForRangeStmt(),`. / 继续一个多行参数列表、初始化器或聚合项：`unless(hasAncestor(stmt(anyOf(forStmt(), cxxForRangeStmt(),`。

### Lines 37-48 / 第 37-48 行

```cpp
37 |                                                  whileStmt(), doStmt())))))))),
38 |       Decl, Context);
39 |   return Matches.empty();
40 | }
41 | 
42 | UnnecessaryValueParamCheck::UnnecessaryValueParamCheck(
43 |     StringRef Name, ClangTidyContext *Context)
44 |     : ClangTidyCheck(Name, Context),
45 |       Inserter(Options.getLocalOrGlobal("IncludeStyle",
46 |                                         utils::IncludeSorter::IS_LLVM),
47 |                areDiagsSelfContained()),
48 |       AllowedTypes(
```

- **L37**: Continues a multi-line argument list, initializer, or aggregate entry: `whileStmt(), doStmt())))))))),`. / 继续一个多行参数列表、初始化器或聚合项：`whileStmt(), doStmt())))))))),`。
- **L38**: Executes a standalone statement or declaration: `Decl, Context);`. / 执行一条独立语句或声明：`Decl, Context);`。
- **L39**: Returns from the current function with `Matches.empty()`. / 以 `Matches.empty()` 从当前函数返回。
- **L40**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L41**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L42**: Continues logic associated with callable symbol `UnnecessaryValueParamCheck`. / 继续与可调用符号 `UnnecessaryValueParamCheck` 相关的逻辑。
- **L43**: Continues the surrounding expression or declaration: `StringRef Name, ClangTidyContext *Context)`. / 继续构造周围的表达式或声明：`StringRef Name, ClangTidyContext *Context)`。
- **L44**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangTidyCheck(Name, Context),`. / 继续一个多行参数列表、初始化器或聚合项：`: ClangTidyCheck(Name, Context),`。
- **L45**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L46**: Continues a multi-line argument list, initializer, or aggregate entry: `utils::IncludeSorter::IS_LLVM),`. / 继续一个多行参数列表、初始化器或聚合项：`utils::IncludeSorter::IS_LLVM),`。
- **L47**: Continues a multi-line argument list, initializer, or aggregate entry: `areDiagsSelfContained()),`. / 继续一个多行参数列表、初始化器或聚合项：`areDiagsSelfContained()),`。
- **L48**: Continues logic associated with callable symbol `AllowedTypes`. / 继续与可调用符号 `AllowedTypes` 相关的逻辑。

### Lines 49-60 / 第 49-60 行

```cpp
49 |           utils::options::parseStringList(Options.get("AllowedTypes", ""))),
50 |       IgnoreCoroutines(Options.get("IgnoreCoroutines", true)) {}
51 | 
52 | void UnnecessaryValueParamCheck::registerMatchers(MatchFinder *Finder) {
53 |   const auto ExpensiveValueParamDecl = parmVarDecl(
54 |       hasType(qualType(hasCanonicalType(matchers::isExpensiveToCopy()),
55 |                        unless(anyOf(hasCanonicalType(referenceType()),
56 |                                     hasDeclaration(namedDecl(
57 |                                         matchers::matchesAnyListedRegexName(
58 |                                             AllowedTypes))))))),
59 |       decl().bind("param"));
60 |   Finder->addMatcher(
```

- **L49**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L50**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L51**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L52**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L53**: Continues logic associated with callable symbol `parmVarDecl`. / 继续与可调用符号 `parmVarDecl` 相关的逻辑。
- **L54**: Continues a multi-line argument list, initializer, or aggregate entry: `hasType(qualType(hasCanonicalType(matchers::isExpensiveToCopy()),`. / 继续一个多行参数列表、初始化器或聚合项：`hasType(qualType(hasCanonicalType(matchers::isExpensiveToCopy()),`。
- **L55**: Continues a multi-line argument list, initializer, or aggregate entry: `unless(anyOf(hasCanonicalType(referenceType()),`. / 继续一个多行参数列表、初始化器或聚合项：`unless(anyOf(hasCanonicalType(referenceType()),`。
- **L56**: Continues logic associated with callable symbol `hasDeclaration`. / 继续与可调用符号 `hasDeclaration` 相关的逻辑。
- **L57**: Continues logic associated with callable symbol `matchesAnyListedRegexName`. / 继续与可调用符号 `matchesAnyListedRegexName` 相关的逻辑。
- **L58**: Continues a multi-line argument list, initializer, or aggregate entry: `AllowedTypes))))))),`. / 继续一个多行参数列表、初始化器或聚合项：`AllowedTypes))))))),`。
- **L59**: Executes a call or declaration centered on `decl`. / 执行以 `decl` 为核心的调用或声明。
- **L60**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。

### Lines 61-72 / 第 61-72 行

```cpp
61 |       traverse(TK_AsIs,
62 |                functionDecl(
63 |                    hasBody(IgnoreCoroutines ? stmt(unless(coroutineBodyStmt()))
64 |                                             : stmt()),
65 |                    isDefinition(), unless(isImplicit()),
66 |                    unless(cxxMethodDecl(anyOf(isOverride(), isFinal()))),
67 |                    has(typeLoc(forEach(ExpensiveValueParamDecl))),
68 |                    decl().bind("functionDecl"))),
69 |       this);
70 | }
71 | 
72 | void UnnecessaryValueParamCheck::check(const MatchFinder::MatchResult &Result) {
```

- **L61**: Continues a multi-line argument list, initializer, or aggregate entry: `traverse(TK_AsIs,`. / 继续一个多行参数列表、初始化器或聚合项：`traverse(TK_AsIs,`。
- **L62**: Continues logic associated with callable symbol `functionDecl`. / 继续与可调用符号 `functionDecl` 相关的逻辑。
- **L63**: Continues logic associated with callable symbol `hasBody`. / 继续与可调用符号 `hasBody` 相关的逻辑。
- **L64**: Continues a multi-line argument list, initializer, or aggregate entry: `: stmt()),`. / 继续一个多行参数列表、初始化器或聚合项：`: stmt()),`。
- **L65**: Continues a multi-line argument list, initializer, or aggregate entry: `isDefinition(), unless(isImplicit()),`. / 继续一个多行参数列表、初始化器或聚合项：`isDefinition(), unless(isImplicit()),`。
- **L66**: Continues a multi-line argument list, initializer, or aggregate entry: `unless(cxxMethodDecl(anyOf(isOverride(), isFinal()))),`. / 继续一个多行参数列表、初始化器或聚合项：`unless(cxxMethodDecl(anyOf(isOverride(), isFinal()))),`。
- **L67**: Continues a multi-line argument list, initializer, or aggregate entry: `has(typeLoc(forEach(ExpensiveValueParamDecl))),`. / 继续一个多行参数列表、初始化器或聚合项：`has(typeLoc(forEach(ExpensiveValueParamDecl))),`。
- **L68**: Continues a multi-line argument list, initializer, or aggregate entry: `decl().bind("functionDecl"))),`. / 继续一个多行参数列表、初始化器或聚合项：`decl().bind("functionDecl"))),`。
- **L69**: Executes a standalone statement or declaration: `this);`. / 执行一条独立语句或声明：`this);`。
- **L70**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L71**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L72**: Starts a function, method, lambda, or structured scope: `void UnnecessaryValueParamCheck::check(const MatchFinder::MatchResult &Result) {`. / 开始一个函数、方法、lambda 或结构化作用域：`void UnnecessaryValueParamCheck::check(const MatchFinder::MatchResult &Result) {`。

### Lines 73-84 / 第 73-84 行

```cpp
73 |   const auto *Param = Result.Nodes.getNodeAs<ParmVarDecl>("param");
74 |   const auto *Function = Result.Nodes.getNodeAs<FunctionDecl>("functionDecl");
75 | 
76 |   const TraversalKindScope RAII(*Result.Context, TK_AsIs);
77 | 
78 |   FunctionParmMutationAnalyzer *Analyzer =
79 |       FunctionParmMutationAnalyzer::getFunctionParmMutationAnalyzer(
80 |           *Function, *Result.Context, MutationAnalyzerCache);
81 |   if (Analyzer->isMutated(Param))
82 |     return;
83 | 
84 |   const bool IsConstQualified =
```

- **L73**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<ParmVarDecl>`. / 执行以 `Result.Nodes.getNodeAs<ParmVarDecl>` 为核心的调用或声明。
- **L74**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<FunctionDecl>`. / 执行以 `Result.Nodes.getNodeAs<FunctionDecl>` 为核心的调用或声明。
- **L75**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L76**: Executes a call or declaration centered on `RAII`. / 执行以 `RAII` 为核心的调用或声明。
- **L77**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L78**: Continues the surrounding expression or declaration: `FunctionParmMutationAnalyzer *Analyzer =`. / 继续构造周围的表达式或声明：`FunctionParmMutationAnalyzer *Analyzer =`。
- **L79**: Continues logic associated with callable symbol `getFunctionParmMutationAnalyzer`. / 继续与可调用符号 `getFunctionParmMutationAnalyzer` 相关的逻辑。
- **L80**: Comment explains nearby logic, intent, or usage: `Function, *Result.Context, MutationAnalyzerCache);`. / 注释说明了附近代码的逻辑、意图或用法：`Function, *Result.Context, MutationAnalyzerCache);`。
- **L81**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L82**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L83**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L84**: Continues the surrounding expression or declaration: `const bool IsConstQualified =`. / 继续构造周围的表达式或声明：`const bool IsConstQualified =`。

### Lines 85-96 / 第 85-96 行

```cpp
85 |       Param->getType().getCanonicalType().isConstQualified();
86 | 
87 |   // If the parameter is non-const, check if it has a move constructor and is
88 |   // only referenced once to copy-construct another object or whether it has a
89 |   // move assignment operator and is only referenced once when copy-assigned.
90 |   // In this case wrap DeclRefExpr with std::move() to avoid the unnecessary
91 |   // copy.
92 |   if (!IsConstQualified) {
93 |     auto AllDeclRefExprs = utils::decl_ref_expr::allDeclRefExprs(
94 |         *Param, *Function, *Result.Context);
95 |     if (AllDeclRefExprs.size() == 1) {
96 |       auto CanonicalType = Param->getType().getCanonicalType();
```

- **L85**: Executes a call or declaration centered on `Param->getType`. / 执行以 `Param->getType` 为核心的调用或声明。
- **L86**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L87**: Comment explains nearby logic, intent, or usage: `If the parameter is non-const, check if it has a move constructor and is`. / 注释说明了附近代码的逻辑、意图或用法：`If the parameter is non-const, check if it has a move constructor and is`。
- **L88**: Comment explains nearby logic, intent, or usage: `only referenced once to copy-construct another object or whether it has a`. / 注释说明了附近代码的逻辑、意图或用法：`only referenced once to copy-construct another object or whether it has a`。
- **L89**: Comment explains nearby logic, intent, or usage: `move assignment operator and is only referenced once when copy-assigned.`. / 注释说明了附近代码的逻辑、意图或用法：`move assignment operator and is only referenced once when copy-assigned.`。
- **L90**: Comment explains nearby logic, intent, or usage: `In this case wrap DeclRefExpr with std::move() to avoid the unnecessary`. / 注释说明了附近代码的逻辑、意图或用法：`In this case wrap DeclRefExpr with std::move() to avoid the unnecessary`。
- **L91**: Comment explains nearby logic, intent, or usage: `copy.`. / 注释说明了附近代码的逻辑、意图或用法：`copy.`。
- **L92**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L93**: Continues logic associated with callable symbol `allDeclRefExprs`. / 继续与可调用符号 `allDeclRefExprs` 相关的逻辑。
- **L94**: Comment explains nearby logic, intent, or usage: `Param, *Function, *Result.Context);`. / 注释说明了附近代码的逻辑、意图或用法：`Param, *Function, *Result.Context);`。
- **L95**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L96**: Initializes variable `CanonicalType` from the right-hand expression. / 使用右侧表达式初始化变量 `CanonicalType`。

### Lines 97-108 / 第 97-108 行

```cpp
 97 |       const auto &DeclRefExpr = **AllDeclRefExprs.begin();
 98 | 
 99 |       if (!hasLoopStmtAncestor(DeclRefExpr, *Function, *Result.Context) &&
100 |           ((utils::type_traits::hasNonTrivialMoveConstructor(CanonicalType) &&
101 |             utils::decl_ref_expr::isCopyConstructorArgument(
102 |                 DeclRefExpr, *Function, *Result.Context)) ||
103 |            (utils::type_traits::hasNonTrivialMoveAssignment(CanonicalType) &&
104 |             utils::decl_ref_expr::isCopyAssignmentArgument(
105 |                 DeclRefExpr, *Function, *Result.Context)))) {
106 |         handleMoveFix(*Param, DeclRefExpr, *Result.Context);
107 |         return;
108 |       }
```

- **L97**: Executes a call or declaration centered on `**AllDeclRefExprs.begin`. / 执行以 `**AllDeclRefExprs.begin` 为核心的调用或声明。
- **L98**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L99**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L100**: Continues logic associated with callable symbol `hasNonTrivialMoveConstructor`. / 继续与可调用符号 `hasNonTrivialMoveConstructor` 相关的逻辑。
- **L101**: Continues logic associated with callable symbol `isCopyConstructorArgument`. / 继续与可调用符号 `isCopyConstructorArgument` 相关的逻辑。
- **L102**: Continues the surrounding expression or declaration: `DeclRefExpr, *Function, *Result.Context)) ||`. / 继续构造周围的表达式或声明：`DeclRefExpr, *Function, *Result.Context)) ||`。
- **L103**: Continues logic associated with callable symbol `hasNonTrivialMoveAssignment`. / 继续与可调用符号 `hasNonTrivialMoveAssignment` 相关的逻辑。
- **L104**: Continues logic associated with callable symbol `isCopyAssignmentArgument`. / 继续与可调用符号 `isCopyAssignmentArgument` 相关的逻辑。
- **L105**: Continues the surrounding expression or declaration: `DeclRefExpr, *Function, *Result.Context)))) {`. / 继续构造周围的表达式或声明：`DeclRefExpr, *Function, *Result.Context)))) {`。
- **L106**: Executes a call or declaration centered on `handleMoveFix`. / 执行以 `handleMoveFix` 为核心的调用或声明。
- **L107**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L108**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 109-120 / 第 109-120 行

```cpp
109 |     }
110 |   }
111 | 
112 |   handleConstRefFix(*Function, *Param, *Result.Context);
113 | }
114 | 
115 | void UnnecessaryValueParamCheck::registerPPCallbacks(
116 |     const SourceManager &SM, Preprocessor *PP, Preprocessor *ModuleExpanderPP) {
117 |   Inserter.registerPreprocessor(PP);
118 | }
119 | 
120 | void UnnecessaryValueParamCheck::storeOptions(
```

- **L109**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L110**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L111**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L112**: Executes a call or declaration centered on `handleConstRefFix`. / 执行以 `handleConstRefFix` 为核心的调用或声明。
- **L113**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L114**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L115**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L116**: Continues the surrounding expression or declaration: `const SourceManager &SM, Preprocessor *PP, Preprocessor *ModuleExpanderPP) {`. / 继续构造周围的表达式或声明：`const SourceManager &SM, Preprocessor *PP, Preprocessor *ModuleExpanderPP) {`。
- **L117**: Executes a call or declaration centered on `Inserter.registerPreprocessor`. / 执行以 `Inserter.registerPreprocessor` 为核心的调用或声明。
- **L118**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L119**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L120**: Continues logic associated with callable symbol `storeOptions`. / 继续与可调用符号 `storeOptions` 相关的逻辑。

### Lines 121-132 / 第 121-132 行

```cpp
121 |     ClangTidyOptions::OptionMap &Opts) {
122 |   Options.store(Opts, "IncludeStyle", Inserter.getStyle());
123 |   Options.store(Opts, "AllowedTypes",
124 |                 utils::options::serializeStringList(AllowedTypes));
125 |   Options.store(Opts, "IgnoreCoroutines", IgnoreCoroutines);
126 | }
127 | 
128 | void UnnecessaryValueParamCheck::onEndOfTranslationUnit() {
129 |   MutationAnalyzerCache.clear();
130 | }
131 | 
132 | void UnnecessaryValueParamCheck::handleConstRefFix(const FunctionDecl &Function,
```

- **L121**: Continues the surrounding expression or declaration: `ClangTidyOptions::OptionMap &Opts) {`. / 继续构造周围的表达式或声明：`ClangTidyOptions::OptionMap &Opts) {`。
- **L122**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L123**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L124**: Executes a call or declaration centered on `utils::options::serializeStringList`. / 执行以 `utils::options::serializeStringList` 为核心的调用或声明。
- **L125**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L126**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L127**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L128**: Starts a function, method, lambda, or structured scope: `void UnnecessaryValueParamCheck::onEndOfTranslationUnit() {`. / 开始一个函数、方法、lambda 或结构化作用域：`void UnnecessaryValueParamCheck::onEndOfTranslationUnit() {`。
- **L129**: Executes a call or declaration centered on `MutationAnalyzerCache.clear`. / 执行以 `MutationAnalyzerCache.clear` 为核心的调用或声明。
- **L130**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L131**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L132**: Continues a multi-line argument list, initializer, or aggregate entry: `void UnnecessaryValueParamCheck::handleConstRefFix(const FunctionDecl &Function,`. / 继续一个多行参数列表、初始化器或聚合项：`void UnnecessaryValueParamCheck::handleConstRefFix(const FunctionDecl &Function,`。

### Lines 133-144 / 第 133-144 行

```cpp
133 |                                                    const ParmVarDecl &Param,
134 |                                                    ASTContext &Context) {
135 |   const size_t Index =
136 |       llvm::find(Function.parameters(), &Param) - Function.parameters().begin();
137 |   const bool IsConstQualified =
138 |       Param.getType().getCanonicalType().isConstQualified();
139 | 
140 |   auto Diag =
141 |       diag(Param.getLocation(),
142 |            "the %select{|const qualified }0parameter %1 of type %2 is copied "
143 |            "for each "
144 |            "invocation%select{ but only used as a const reference|}0; consider "
```

- **L133**: Continues a multi-line argument list, initializer, or aggregate entry: `const ParmVarDecl &Param,`. / 继续一个多行参数列表、初始化器或聚合项：`const ParmVarDecl &Param,`。
- **L134**: Continues the surrounding expression or declaration: `ASTContext &Context) {`. / 继续构造周围的表达式或声明：`ASTContext &Context) {`。
- **L135**: Continues the surrounding expression or declaration: `const size_t Index =`. / 继续构造周围的表达式或声明：`const size_t Index =`。
- **L136**: Executes a call or declaration centered on `llvm::find`. / 执行以 `llvm::find` 为核心的调用或声明。
- **L137**: Continues the surrounding expression or declaration: `const bool IsConstQualified =`. / 继续构造周围的表达式或声明：`const bool IsConstQualified =`。
- **L138**: Executes a call or declaration centered on `Param.getType`. / 执行以 `Param.getType` 为核心的调用或声明。
- **L139**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L140**: Continues the surrounding expression or declaration: `auto Diag =`. / 继续构造周围的表达式或声明：`auto Diag =`。
- **L141**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L142**: Continues the surrounding expression or declaration: `"the %select{|const qualified }0parameter %1 of type %2 is copied "`. / 继续构造周围的表达式或声明：`"the %select{|const qualified }0parameter %1 of type %2 is copied "`。
- **L143**: Continues the surrounding expression or declaration: `"for each "`. / 继续构造周围的表达式或声明：`"for each "`。
- **L144**: Continues the surrounding expression or declaration: `"invocation%select{ but only used as a const reference|}0; consider "`. / 继续构造周围的表达式或声明：`"invocation%select{ but only used as a const reference|}0; consider "`。

### Lines 145-156 / 第 145-156 行

```cpp
145 |            "making it a %select{const |}0reference")
146 |       << IsConstQualified << paramNameOrIndex(Param.getName(), Index)
147 |       << Param.getType();
148 |   // Do not propose fixes when:
149 |   // 1. the ParmVarDecl is in a macro, since we cannot place them correctly
150 |   // 2. the function is virtual as it might break overrides
151 |   // 3. the function is an explicit template/ specialization.
152 |   const auto *Method = dyn_cast<CXXMethodDecl>(&Function);
153 |   if (Param.getBeginLoc().isMacroID() || (Method && Method->isVirtual()) ||
154 |       Function.getTemplateSpecializationKind() == TSK_ExplicitSpecialization)
155 |     return;
156 |   for (const auto *FunctionDecl = &Function; FunctionDecl != nullptr;
```

- **L145**: Continues the surrounding expression or declaration: `"making it a %select{const |}0reference")`. / 继续构造周围的表达式或声明：`"making it a %select{const |}0reference")`。
- **L146**: Continues logic associated with callable symbol `paramNameOrIndex`. / 继续与可调用符号 `paramNameOrIndex` 相关的逻辑。
- **L147**: Executes a call or declaration centered on `Param.getType`. / 执行以 `Param.getType` 为核心的调用或声明。
- **L148**: Comment explains nearby logic, intent, or usage: `Do not propose fixes when:`. / 注释说明了附近代码的逻辑、意图或用法：`Do not propose fixes when:`。
- **L149**: Comment explains nearby logic, intent, or usage: `1. the ParmVarDecl is in a macro, since we cannot place them correctly`. / 注释说明了附近代码的逻辑、意图或用法：`1. the ParmVarDecl is in a macro, since we cannot place them correctly`。
- **L150**: Comment explains nearby logic, intent, or usage: `2. the function is virtual as it might break overrides`. / 注释说明了附近代码的逻辑、意图或用法：`2. the function is virtual as it might break overrides`。
- **L151**: Comment explains nearby logic, intent, or usage: `3. the function is an explicit template/ specialization.`. / 注释说明了附近代码的逻辑、意图或用法：`3. the function is an explicit template/ specialization.`。
- **L152**: Executes a call or declaration centered on `dyn_cast<CXXMethodDecl>`. / 执行以 `dyn_cast<CXXMethodDecl>` 为核心的调用或声明。
- **L153**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L154**: Continues logic associated with callable symbol `getTemplateSpecializationKind`. / 继续与可调用符号 `getTemplateSpecializationKind` 相关的逻辑。
- **L155**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L156**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。

### Lines 157-168 / 第 157-168 行

```cpp
157 |        FunctionDecl = FunctionDecl->getPreviousDecl()) {
158 |     const auto &CurrentParam = *FunctionDecl->getParamDecl(Index);
159 |     Diag << utils::fixit::changeVarDeclToReference(CurrentParam, Context);
160 |     // The parameter of each declaration needs to be checked individually as to
161 |     // whether it is const or not as constness can differ between definition and
162 |     // declaration.
163 |     if (!CurrentParam.getType().getCanonicalType().isConstQualified()) {
164 |       if (std::optional<FixItHint> Fix = utils::fixit::addQualifierToVarDecl(
165 |               CurrentParam, Context, Qualifiers::Const))
166 |         Diag << *Fix;
167 |     }
168 |   }
```

- **L157**: Starts a function, method, lambda, or structured scope: `FunctionDecl = FunctionDecl->getPreviousDecl()) {`. / 开始一个函数、方法、lambda 或结构化作用域：`FunctionDecl = FunctionDecl->getPreviousDecl()) {`。
- **L158**: Executes a call or declaration centered on `*FunctionDecl->getParamDecl`. / 执行以 `*FunctionDecl->getParamDecl` 为核心的调用或声明。
- **L159**: Executes a call or declaration centered on `utils::fixit::changeVarDeclToReference`. / 执行以 `utils::fixit::changeVarDeclToReference` 为核心的调用或声明。
- **L160**: Comment explains nearby logic, intent, or usage: `The parameter of each declaration needs to be checked individually as to`. / 注释说明了附近代码的逻辑、意图或用法：`The parameter of each declaration needs to be checked individually as to`。
- **L161**: Comment explains nearby logic, intent, or usage: `whether it is const or not as constness can differ between definition and`. / 注释说明了附近代码的逻辑、意图或用法：`whether it is const or not as constness can differ between definition and`。
- **L162**: Comment explains nearby logic, intent, or usage: `declaration.`. / 注释说明了附近代码的逻辑、意图或用法：`declaration.`。
- **L163**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L164**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L165**: Continues the surrounding expression or declaration: `CurrentParam, Context, Qualifiers::Const))`. / 继续构造周围的表达式或声明：`CurrentParam, Context, Qualifiers::Const))`。
- **L166**: Executes a standalone statement or declaration: `Diag << *Fix;`. / 执行一条独立语句或声明：`Diag << *Fix;`。
- **L167**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L168**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 169-180 / 第 169-180 行

```cpp
169 | }
170 | 
171 | void UnnecessaryValueParamCheck::handleMoveFix(const ParmVarDecl &Param,
172 |                                                const DeclRefExpr &CopyArgument,
173 |                                                ASTContext &Context) {
174 |   auto Diag =
175 |       diag(CopyArgument.getBeginLoc(),
176 |            "parameter %0 of type %1 is passed by value and only copied once; "
177 |            "consider moving it to avoid unnecessary copies")
178 |       << &Param << Param.getType();
179 |   // Do not propose fixes in macros since we cannot place them correctly.
180 |   if (CopyArgument.getBeginLoc().isMacroID())
```

- **L169**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L170**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L171**: Continues a multi-line argument list, initializer, or aggregate entry: `void UnnecessaryValueParamCheck::handleMoveFix(const ParmVarDecl &Param,`. / 继续一个多行参数列表、初始化器或聚合项：`void UnnecessaryValueParamCheck::handleMoveFix(const ParmVarDecl &Param,`。
- **L172**: Continues a multi-line argument list, initializer, or aggregate entry: `const DeclRefExpr &CopyArgument,`. / 继续一个多行参数列表、初始化器或聚合项：`const DeclRefExpr &CopyArgument,`。
- **L173**: Continues the surrounding expression or declaration: `ASTContext &Context) {`. / 继续构造周围的表达式或声明：`ASTContext &Context) {`。
- **L174**: Continues the surrounding expression or declaration: `auto Diag =`. / 继续构造周围的表达式或声明：`auto Diag =`。
- **L175**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L176**: Continues the surrounding expression or declaration: `"parameter %0 of type %1 is passed by value and only copied once; "`. / 继续构造周围的表达式或声明：`"parameter %0 of type %1 is passed by value and only copied once; "`。
- **L177**: Continues the surrounding expression or declaration: `"consider moving it to avoid unnecessary copies")`. / 继续构造周围的表达式或声明：`"consider moving it to avoid unnecessary copies")`。
- **L178**: Executes a call or declaration centered on `Param.getType`. / 执行以 `Param.getType` 为核心的调用或声明。
- **L179**: Comment explains nearby logic, intent, or usage: `Do not propose fixes in macros since we cannot place them correctly.`. / 注释说明了附近代码的逻辑、意图或用法：`Do not propose fixes in macros since we cannot place them correctly.`。
- **L180**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。

### Lines 181-191 / 第 181-191 行

```cpp
181 |     return;
182 |   const auto &SM = Context.getSourceManager();
183 |   auto EndLoc = Lexer::getLocForEndOfToken(CopyArgument.getLocation(), 0, SM,
184 |                                            Context.getLangOpts());
185 |   Diag << FixItHint::CreateInsertion(CopyArgument.getBeginLoc(), "std::move(")
186 |        << FixItHint::CreateInsertion(EndLoc, ")")
187 |        << Inserter.createIncludeInsertion(
188 |               SM.getFileID(CopyArgument.getBeginLoc()), "<utility>");
189 | }
190 | 
191 | } // namespace clang::tidy::performance
```

- **L181**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L182**: Executes a call or declaration centered on `Context.getSourceManager`. / 执行以 `Context.getSourceManager` 为核心的调用或声明。
- **L183**: Continues a multi-line argument list, initializer, or aggregate entry: `auto EndLoc = Lexer::getLocForEndOfToken(CopyArgument.getLocation(), 0, SM,`. / 继续一个多行参数列表、初始化器或聚合项：`auto EndLoc = Lexer::getLocForEndOfToken(CopyArgument.getLocation(), 0, SM,`。
- **L184**: Executes a call or declaration centered on `Context.getLangOpts`. / 执行以 `Context.getLangOpts` 为核心的调用或声明。
- **L185**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L186**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L187**: Continues logic associated with callable symbol `createIncludeInsertion`. / 继续与可调用符号 `createIncludeInsertion` 相关的逻辑。
- **L188**: Executes a call or declaration centered on `SM.getFileID`. / 执行以 `SM.getFileID` 为核心的调用或声明。
- **L189**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L190**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L191**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::performance`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::performance`。

## Key Concepts / 关键概念

- **Performance diagnostics / 性能诊断**:
  - **EN**: Looks for unnecessary copies, allocations, conversions, or other avoidable costs.
  - **CN**: 查找不必要的拷贝、分配、转换或其他可避免的开销。
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

## Dependencies / 依赖关系

- `UnnecessaryValueParamCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `../utils/DeclRefExprUtils.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `../utils/FixItHintUtils.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `../utils/Matchers.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `../utils/OptionsUtils.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `../utils/TypeTraits.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `clang/Frontend/CompilerInstance.h`: Provides frontend action and compiler-instance APIs. / 提供前端动作与编译器实例 API。
- `clang/Lex/Lexer.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
- `clang/Lex/Preprocessor.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
- `optional`: Provides C or C++ standard library facilities. / 提供C 或 C++ 标准库设施。
