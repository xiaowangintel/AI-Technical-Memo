# IncludeInserter.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/utils/IncludeInserter.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Provides shared helper utilities reused by multiple clang-tidy checks.
  - **CN**: 提供多个 clang-tidy 检查共享复用的辅助工具。

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
 9 | #ifndef LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_UTILS_INCLUDEINSERTER_H
10 | #define LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_UTILS_INCLUDEINSERTER_H
11 | 
12 | #include "IncludeSorter.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Starts a preprocessor conditional block: `#ifndef LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_UTILS_INCLUDEINSERTER_H`. / 开始一个预处理条件块：`#ifndef LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_UTILS_INCLUDEINSERTER_H`。
- **L10**: Defines macro `LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_UTILS_INCLUDEINSERTER_H` for compile-time control or shorthand. / 定义宏 `LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_UTILS_INCLUDEINSERTER_H`，用于编译期控制或简写。
- **L11**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L12**: Includes "IncludeSorter.h" to access local declarations from the current tool or check. / 引入 "IncludeSorter.h" 以使用当前工具或检查的本地声明。

### Lines 13-24 / 第 13-24 行

```cpp
13 | #include "clang/Basic/Diagnostic.h"
14 | #include "llvm/ADT/StringSet.h"
15 | #include <memory>
16 | #include <optional>
17 | 
18 | namespace clang {
19 | class Preprocessor;
20 | namespace tidy::utils {
21 | 
22 | /// Produces fixes to insert specified includes to source files, if not
23 | /// yet present.
24 | ///
```

- **L13**: Includes "clang/Basic/Diagnostic.h" to access basic source, diagnostic, and language-option support. / 引入 "clang/Basic/Diagnostic.h" 以使用基础源码、诊断与语言选项支持。
- **L14**: Includes "llvm/ADT/StringSet.h" to access LLVM ADT containers and helper types. / 引入 "llvm/ADT/StringSet.h" 以使用LLVM ADT 容器与辅助类型。
- **L15**: Includes <memory> to access C or C++ standard library facilities. / 引入 <memory> 以使用C 或 C++ 标准库设施。
- **L16**: Includes <optional> to access C or C++ standard library facilities. / 引入 <optional> 以使用C 或 C++ 标准库设施。
- **L17**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L18**: Opens namespace scope `clang`. / 打开命名空间作用域 `clang`。
- **L19**: Declares class `Preprocessor;`. / 声明类 `Preprocessor;`。
- **L20**: Opens namespace scope `tidy::utils`. / 打开命名空间作用域 `tidy::utils`。
- **L21**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L22**: Comment explains nearby logic, intent, or usage: `/ Produces fixes to insert specified includes to source files, if not`. / 注释说明了附近代码的逻辑、意图或用法：`/ Produces fixes to insert specified includes to source files, if not`。
- **L23**: Comment explains nearby logic, intent, or usage: `/ yet present.`. / 注释说明了附近代码的逻辑、意图或用法：`/ yet present.`。
- **L24**: Comment explains nearby logic, intent, or usage: `/`. / 注释说明了附近代码的逻辑、意图或用法：`/`。

### Lines 25-36 / 第 25-36 行

```cpp
25 | /// ``IncludeInserter`` can be used in clang-tidy checks in the following way:
26 | /// \code
27 | /// #include "../ClangTidyCheck.h"
28 | /// #include "../utils/IncludeInserter.h"
29 | ///
30 | /// namespace clang {
31 | /// namespace tidy {
32 | ///
33 | /// class MyCheck : public ClangTidyCheck {
34 | ///  public:
35 | ///   void registerPPCallbacks(const SourceManager &SM, Preprocessor *PP,
36 | ///                            Preprocessor *ModuleExpanderPP) override {
```

