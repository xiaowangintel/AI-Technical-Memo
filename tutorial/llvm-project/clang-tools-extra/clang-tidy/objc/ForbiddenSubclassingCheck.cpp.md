# ForbiddenSubclassingCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/objc/ForbiddenSubclassingCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `ForbiddenSubclassingCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `ForbiddenSubclassingCheck`，包括 AST 匹配、诊断与自动修复行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行

```cpp
 1 | //===----------------------------------------------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | 
 9 | #include "ForbiddenSubclassingCheck.h"
10 | #include "../utils/OptionsUtils.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "ForbiddenSubclassingCheck.h" to access local declarations from the current tool or check. / 引入 "ForbiddenSubclassingCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Includes "../utils/OptionsUtils.h" to access shared clang-tidy utility helpers. / 引入 "../utils/OptionsUtils.h" 以使用共享 clang-tidy 工具辅助逻辑。

### Lines 11-20 / 第 11-20 行

```cpp
11 | #include "clang/AST/ASTContext.h"
12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
13 | 
14 | using namespace clang::ast_matchers;
15 | 
16 | namespace clang::tidy::objc {
17 | 
18 | namespace {
19 | 
20 | constexpr char DefaultForbiddenSuperClassNames[] =
```

- **L11**: Includes "clang/AST/ASTContext.h" to access Clang AST node and semantic interfaces. / 引入 "clang/AST/ASTContext.h" 以使用Clang AST 节点与语义接口。
- **L12**: Includes "clang/ASTMatchers/ASTMatchFinder.h" to access AST matcher construction helpers. / 引入 "clang/ASTMatchers/ASTMatchFinder.h" 以使用AST 匹配器构造辅助逻辑。
- **L13**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L14**: Brings namespace `clang::ast_matchers` into the local scope. / 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **L15**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L16**: Opens namespace scope `clang::tidy::objc`. / 打开命名空间作用域 `clang::tidy::objc`。
- **L17**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L18**: Opens an anonymous namespace for internal linkage. / 打开匿名命名空间以提供内部链接。
- **L19**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L20**: Continues the surrounding expression or declaration: `constexpr char DefaultForbiddenSuperClassNames[] =`. / 继续构造周围的表达式或声明：`constexpr char DefaultForbiddenSuperClassNames[] =`。

### Lines 21-30 / 第 21-30 行

```cpp
21 |     "ABNewPersonViewController;"
22 |     "ABPeoplePickerNavigationController;"
23 |     "ABPersonViewController;"
24 |     "ABUnknownPersonViewController;"
25 |     "NSHashTable;"
26 |     "NSMapTable;"
27 |     "NSPointerArray;"
28 |     "NSPointerFunctions;"
29 |     "NSTimer;"
30 |     "UIActionSheet;"
```

- **L21**: Continues the surrounding expression or declaration: `"ABNewPersonViewController;"`. / 继续构造周围的表达式或声明：`"ABNewPersonViewController;"`。
- **L22**: Continues the surrounding expression or declaration: `"ABPeoplePickerNavigationController;"`. / 继续构造周围的表达式或声明：`"ABPeoplePickerNavigationController;"`。
- **L23**: Continues the surrounding expression or declaration: `"ABPersonViewController;"`. / 继续构造周围的表达式或声明：`"ABPersonViewController;"`。
- **L24**: Continues the surrounding expression or declaration: `"ABUnknownPersonViewController;"`. / 继续构造周围的表达式或声明：`"ABUnknownPersonViewController;"`。
- **L25**: Continues the surrounding expression or declaration: `"NSHashTable;"`. / 继续构造周围的表达式或声明：`"NSHashTable;"`。
- **L26**: Continues the surrounding expression or declaration: `"NSMapTable;"`. / 继续构造周围的表达式或声明：`"NSMapTable;"`。
- **L27**: Continues the surrounding expression or declaration: `"NSPointerArray;"`. / 继续构造周围的表达式或声明：`"NSPointerArray;"`。
- **L28**: Continues the surrounding expression or declaration: `"NSPointerFunctions;"`. / 继续构造周围的表达式或声明：`"NSPointerFunctions;"`。
- **L29**: Continues the surrounding expression or declaration: `"NSTimer;"`. / 继续构造周围的表达式或声明：`"NSTimer;"`。
- **L30**: Continues the surrounding expression or declaration: `"UIActionSheet;"`. / 继续构造周围的表达式或声明：`"UIActionSheet;"`。

### Lines 31-40 / 第 31-40 行

```cpp
31 |     "UIAlertView;"
32 |     "UIImagePickerController;"
33 |     "UITextInputMode;"
34 |     "UIWebView";
35 | 
36 | } // namespace
37 | 
38 | ForbiddenSubclassingCheck::ForbiddenSubclassingCheck(StringRef Name,
39 |                                                      ClangTidyContext *Context)
40 |     : ClangTidyCheck(Name, Context),
```

- **L31**: Continues the surrounding expression or declaration: `"UIAlertView;"`. / 继续构造周围的表达式或声明：`"UIAlertView;"`。
- **L32**: Continues the surrounding expression or declaration: `"UIImagePickerController;"`. / 继续构造周围的表达式或声明：`"UIImagePickerController;"`。
- **L33**: Continues the surrounding expression or declaration: `"UITextInputMode;"`. / 继续构造周围的表达式或声明：`"UITextInputMode;"`。
- **L34**: Executes a standalone statement or declaration: `"UIWebView";`. / 执行一条独立语句或声明：`"UIWebView";`。
- **L35**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L36**: Closes a namespace scope while preserving the trailing comment: `} // namespace`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L37**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L38**: Continues a multi-line argument list, initializer, or aggregate entry: `ForbiddenSubclassingCheck::ForbiddenSubclassingCheck(StringRef Name,`. / 继续一个多行参数列表、初始化器或聚合项：`ForbiddenSubclassingCheck::ForbiddenSubclassingCheck(StringRef Name,`。
- **L39**: Continues the surrounding expression or declaration: `ClangTidyContext *Context)`. / 继续构造周围的表达式或声明：`ClangTidyContext *Context)`。
- **L40**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangTidyCheck(Name, Context),`. / 继续一个多行参数列表、初始化器或聚合项：`: ClangTidyCheck(Name, Context),`。

### Lines 41-50 / 第 41-50 行

```cpp
41 |       ForbiddenSuperClassNames(utils::options::parseStringList(
42 |           Options.get("ClassNames", DefaultForbiddenSuperClassNames))) {}
43 | 
44 | void ForbiddenSubclassingCheck::registerMatchers(MatchFinder *Finder) {
45 |   Finder->addMatcher(
46 |       objcInterfaceDecl(
47 |           isDerivedFrom(objcInterfaceDecl(hasAnyName(ForbiddenSuperClassNames))
48 |                             .bind("superclass")))
49 |           .bind("subclass"),
50 |       this);
```

- **L41**: Continues logic associated with callable symbol `ForbiddenSuperClassNames`. / 继续与可调用符号 `ForbiddenSuperClassNames` 相关的逻辑。
- **L42**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L43**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L44**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L45**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L46**: Continues logic associated with callable symbol `objcInterfaceDecl`. / 继续与可调用符号 `objcInterfaceDecl` 相关的逻辑。
- **L47**: Continues logic associated with callable symbol `isDerivedFrom`. / 继续与可调用符号 `isDerivedFrom` 相关的逻辑。
- **L48**: Continues logic associated with callable symbol `bind`. / 继续与可调用符号 `bind` 相关的逻辑。
- **L49**: Continues a multi-line argument list, initializer, or aggregate entry: `.bind("subclass"),`. / 继续一个多行参数列表、初始化器或聚合项：`.bind("subclass"),`。
- **L50**: Executes a standalone statement or declaration: `this);`. / 执行一条独立语句或声明：`this);`。

### Lines 51-60 / 第 51-60 行

```cpp
51 | }
52 | 
53 | void ForbiddenSubclassingCheck::check(const MatchFinder::MatchResult &Result) {
54 |   const auto *SubClass = Result.Nodes.getNodeAs<ObjCInterfaceDecl>("subclass");
55 |   assert(SubClass != nullptr);
56 |   const auto *SuperClass =
57 |       Result.Nodes.getNodeAs<ObjCInterfaceDecl>("superclass");
58 |   assert(SuperClass != nullptr);
59 |   diag(SubClass->getLocation(),
60 |        "Objective-C interface %0 subclasses %1, which is not "
```

- **L51**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L52**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L53**: Starts a function, method, lambda, or structured scope: `void ForbiddenSubclassingCheck::check(const MatchFinder::MatchResult &Result) {`. / 开始一个函数、方法、lambda 或结构化作用域：`void ForbiddenSubclassingCheck::check(const MatchFinder::MatchResult &Result) {`。
- **L54**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<ObjCInterfaceDecl>`. / 执行以 `Result.Nodes.getNodeAs<ObjCInterfaceDecl>` 为核心的调用或声明。
- **L55**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L56**: Continues the surrounding expression or declaration: `const auto *SuperClass =`. / 继续构造周围的表达式或声明：`const auto *SuperClass =`。
- **L57**: Executes a call or declaration centered on `Result.Nodes.getNodeAs<ObjCInterfaceDecl>`. / 执行以 `Result.Nodes.getNodeAs<ObjCInterfaceDecl>` 为核心的调用或声明。
- **L58**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L59**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L60**: Continues the surrounding expression or declaration: `"Objective-C interface %0 subclasses %1, which is not "`. / 继续构造周围的表达式或声明：`"Objective-C interface %0 subclasses %1, which is not "`。

### Lines 61-70 / 第 61-70 行

```cpp
61 |        "intended to be subclassed")
62 |       << SubClass << SuperClass;
63 | }
64 | 
65 | void ForbiddenSubclassingCheck::storeOptions(
66 |     ClangTidyOptions::OptionMap &Opts) {
67 |   Options.store(Opts, "ForbiddenSuperClassNames",
68 |                 utils::options::serializeStringList(ForbiddenSuperClassNames));
69 | }
70 | 
```

- **L61**: Continues the surrounding expression or declaration: `"intended to be subclassed")`. / 继续构造周围的表达式或声明：`"intended to be subclassed")`。
- **L62**: Executes a standalone statement or declaration: `<< SubClass << SuperClass;`. / 执行一条独立语句或声明：`<< SubClass << SuperClass;`。
- **L63**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L64**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L65**: Continues logic associated with callable symbol `storeOptions`. / 继续与可调用符号 `storeOptions` 相关的逻辑。
- **L66**: Continues the surrounding expression or declaration: `ClangTidyOptions::OptionMap &Opts) {`. / 继续构造周围的表达式或声明：`ClangTidyOptions::OptionMap &Opts) {`。
- **L67**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L68**: Executes a call or declaration centered on `utils::options::serializeStringList`. / 执行以 `utils::options::serializeStringList` 为核心的调用或声明。
- **L69**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L70**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 71-71 / 第 71-71 行

```cpp
71 | } // namespace clang::tidy::objc
```

- **L71**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::objc`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::objc`。

## Key Concepts / 关键概念

- **Objective-C analysis / Objective-C 分析**:
  - **EN**: Matches Objective-C declarations and messaging patterns to enforce project rules.
  - **CN**: 匹配 Objective-C 声明与消息发送模式，以执行项目规则。
- **AST matcher registration / AST 匹配器注册**:
  - **EN**: Builds matcher trees that let the check find relevant AST patterns.
  - **CN**: 构建匹配器树，使检查能够找到相关 AST 模式。
- **AST matcher callbacks / AST 匹配回调**:
  - **EN**: Routes matched AST nodes into check-specific callback logic.
  - **CN**: 把匹配到的 AST 节点路由到检查特定的回调逻辑。
- **Diagnostic emission / 诊断信息发出**:
  - **EN**: Reports issues at source locations with message parameters.
  - **CN**: 在源码位置上携带消息参数报告问题。
- **Persistent options / 持久化选项**:
  - **EN**: Saves configurable behavior so checks can be tuned from .clang-tidy.
  - **CN**: 保存可配置行为，以便从 .clang-tidy 调整检查。

## Dependencies / 依赖关系

- `ForbiddenSubclassingCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `../utils/OptionsUtils.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `clang/AST/ASTContext.h`: Provides Clang AST node and semantic interfaces. / 提供Clang AST 节点与语义接口。
- `clang/ASTMatchers/ASTMatchFinder.h`: Provides AST matcher construction helpers. / 提供AST 匹配器构造辅助逻辑。
