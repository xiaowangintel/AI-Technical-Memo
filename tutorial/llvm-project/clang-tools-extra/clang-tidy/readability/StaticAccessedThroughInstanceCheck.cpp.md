# StaticAccessedThroughInstanceCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/readability/StaticAccessedThroughInstanceCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `StaticAccessedThroughInstanceCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `StaticAccessedThroughInstanceCheck`，包括 AST 匹配、诊断与自动修复行为。

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
 9 | #include "StaticAccessedThroughInstanceCheck.h"
10 | #include "clang/AST/ASTContext.h"
11 | #include "clang/ASTMatchers/ASTMatchFinder.h"
12 | #include "llvm/ADT/StringRef.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "StaticAccessedThroughInstanceCheck.h" to access local declarations from the current tool or check. / 引入 "StaticAccessedThroughInstanceCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Includes "clang/AST/ASTContext.h" to access Clang AST node and semantic interfaces. / 引入 "clang/AST/ASTContext.h" 以使用Clang AST 节点与语义接口。
- **L11**: Includes "clang/ASTMatchers/ASTMatchFinder.h" to access AST matcher construction helpers. / 引入 "clang/ASTMatchers/ASTMatchFinder.h" 以使用AST 匹配器构造辅助逻辑。
- **L12**: Includes "llvm/ADT/StringRef.h" to access LLVM ADT containers and helper types. / 引入 "llvm/ADT/StringRef.h" 以使用LLVM ADT 容器与辅助类型。

### Lines 13-24 / 第 13-24 行

```cpp
13 | 
14 | using namespace clang::ast_matchers;
15 | 
16 | namespace clang::tidy::readability {
17 | 
18 | namespace {
19 | AST_MATCHER(CXXMethodDecl, isStatic) { return Node.isStatic(); }
20 | } // namespace
21 | 
22 | static unsigned getNameSpecifierNestingLevel(QualType QType) {
23 |   unsigned NameSpecifierNestingLevel = 1;
24 |   for (NestedNameSpecifier Qualifier = QType->getPrefix(); /**/;
```

- **L13**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L14**: Brings namespace `clang::ast_matchers` into the local scope. / 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **L15**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L16**: Opens namespace scope `clang::tidy::readability`. / 打开命名空间作用域 `clang::tidy::readability`。
- **L17**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L18**: Opens an anonymous namespace for internal linkage. / 打开匿名命名空间以提供内部链接。
- **L19**: Defines a reusable AST matcher helper for later pattern matching logic. / 定义一个可复用的 AST 匹配器辅助组件，供后续模式匹配逻辑使用。
- **L20**: Closes a namespace scope while preserving the trailing comment: `} // namespace`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L21**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L22**: Starts a function, method, lambda, or structured scope: `static unsigned getNameSpecifierNestingLevel(QualType QType) {`. / 开始一个函数、方法、lambda 或结构化作用域：`static unsigned getNameSpecifierNestingLevel(QualType QType) {`。
- **L23**: Initializes variable `NameSpecifierNestingLevel` from the right-hand expression. / 使用右侧表达式初始化变量 `NameSpecifierNestingLevel`。
- **L24**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。

### Lines 25-36 / 第 25-36 行

```cpp
25 |        ++NameSpecifierNestingLevel) {
26 |     switch (Qualifier.getKind()) {
27 |     case NestedNameSpecifier::Kind::Null:
28 |       return NameSpecifierNestingLevel;
29 |     case NestedNameSpecifier::Kind::Global:
30 |     case NestedNameSpecifier::Kind::MicrosoftSuper:
31 |       return NameSpecifierNestingLevel + 1;
32 |     case NestedNameSpecifier::Kind::Namespace:
33 |       Qualifier = Qualifier.getAsNamespaceAndPrefix().Prefix;
34 |       continue;
35 |     case NestedNameSpecifier::Kind::Type:
36 |       Qualifier = Qualifier.getAsType()->getPrefix();
```

