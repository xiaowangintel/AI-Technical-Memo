# ClangTidyModule.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `clang-tools-extra/clang-tidy/ClangTidyModule.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: Declares the base classes and factories used to define clang-tidy modules.
- **Purpose (CN)**: 声明定义 clang-tidy 模块所需的基类与工厂接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14 / 第 1-14 行

```cpp
   1: //===----------------------------------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_CLANGTIDYMODULE_H
  10: #define LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_CLANGTIDYMODULE_H
  11: 
  12: #include "ClangTidyOptions.h"
  13: #include "llvm/ADT/StringMap.h"
  14: #include "llvm/ADT/StringRef.h"
```
- **Line 1 / 第 1 行**: EN: Banner comment marking a file or section boundary. CN: 横幅注释，用于标记文件或章节边界。
- **Line 2 / 第 2 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 3 / 第 3 行**: EN: Comment describing intent, behavior, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. CN: 用于说明意图、行为或元数据的注释：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **Line 4 / 第 4 行**: EN: Comment describing intent, behavior, or metadata: `See https://llvm.org/LICENSE.txt for license information.`. CN: 用于说明意图、行为或元数据的注释：`See https://llvm.org/LICENSE.txt for license information.`。
- **Line 5 / 第 5 行**: EN: Comment describing intent, behavior, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. CN: 用于说明意图、行为或元数据的注释：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **Line 6 / 第 6 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 7 / 第 7 行**: EN: Banner comment marking a file or section boundary. CN: 横幅注释，用于标记文件或章节边界。
- **Line 8 / 第 8 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 9 / 第 9 行**: EN: Starts a conditional-compilation guard or branch. CN: 开始一个条件编译保护块或分支。
- **Line 10 / 第 10 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 11 / 第 11 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 12 / 第 12 行**: EN: Includes "ClangTidyOptions.h" so this file can use local declarations that pair with this file. CN: 包含 "ClangTidyOptions.h"，以便当前文件使用与该文件配套的本地声明。
- **Line 13 / 第 13 行**: EN: Includes "llvm/ADT/StringMap.h" so this file can use LLVM ADT containers and low-level utilities. CN: 包含 "llvm/ADT/StringMap.h"，以便当前文件使用LLVM ADT 容器与底层工具。
- **Line 14 / 第 14 行**: EN: Includes "llvm/ADT/StringRef.h" so this file can use LLVM ADT containers and low-level utilities. CN: 包含 "llvm/ADT/StringRef.h"，以便当前文件使用LLVM ADT 容器与底层工具。

### Lines 15-28 / 第 15-28 行

```cpp
  15: #include "llvm/Support/Registry.h"
  16: #include <functional>
  17: #include <memory>
  18: 
  19: namespace clang::tidy {
  20: 
  21: class ClangTidyCheck;
  22: class ClangTidyContext;
  23: 
  24: /// A collection of \c ClangTidyCheckFactory instances.
  25: ///
  26: /// All clang-tidy modules register their check factories with an instance of
  27: /// this object.
  28: class ClangTidyCheckFactories {
```
- **Line 15 / 第 15 行**: EN: Includes "llvm/Support/Registry.h" so this file can use LLVM support utilities such as diagnostics, filesystem, and strings. CN: 包含 "llvm/Support/Registry.h"，以便当前文件使用LLVM 支持工具，例如诊断、文件系统和字符串设施。
- **Line 16 / 第 16 行**: EN: Includes <functional> so this file can use supporting declarations or standard-library facilities. CN: 包含 <functional>，以便当前文件使用辅助声明或标准库设施。
- **Line 17 / 第 17 行**: EN: Includes <memory> so this file can use supporting declarations or standard-library facilities. CN: 包含 <memory>，以便当前文件使用辅助声明或标准库设施。
- **Line 18 / 第 18 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 19 / 第 19 行**: EN: Opens namespace `clang::tidy` to scope related declarations. CN: 打开命名空间 `clang::tidy`，为相关声明建立作用域。
- **Line 20 / 第 20 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 21 / 第 21 行**: EN: Begins the declaration of class `ClangTidyCheck`. CN: 开始声明 class `ClangTidyCheck`。
- **Line 22 / 第 22 行**: EN: Begins the declaration of class `ClangTidyContext`. CN: 开始声明 class `ClangTidyContext`。
- **Line 23 / 第 23 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 24 / 第 24 行**: EN: Comment describing intent, behavior, or metadata: `A collection of \c ClangTidyCheckFactory instances.`. CN: 用于说明意图、行为或元数据的注释：`A collection of \c ClangTidyCheckFactory instances.`。
- **Line 25 / 第 25 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 26 / 第 26 行**: EN: Comment describing intent, behavior, or metadata: `All clang-tidy modules register their check factories with an instance of`. CN: 用于说明意图、行为或元数据的注释：`All clang-tidy modules register their check factories with an instance of`。
- **Line 27 / 第 27 行**: EN: Comment describing intent, behavior, or metadata: `this object.`. CN: 用于说明意图、行为或元数据的注释：`this object.`。
- **Line 28 / 第 28 行**: EN: Begins the declaration of class `ClangTidyCheckFactories`. CN: 开始声明 class `ClangTidyCheckFactories`。

