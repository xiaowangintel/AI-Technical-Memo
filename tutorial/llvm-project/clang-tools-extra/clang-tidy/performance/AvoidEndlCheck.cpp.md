# AvoidEndlCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/performance/AvoidEndlCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `AvoidEndlCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `AvoidEndlCheck`，包括 AST 匹配、诊断与自动修复行为。

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
 9 | #include "AvoidEndlCheck.h"
10 | #include "clang/AST/ASTContext.h"
11 | #include "clang/AST/DeclCXX.h"
12 | #include "clang/AST/Expr.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "AvoidEndlCheck.h" to access local declarations from the current tool or check. / 引入 "AvoidEndlCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Includes "clang/AST/ASTContext.h" to access Clang AST node and semantic interfaces. / 引入 "clang/AST/ASTContext.h" 以使用Clang AST 节点与语义接口。
- **L11**: Includes "clang/AST/DeclCXX.h" to access Clang AST node and semantic interfaces. / 引入 "clang/AST/DeclCXX.h" 以使用Clang AST 节点与语义接口。
- **L12**: Includes "clang/AST/Expr.h" to access Clang AST node and semantic interfaces. / 引入 "clang/AST/Expr.h" 以使用Clang AST 节点与语义接口。

### Lines 13-24 / 第 13-24 行

```cpp
13 | #include "clang/AST/ExprCXX.h"
14 | #include "clang/ASTMatchers/ASTMatchFinder.h"
15 | #include "clang/ASTMatchers/ASTMatchers.h"
16 | #include "clang/Lex/Lexer.h"
17 | 
18 | using namespace clang::ast_matchers;
19 | 
20 | namespace clang::tidy::performance {
21 | 
22 | void AvoidEndlCheck::registerMatchers(MatchFinder *Finder) {
23 |   Finder->addMatcher(
24 |       callExpr(
```

- **L13**: Includes "clang/AST/ExprCXX.h" to access Clang AST node and semantic interfaces. / 引入 "clang/AST/ExprCXX.h" 以使用Clang AST 节点与语义接口。
- **L14**: Includes "clang/ASTMatchers/ASTMatchFinder.h" to access AST matcher construction helpers. / 引入 "clang/ASTMatchers/ASTMatchFinder.h" 以使用AST 匹配器构造辅助逻辑。
- **L15**: Includes "clang/ASTMatchers/ASTMatchers.h" to access AST matcher construction helpers. / 引入 "clang/ASTMatchers/ASTMatchers.h" 以使用AST 匹配器构造辅助逻辑。
- **L16**: Includes "clang/Lex/Lexer.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/Lexer.h" 以使用词法分析器与预处理器接口。
- **L17**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L18**: Brings namespace `clang::ast_matchers` into the local scope. / 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **L19**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L20**: Opens namespace scope `clang::tidy::performance`. / 打开命名空间作用域 `clang::tidy::performance`。
- **L21**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L22**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L23**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L24**: Continues logic associated with callable symbol `callExpr`. / 继续与可调用符号 `callExpr` 相关的逻辑。

### Lines 25-36 / 第 25-36 行

```cpp
25 |           anyOf(cxxOperatorCallExpr(
26 |                     hasOverloadedOperatorName("<<"),
27 |                     hasRHS(declRefExpr(to(namedDecl(hasName("::std::endl"))))
28 |                                .bind("expr"))),
29 |                 callExpr(argumentCountIs(1),
30 |                          callee(functionDecl(hasName("::std::endl"))))
31 |                     .bind("expr"))),
32 |       this);
33 | }
34 | 
35 | void AvoidEndlCheck::check(const MatchFinder::MatchResult &Result) {
36 |   const auto *Expression = Result.Nodes.getNodeAs<Expr>("expr");
```