- **L25**: Comment explains nearby logic, intent, or usage: `/ \`\`IncludeInserter\`\` can be used in clang-tidy checks in the following way:`. / 注释说明了附近代码的逻辑、意图或用法：`/ \`\`IncludeInserter\`\` can be used in clang-tidy checks in the following way:`。
- **L26**: Comment explains nearby logic, intent, or usage: `/ \code`. / 注释说明了附近代码的逻辑、意图或用法：`/ \code`。
- **L27**: Comment explains nearby logic, intent, or usage: `/ #include "../ClangTidyCheck.h"`. / 注释说明了附近代码的逻辑、意图或用法：`/ #include "../ClangTidyCheck.h"`。
- **L28**: Comment explains nearby logic, intent, or usage: `/ #include "../utils/IncludeInserter.h"`. / 注释说明了附近代码的逻辑、意图或用法：`/ #include "../utils/IncludeInserter.h"`。
- **L29**: Comment explains nearby logic, intent, or usage: `/`. / 注释说明了附近代码的逻辑、意图或用法：`/`。
- **L30**: Comment explains nearby logic, intent, or usage: `/ namespace clang {`. / 注释说明了附近代码的逻辑、意图或用法：`/ namespace clang {`。
- **L31**: Comment explains nearby logic, intent, or usage: `/ namespace tidy {`. / 注释说明了附近代码的逻辑、意图或用法：`/ namespace tidy {`。
- **L32**: Comment explains nearby logic, intent, or usage: `/`. / 注释说明了附近代码的逻辑、意图或用法：`/`。
- **L33**: Comment explains nearby logic, intent, or usage: `/ class MyCheck : public ClangTidyCheck {`. / 注释说明了附近代码的逻辑、意图或用法：`/ class MyCheck : public ClangTidyCheck {`。
- **L34**: Comment explains nearby logic, intent, or usage: `/  public:`. / 注释说明了附近代码的逻辑、意图或用法：`/  public:`。
- **L35**: Comment explains nearby logic, intent, or usage: `/   void registerPPCallbacks(const SourceManager &SM, Preprocessor *PP,`. / 注释说明了附近代码的逻辑、意图或用法：`/   void registerPPCallbacks(const SourceManager &SM, Preprocessor *PP,`。
- **L36**: Comment explains nearby logic, intent, or usage: `/                            Preprocessor *ModuleExpanderPP) override {`. / 注释说明了附近代码的逻辑、意图或用法：`/                            Preprocessor *ModuleExpanderPP) override {`。

### Lines 37-48 / 第 37-48 行

```cpp
37 | ///     Inserter.registerPreprocessor(PP);
38 | ///   }
39 | ///
40 | ///   void registerMatchers(ast_matchers::MatchFinder* Finder) override { ... }
41 | ///
42 | ///   void check(
43 | ///       const ast_matchers::MatchFinder::MatchResult& Result) override {
44 | ///     ...
45 | ///     Inserter.createMainFileIncludeInsertion("path/to/Header.h");
46 | ///     ...
47 | ///   }
48 | ///
```

- **L37**: Comment explains nearby logic, intent, or usage: `/     Inserter.registerPreprocessor(PP);`. / 注释说明了附近代码的逻辑、意图或用法：`/     Inserter.registerPreprocessor(PP);`。
- **L38**: Comment explains nearby logic, intent, or usage: `/   }`. / 注释说明了附近代码的逻辑、意图或用法：`/   }`。
- **L39**: Comment explains nearby logic, intent, or usage: `/`. / 注释说明了附近代码的逻辑、意图或用法：`/`。
- **L40**: Comment explains nearby logic, intent, or usage: `/   void registerMatchers(ast_matchers::MatchFinder* Finder) override { ... }`. / 注释说明了附近代码的逻辑、意图或用法：`/   void registerMatchers(ast_matchers::MatchFinder* Finder) override { ... }`。
- **L41**: Comment explains nearby logic, intent, or usage: `/`. / 注释说明了附近代码的逻辑、意图或用法：`/`。
- **L42**: Comment explains nearby logic, intent, or usage: `/   void check(`. / 注释说明了附近代码的逻辑、意图或用法：`/   void check(`。
- **L43**: Comment explains nearby logic, intent, or usage: `/       const ast_matchers::MatchFinder::MatchResult& Result) override {`. / 注释说明了附近代码的逻辑、意图或用法：`/       const ast_matchers::MatchFinder::MatchResult& Result) override {`。
- **L44**: Comment explains nearby logic, intent, or usage: `/     ...`. / 注释说明了附近代码的逻辑、意图或用法：`/     ...`。
- **L45**: Comment explains nearby logic, intent, or usage: `/     Inserter.createMainFileIncludeInsertion("path/to/Header.h");`. / 注释说明了附近代码的逻辑、意图或用法：`/     Inserter.createMainFileIncludeInsertion("path/to/Header.h");`。
- **L46**: Comment explains nearby logic, intent, or usage: `/     ...`. / 注释说明了附近代码的逻辑、意图或用法：`/     ...`。
- **L47**: Comment explains nearby logic, intent, or usage: `/   }`. / 注释说明了附近代码的逻辑、意图或用法：`/   }`。
- **L48**: Comment explains nearby logic, intent, or usage: `/`. / 注释说明了附近代码的逻辑、意图或用法：`/`。

