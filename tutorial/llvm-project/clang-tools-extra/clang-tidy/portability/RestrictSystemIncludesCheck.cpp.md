# RestrictSystemIncludesCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/portability/RestrictSystemIncludesCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `RestrictSystemIncludesCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `RestrictSystemIncludesCheck`，包括 AST 匹配、诊断与自动修复行为。

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
 9 | #include "RestrictSystemIncludesCheck.h"
10 | #include "clang/Frontend/CompilerInstance.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "RestrictSystemIncludesCheck.h" to access local declarations from the current tool or check. / 引入 "RestrictSystemIncludesCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Includes "clang/Frontend/CompilerInstance.h" to access frontend action and compiler-instance APIs. / 引入 "clang/Frontend/CompilerInstance.h" 以使用前端动作与编译器实例 API。

### Lines 11-20 / 第 11-20 行

```cpp
11 | #include "clang/Lex/HeaderSearch.h"
12 | #include "clang/Lex/PPCallbacks.h"
13 | #include "clang/Lex/Preprocessor.h"
14 | #include "llvm/ADT/DenseMap.h"
15 | #include "llvm/ADT/SmallVector.h"
16 | #include "llvm/Support/Path.h"
17 | #include <cstring>
18 | 
19 | namespace clang::tidy::portability {
20 | 
```

- **L11**: Includes "clang/Lex/HeaderSearch.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/HeaderSearch.h" 以使用词法分析器与预处理器接口。
- **L12**: Includes "clang/Lex/PPCallbacks.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/PPCallbacks.h" 以使用词法分析器与预处理器接口。
- **L13**: Includes "clang/Lex/Preprocessor.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/Preprocessor.h" 以使用词法分析器与预处理器接口。
- **L14**: Includes "llvm/ADT/DenseMap.h" to access LLVM ADT containers and helper types. / 引入 "llvm/ADT/DenseMap.h" 以使用LLVM ADT 容器与辅助类型。
- **L15**: Includes "llvm/ADT/SmallVector.h" to access LLVM ADT containers and helper types. / 引入 "llvm/ADT/SmallVector.h" 以使用LLVM ADT 容器与辅助类型。
- **L16**: Includes "llvm/Support/Path.h" to access LLVM support-library facilities. / 引入 "llvm/Support/Path.h" 以使用LLVM Support 库设施。
- **L17**: Includes <cstring> to access C or C++ standard library facilities. / 引入 <cstring> 以使用C 或 C++ 标准库设施。
- **L18**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L19**: Opens namespace scope `clang::tidy::portability`. / 打开命名空间作用域 `clang::tidy::portability`。
- **L20**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 21-30 / 第 21-30 行

```cpp
21 | void RestrictedIncludesPPCallbacks::InclusionDirective(
22 |     SourceLocation HashLoc, const Token &IncludeTok, StringRef FileName,
23 |     bool IsAngled, CharSourceRange FilenameRange, OptionalFileEntryRef File,
24 |     StringRef SearchPath, StringRef RelativePath, const Module *SuggestedModule,
25 |     bool ModuleImported, SrcMgr::CharacteristicKind FileType) {
26 |   if (!Check.contains(FileName) && SrcMgr::isSystem(FileType)) {
27 |     SmallString<256> FullPath;
28 |     llvm::sys::path::append(FullPath, SearchPath);
29 |     llvm::sys::path::append(FullPath, RelativePath);
30 |     // Bucket the allowed include directives by the id of the file they were
```