- **L25**: Continues logic associated with callable symbol `anyOf`. / 继续与可调用符号 `anyOf` 相关的逻辑。
- **L26**: Continues a multi-line argument list, initializer, or aggregate entry: `hasOverloadedOperatorName("<<"),`. / 继续一个多行参数列表、初始化器或聚合项：`hasOverloadedOperatorName("<<"),`。
- **L27**: Continues logic associated with callable symbol `hasRHS`. / 继续与可调用符号 `hasRHS` 相关的逻辑。
- **L28**: Continues a multi-line argument list, initializer, or aggregate entry: `.bind("expr"))),`. / 继续一个多行参数列表、初始化器或聚合项：`.bind("expr"))),`。
- **L29**: Continues a multi-line argument list, initializer, or aggregate entry: `callExpr(argumentCountIs(1),`. / 继续一个多行参数列表、初始化器或聚合项：`callExpr(argumentCountIs(1),`。
- **L30**: Continues logic associated with callable symbol `callee`. / 继续与可调用符号 `callee` 相关的逻辑。
- **L31**: Continues a multi-line argument list, initializer, or aggregate entry: `.bind("expr"))),`. / 继续一个多行参数列表、初始化器或聚合项：`.bind("expr"))),`。
- **L32**: Executes a standalone statement or declaration: `this);`. / 执行一条独立语句或声明：`this);`。
- **L33**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L34**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L35**: Starts a function, method, lambda, or structured scope: `void AvoidEndlCheck::check(const MatchFinder::MatchResult &Result) {`. / 开始一个函数、方法、lambda 或结构化作用域：`void AvoidEndlCheck::check(const MatchFinder::MatchResult &Result) {`。
- **L36**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<Expr>`. / 执行以 `Result.Nodes.getNodeAs<Expr>` 为核心的调用或声明。

### Lines 37-48 / 第 37-48 行

```cpp
37 |   assert(Expression);
38 |   assert(isa<DeclRefExpr>(Expression) || isa<CallExpr>(Expression));
39 | 
40 |   // FIXME: It would be great if we could transform
41 |   // 'std::cout << "Hi" << std::endl;' into
42 |   // 'std::cout << "Hi\n"';
43 | 
44 |   if (isa<DeclRefExpr>(Expression)) {
45 |     // Handle the more common streaming '... << std::endl' case
46 |     const CharSourceRange TokenRange =
47 |         CharSourceRange::getTokenRange(Expression->getSourceRange());
48 |     StringRef SourceText = Lexer::getSourceText(
```

- **L37**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L38**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L39**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L40**: Comment records a pending task or caution: `FIXME: It would be great if we could transform`. / 注释记录了待办事项或注意点：`FIXME: It would be great if we could transform`。
- **L41**: Comment explains nearby logic, intent, or usage: `'std::cout << "Hi" << std::endl;' into`. / 注释说明了附近代码的逻辑、意图或用法：`'std::cout << "Hi" << std::endl;' into`。
- **L42**: Comment explains nearby logic, intent, or usage: `'std::cout << "Hi\n"';`. / 注释说明了附近代码的逻辑、意图或用法：`'std::cout << "Hi\n"';`。
- **L43**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L44**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L45**: Comment explains nearby logic, intent, or usage: `Handle the more common streaming '... << std::endl' case`. / 注释说明了附近代码的逻辑、意图或用法：`Handle the more common streaming '... << std::endl' case`。
- **L46**: Continues the surrounding expression or declaration: `const CharSourceRange TokenRange =`. / 继续构造周围的表达式或声明：`const CharSourceRange TokenRange =`。
- **L47**: Executes a call or declaration centered on `CharSourceRange::getTokenRange`. / 执行以 `CharSourceRange::getTokenRange` 为核心的调用或声明。
- **L48**: Continues logic associated with callable symbol `getSourceText`. / 继续与可调用符号 `getSourceText` 相关的逻辑。

### Lines 49-60 / 第 49-60 行

```cpp
49 |         TokenRange, *Result.SourceManager, Result.Context->getLangOpts());
50 |     if (SourceText.empty())
51 |       SourceText = "std::endl";
52 |     auto Diag = diag(Expression->getBeginLoc(),
53 |                      "do not use '%0' with streams; use '\\n' instead")
54 |                 << SourceText;
55 |     if (TokenRange.isValid())
56 |       Diag << FixItHint::CreateReplacement(TokenRange, "'\\n'");
57 |   } else {
58 |     // Handle the less common function call 'std::endl(...)' case
59 |     const auto *CallExpression = cast<CallExpr>(Expression);
60 |     assert(CallExpression->getNumArgs() == 1);
```

- **L49**: Executes a call or declaration centered on `Result.Context->getLangOpts`. / 执行以 `Result.Context->getLangOpts` 为核心的调用或声明。
- **L50**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L51**: Assigns new state to `SourceText` for later logic. / 为后续逻辑给 `SourceText` 赋予新状态。
- **L52**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L53**: Continues the surrounding expression or declaration: `"do not use '%0' with streams; use '\\n' instead")`. / 继续构造周围的表达式或声明：`"do not use '%0' with streams; use '\\n' instead")`。
- **L54**: Executes a standalone statement or declaration: `<< SourceText;`. / 执行一条独立语句或声明：`<< SourceText;`。
- **L55**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L56**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L57**: Continues the surrounding expression or declaration: `} else {`. / 继续构造周围的表达式或声明：`} else {`。
- **L58**: Comment explains nearby logic, intent, or usage: `Handle the less common function call 'std::endl(...)' case`. / 注释说明了附近代码的逻辑、意图或用法：`Handle the less common function call 'std::endl(...)' case`。
- **L59**: Executes a call or declaration centered on `cast<CallExpr>`. / 执行以 `cast<CallExpr>` 为核心的调用或声明。
- **L60**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。

### Lines 61-72 / 第 61-72 行

```cpp
61 | 
62 |     StringRef SourceText = Lexer::getSourceText(
63 |         CharSourceRange::getTokenRange(
64 |             CallExpression->getCallee()->getSourceRange()),
65 |         *Result.SourceManager, Result.Context->getLangOpts());
66 |     if (SourceText.empty())
67 |       SourceText = "std::endl";
68 |     auto Diag = diag(CallExpression->getBeginLoc(),
69 |                      "do not use '%0' with streams; use '\\n' instead")
70 |                 << SourceText;
71 | 
72 |     const CharSourceRange ArgTokenRange = CharSourceRange::getTokenRange(
```

- **L61**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L62**: Continues logic associated with callable symbol `getSourceText`. / 继续与可调用符号 `getSourceText` 相关的逻辑。
- **L63**: Continues logic associated with callable symbol `getTokenRange`. / 继续与可调用符号 `getTokenRange` 相关的逻辑。
- **L64**: Continues a multi-line argument list, initializer, or aggregate entry: `CallExpression->getCallee()->getSourceRange()),`. / 继续一个多行参数列表、初始化器或聚合项：`CallExpression->getCallee()->getSourceRange()),`。
- **L65**: Comment explains nearby logic, intent, or usage: `Result.SourceManager, Result.Context->getLangOpts());`. / 注释说明了附近代码的逻辑、意图或用法：`Result.SourceManager, Result.Context->getLangOpts());`。
- **L66**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L67**: Assigns new state to `SourceText` for later logic. / 为后续逻辑给 `SourceText` 赋予新状态。
- **L68**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L69**: Continues the surrounding expression or declaration: `"do not use '%0' with streams; use '\\n' instead")`. / 继续构造周围的表达式或声明：`"do not use '%0' with streams; use '\\n' instead")`。
- **L70**: Executes a standalone statement or declaration: `<< SourceText;`. / 执行一条独立语句或声明：`<< SourceText;`。
- **L71**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L72**: Continues logic associated with callable symbol `getTokenRange`. / 继续与可调用符号 `getTokenRange` 相关的逻辑。

### Lines 73-84 / 第 73-84 行

```cpp
73 |         CallExpression->getArg(0)->getSourceRange());
74 |     const StringRef ArgSourceText = Lexer::getSourceText(
75 |         ArgTokenRange, *Result.SourceManager, Result.Context->getLangOpts());
76 |     const CharSourceRange ReplacementRange =
77 |         CharSourceRange::getTokenRange(CallExpression->getSourceRange());
78 |     if (!ArgSourceText.empty() && ReplacementRange.isValid()) {
79 |       const std::string ReplacementString =
80 |           std::string(ArgSourceText) + " << '\\n'";
81 |       Diag << FixItHint::CreateReplacement(ReplacementRange, ReplacementString);
82 |     }
83 |   }
84 | }
```

- **L73**: Executes a call or declaration centered on `CallExpression->getArg`. / 执行以 `CallExpression->getArg` 为核心的调用或声明。
- **L74**: Continues logic associated with callable symbol `getSourceText`. / 继续与可调用符号 `getSourceText` 相关的逻辑。
- **L75**: Executes a call or declaration centered on `Result.Context->getLangOpts`. / 执行以 `Result.Context->getLangOpts` 为核心的调用或声明。
- **L76**: Continues the surrounding expression or declaration: `const CharSourceRange ReplacementRange =`. / 继续构造周围的表达式或声明：`const CharSourceRange ReplacementRange =`。
- **L77**: Executes a call or declaration centered on `CharSourceRange::getTokenRange`. / 执行以 `CharSourceRange::getTokenRange` 为核心的调用或声明。
- **L78**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L79**: Continues the surrounding expression or declaration: `const std::string ReplacementString =`. / 继续构造周围的表达式或声明：`const std::string ReplacementString =`。
- **L80**: Executes a call or declaration centered on `std::string`. / 执行以 `std::string` 为核心的调用或声明。
- **L81**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L82**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L83**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L84**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 85-86 / 第 85-86 行

```cpp
85 | 
86 | } // namespace clang::tidy::performance
```

- **L85**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L86**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::performance`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::performance`。

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

## Dependencies / 依赖关系

- `AvoidEndlCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `clang/AST/ASTContext.h`: Provides Clang AST node and semantic interfaces. / 提供Clang AST 节点与语义接口。
- `clang/AST/DeclCXX.h`: Provides Clang AST node and semantic interfaces. / 提供Clang AST 节点与语义接口。
- `clang/AST/Expr.h`: Provides Clang AST node and semantic interfaces. / 提供Clang AST 节点与语义接口。
- `clang/AST/ExprCXX.h`: Provides Clang AST node and semantic interfaces. / 提供Clang AST 节点与语义接口。
- `clang/ASTMatchers/ASTMatchFinder.h`: Provides AST matcher construction helpers. / 提供AST 匹配器构造辅助逻辑。
- `clang/ASTMatchers/ASTMatchers.h`: Provides AST matcher construction helpers. / 提供AST 匹配器构造辅助逻辑。
- `clang/Lex/Lexer.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