### Lines 29-42 / 第 29-42 行

```cpp
  29: public:
  30:   using CheckFactory = std::function<std::unique_ptr<ClangTidyCheck>(
  31:       StringRef Name, ClangTidyContext *Context)>;
  32: 
  33:   /// Registers check \p Factory with name \p Name.
  34:   ///
  35:   /// For all checks that have default constructors, use \c registerCheck.
  36:   void registerCheckFactory(StringRef Name, CheckFactory Factory);
  37: 
  38:   /// Registers the \c CheckType with the name \p Name.
  39:   ///
  40:   /// This method should be used for all \c ClangTidyChecks that don't require
  41:   /// constructor parameters.
  42:   ///
```
- **Line 29 / 第 29 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 30 / 第 30 行**: EN: Continues logic associated with callable symbol `unique_ptr<ClangTidyCheck>`. CN: 继续与可调用符号 `unique_ptr<ClangTidyCheck>` 相关的逻辑。
- **Line 31 / 第 31 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 32 / 第 32 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 33 / 第 33 行**: EN: Comment describing intent, behavior, or metadata: `Registers check \p Factory with name \p Name.`. CN: 用于说明意图、行为或元数据的注释：`Registers check \p Factory with name \p Name.`。
- **Line 34 / 第 34 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 35 / 第 35 行**: EN: Comment describing intent, behavior, or metadata: `For all checks that have default constructors, use \c registerCheck.`. CN: 用于说明意图、行为或元数据的注释：`For all checks that have default constructors, use \c registerCheck.`。
- **Line 36 / 第 36 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 37 / 第 37 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 38 / 第 38 行**: EN: Comment describing intent, behavior, or metadata: `Registers the \c CheckType with the name \p Name.`. CN: 用于说明意图、行为或元数据的注释：`Registers the \c CheckType with the name \p Name.`。
- **Line 39 / 第 39 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 40 / 第 40 行**: EN: Comment describing intent, behavior, or metadata: `This method should be used for all \c ClangTidyChecks that don't require`. CN: 用于说明意图、行为或元数据的注释：`This method should be used for all \c ClangTidyChecks that don't require`。
- **Line 41 / 第 41 行**: EN: Comment describing intent, behavior, or metadata: `constructor parameters.`. CN: 用于说明意图、行为或元数据的注释：`constructor parameters.`。
- **Line 42 / 第 42 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。

### Lines 43-56 / 第 43-56 行