- **L21**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L22**: Continues a multi-line argument list, initializer, or aggregate entry: `SourceLocation HashLoc, const Token &IncludeTok, StringRef FileName,`. / 继续一个多行参数列表、初始化器或聚合项：`SourceLocation HashLoc, const Token &IncludeTok, StringRef FileName,`。
- **L23**: Continues a multi-line argument list, initializer, or aggregate entry: `bool IsAngled, CharSourceRange FilenameRange, OptionalFileEntryRef File,`. / 继续一个多行参数列表、初始化器或聚合项：`bool IsAngled, CharSourceRange FilenameRange, OptionalFileEntryRef File,`。
- **L24**: Continues a multi-line argument list, initializer, or aggregate entry: `StringRef SearchPath, StringRef RelativePath, const Module *SuggestedModule,`. / 继续一个多行参数列表、初始化器或聚合项：`StringRef SearchPath, StringRef RelativePath, const Module *SuggestedModule,`。
- **L25**: Continues the surrounding expression or declaration: `bool ModuleImported, SrcMgr::CharacteristicKind FileType) {`. / 继续构造周围的表达式或声明：`bool ModuleImported, SrcMgr::CharacteristicKind FileType) {`。
- **L26**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L27**: Executes a standalone statement or declaration: `SmallString<256> FullPath;`. / 执行一条独立语句或声明：`SmallString<256> FullPath;`。
- **L28**: Executes a call or declaration centered on `llvm::sys::path::append`. / 执行以 `llvm::sys::path::append` 为核心的调用或声明。
- **L29**: Executes a call or declaration centered on `llvm::sys::path::append`. / 执行以 `llvm::sys::path::append` 为核心的调用或声明。
- **L30**: Comment explains nearby logic, intent, or usage: `Bucket the allowed include directives by the id of the file they were`. / 注释说明了附近代码的逻辑、意图或用法：`Bucket the allowed include directives by the id of the file they were`。

### Lines 31-40 / 第 31-40 行

```cpp
31 |     // declared in.
32 |     IncludeDirectives[SM.getFileID(HashLoc)].emplace_back(
33 |         HashLoc, FilenameRange, FileName, FullPath.str(),
34 |         SM.isInMainFile(HashLoc));
35 |   }
36 | }
37 | 
38 | void RestrictedIncludesPPCallbacks::EndOfMainFile() {
39 |   for (const auto &Bucket : IncludeDirectives) {
40 |     const FileIncludes &FileDirectives = Bucket.second;
```

- **L31**: Comment explains nearby logic, intent, or usage: `declared in.`. / 注释说明了附近代码的逻辑、意图或用法：`declared in.`。
- **L32**: Continues logic associated with callable symbol `getFileID`. / 继续与可调用符号 `getFileID` 相关的逻辑。
- **L33**: Continues a multi-line argument list, initializer, or aggregate entry: `HashLoc, FilenameRange, FileName, FullPath.str(),`. / 继续一个多行参数列表、初始化器或聚合项：`HashLoc, FilenameRange, FileName, FullPath.str(),`。
- **L34**: Executes a call or declaration centered on `SM.isInMainFile`. / 执行以 `SM.isInMainFile` 为核心的调用或声明。
- **L35**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L36**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L37**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L38**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L39**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L40**: Executes a standalone statement or declaration: `const FileIncludes &FileDirectives = Bucket.second;`. / 执行一条独立语句或声明：`const FileIncludes &FileDirectives = Bucket.second;`。

### Lines 41-50 / 第 41-50 行

```cpp
41 | 
42 |     // Emit fixits for all restricted includes.
43 |     for (const auto &Include : FileDirectives) {
44 |       // Fetch the length of the include statement from the start to just after
45 |       // the newline, for finding the end (including the newline).
46 |       const unsigned ToLen =
47 |           std::strcspn(SM.getCharacterData(Include.Loc), "\n") + 1;
48 |       const CharSourceRange ToRange = CharSourceRange::getCharRange(
49 |           Include.Loc, Include.Loc.getLocWithOffset(ToLen));
50 | 
```

- **L41**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L42**: Comment explains nearby logic, intent, or usage: `Emit fixits for all restricted includes.`. / 注释说明了附近代码的逻辑、意图或用法：`Emit fixits for all restricted includes.`。
- **L43**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L44**: Comment explains nearby logic, intent, or usage: `Fetch the length of the include statement from the start to just after`. / 注释说明了附近代码的逻辑、意图或用法：`Fetch the length of the include statement from the start to just after`。
- **L45**: Comment explains nearby logic, intent, or usage: `the newline, for finding the end (including the newline).`. / 注释说明了附近代码的逻辑、意图或用法：`the newline, for finding the end (including the newline).`。
- **L46**: Continues the surrounding expression or declaration: `const unsigned ToLen =`. / 继续构造周围的表达式或声明：`const unsigned ToLen =`。
- **L47**: Executes a call or declaration centered on `std::strcspn`. / 执行以 `std::strcspn` 为核心的调用或声明。
- **L48**: Continues logic associated with callable symbol `getCharRange`. / 继续与可调用符号 `getCharRange` 相关的逻辑。
- **L49**: Executes a call or declaration centered on `Include.Loc.getLocWithOffset`. / 执行以 `Include.Loc.getLocWithOffset` 为核心的调用或声明。
- **L50**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 51-60 / 第 51-60 行