### Lines 49-60 / 第 49-60 行

```cpp
49 | ///  private:
50 | ///   utils::IncludeInserter Inserter{utils::IncludeSorter::IS_Google};
51 | /// };
52 | /// } // namespace tidy
53 | /// } // namespace clang
54 | /// \endcode
55 | class IncludeInserter {
56 | public:
57 |   /// Initializes the IncludeInserter using the IncludeStyle \p Style.
58 |   /// In most cases the \p Style will be retrieved from the ClangTidyOptions
59 |   /// using \code
60 |   ///   Options.getLocalOrGlobal("IncludeStyle", <DefaultStyle>)
```

- **L49**: Comment explains nearby logic, intent, or usage: `/  private:`. / 注释说明了附近代码的逻辑、意图或用法：`/  private:`。
- **L50**: Comment explains nearby logic, intent, or usage: `/   utils::IncludeInserter Inserter{utils::IncludeSorter::IS_Google};`. / 注释说明了附近代码的逻辑、意图或用法：`/   utils::IncludeInserter Inserter{utils::IncludeSorter::IS_Google};`。
- **L51**: Comment explains nearby logic, intent, or usage: `/ };`. / 注释说明了附近代码的逻辑、意图或用法：`/ };`。
- **L52**: Comment explains nearby logic, intent, or usage: `/ } // namespace tidy`. / 注释说明了附近代码的逻辑、意图或用法：`/ } // namespace tidy`。
- **L53**: Comment explains nearby logic, intent, or usage: `/ } // namespace clang`. / 注释说明了附近代码的逻辑、意图或用法：`/ } // namespace clang`。
- **L54**: Comment explains nearby logic, intent, or usage: `/ \endcode`. / 注释说明了附近代码的逻辑、意图或用法：`/ \endcode`。
- **L55**: Declares class `IncludeInserter`. / 声明类 `IncludeInserter`。
- **L56**: Sets the following members to `public` access. / 将后续成员的访问级别设为 `public`。
- **L57**: Comment explains nearby logic, intent, or usage: `/ Initializes the IncludeInserter using the IncludeStyle \p Style.`. / 注释说明了附近代码的逻辑、意图或用法：`/ Initializes the IncludeInserter using the IncludeStyle \p Style.`。
- **L58**: Comment explains nearby logic, intent, or usage: `/ In most cases the \p Style will be retrieved from the ClangTidyOptions`. / 注释说明了附近代码的逻辑、意图或用法：`/ In most cases the \p Style will be retrieved from the ClangTidyOptions`。
- **L59**: Comment explains nearby logic, intent, or usage: `/ using \code`. / 注释说明了附近代码的逻辑、意图或用法：`/ using \code`。
- **L60**: Comment explains nearby logic, intent, or usage: `/   Options.getLocalOrGlobal("IncludeStyle", <DefaultStyle>)`. / 注释说明了附近代码的逻辑、意图或用法：`/   Options.getLocalOrGlobal("IncludeStyle", <DefaultStyle>)`。

### Lines 61-72 / 第 61-72 行

```cpp
61 |   /// \endcode
62 |   explicit IncludeInserter(IncludeSorter::IncludeStyle Style,
63 |                            bool SelfContainedDiags);
64 | 
65 |   /// Registers this with the Preprocessor \p PP, must be called before this
66 |   /// class is used.
67 |   void registerPreprocessor(Preprocessor *PP);
68 | 
69 |   /// Creates a \p Header inclusion directive fixit in the File \p FileID.
70 |   /// When \p Header is enclosed in angle brackets, uses angle brackets in the
71 |   /// inclusion directive, otherwise uses quotes.
72 |   /// Returns ``std::nullopt`` on error or if the inclusion directive already
```