```cpp
  43:   /// For example, if have a clang-tidy check like:
  44:   /// \code
  45:   /// class MyTidyCheck : public ClangTidyCheck {
  46:   ///   void registerMatchers(ast_matchers::MatchFinder *Finder) override {
  47:   ///     ..
  48:   ///   }
  49:   /// };
  50:   /// \endcode
  51:   /// you can register it with:
  52:   /// \code
  53:   /// class MyModule : public ClangTidyModule {
  54:   ///   void addCheckFactories(ClangTidyCheckFactories &Factories) override {
  55:   ///     Factories.registerCheck<MyTidyCheck>("myproject-my-check");
  56:   ///   }
```
- **Line 43 / 第 43 行**: EN: Comment describing intent, behavior, or metadata: `For example, if have a clang-tidy check like:`. CN: 用于说明意图、行为或元数据的注释：`For example, if have a clang-tidy check like:`。
- **Line 44 / 第 44 行**: EN: Comment describing intent, behavior, or metadata: `\code`. CN: 用于说明意图、行为或元数据的注释：`\code`。
- **Line 45 / 第 45 行**: EN: Comment describing intent, behavior, or metadata: `class MyTidyCheck : public ClangTidyCheck {`. CN: 用于说明意图、行为或元数据的注释：`class MyTidyCheck : public ClangTidyCheck {`。
- **Line 46 / 第 46 行**: EN: Comment describing intent, behavior, or metadata: `void registerMatchers(ast_matchers::MatchFinder *Finder) override {`. CN: 用于说明意图、行为或元数据的注释：`void registerMatchers(ast_matchers::MatchFinder *Finder) override {`。
- **Line 47 / 第 47 行**: EN: Comment describing intent, behavior, or metadata: `..`. CN: 用于说明意图、行为或元数据的注释：`..`。
- **Line 48 / 第 48 行**: EN: Comment describing intent, behavior, or metadata: `}`. CN: 用于说明意图、行为或元数据的注释：`}`。
- **Line 49 / 第 49 行**: EN: Comment describing intent, behavior, or metadata: `};`. CN: 用于说明意图、行为或元数据的注释：`};`。
- **Line 50 / 第 50 行**: EN: Comment describing intent, behavior, or metadata: `\endcode`. CN: 用于说明意图、行为或元数据的注释：`\endcode`。
- **Line 51 / 第 51 行**: EN: Comment describing intent, behavior, or metadata: `you can register it with:`. CN: 用于说明意图、行为或元数据的注释：`you can register it with:`。
- **Line 52 / 第 52 行**: EN: Comment describing intent, behavior, or metadata: `\code`. CN: 用于说明意图、行为或元数据的注释：`\code`。
- **Line 53 / 第 53 行**: EN: Comment describing intent, behavior, or metadata: `class MyModule : public ClangTidyModule {`. CN: 用于说明意图、行为或元数据的注释：`class MyModule : public ClangTidyModule {`。
- **Line 54 / 第 54 行**: EN: Comment describing intent, behavior, or metadata: `void addCheckFactories(ClangTidyCheckFactories &Factories) override {`. CN: 用于说明意图、行为或元数据的注释：`void addCheckFactories(ClangTidyCheckFactories &Factories) override {`。
- **Line 55 / 第 55 行**: EN: Comment describing intent, behavior, or metadata: `Factories.registerCheck<MyTidyCheck>("myproject-my-check");`. CN: 用于说明意图、行为或元数据的注释：`Factories.registerCheck<MyTidyCheck>("myproject-my-check");`。
- **Line 56 / 第 56 行**: EN: Comment describing intent, behavior, or metadata: `}`. CN: 用于说明意图、行为或元数据的注释：`}`。

### Lines 57-70 / 第 57-70 行