```cpp
51 |       if (!Include.IsInMainFile) {
52 |         auto D = Check.diag(
53 |             Include.Loc,
54 |             "system include %0 not allowed, transitively included from %1");
55 |         D << Include.IncludeFile << SM.getFilename(Include.Loc);
56 |         D << FixItHint::CreateRemoval(ToRange);
57 |         continue;
58 |       }
59 |       auto D = Check.diag(Include.Loc, "system include %0 not allowed");
60 |       D << Include.IncludeFile;
```

- **L51**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L52**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L53**: Continues a multi-line argument list, initializer, or aggregate entry: `Include.Loc,`. / 继续一个多行参数列表、初始化器或聚合项：`Include.Loc,`。
- **L54**: Executes a standalone statement or declaration: `"system include %0 not allowed, transitively included from %1");`. / 执行一条独立语句或声明：`"system include %0 not allowed, transitively included from %1");`。
- **L55**: Executes a call or declaration centered on `SM.getFilename`. / 执行以 `SM.getFilename` 为核心的调用或声明。
- **L56**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L57**: Skips to the next loop iteration or continues the enclosing flow. / 跳到下一次循环迭代，或继续外层流程。
- **L58**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L59**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L60**: Executes a standalone statement or declaration: `D << Include.IncludeFile;`. / 执行一条独立语句或声明：`D << Include.IncludeFile;`。

### Lines 61-70 / 第 61-70 行

```cpp
61 |       D << FixItHint::CreateRemoval(ToRange);
62 |     }
63 |   }
64 | }
65 | 
66 | void RestrictSystemIncludesCheck::registerPPCallbacks(
67 |     const SourceManager &SM, Preprocessor *PP, Preprocessor *ModuleExpanderPP) {
68 |   PP->addPPCallbacks(
69 |       std::make_unique<RestrictedIncludesPPCallbacks>(*this, SM));
70 | }
```

- **L61**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L62**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L63**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L64**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L65**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L66**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L67**: Continues the surrounding expression or declaration: `const SourceManager &SM, Preprocessor *PP, Preprocessor *ModuleExpanderPP) {`. / 继续构造周围的表达式或声明：`const SourceManager &SM, Preprocessor *PP, Preprocessor *ModuleExpanderPP) {`。
- **L68**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L69**: Hooks into preprocessor callback flow to observe directives or macro activity. / 挂接到预处理器回调流程，以观察指令或宏活动。
- **L70**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 71-77 / 第 71-77 行

```cpp
71 | 
72 | void RestrictSystemIncludesCheck::storeOptions(
73 |     ClangTidyOptions::OptionMap &Opts) {
74 |   Options.store(Opts, "Includes", AllowedIncludes);
75 | }
76 | 
77 | } // namespace clang::tidy::portability
```

- **L71**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L72**: Continues logic associated with callable symbol `storeOptions`. / 继续与可调用符号 `storeOptions` 相关的逻辑。
- **L73**: Continues the surrounding expression or declaration: `ClangTidyOptions::OptionMap &Opts) {`. / 继续构造周围的表达式或声明：`ClangTidyOptions::OptionMap &Opts) {`。
- **L74**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L75**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L76**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L77**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::portability`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::portability`。

## Key Concepts / 关键概念

- **Portability checking / 可移植性检查**:
  - **EN**: Flags code that depends on compiler, platform, or ABI-specific behavior.
  - **CN**: 标记依赖编译器、平台或 ABI 特定行为的代码。
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

- `RestrictSystemIncludesCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `clang/Frontend/CompilerInstance.h`: Provides frontend action and compiler-instance APIs. / 提供前端动作与编译器实例 API。
- `clang/Lex/HeaderSearch.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
- `clang/Lex/PPCallbacks.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
- `clang/Lex/Preprocessor.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
- `llvm/ADT/DenseMap.h`: Provides LLVM ADT containers and helper types. / 提供LLVM ADT 容器与辅助类型。
- `llvm/ADT/SmallVector.h`: Provides LLVM ADT containers and helper types. / 提供LLVM ADT 容器与辅助类型。
- `llvm/Support/Path.h`: Provides LLVM support-library facilities. / 提供LLVM Support 库设施。
- `cstring`: Provides C or C++ standard library facilities. / 提供C 或 C++ 标准库设施。