- **L25**: Continues the surrounding expression or declaration: `++NameSpecifierNestingLevel) {`. / 继续构造周围的表达式或声明：`++NameSpecifierNestingLevel) {`。
- **L26**: Begins a `switch` control-flow statement and evaluates its condition. / 开始 `switch` 控制流语句并计算其条件。
- **L27**: Introduces a switch dispatch label: `case NestedNameSpecifier::Kind::Null:`. / 引入一个 switch 分发标签：`case NestedNameSpecifier::Kind::Null:`。
- **L28**: Returns from the current function with `NameSpecifierNestingLevel`. / 以 `NameSpecifierNestingLevel` 从当前函数返回。
- **L29**: Introduces a switch dispatch label: `case NestedNameSpecifier::Kind::Global:`. / 引入一个 switch 分发标签：`case NestedNameSpecifier::Kind::Global:`。
- **L30**: Introduces a switch dispatch label: `case NestedNameSpecifier::Kind::MicrosoftSuper:`. / 引入一个 switch 分发标签：`case NestedNameSpecifier::Kind::MicrosoftSuper:`。
- **L31**: Returns from the current function with `NameSpecifierNestingLevel + 1`. / 以 `NameSpecifierNestingLevel + 1` 从当前函数返回。
- **L32**: Introduces a switch dispatch label: `case NestedNameSpecifier::Kind::Namespace:`. / 引入一个 switch 分发标签：`case NestedNameSpecifier::Kind::Namespace:`。
- **L33**: Assigns new state to `Qualifier` for later logic. / 为后续逻辑给 `Qualifier` 赋予新状态。
- **L34**: Skips to the next loop iteration or continues the enclosing flow. / 跳到下一次循环迭代，或继续外层流程。
- **L35**: Introduces a switch dispatch label: `case NestedNameSpecifier::Kind::Type:`. / 引入一个 switch 分发标签：`case NestedNameSpecifier::Kind::Type:`。
- **L36**: Assigns new state to `Qualifier` for later logic. / 为后续逻辑给 `Qualifier` 赋予新状态。

### Lines 37-48 / 第 37-48 行

```cpp
37 |       continue;
38 |     }
39 |     llvm_unreachable("unhandled nested name specifier kind");
40 |   }
41 | }
42 | 
43 | void StaticAccessedThroughInstanceCheck::storeOptions(
44 |     ClangTidyOptions::OptionMap &Opts) {
45 |   Options.store(Opts, "NameSpecifierNestingThreshold",
46 |                 NameSpecifierNestingThreshold);
47 | }
48 | 
```

- **L37**: Skips to the next loop iteration or continues the enclosing flow. / 跳到下一次循环迭代，或继续外层流程。
- **L38**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L39**: Marks this control path as unreachable to LLVM. / 将该控制路径标记为 LLVM 认为不可达。
- **L40**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L41**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L42**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L43**: Continues logic associated with callable symbol `storeOptions`. / 继续与可调用符号 `storeOptions` 相关的逻辑。
- **L44**: Continues the surrounding expression or declaration: `ClangTidyOptions::OptionMap &Opts) {`. / 继续构造周围的表达式或声明：`ClangTidyOptions::OptionMap &Opts) {`。
- **L45**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L46**: Executes a standalone statement or declaration: `NameSpecifierNestingThreshold);`. / 执行一条独立语句或声明：`NameSpecifierNestingThreshold);`。
- **L47**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L48**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 49-60 / 第 49-60 行

```cpp
49 | void StaticAccessedThroughInstanceCheck::registerMatchers(MatchFinder *Finder) {
50 |   Finder->addMatcher(
51 |       memberExpr(hasDeclaration(anyOf(cxxMethodDecl(isStatic()),
52 |                                       varDecl(hasStaticStorageDuration()),
53 |                                       enumConstantDecl())))
54 |           .bind("memberExpression"),
55 |       this);
56 | }
57 | 
58 | void StaticAccessedThroughInstanceCheck::check(
59 |     const MatchFinder::MatchResult &Result) {
60 |   const auto *MemberExpression =
```

- **L49**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L50**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L51**: Continues a multi-line argument list, initializer, or aggregate entry: `memberExpr(hasDeclaration(anyOf(cxxMethodDecl(isStatic()),`. / 继续一个多行参数列表、初始化器或聚合项：`memberExpr(hasDeclaration(anyOf(cxxMethodDecl(isStatic()),`。
- **L52**: Continues a multi-line argument list, initializer, or aggregate entry: `varDecl(hasStaticStorageDuration()),`. / 继续一个多行参数列表、初始化器或聚合项：`varDecl(hasStaticStorageDuration()),`。
- **L53**: Continues logic associated with callable symbol `enumConstantDecl`. / 继续与可调用符号 `enumConstantDecl` 相关的逻辑。
- **L54**: Continues a multi-line argument list, initializer, or aggregate entry: `.bind("memberExpression"),`. / 继续一个多行参数列表、初始化器或聚合项：`.bind("memberExpression"),`。
- **L55**: Executes a standalone statement or declaration: `this);`. / 执行一条独立语句或声明：`this);`。
- **L56**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L57**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L58**: Continues logic associated with callable symbol `check`. / 继续与可调用符号 `check` 相关的逻辑。
- **L59**: Continues the surrounding expression or declaration: `const MatchFinder::MatchResult &Result) {`. / 继续构造周围的表达式或声明：`const MatchFinder::MatchResult &Result) {`。
- **L60**: Continues the surrounding expression or declaration: `const auto *MemberExpression =`. / 继续构造周围的表达式或声明：`const auto *MemberExpression =`。