```cpp
  57:   /// };
  58:   /// \endcode
  59:   template <typename CheckType> void registerCheck(StringRef CheckName) {
  60:     registerCheckFactory(CheckName,
  61:                          [](StringRef Name, ClangTidyContext *Context) {
  62:                            return std::make_unique<CheckType>(Name, Context);
  63:                          });
  64:   }
  65: 
  66:   void eraseCheck(StringRef CheckName) { Factories.erase(CheckName); }
  67: 
  68:   /// Create instances of checks that are enabled.
  69:   std::vector<std::unique_ptr<ClangTidyCheck>>
  70:   createChecks(ClangTidyContext *Context) const;
```
- **Line 57 / 第 57 行**: EN: Comment describing intent, behavior, or metadata: `};`. CN: 用于说明意图、行为或元数据的注释：`};`。
- **Line 58 / 第 58 行**: EN: Comment describing intent, behavior, or metadata: `\endcode`. CN: 用于说明意图、行为或元数据的注释：`\endcode`。
- **Line 59 / 第 59 行**: EN: Introduces template parameters or specialization context. CN: 引入模板参数或特化上下文。
- **Line 60 / 第 60 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 61 / 第 61 行**: EN: Defines function or method `callable`. CN: 定义函数或方法 `callable`。
- **Line 62 / 第 62 行**: EN: Returns a value or transfers control to the caller with `std::make_unique<CheckType>(Name, Context)`. CN: 返回一个值，或以 `std::make_unique<CheckType>(Name, Context)` 将控制权交还给调用者。
- **Line 63 / 第 63 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 64 / 第 64 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 65 / 第 65 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 66 / 第 66 行**: EN: Continues logic associated with callable symbol `eraseCheck`. CN: 继续与可调用符号 `eraseCheck` 相关的逻辑。
- **Line 67 / 第 67 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 68 / 第 68 行**: EN: Comment describing intent, behavior, or metadata: `Create instances of checks that are enabled.`. CN: 用于说明意图、行为或元数据的注释：`Create instances of checks that are enabled.`。
- **Line 69 / 第 69 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 70 / 第 70 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。

### Lines 71-84 / 第 71-84 行

```cpp
  71: 
  72:   /// Create instances of checks that are enabled for the current Language.
  73:   std::vector<std::unique_ptr<ClangTidyCheck>>
  74:   createChecksForLanguage(ClangTidyContext *Context) const;
  75: 
  76:   using FactoryMap = llvm::StringMap<CheckFactory>;
  77:   FactoryMap::const_iterator begin() const { return Factories.begin(); }
  78:   FactoryMap::const_iterator end() const { return Factories.end(); }
  79:   bool empty() const { return Factories.empty(); }
  80: 
  81: private:
  82:   FactoryMap Factories;
  83: };
  84: 
```
- **Line 71 / 第 71 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 72 / 第 72 行**: EN: Comment describing intent, behavior, or metadata: `Create instances of checks that are enabled for the current Language.`. CN: 用于说明意图、行为或元数据的注释：`Create instances of checks that are enabled for the current Language.`。
- **Line 73 / 第 73 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 74 / 第 74 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 75 / 第 75 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 76 / 第 76 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 77 / 第 77 行**: EN: Continues logic associated with callable symbol `begin`. CN: 继续与可调用符号 `begin` 相关的逻辑。
- **Line 78 / 第 78 行**: EN: Continues logic associated with callable symbol `end`. CN: 继续与可调用符号 `end` 相关的逻辑。
- **Line 79 / 第 79 行**: EN: Continues logic associated with callable symbol `empty`. CN: 继续与可调用符号 `empty` 相关的逻辑。
- **Line 80 / 第 80 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 81 / 第 81 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 82 / 第 82 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 83 / 第 83 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 84 / 第 84 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 85-98 / 第 85-98 行

