# AvoidPragmaOnceCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/portability/AvoidPragmaOnceCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `AvoidPragmaOnceCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `AvoidPragmaOnceCheck`，包括 AST 匹配、诊断与自动修复行为。

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
 9 | #include "AvoidPragmaOnceCheck.h"
10 | 
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "AvoidPragmaOnceCheck.h" to access local declarations from the current tool or check. / 引入 "AvoidPragmaOnceCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 11-20 / 第 11-20 行

```cpp
11 | #include "clang/Basic/SourceManager.h"
12 | #include "clang/Lex/PPCallbacks.h"
13 | #include "clang/Lex/Preprocessor.h"
14 | #include "llvm/ADT/StringRef.h"
15 | 
16 | namespace clang::tidy::portability {
17 | 
18 | namespace {
19 | 
20 | class PragmaOnceCallbacks : public PPCallbacks {
```

- **L11**: Includes "clang/Basic/SourceManager.h" to access basic source, diagnostic, and language-option support. / 引入 "clang/Basic/SourceManager.h" 以使用基础源码、诊断与语言选项支持。
- **L12**: Includes "clang/Lex/PPCallbacks.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/PPCallbacks.h" 以使用词法分析器与预处理器接口。
- **L13**: Includes "clang/Lex/Preprocessor.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/Preprocessor.h" 以使用词法分析器与预处理器接口。
- **L14**: Includes "llvm/ADT/StringRef.h" to access LLVM ADT containers and helper types. / 引入 "llvm/ADT/StringRef.h" 以使用LLVM ADT 容器与辅助类型。
- **L15**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L16**: Opens namespace scope `clang::tidy::portability`. / 打开命名空间作用域 `clang::tidy::portability`。
- **L17**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L18**: Opens an anonymous namespace for internal linkage. / 打开匿名命名空间以提供内部链接。
- **L19**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L20**: Declares class `PragmaOnceCallbacks`. / 声明类 `PragmaOnceCallbacks`。

### Lines 21-30 / 第 21-30 行

```cpp
21 | public:
22 |   PragmaOnceCallbacks(AvoidPragmaOnceCheck *Check, const SourceManager &SM)
23 |       : Check(Check), SM(SM) {}
24 |   void PragmaDirective(SourceLocation Loc,
25 |                        PragmaIntroducerKind Introducer) override {
26 |     auto Str = StringRef(SM.getCharacterData(Loc));
27 |     if (!Str.consume_front("#"))
28 |       return;
29 |     Str = Str.trim();
30 |     if (!Str.consume_front("pragma"))
```

- **L21**: Sets the following members to `public` access. / 将后续成员的访问级别设为 `public`。
- **L22**: Continues logic associated with callable symbol `PragmaOnceCallbacks`. / 继续与可调用符号 `PragmaOnceCallbacks` 相关的逻辑。
- **L23**: Continues logic associated with callable symbol `Check`. / 继续与可调用符号 `Check` 相关的逻辑。
- **L24**: Continues a multi-line argument list, initializer, or aggregate entry: `void PragmaDirective(SourceLocation Loc,`. / 继续一个多行参数列表、初始化器或聚合项：`void PragmaDirective(SourceLocation Loc,`。
- **L25**: Continues the surrounding expression or declaration: `PragmaIntroducerKind Introducer) override {`. / 继续构造周围的表达式或声明：`PragmaIntroducerKind Introducer) override {`。
- **L26**: Initializes variable `Str` from the right-hand expression. / 使用右侧表达式初始化变量 `Str`。
- **L27**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L28**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L29**: Assigns new state to `Str` for later logic. / 为后续逻辑给 `Str` 赋予新状态。
- **L30**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。

### Lines 31-40 / 第 31-40 行

```cpp
31 |       return;
32 |     Str = Str.trim();
33 |     if (Str.starts_with("once"))
34 |       Check->diag(Loc,
35 |                   "avoid 'pragma once' directive; use include guards instead");
36 |   }
37 | 
38 | private:
39 |   AvoidPragmaOnceCheck *Check;
40 |   const SourceManager &SM;
```

- **L31**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L32**: Assigns new state to `Str` for later logic. / 为后续逻辑给 `Str` 赋予新状态。
- **L33**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L34**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L35**: Executes a standalone statement or declaration: `"avoid 'pragma once' directive; use include guards instead");`. / 执行一条独立语句或声明：`"avoid 'pragma once' directive; use include guards instead");`。
- **L36**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L37**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L38**: Sets the following members to `private` access. / 将后续成员的访问级别设为 `private`。
- **L39**: Executes a standalone statement or declaration: `AvoidPragmaOnceCheck *Check;`. / 执行一条独立语句或声明：`AvoidPragmaOnceCheck *Check;`。
- **L40**: Executes a standalone statement or declaration: `const SourceManager &SM;`. / 执行一条独立语句或声明：`const SourceManager &SM;`。

### Lines 41-50 / 第 41-50 行

```cpp
41 | };
42 | 
43 | } // namespace
44 | 
45 | void AvoidPragmaOnceCheck::registerPPCallbacks(const SourceManager &SM,
46 |                                                Preprocessor *PP,
47 |                                                Preprocessor *ModuleExpanderPP) {
48 |   PP->addPPCallbacks(std::make_unique<PragmaOnceCallbacks>(this, SM));
49 | }
50 | 
```

- **L41**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L42**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L43**: Closes a namespace scope while preserving the trailing comment: `} // namespace`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L44**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L45**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L46**: Continues a multi-line argument list, initializer, or aggregate entry: `Preprocessor *PP,`. / 继续一个多行参数列表、初始化器或聚合项：`Preprocessor *PP,`。
- **L47**: Continues the surrounding expression or declaration: `Preprocessor *ModuleExpanderPP) {`. / 继续构造周围的表达式或声明：`Preprocessor *ModuleExpanderPP) {`。
- **L48**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L49**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L50**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 51-51 / 第 51-51 行

```cpp
51 | } // namespace clang::tidy::portability
```

- **L51**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::portability`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::portability`。

## Key Concepts / 关键概念

- **Portability checking / 可移植性检查**:
  - **EN**: Flags code that depends on compiler, platform, or ABI-specific behavior.
  - **CN**: 标记依赖编译器、平台或 ABI 特定行为的代码。
- **Diagnostic emission / 诊断信息发出**:
  - **EN**: Reports issues at source locations with message parameters.
  - **CN**: 在源码位置上携带消息参数报告问题。
- **Preprocessor callbacks / 预处理器回调**:
  - **EN**: Listens to macro and include events before the AST is fully formed.
  - **CN**: 在 AST 完全形成前监听宏与包含事件。
- **Preprocessor integration / 预处理器集成**:
  - **EN**: Uses Clang preprocessor state to reason about directives and tokens.
  - **CN**: 利用 Clang 预处理器状态推断指令与记号。

## Dependencies / 依赖关系

- `AvoidPragmaOnceCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `clang/Basic/SourceManager.h`: Provides basic source, diagnostic, and language-option support. / 提供基础源码、诊断与语言选项支持。
- `clang/Lex/PPCallbacks.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
- `clang/Lex/Preprocessor.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
- `llvm/ADT/StringRef.h`: Provides LLVM ADT containers and helper types. / 提供LLVM ADT 容器与辅助类型。