### Lines 61-72 / 第 61-72 行

```cpp
61 |       Result.Nodes.getNodeAs<MemberExpr>("memberExpression");
62 | 
63 |   if (MemberExpression->getBeginLoc().isMacroID())
64 |     return;
65 | 
66 |   const Expr *BaseExpr = MemberExpression->getBase();
67 | 
68 |   const QualType BaseType =
69 |       BaseExpr->getType()->isPointerType()
70 |           ? BaseExpr->getType()->getPointeeType().getUnqualifiedType()
71 |           : BaseExpr->getType().getUnqualifiedType();
72 | 
```

- **L61**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<MemberExpr>`. / 执行以 `Result.Nodes.getNodeAs<MemberExpr>` 为核心的调用或声明。
- **L62**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L63**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L64**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L65**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L66**: Executes a call or declaration centered on `MemberExpression->getBase`. / 执行以 `MemberExpression->getBase` 为核心的调用或声明。
- **L67**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L68**: Continues the surrounding expression or declaration: `const QualType BaseType =`. / 继续构造周围的表达式或声明：`const QualType BaseType =`。
- **L69**: Continues logic associated with callable symbol `getType`. / 继续与可调用符号 `getType` 相关的逻辑。
- **L70**: Continues logic associated with callable symbol `getType`. / 继续与可调用符号 `getType` 相关的逻辑。
- **L71**: Executes a call or declaration centered on `BaseExpr->getType`. / 执行以 `BaseExpr->getType` 为核心的调用或声明。
- **L72**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 73-84 / 第 73-84 行

```cpp
73 |   const ASTContext *AstContext = Result.Context;
74 |   PrintingPolicy PrintingPolicyWithSuppressedTag(AstContext->getLangOpts());
75 |   PrintingPolicyWithSuppressedTag.SuppressTagKeyword = true;
76 |   PrintingPolicyWithSuppressedTag.SuppressUnwrittenScope = true;
77 | 
78 |   PrintingPolicyWithSuppressedTag.PrintAsCanonical =
79 |       !BaseExpr->getType()->isTypedefNameType();
80 | 
81 |   std::string BaseTypeName =
82 |       BaseType.getAsString(PrintingPolicyWithSuppressedTag);
83 | 
84 |   // Ignore anonymous structs/classes which will not have an identifier
```

- **L73**: Executes a standalone statement or declaration: `const ASTContext *AstContext = Result.Context;`. / 执行一条独立语句或声明：`const ASTContext *AstContext = Result.Context;`。
- **L74**: Executes a call or declaration centered on `PrintingPolicyWithSuppressedTag`. / 执行以 `PrintingPolicyWithSuppressedTag` 为核心的调用或声明。
- **L75**: Executes a standalone statement or declaration: `PrintingPolicyWithSuppressedTag.SuppressTagKeyword = true;`. / 执行一条独立语句或声明：`PrintingPolicyWithSuppressedTag.SuppressTagKeyword = true;`。
- **L76**: Executes a standalone statement or declaration: `PrintingPolicyWithSuppressedTag.SuppressUnwrittenScope = true;`. / 执行一条独立语句或声明：`PrintingPolicyWithSuppressedTag.SuppressUnwrittenScope = true;`。
- **L77**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L78**: Continues the surrounding expression or declaration: `PrintingPolicyWithSuppressedTag.PrintAsCanonical =`. / 继续构造周围的表达式或声明：`PrintingPolicyWithSuppressedTag.PrintAsCanonical =`。
- **L79**: Executes a call or declaration centered on `!BaseExpr->getType`. / 执行以 `!BaseExpr->getType` 为核心的调用或声明。
- **L80**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L81**: Continues the surrounding expression or declaration: `std::string BaseTypeName =`. / 继续构造周围的表达式或声明：`std::string BaseTypeName =`。
- **L82**: Executes a call or declaration centered on `BaseType.getAsString`. / 执行以 `BaseType.getAsString` 为核心的调用或声明。
- **L83**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L84**: Comment explains nearby logic, intent, or usage: `Ignore anonymous structs/classes which will not have an identifier`. / 注释说明了附近代码的逻辑、意图或用法：`Ignore anonymous structs/classes which will not have an identifier`。

### Lines 85-96 / 第 85-96 行

```cpp
85 |   const RecordDecl *RecDecl = BaseType->getAsCXXRecordDecl();
86 |   if (!RecDecl || RecDecl->getIdentifier() == nullptr)
87 |     return;
88 | 
89 |   // Do not warn for CUDA built-in variables.
90 |   if (StringRef(BaseTypeName).starts_with("__cuda_builtin_"))
91 |     return;
92 | 
93 |   SourceLocation MemberExprStartLoc = MemberExpression->getBeginLoc();
94 |   auto CreateFix = [&] {
95 |     return FixItHint::CreateReplacement(
96 |         CharSourceRange::getCharRange(MemberExprStartLoc,
```

- **L85**: Executes a call or declaration centered on `BaseType->getAsCXXRecordDecl`. / 执行以 `BaseType->getAsCXXRecordDecl` 为核心的调用或声明。
- **L86**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L87**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L88**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L89**: Comment explains nearby logic, intent, or usage: `Do not warn for CUDA built-in variables.`. / 注释说明了附近代码的逻辑、意图或用法：`Do not warn for CUDA built-in variables.`。
- **L90**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L91**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L92**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L93**: Initializes variable `MemberExprStartLoc` from the right-hand expression. / 使用右侧表达式初始化变量 `MemberExprStartLoc`。
- **L94**: Continues the surrounding expression or declaration: `auto CreateFix = [&] {`. / 继续构造周围的表达式或声明：`auto CreateFix = [&] {`。
- **L95**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L96**: Continues a multi-line argument list, initializer, or aggregate entry: `CharSourceRange::getCharRange(MemberExprStartLoc,`. / 继续一个多行参数列表、初始化器或聚合项：`CharSourceRange::getCharRange(MemberExprStartLoc,`。

### Lines 97-108 / 第 97-108 行

```cpp
 97 |                                       MemberExpression->getMemberLoc()),
 98 |         BaseTypeName + "::");
 99 |   };
100 | 
101 |   {
102 |     auto Diag =
103 |         diag(MemberExprStartLoc, "static member accessed through instance");
104 | 
105 |     if (getNameSpecifierNestingLevel(BaseType) > NameSpecifierNestingThreshold)
106 |       return;
107 | 
108 |     if (!BaseExpr->HasSideEffects(*AstContext,
```

- **L97**: Continues a multi-line argument list, initializer, or aggregate entry: `MemberExpression->getMemberLoc()),`. / 继续一个多行参数列表、初始化器或聚合项：`MemberExpression->getMemberLoc()),`。
- **L98**: Executes a standalone statement or declaration: `BaseTypeName + "::");`. / 执行一条独立语句或声明：`BaseTypeName + "::");`。
- **L99**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L100**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L101**: Opens a new lexical scope or compound statement. / 打开一个新的词法作用域或复合语句块。
- **L102**: Continues the surrounding expression or declaration: `auto Diag =`. / 继续构造周围的表达式或声明：`auto Diag =`。
- **L103**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L104**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L105**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L106**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L107**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L108**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。

### Lines 109-120 / 第 109-120 行

```cpp
109 |                                   /* IncludePossibleEffects =*/true)) {
110 |       Diag << CreateFix();
111 |       return;
112 |     }
113 |   }
114 | 
115 |   diag(MemberExprStartLoc, "member base expression may carry some side effects",
116 |        DiagnosticIDs::Level::Note)
117 |       << BaseExpr->getSourceRange() << CreateFix();
118 | }
119 | 
120 | } // namespace clang::tidy::readability
```

- **L109**: Comment explains nearby logic, intent, or usage: `IncludePossibleEffects =*/true)) {`. / 注释说明了附近代码的逻辑、意图或用法：`IncludePossibleEffects =*/true)) {`。
- **L110**: Executes a call or declaration centered on `CreateFix`. / 执行以 `CreateFix` 为核心的调用或声明。
- **L111**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L112**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L113**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L114**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L115**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L116**: Continues the surrounding expression or declaration: `DiagnosticIDs::Level::Note)`. / 继续构造周围的表达式或声明：`DiagnosticIDs::Level::Note)`。
- **L117**: Executes a call or declaration centered on `BaseExpr->getSourceRange`. / 执行以 `BaseExpr->getSourceRange` 为核心的调用或声明。
- **L118**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L119**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L120**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::readability`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::readability`。

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

- `StaticAccessedThroughInstanceCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `clang/AST/ASTContext.h`: Provides Clang AST node and semantic interfaces. / 提供Clang AST 节点与语义接口。
- `clang/ASTMatchers/ASTMatchFinder.h`: Provides AST matcher construction helpers. / 提供AST 匹配器构造辅助逻辑。
- `llvm/ADT/StringRef.h`: Provides LLVM ADT containers and helper types. / 提供LLVM ADT 容器与辅助类型。