```cpp
  85: /// A clang-tidy module groups a number of \c ClangTidyChecks and gives
  86: /// them a prefixed name.
  87: class ClangTidyModule {
  88: public:
  89:   virtual ~ClangTidyModule() = default;
  90: 
  91:   /// Implement this function in order to register all \c CheckFactories
  92:   /// belonging to this module.
  93:   virtual void addCheckFactories(ClangTidyCheckFactories &CheckFactories) = 0;
  94: 
  95:   /// Gets default options for checks defined in this module.
  96:   virtual ClangTidyOptions getModuleOptions();
  97: };
  98: 
```
- **Line 85 / 第 85 行**: EN: Comment describing intent, behavior, or metadata: `A clang-tidy module groups a number of \c ClangTidyChecks and gives`. CN: 用于说明意图、行为或元数据的注释：`A clang-tidy module groups a number of \c ClangTidyChecks and gives`。
- **Line 86 / 第 86 行**: EN: Comment describing intent, behavior, or metadata: `them a prefixed name.`. CN: 用于说明意图、行为或元数据的注释：`them a prefixed name.`。
- **Line 87 / 第 87 行**: EN: Begins the declaration of class `ClangTidyModule`. CN: 开始声明 class `ClangTidyModule`。
- **Line 88 / 第 88 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 89 / 第 89 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 90 / 第 90 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 91 / 第 91 行**: EN: Comment describing intent, behavior, or metadata: `Implement this function in order to register all \c CheckFactories`. CN: 用于说明意图、行为或元数据的注释：`Implement this function in order to register all \c CheckFactories`。
- **Line 92 / 第 92 行**: EN: Comment describing intent, behavior, or metadata: `belonging to this module.`. CN: 用于说明意图、行为或元数据的注释：`belonging to this module.`。
- **Line 93 / 第 93 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 94 / 第 94 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 95 / 第 95 行**: EN: Comment describing intent, behavior, or metadata: `Gets default options for checks defined in this module.`. CN: 用于说明意图、行为或元数据的注释：`Gets default options for checks defined in this module.`。
- **Line 96 / 第 96 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 97 / 第 97 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 98 / 第 98 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 99-107 / 第 99-107 行

```cpp
  99: using ClangTidyModuleRegistry = llvm::Registry<ClangTidyModule>;
 100: 
 101: } // namespace clang::tidy
 102: 
 103: namespace llvm {
 104: extern template class Registry<clang::tidy::ClangTidyModule>;
 105: } // namespace llvm
 106: 
 107: #endif // LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_CLANGTIDYMODULE_H
```
- **Line 99 / 第 99 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 100 / 第 100 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 101 / 第 101 行**: EN: Closes a namespace scope and documents which namespace ended. CN: 结束一个命名空间作用域，并说明被关闭的命名空间。
- **Line 102 / 第 102 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 103 / 第 103 行**: EN: Opens namespace `llvm` to scope related declarations. CN: 打开命名空间 `llvm`，为相关声明建立作用域。
- **Line 104 / 第 104 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 105 / 第 105 行**: EN: Closes a namespace scope and documents which namespace ended. CN: 结束一个命名空间作用域，并说明被关闭的命名空间。
- **Line 106 / 第 106 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 107 / 第 107 行**: EN: Closes the current preprocessor conditional block. CN: 结束当前预处理条件块。

## Key Concepts / 关键概念
- **Clang-Tidy framework / Clang-Tidy 框架**: Participates in the clang-tidy architecture that wires checks, options, and diagnostics together. / 参与 clang-tidy 架构，把检查、选项和诊断连接在一起。
- **Clang-Tidy check lifecycle / Clang-Tidy 检查生命周期**: Defines or uses the standard hook points of a clang-tidy check. / 定义或使用 clang-tidy 检查的标准钩子。
- **Shared analysis context / 共享分析上下文**: Carries options, diagnostics, language mode, and per-run shared state. / 承载选项、诊断、语言模式以及每次运行的共享状态。
- **Configurable check options / 可配置检查选项**: Reads, stores, or merges user-visible configuration knobs. / 读取、存储或合并面向用户的配置项。
- **Module registration / 模块注册**: Groups checks into a named clang-tidy module and registers factories. / 将检查分组到命名 clang-tidy 模块中并注册工厂。
- **Check factory dispatch / 检查工厂分发**: Builds check instances from registered factories. / 从已注册工厂构建检查实例。

## Dependencies / 依赖关系
- **Clang/LLVM and local headers / Clang/LLVM 与本地头文件**: `ClangTidyOptions.h`, `llvm/ADT/StringMap.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Registry.h`
- **Standard library headers / 标准库头文件**: `<functional>`, `<memory>`