- **L61**: Comment explains nearby logic, intent, or usage: `/ \endcode`. / 注释说明了附近代码的逻辑、意图或用法：`/ \endcode`。
- **L62**: Uses include-insertion support to keep rewritten code compilable. / 使用头文件插入支持，以保持重写后代码可编译。
- **L63**: Executes a standalone statement or declaration: `bool SelfContainedDiags);`. / 执行一条独立语句或声明：`bool SelfContainedDiags);`。
- **L64**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L65**: Comment explains nearby logic, intent, or usage: `/ Registers this with the Preprocessor \p PP, must be called before this`. / 注释说明了附近代码的逻辑、意图或用法：`/ Registers this with the Preprocessor \p PP, must be called before this`。
- **L66**: Comment explains nearby logic, intent, or usage: `/ class is used.`. / 注释说明了附近代码的逻辑、意图或用法：`/ class is used.`。
- **L67**: Executes a call or declaration centered on `registerPreprocessor`. / 执行以 `registerPreprocessor` 为核心的调用或声明。
- **L68**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L69**: Comment explains nearby logic, intent, or usage: `/ Creates a \p Header inclusion directive fixit in the File \p FileID.`. / 注释说明了附近代码的逻辑、意图或用法：`/ Creates a \p Header inclusion directive fixit in the File \p FileID.`。
- **L70**: Comment explains nearby logic, intent, or usage: `/ When \p Header is enclosed in angle brackets, uses angle brackets in the`. / 注释说明了附近代码的逻辑、意图或用法：`/ When \p Header is enclosed in angle brackets, uses angle brackets in the`。
- **L71**: Comment explains nearby logic, intent, or usage: `/ inclusion directive, otherwise uses quotes.`. / 注释说明了附近代码的逻辑、意图或用法：`/ inclusion directive, otherwise uses quotes.`。
- **L72**: Comment explains nearby logic, intent, or usage: `/ Returns \`\`std::nullopt\`\` on error or if the inclusion directive already`. / 注释说明了附近代码的逻辑、意图或用法：`/ Returns \`\`std::nullopt\`\` on error or if the inclusion directive already`。

### Lines 73-84 / 第 73-84 行

```cpp
73 |   /// exists.
74 |   std::optional<FixItHint> createIncludeInsertion(FileID FileID,
75 |                                                   StringRef Header);
76 | 
77 |   /// Creates a \p Header inclusion directive fixit in the main file.
78 |   /// When \p Header is enclosed in angle brackets, uses angle brackets in the
79 |   /// inclusion directive, otherwise uses quotes.
80 |   /// Returns ``std::nullopt`` on error or if the inclusion directive already
81 |   /// exists.
82 |   std::optional<FixItHint> createMainFileIncludeInsertion(StringRef Header);
83 | 
84 |   IncludeSorter::IncludeStyle getStyle() const { return Style; }
```

- **L73**: Comment explains nearby logic, intent, or usage: `/ exists.`. / 注释说明了附近代码的逻辑、意图或用法：`/ exists.`。
- **L74**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L75**: Executes a standalone statement or declaration: `StringRef Header);`. / 执行一条独立语句或声明：`StringRef Header);`。
- **L76**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L77**: Comment explains nearby logic, intent, or usage: `/ Creates a \p Header inclusion directive fixit in the main file.`. / 注释说明了附近代码的逻辑、意图或用法：`/ Creates a \p Header inclusion directive fixit in the main file.`。
- **L78**: Comment explains nearby logic, intent, or usage: `/ When \p Header is enclosed in angle brackets, uses angle brackets in the`. / 注释说明了附近代码的逻辑、意图或用法：`/ When \p Header is enclosed in angle brackets, uses angle brackets in the`。
- **L79**: Comment explains nearby logic, intent, or usage: `/ inclusion directive, otherwise uses quotes.`. / 注释说明了附近代码的逻辑、意图或用法：`/ inclusion directive, otherwise uses quotes.`。
- **L80**: Comment explains nearby logic, intent, or usage: `/ Returns \`\`std::nullopt\`\` on error or if the inclusion directive already`. / 注释说明了附近代码的逻辑、意图或用法：`/ Returns \`\`std::nullopt\`\` on error or if the inclusion directive already`。
- **L81**: Comment explains nearby logic, intent, or usage: `/ exists.`. / 注释说明了附近代码的逻辑、意图或用法：`/ exists.`。
- **L82**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L83**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L84**: Continues logic associated with callable symbol `getStyle`. / 继续与可调用符号 `getStyle` 相关的逻辑。

### Lines 85-96 / 第 85-96 行

```cpp
85 | 
86 | private:
87 |   void addInclude(StringRef FileName, bool IsAngled,
88 |                   SourceLocation HashLocation, SourceLocation EndLocation);
89 | 
90 |   IncludeSorter &getOrCreate(FileID FileID);
91 | 
92 |   llvm::DenseMap<FileID, std::unique_ptr<IncludeSorter>> IncludeSorterByFile;
93 |   llvm::DenseMap<FileID, llvm::StringSet<>> InsertedHeaders;
94 |   const SourceManager *SourceMgr{nullptr};
95 |   const IncludeSorter::IncludeStyle Style;
96 |   const bool SelfContainedDiags;
```

- **L85**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L86**: Sets the following members to `private` access. / 将后续成员的访问级别设为 `private`。
- **L87**: Continues a multi-line argument list, initializer, or aggregate entry: `void addInclude(StringRef FileName, bool IsAngled,`. / 继续一个多行参数列表、初始化器或聚合项：`void addInclude(StringRef FileName, bool IsAngled,`。
- **L88**: Executes a standalone statement or declaration: `SourceLocation HashLocation, SourceLocation EndLocation);`. / 执行一条独立语句或声明：`SourceLocation HashLocation, SourceLocation EndLocation);`。
- **L89**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L90**: Executes a call or declaration centered on `&getOrCreate`. / 执行以 `&getOrCreate` 为核心的调用或声明。
- **L91**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L92**: Executes a standalone statement or declaration: `llvm::DenseMap<FileID, std::unique_ptr<IncludeSorter>> IncludeSorterByFile;`. / 执行一条独立语句或声明：`llvm::DenseMap<FileID, std::unique_ptr<IncludeSorter>> IncludeSorterByFile;`。
- **L93**: Executes a standalone statement or declaration: `llvm::DenseMap<FileID, llvm::StringSet<>> InsertedHeaders;`. / 执行一条独立语句或声明：`llvm::DenseMap<FileID, llvm::StringSet<>> InsertedHeaders;`。
- **L94**: Executes a standalone statement or declaration: `const SourceManager *SourceMgr{nullptr};`. / 执行一条独立语句或声明：`const SourceManager *SourceMgr{nullptr};`。
- **L95**: Executes a standalone statement or declaration: `const IncludeSorter::IncludeStyle Style;`. / 执行一条独立语句或声明：`const IncludeSorter::IncludeStyle Style;`。
- **L96**: Executes a standalone statement or declaration: `const bool SelfContainedDiags;`. / 执行一条独立语句或声明：`const bool SelfContainedDiags;`。

### Lines 97-102 / 第 97-102 行

```cpp
 97 |   friend class IncludeInserterCallback;
 98 | };
 99 | 
100 | } // namespace tidy::utils
101 | } // namespace clang
102 | #endif // LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_UTILS_INCLUDEINSERTER_H
```

- **L97**: Uses include-insertion support to keep rewritten code compilable. / 使用头文件插入支持，以保持重写后代码可编译。
- **L98**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L99**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L100**: Closes a namespace scope while preserving the trailing comment: `} // namespace tidy::utils`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace tidy::utils`。
- **L101**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang`。
- **L102**: Closes the current preprocessor conditional block. / 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Shared clang-tidy helpers / 共享 clang-tidy 辅助逻辑**:
  - **EN**: Centralizes reusable matcher, lexer, option, and replacement utilities.
  - **CN**: 集中管理可复用的 matcher、词法、选项与替换辅助逻辑。
- **AST matcher registration / AST 匹配器注册**:
  - **EN**: Builds matcher trees that let the check find relevant AST patterns.
  - **CN**: 构建匹配器树，使检查能够找到相关 AST 模式。
- **AST matcher callbacks / AST 匹配回调**:
  - **EN**: Routes matched AST nodes into check-specific callback logic.
  - **CN**: 把匹配到的 AST 节点路由到检查特定的回调逻辑。
- **Automated fix-its / 自动修复提示**:
  - **EN**: Attaches machine-applicable edits to diagnostics whenever the transformation is safe.
  - **CN**: 在转换安全时为诊断附加可由机器应用的编辑。
- **Persistent options / 持久化选项**:
  - **EN**: Reads configurable behavior so checks can adapt to project policy.
  - **CN**: 读取可配置行为，使检查能够适配项目策略。
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

- `IncludeSorter.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `clang/Basic/Diagnostic.h`: Provides basic source, diagnostic, and language-option support. / 提供基础源码、诊断与语言选项支持。
- `llvm/ADT/StringSet.h`: Provides LLVM ADT containers and helper types. / 提供LLVM ADT 容器与辅助类型。
- `memory`: Provides C or C++ standard library facilities. / 提供C 或 C++ 标准库设施。
- `optional`: Provides C or C++ standard library facilities. / 提供C 或 C++ 标准库设施。
